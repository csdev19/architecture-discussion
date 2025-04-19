# 📘 1. Entity (Entidad) – Domain-Driven Design

---

## 📖 ¿Qué es una Entity?

Una **Entity** en Domain-Driven Design es un objeto del dominio que:

- ✅ Tiene una **identidad única** (`id`) que no cambia a lo largo del tiempo
- ✅ Tiene un **ciclo de vida** (puede cambiar su estado o propiedades)
- ✅ Contiene **comportamiento del negocio**
- ❌ No se compara por atributos, sino por su `id`

> 📌 Ejemplo real: un `User`, una `Order`, una `Company`.  
Aunque cambien sus propiedades, siguen siendo **la misma entidad** si tienen el mismo ID.

---

## 🧠 Diferencia con Value Object

| Característica          | Entity                    | Value Object            |
|-------------------------|---------------------------|-------------------------|
| Tiene `id`              | ✅ Sí                     | ❌ No                   |
| Puede mutar             | ✅ Sí                     | ❌ No (inmutable)       |
| Comparación             | Por identidad (ID)        | Por valor (atributos)   |
| Ciclo de vida           | ✅ Tiene                  | ❌ No aplica            |
| Tiene comportamiento    | ✅ Complejo               | ✅ Acotado              |

---

## 💻 Ejemplo de Entity: `Account`

```ts
import { Email } from '../shared/value-objects/email.vo';

export class Account {
  constructor(
    public readonly id: string,
    private email: Email,
    private isActive: boolean = false
  ) {}

  activate() {
    if (this.isActive) throw new Error('Cuenta ya activa');
    this.isActive = true;
  }

  suspend() {
    this.isActive = false;
  }

  getEmail(): string {
    return this.email.getValue();
  }

  isActivated(): boolean {
    return this.isActive;
  }
}
```

---

## 🧪 Ejemplo de uso

```ts
const email = Email.create('cliente@correo.com');
const account = new Account('abc-123', email);

account.activate();
console.log(account.isActivated()); // true

account.suspend();
console.log(account.isActivated()); // false
```

---

## ✅ Buenas prácticas con Entities

| Recomendación                          | ¿Por qué? |
|---------------------------------------|-----------|
| Usa un `id` único                     | Para comparar correctamente y persistir |
| Encapsula estado mediante métodos     | Protege invariantes |
| No expongas propiedades directamente  | Evita inconsistencias desde fuera |
| Usa Value Objects como atributos      | Para mayor expresividad y validación |

---

## 🛡️ Gestión de identidad en Entities

### 🎯 ¿Qué es el `id` en una Entity?

- Es el **identificador único** del objeto en el dominio
- Es lo que permite decir si **dos instancias son la misma cosa**
- Debe ser **estable**, **persistente**, y **no cambiante**

---

## 🧠 ¿UUID o ID de base de datos?

| Estrategia               | Cuándo usarla                           | Ventajas                     | Consideraciones        |
|--------------------------|-----------------------------------------|------------------------------|------------------------|
| ✅ UUID generado por el dominio | Siempre que quieras identidad desde el inicio | Desacopla del motor de base de datos | UUIDs son más largos  |
| ⚠️ SERIAL / AUTO_INCREMENT     | Sistemas chicos o tightly-coupled    | Simple de manejar             | El ID se obtiene después del insert |
| ✅ VO para ID (`UserId`)        | Si quieres tipado fuerte + semántica | Legibilidad + protección de tipo | Agrega capa de código, pero vale la pena |

---

### 🧱 Ejemplo recomendado con UUID + VO

```ts
import { randomUUID } from 'crypto';

export class AccountId {
  private constructor(private readonly value: string) {}

  static create(value?: string): AccountId {
    return new AccountId(value ?? randomUUID());
  }

  equals(other: AccountId): boolean {
    return this.value === other.value;
  }

  toString(): string {
    return this.value;
  }
}
```

Y en tu Entity:

```ts
export class Account {
  constructor(public readonly id: AccountId, private email: Email) {}
}
```

---

## 🧾 Integración con bases de datos relacionales

### 🧩 Si el dominio genera el ID (DDD-style)

**En la base de datos (PostgreSQL):**

```sql
CREATE TABLE accounts (
  id UUID PRIMARY KEY,
  email TEXT NOT NULL
);
```

**En código:**

```ts
await db.insert(accounts).values({
  id: account.id.toString(),
  email: account.getEmail(),
});
```

---

### 🧩 Si la base de datos genera el ID

**En la DB (PostgreSQL):**

```sql
CREATE TABLE accounts (
  id SERIAL PRIMARY KEY,
  email TEXT NOT NULL
);
```

**En el repositorio:**

```ts
const [result] = await db.insert(accounts).values({ email }).returning();
const account = Account.withId(result.id, Email.create(email));
```

---

## ✅ Conclusión

> En DDD, una Entity **representa un objeto del negocio que tiene identidad y comportamiento**.  
> Para hacerlo bien:
- Usa un `id` persistente y consistente (idealmente como VO)
- No mezcles responsabilidades
- Encapsula su estado
- Y no dejes que la infraestructura controle su identidad (si puedes evitarlo)
