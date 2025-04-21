# 📘 6. Application Service – Domain-Driven Design

---

## ¿Qué es un Application Service?

Un **Application Service** es una capa que:

- Orquesta **interacciones entre objetos del dominio**
- Se encarga de **coordinar**, **validar entradas** y **ejecutar comandos**
- No contiene lógica de negocio en sí, pero **activa** la lógica del dominio

> 📌 No representa una entidad del negocio. Representa una **acción del sistema**.

---

## ¿Para qué sirve?

| Rol del Application Service       | Ejemplo                               |
| --------------------------------- | ------------------------------------- |
| Recibe una solicitud del exterior | Crear pedido, actualizar perfil       |
| Valida el input                   | ¿Faltan datos? ¿Tiene formato válido? |
| Usa repositorios o servicios      | Guarda entidades, envía eventos       |
| Orquesta múltiples acciones       | Crear, notificar, guardar, etc.       |

---

## ¿Qué NO debe hacer?

- Contener lógica de negocio compleja directamente
- Mutar el estado de Entities por fuera
- Preparar DTOs de presentación

---

## ¿Qué SÍ debe hacer?

- Validar input inicial
- Recuperar y persistir Aggregates
- Llamar métodos del dominio
- Ejecutar varios casos juntos si hace falta

---

## Ejemplo: Crear un usuario

### Entity y Value Object

```ts
export class Email {
  static create(value: string): Email {
    if (!value.includes("@")) throw new Error("Email inválido");
    return new Email(value.trim().toLowerCase());
  }
  constructor(private readonly value: string) {}
  getValue() {
    return this.value;
  }
}

export class User {
  constructor(
    public readonly id: string,
    private name: string,
    private email: Email
  ) {}

  getName(): string {
    return this.name;
  }
  getEmail(): Email {
    return this.email;
  }
}
```

---

### Repositorio

```ts
export interface IUserRepository {
  save(user: User): Promise<void>;
  existsByEmail(email: Email): Promise<boolean>;
}
```

---

### Application Service

```ts
export class CreateUser {
  constructor(private readonly userRepo: IUserRepository) {}

  async execute(input: { name: string; email: string }): Promise<string> {
    const email = Email.create(input.email);

    const exists = await this.userRepo.existsByEmail(email);
    if (exists) throw new Error("El email ya está en uso");

    const id = crypto.randomUUID();
    const user = new User(id, input.name, email);
    await this.userRepo.save(user);

    return id;
  }
}
```

---

## 📦 Ubicación sugerida en tu monorepo

```bash
packages/
  application/
    users/
      create-user.service.ts   ← Application Service
  domain/
    users/
      user.entity.ts
      user.repository.ts
  infrastructure/
    users/
      user.repository.pg.ts
  interfaces/
    http/
      users.controller.ts
```

---

## 📌 ¿Dónde debe ir la lógica de negocio?

### 🎯 Regla general

> ✅ Si la regla depende del negocio y del estado del modelo → va en el dominio (Entity o Domain Service).  
> ✅ Si la regla depende del input del usuario o del contexto externo → va en el Application Service.

---

## 🧱 Ejemplo realista: Sistema de pedidos

### ❌ Mal ejemplo (toda la lógica en el Application Service)

```ts
if (product.stock < input.quantity) {
  throw new Error("Stock insuficiente");
}
```

Esto rompe la encapsulación del dominio.

---

### ✅ Buen ejemplo (reglas en el dominio)

```ts
// order.entity.ts
export class Order {
  static create(product: Product, quantity: number): Order {
    if (!product.hasStockFor(quantity)) {
      throw new Error("No hay suficiente stock");
    }
    product.decreaseStock(quantity);
    return new Order(product.id, quantity);
  }

  constructor(
    public readonly productId: string,
    public readonly quantity: number
  ) {}
}

// product.entity.ts
export class Product {
  hasStockFor(qty: number): boolean {
    return this.stock >= qty;
  }

  decreaseStock(qty: number): void {
    if (!this.hasStockFor(qty)) {
      throw new Error("Stock insuficiente");
    }
    this.stock -= qty;
  }
}
```

---

### Application Service limpio

```ts
const product = await this.productRepo.findById(input.productId);
const order = Order.create(product, input.quantity);
await this.orderRepo.save(order);
```

---

## ✅ Conclusión

Un **Application Service**:

- No es parte del dominio
- No tiene reglas de negocio
- Pero es crítico para mantener limpia tu arquitectura
- Es el puente entre el mundo externo y tu modelo de dominio

> Usa esta capa para que tu dominio hable limpio, claro y sin ruidos de HTTP, formularios o bases de datos.
