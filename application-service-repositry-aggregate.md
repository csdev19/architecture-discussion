# 📘 Guía de Conceptos del Dominio: Application Service, Repository, Aggregate

Esta guía continúa con los conceptos centrales de Domain-Driven Design (DDD) y cómo aplicarlos en un proyecto real.

---

## ⚙️ Application Service

Un **Application Service** es un orquestador. No contiene lógica de negocio, sino que coordina llamadas a:

- Repositorios
- Aggregates
- Domain Services
- Value Objects

### Características:

| Propiedad | Valor |
|-----------|-------|
| ¿Tiene lógica de negocio? | ❌ No |
| ¿Coordina flujo del negocio? | ✅ Sí |
| ¿Es parte del dominio? | ⚠️ No, está en la capa de aplicación |
| ¿Llama a entidades o domain services? | ✅ Sí |

### Ejemplo:

```ts
export class CreateOrder {
  constructor(private readonly repo: IOrderRepository) {}

  async execute(input: CreateOrderInput): Promise<void> {
    const order = new Order(input.id);
    for (const item of input.items) {
      const price = Price.create(item.price);
      order.addItem(new OrderItem(item.productId, item.quantity, price));
    }
    await this.repo.save(order);
  }
}
```

---

## 📦 Repository

El **Repository** es la capa que separa el dominio de la infraestructura (DB, APIs).

### Características:

| Propiedad | Valor |
|-----------|-------|
| ¿Pertenece al dominio? | ✅ La interfaz sí |
| ¿Accede a DB? | ✅ La implementación |
| ¿Guarda agregados completos? | ✅ Sí |
| ¿Usa el Aggregate Root? | ✅ Siempre |

### Ejemplo: interfaz (en `domain/`)

```ts
export interface IOrderRepository {
  findById(id: string): Promise<Order | null>;
  save(order: Order): Promise<void>;
}
```

### Ejemplo: implementación (en `database/`)

```ts
export class OrderRepository implements IOrderRepository {
  async findById(id: string): Promise<Order | null> {
    const row = await db.query.orders.findFirst({ where: eq(orders.id, id) });
    return row ? new Order(row.id) : null;
  }

  async save(order: Order): Promise<void> {
    await db.insert(orders).values({ id: order.id, total: order.getTotal() });
  }
}
```

---

## 🧱 Aggregate

Un **Aggregate** es una unidad consistente de reglas de negocio.

Contiene:

- Una **Aggregate Root**
- Entidades internas (opcional)
- Value Objects (opcional)

### Características:

| Propiedad | Valor |
|-----------|-------|
| ¿Tiene una Root? | ✅ Sí |
| ¿Tiene otras entidades? | ✅ A veces |
| ¿Tiene lógica de negocio? | ✅ Sí |
| ¿Se modifica por fuera? | ❌ No, solo a través del Root |

### Ejemplo:

```ts
export class Order {
  private items: OrderItem[] = [];
  private total: Price = Price.create(0);

  addItem(item: OrderItem) {
    this.items.push(item);
    this.total = this.total.add(item.getTotal());
  }

  getTotal(): number {
    return this.total.getValue();
  }
}
```

---

## 🔁 Relación entre los tres

```plaintext
Application Service
  ↓
Domain (Aggregate / VO / Domain Service)
  ↓
Repository
  ↔ DB / Infraestructura
```

---

## 📦 Ubicación recomendada en monorepo

| Tipo              | Ubicación sugerida                  |
|-------------------|-------------------------------------|
| Application Service | `packages/application/<context>/` |
| Repository interface | `packages/domain/<context>/`     |
| Repository impl     | `packages/database/<context>/`    |
| Aggregates         | `packages/domain/<context>/`       |

---

¿Dudas? Recuerda:

- ¿Orquestación y flujo? → Application Service
- ¿Persistencia de aggregates? → Repository
- ¿Reglas de negocio centrales? → Aggregate Root
