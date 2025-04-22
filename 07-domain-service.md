# 📘 7. Domain Service – Domain-Driven Design

---

## 📖 ¿Qué es un Domain Service?

Un **Domain Service** es una clase dentro del dominio que:

- **Contiene lógica de negocio importante**
- **No pertenece naturalmente a ninguna Entity o Value Object**
- Es **stateless** (no guarda estado interno)
- Usa y coordina **varias entidades o VOs** para cumplir una regla del negocio

> 🧠 Si la lógica no tiene una casa clara entre tus Entities, probablemente es un Domain Service.

---

## 🎯 ¿Cuándo usar un Domain Service?

| Situación                                             | ¿Usar Domain Service?                      |
| ----------------------------------------------------- | ------------------------------------------ |
| La lógica depende de más de una Entity                | ✅ Sí                                      |
| No hay una entidad clara que pueda ejecutar la lógica | ✅ Sí                                      |
| Solo es lógica técnica, sin reglas del negocio        | ❌ No (eso es Application Service o Infra) |
| La lógica es interna de una entidad                   | ❌ No (debe ir en esa Entity)              |

---

## ✅ Ejemplo clásico: transferencia de saldo

### Entity: `Account`

```ts
export class Account {
  constructor(public readonly id: string, private balance: number) {}

  debit(amount: number): void {
    if (this.balance < amount) {
      throw new Error("Fondos insuficientes");
    }
    this.balance -= amount;
  }

  credit(amount: number): void {
    this.balance += amount;
  }

  getBalance(): number {
    return this.balance;
  }
}
```

### Domain Service

```ts
export class TransferService {
  transfer(from: Account, to: Account, amount: number): void {
    from.debit(amount);
    to.credit(amount);
  }
}
```

---

## 🧪 Ejemplo completo: Crear una orden con reglas + notificación

### Entity: `Order`

```ts
export class Order {
  private total: number;
  private isPriority: boolean = false;

  constructor(
    public readonly id: string,
    private customerEmail: string,
    private items: { productId: string; price: number; qty: number }[]
  ) {
    this.total = items.reduce((sum, i) => sum + i.price * i.qty, 0);
  }

  markAsPriority() {
    this.isPriority = true;
  }

  getTotal(): number {
    return this.total;
  }

  getCustomerEmail(): string {
    return this.customerEmail;
  }

  isHighPriority(): boolean {
    return this.isPriority;
  }
}
```

---

### Domain Service: `OrderEvaluator`

```ts
export class OrderEvaluator {
  static PRIORITY_THRESHOLD = 500;

  static evaluate(order: Order) {
    if (order.getTotal() > OrderEvaluator.PRIORITY_THRESHOLD) {
      order.markAsPriority();
    }
  }
}
```

---

### Servicio técnico: `EmailService`

```ts
export interface IEmailService {
  send(to: string, subject: string, body: string): Promise<void>;
}

export class SimpleEmailService implements IEmailService {
  async send(to: string, subject: string, body: string) {
    console.log(`Enviando correo a ${to}: ${subject}`);
  }
}
```

---

### Application Service

```ts
export class CreateOrder {
  constructor(
    private readonly orderRepo: IOrderRepository,
    private readonly emailService: IEmailService
  ) {}

  async execute(input: {
    customerEmail: string;
    items: { productId: string; price: number; qty: number }[];
  }): Promise<string> {
    const order = new Order(
      crypto.randomUUID(),
      input.customerEmail,
      input.items
    );

    OrderEvaluator.evaluate(order); // dominio

    await this.orderRepo.save(order);

    await this.emailService.send(
      order.getCustomerEmail(),
      "Tu orden fue recibida",
      order.isHighPriority()
        ? "Gracias por tu compra. Tu orden es prioritaria 🚀"
        : "Gracias por tu compra. Pronto la procesaremos."
    );

    return order.id;
  }
}
```

---

## 📦 Estructura de carpetas sugerida

```bash
packages/
  domain/
    orders/
      order.entity.ts
      order-evaluator.service.ts    ← Domain Service
  application/
    orders/
      create-order.use-case.ts      ← Application Service
  infrastructure/
    email/
      email.service.ts              ← Servicio técnico
  interfaces/
    http/
      orders.controller.ts
```

---

## ✅ Reglas del Domain Service

| Regla                               | Descripción                                |
| ----------------------------------- | ------------------------------------------ |
| No tiene estado interno             | Es stateless                               |
| Su único rol es lógica del dominio  | No depende de detalles técnicos            |
| No reemplaza a las Entities         | Solo se usa cuando no hay una Entity clara |
| Se invoca desde Application Service | Nunca desde controller directamente        |

---

## ❌ Errores comunes

- Usar Domain Service como un Service técnico (como enviar emails)
- Mezclar lógica de presentación con lógica de dominio
- Agregar estado o dependencias externas
- Llamarlo desde el controller directamente

---

## ✅ Conclusión

Un **Domain Service** existe para **reunir lógica de negocio significativa** que no pertenece a ninguna Entity por sí sola, pero **sí al dominio**.

> Usa un Domain Service si:
>
> - Estás cruzando límites de entidades
> - No quieres duplicar lógica entre Aggregates
> - Estás expresando una operación del dominio, no una operación técnica
