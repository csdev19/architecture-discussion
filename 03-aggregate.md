# 📘 3. Aggregate – Domain-Driven Design

---

## ¿Qué es un Aggregate?

Un **Aggregate** es una **unidad de consistencia y transacción en el dominio**. Está compuesto por:

- Una **entidad principal** llamada **Aggregate Root**
- Otras entidades internas (opcional)
- Value Objects (opcional)
- Reglas del negocio que afectan al conjunto

Es la forma en que el dominio mantiene **invariantes**, protege el estado válido y asegura que todas las modificaciones pasen por un único punto de control.

---

## ¿Por qué existen los Aggregates?

Porque en sistemas complejos:

- Tienes grupos de entidades que están relacionadas
- Las acciones sobre una afectan a otras
- Necesitas evitar inconsistencias y efectos secundarios
- Necesitas guardar o modificar estas entidades como un **bloque coherente**

---

## Componentes de un Aggregate

- Aggregate Root (obligatorio)
- Entidades internas (opcional)
- Value Objects (opcional)

---

## Ejemplo de Aggregate: Order

Supongamos que estás modelando un sistema de pedidos (`Order`).

### Estructura

- `Order` → Aggregate Root
- `OrderItem[]` → entidad interna
- `Money` → Value Object usado como precio y total

---

### Value Object: Money

```ts
export class Money {
  private constructor(private readonly value: number) {}

  static create(value: number): Money {
    if (value < 0) throw new Error('Monto inválido');
    return new Money(value);
  }

  add(other: Money): Money {
    return Money.create(this.value + other.value);
  }

  getValue(): number {
    return this.value;
  }
}
```

---

### Entidad interna: OrderItem

```ts
export class OrderItem {
  constructor(
    public readonly productId: string,
    public readonly quantity: number,
    public readonly price: Money
  ) {}

  getTotal(): Money {
    return Money.create(this.price.getValue() * this.quantity);
  }
}
```

---

### Aggregate Root: Order

```ts
export class Order {
  private items: OrderItem[] = [];
  private total: Money = Money.create(0);

  constructor(public readonly id: string) {}

  addItem(item: OrderItem) {
    const exists = this.items.find(i => i.productId === item.productId);
    if (exists) throw new Error('Producto duplicado');

    this.items.push(item);
    this.total = this.total.add(item.getTotal());
  }

  getItems(): OrderItem[] {
    return this.items;
  }

  getTotal(): number {
    return this.total.getValue();
  }
}
```

---

## Ejemplo de uso

```ts
const order = new Order('order-123');

order.addItem(new OrderItem('p1', 2, Money.create(50))); // $100
order.addItem(new OrderItem('p2', 1, Money.create(30))); // $30

console.log(order.getTotal()); // 130
```

---

## Reglas clave de un Aggregate

- Toda modificación pasa por el Root
- Solo el Root es accesible desde afuera
- Se guarda o carga como unidad completa
- No se modifican entidades internas directamente

---

## Invariantes del negocio

Una **invariante** es una regla que siempre debe cumplirse.  
Ejemplo: “Una orden no puede tener ítems duplicados”.  
→ Esa lógica vive en el método `addItem()` del Root, no fuera del Aggregate.

---

## 📦 Ubicación sugerida en tu monorepo

```bash
packages/
  domain/
    orders/
      order.aggregate.ts         ← contiene el Root
      order-item.entity.ts       ← entidad interna
    shared/
      value-objects/
        money.vo.ts              ← VO compartido
```

---

## Conclusión

Un **Aggregate** te ayuda a:

- Proteger el dominio de inconsistencias
- Agrupar objetos que deben mantenerse sincronizados
- Exponer un punto de entrada único (el Root)
- Centralizar reglas complejas que cruzan múltiples entidades

Piénsalo como un pequeño universo del dominio, donde todo debe entrar y salir controladamente a través del Root.
