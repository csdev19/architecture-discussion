# 📘 2. Value Object (VO) – Domain-Driven Design

---

## 📖 ¿Qué es un Value Object?

Un **Value Object (VO)** es un tipo del dominio que:

- ❌ No tiene identidad (`id`)
- ✅ Es inmutable
- ✅ Se compara por valor
- ✅ Contiene su propia lógica de validación o transformación
- ✅ Puede tener comportamiento propio, pero siempre acotado

> Su objetivo es representar un concepto atómico del dominio que tenga reglas claras y no necesite ser rastreado por sí solo.

---

## 🧠 Ejemplos comunes de VOs

| VO            | Qué representa               |
|---------------|------------------------------|
| `Email`       | Una dirección válida de correo electrónico |
| `Money`       | Monto + moneda, con validación y cálculo |
| `DateRange`   | Intervalo entre dos fechas   |
| `PhoneNumber` | Número con formato específico |
| `Slug`        | Identificador legible en URLs |

---

## 💡 Características clave

| Propiedad            | ¿Aplica a VO? |
|----------------------|----------------|
| Tiene `id`           | ❌ No           |
| Es inmutable         | ✅ Sí           |
| Se compara por valor | ✅ Sí           |
| Tiene lógica interna | ✅ Sí           |
| Puede vivir en DB    | ⚠️ Solo como parte de una Entity |

---

## 💻 Ejemplo de VO: `Email`

```ts
export class Email {
  private constructor(private readonly value: string) {}

  static create(value: string): Email {
    const normalized = value.trim().toLowerCase();
    if (!normalized.includes('@')) {
      throw new Error('Email inválido');
    }
    return new Email(normalized);
  }

  getValue(): string {
    return this.value;
  }

  equals(other: Email): boolean {
    return this.value === other.value;
  }
}
```

---

## 🧪 Ejemplo de uso

```ts
const email1 = Email.create('Usuario@Dominio.com');
const email2 = Email.create('usuario@dominio.com');

console.log(email1.equals(email2)); // true

const email3 = Email.create('otro@dominio.com');
console.log(email1.equals(email3)); // false
```

---

## ✅ Buenas prácticas con VO

| Recomendación                          | ¿Por qué? |
|---------------------------------------|-----------|
| Inmutables                            | Evita errores de estado inesperado |
| Encapsulan su lógica                  | Centraliza validación y normalización |
| Se crean con un `create()` o `from()` | Controlan el estado desde el origen |
| Se comparan con `equals()`            | No uses `===`, ya que son instancias distintas |

---

## 🔐 ¿Por qué usar VOs en lugar de primitivos?

> En vez de usar `string` para emails o `number` para montos…

```ts
// ❌ vulnerable y sin validación
class Account {
  constructor(public readonly email: string) {}
}

// ✅ fuerte y validado
class Account {
  constructor(public readonly email: Email) {}
}
```

Esto te permite:

- Reutilizar lógica
- Evitar bugs
- Hacer que el modelo del dominio sea más expresivo

---

## 🔁 Los VOs deben devolver **nuevas instancias**

> Porque son inmutables, nunca debes hacer:

```ts
email.value = 'nuevo@correo.com' // ❌ NO
```

Sino:

```ts
const newEmail = Email.create('nuevo@correo.com') // ✅
```

---

## 🔎 ¿Un booleano puede ser un VO?

Por sí solo, no.  
Pero si tiene semántica y lógica en el dominio, sí puede y debe ser un VO.

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

## 📦 Ubicación sugerida en tu monorepo

```bash
packages/
  domain/
    shared/
      value-objects/
        email.vo.ts
        money.vo.ts
        date-range.vo.ts
```

---

## ✅ Conclusión

Un **Value Object** representa un dato rico con reglas y semántica.  
No es solo “un string” o “un number”. Es una parte importante del lenguaje del dominio.

> Si una pieza de información:
> - No necesita ser identificada
> - Tiene reglas
> - Y puede compararse por su valor  
> → entonces es un **VO perfecto**
