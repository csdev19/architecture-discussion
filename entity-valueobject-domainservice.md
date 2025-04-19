# 📘 Guía de Conceptos del Dominio: Entity, Value Object, Domain Service

Esta guía resume **cuándo y cómo** usar los elementos principales del dominio en una arquitectura DDD:

- Entity
- Value Object (VO)
- Domain Service

---

## ✅ ¿Qué es una Entity?

Una **Entity** es un objeto del dominio que:

- Tiene una **identidad única** (`id`)
- Cambia en el tiempo
- Contiene comportamiento y reglas
- Se compara por su `id`

### Ejemplo:

```ts
export class Account {
  constructor(public readonly id: string, private balance: Money) {}

  debit(amount: Money) {
    this.balance = this.balance.subtract(amount);
  }
}
```

---

## ✅ ¿Qué es un Value Object (VO)?

Un **Value Object** es:

- Inmutable
- Sin identidad
- Se compara por valor
- Puede contener lógica

### Reglas clave:

| Característica     | ¿Aplica a VO? |
|--------------------|---------------|
| Tiene ID           | ❌ No         |
| Inmutable          | ✅ Sí         |
| Lógica interna     | ✅ Sí         |
| Se compara por valor | ✅ Sí      |

### Ejemplo:

```ts
export class Money {
  private constructor(private readonly value: number) {}

  static create(value: number): Money {
    if (value < 0) throw new Error('Invalid amount');
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

## ✅ ¿Qué es un Domain Service?

Un **Domain Service** se usa cuando:

- La lógica **no pertenece claramente a una sola Entity**
- La operación involucra **múltiples entidades o VOs**
- Se debe encapsular una **regla importante del dominio**
- Es **stateless**

### Reglas clave:

| Característica     | ¿Aplica a Domain Service? |
|--------------------|---------------------------|
| Tiene estado propio | ❌ No                    |
| Accede a entidades  | ✅ Sí                    |
| Contiene lógica de negocio | ✅ Sí             |
| Se encarga de orquestar reglas del dominio | ✅ Sí |

### Ejemplo:

```ts
export class TransferService {
  transfer(from: Account, to: Account, amount: Money) {
    if (from.getBalance().lessThan(amount)) {
      throw new Error('Fondos insuficientes');
    }

    from.debit(amount);
    to.credit(amount);
  }
}
```

---

## 🎯 ¿Un booleano es un VO?

Por sí solo, **no**. Pero **puede ser envuelto en un VO** si tiene lógica del dominio.

### Ejemplo:

```ts
export class ActiveStatus {
  private constructor(private readonly active: boolean) {}

  static active() {
    return new ActiveStatus(true);
  }

  static inactive() {
    return new ActiveStatus(false);
  }

  toggle(): ActiveStatus {
    return new ActiveStatus(!this.active);
  }

  isActive(): boolean {
    return this.active;
  }
}
```

---

## ✅ ¿Qué NO debe contener el Application Service?

| No es su responsabilidad... | Porque... |
|-----------------------------|-----------|
| Validar lógica del dominio  | Eso va en entidades o domain services |
| Manipular directamente objetos sin reglas | Puede romper invariantes |
| Acceder a la DB directamente | Lo hace a través de repositorios |

---

## 📦 Ubicación sugerida en tu monorepo

| Tipo             | Ubicación                           |
|------------------|-------------------------------------|
| Entities         | `packages/domain/<context>/`        |
| Value Objects    | `packages/domain/shared/value-objects/` |
| Domain Services  | `packages/domain/<context>/services/` |

---

## 🔁 Recordatorio visual

```plaintext
Application Service → llama a → Domain Service / Entity / VO
                        ↓
                     Repository ←→ DB
```

---

¿Tienes dudas sobre qué patrón usar para un caso específico?
Recuerda:  
- ¿La lógica pertenece a una sola entidad? → Entity  
- ¿Es un dato atómico y sin identidad? → VO  
- ¿La lógica cruza varios objetos y no cabe en una sola parte? → Domain Service
