# 📘 5. Repository – Domain-Driven Design

---

## ¿Qué es un Repository?

Un **Repository** es un patrón que actúa como **puente entre el dominio y la infraestructura de datos**.  
Su propósito es:

- **Cargar y guardar Aggregates o Entities**
- Esconder los detalles de persistencia (ORM, SQL, APIs, etc.)
- **Simular una colección en memoria** (como si tus entidades vivieran en RAM)

> “Desde el punto de vista del dominio, es como tener una lista de objetos, aunque estén en una base de datos.”

---

## ¿Qué hace un Repository?

| Acción            | Ejemplo en código         |
|-------------------|---------------------------|
| Guardar           | `userRepository.save(user)` |
| Recuperar por ID  | `userRepository.findById(id)` |
| Borrar (opcional) | `userRepository.delete(id)`  |

Pero **no** debe usarse para consultas complejas, filtros ni joins optimizados para UI. Eso va en otro lugar.

---

## ¿Qué NO debe hacer?

- No debe devolver DTOs
- No debe devolver raw data de SQL
- No debe incluir lógica de presentación
- No debe hacer múltiples joins innecesarios

---

## Ejemplo de Repository de Usuario

### Interfaz (capa de dominio)

```ts
export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
}
```

### Implementación (infraestructura con ORM)

```ts
// originalmente era PgUserRepository, pero lo quise actualizar a UserRepository
export class UserRepository implements IUserRepository {
  async findById(id: string): Promise<User | null> {
    const row = await db.query.users.findFirst({ where: eq(users.id, id) });
    return row ? new User(row.id, row.name, Email.create(row.email)) : null;
  }

  async save(user: User): Promise<void> {
    await db.insert(users).values({
      id: user.id,
      name: user.getName(),
      email: user.getEmail().getValue()
    });
  }
}
```

---

## ¿Cómo se usa un Repository con una Entity + Value Object?

### Value Object: Email

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

### Entity: User

```ts
export class User {
  constructor(
    public readonly id: string,
    private name: string,
    private email: Email
  ) {}

  changeName(newName: string): void {
    if (newName.length < 2) {
      throw new Error('Nombre inválido');
    }
    this.name = newName;
  }

  getName(): string {
    return this.name;
  }

  getEmail(): Email {
    return this.email;
  }
}
```

---

## Cómo mapear datos del ORM a entidades del dominio

Cuando usas ORMs como Prisma o Drizzle, **no devuelvas directamente los objetos del ORM**.  
En su lugar, mapea la data manualmente para reconstruir una instancia de tu Entity con sus Value Objects.

```ts
const row = await db.query.users.findFirst({ where: eq(users.id, id) });

return row
  ? new User(
      row.id,
      row.name,
      Email.create(row.email)
    )
  : null;
```

Este patrón asegura que:

- Tu dominio siga **desacoplado del ORM**
- La entidad se mantenga **válida y completa**
- Puedes testear, mutar y usar sus métodos normalmente

---

### Bonus: usar un mapper si se repite mucho

```ts
export class UserMapper {
  static toEntity(row: any): User {
    return new User(row.id, row.name, Email.create(row.email));
  }

  static toPersistence(user: User): any {
    return {
      id: user.id,
      name: user.getName(),
      email: user.getEmail().getValue()
    };
  }
}
```

---

## Reglas del patrón Repository

| Regla                                   | ¿Por qué? |
|----------------------------------------|-----------|
| Solo trabaja con Entities o Aggregates | Para proteger la capa de dominio |
| No devuelve estructuras planas         | Eso lo hace un QueryService |
| No se usa desde los controladores      | Lo usan los Application Services o Commands |
| Su implementación puede variar         | Pero su interfaz se mantiene estable |

---

## Ubicación sugerida en tu monorepo

```bash
packages/
  domain/
    users/
      user.entity.ts
      user.repository.ts     ← interfaz
  infrastructure/
    users/
      user.repository.pg.ts  ← implementación concreta
```

---

## Conclusión

El **Repository** es esencial para:

- Separar el dominio de la base de datos
- Mantener limpio tu modelo
- Proteger tus Aggregates y sus reglas
- Facilitar el testing y el mantenimiento

Si una Entity vive en el dominio, su persistencia debe estar encapsulada en un Repository.


## Notas extra:

hay temas de no hemos tocado 
