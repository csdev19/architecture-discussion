# 🧭 Bounded Contexts en DDD: Guía Completa + Ejemplo real

## 📖 ¿Qué es un Bounded Context?

Un **Bounded Context** (o Contexto Delimitado) es una **frontera explícita** dentro de tu sistema donde:

- El modelo del dominio **tiene un significado coherente y consistente**
- Se define un **lenguaje específico** (Ubiquitous Language)
- Las reglas, entidades y casos de uso **no se mezclan con otros contextos**

> ✅ Dentro de un Bounded Context, `Account`, `User`, `Invoice`, etc., tienen un único significado  
> ❌ Fuera de ese contexto, pueden representar cosas diferentes

---

## 🧠 ¿Cómo se relaciona con el resto del sistema?

Cada Bounded Context define su propio set de objetos:

| Elemento             | Vive dentro del contexto |
|----------------------|--------------------------|
| Entities             | ✅                       |
| Value Objects        | ✅ (a veces compartidos si es seguro) |
| Aggregates           | ✅                       |
| Repositories         | ✅                       |
| Domain Services      | ✅                       |
| Application Services | ✅                       |

---

## 🧪 Estructura sugerida en un monorepo

```bash
packages/
  domain/
    scheduling/
    billing/
    identity/
    business/
  application/
    scheduling/
    billing/
    identity/
    business/
```

---

## ✅ ¿Qué problema resuelve?

Evita colisiones semánticas como:

- Mismo nombre, distintos significados
- Entidades con demasiadas responsabilidades
- Mezclas caóticas de lógica entre módulos

---

## 🎯 Ejemplo práctico: Mismo término, distintos significados

### 🔷 `Identity` Context

Maneja:
- Login
- Roles
- Seguridad

Lenguaje:
- `User`: persona que inicia sesión
- `Account`: credenciales de acceso

### 🟢 `BusinessManagement` Context

Maneja:
- Empresas, colaboradores, negocios

Lenguaje:
- `User`: empleado de una empresa
- `Account`: una empresa registrada

---

### 🤯 Conflicto

| Término    | Identity                     | BusinessManagement            |
|------------|------------------------------|-------------------------------|
| `User`     | Persona autenticada          | Colaborador del negocio       |
| `Account`  | Objeto de autenticación      | Empresa / organización        |

---

## ✅ ¿Cómo lo resuelve DDD?

Separando por **Bounded Contexts** y usando nombres explícitos.

### 🧱 En el dominio

```ts
// domain/identity/user.entity.ts
export class IdentityUser {
  constructor(
    public readonly id: string,
    private email: Email,
    private password: EncryptedPassword
  ) {}

  authenticate(password: string): boolean {
    return this.password.matches(password);
  }
}
```

```ts
// domain/business/user.entity.ts
export class BusinessUser {
  constructor(
    public readonly id: string,
    public readonly role: 'owner' | 'collaborator',
    public readonly companyId: string
  ) {}

  isOwner(): boolean {
    return this.role === 'owner';
  }
}
```

---

### 🧪 En la aplicación

```ts
// application/identity/authenticate-user.ts
const user = identityUserRepo.findByEmail(email);
if (!user.authenticate(password)) throw new Error('Unauthorized');
```

```ts
// application/business/get-collaborators.ts
const users = businessUserRepo.findByCompanyId(companyId);
```

---

## 🧠 Estrategias para evitar confusión

| Estrategia                     | Ejemplo                            |
|--------------------------------|-------------------------------------|
| Prefijar nombres               | `IdentityUser`, `BusinessUser`     |
| Separar archivos y carpetas    | `domain/identity/`, `domain/business/` |
| Usar tipos estrictos           | `UserId` en ambos contextos, diferentes |
| Comunicar entre contextos por eventos o contratos | `UserLoggedInEvent`, `UserDTO` |

---

## 🔁 Comunicación entre contextos

- Por eventos:

```plaintext
IdentityUserLoggedIn → Event
BusinessContext → lo escucha y reacciona
```

- Por contratos (DTOs, APIs):

```ts
interface IdentityUserDTO {
  id: string;
  email: string;
}

const externalUser = identityApi.getUser(authId);
const businessUser = userMapper.fromIdentityUser(externalUser);
```

---

## ✅ Conclusión

- Un **Bounded Context** protege el modelo del dominio de ambigüedades
- Puedes usar los mismos nombres (`User`, `Account`) sin generar conflictos
- Te permite **escalar**, **separar equipos**, y **pensar con claridad**
