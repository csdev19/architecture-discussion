# 📘 4. Aggregate Root – Domain-Driven Design

---

## ¿Qué es el Aggregate Root?

El **Aggregate Root** es la **entidad principal** dentro de un Aggregate.  
Actúa como:

- La **única entrada** al Aggregate
- El **guardián de las invariantes**
- El **único objeto que puede ser accedido desde el exterior**
- El **objeto que se guarda y carga desde un Repository**

> Si el Aggregate es el universo, el Root es la puerta única de entrada y salida.

---

## ¿Por qué es tan importante?

Porque garantiza que:

- Las reglas del negocio se apliquen correctamente
- El estado interno se mantenga válido
- No se modifiquen objetos internos sin pasar por las reglas del dominio

---

## Reglas clave del Aggregate Root

| Regla | ¿Por qué? |
|-------|-----------|
| Toda modificación del Aggregate debe pasar por el Root | Para validar reglas de negocio e invariantes |
| No se accede directamente a las entidades internas | Protege el encapsulamiento |
| El Repository guarda y carga solo el Root | El resto del Aggregate se reconstruye a partir del Root |
| El Root representa todo el Aggregate públicamente | Es la fachada del modelo de dominio |

---

## ¿Es una clase separada?

No necesariamente. El Root es una Entity, pero con una responsabilidad mayor:  
Representa al conjunto completo (el Aggregate), y su diseño y métodos están pensados para proteger y controlar el todo.

---

## ¿Cómo se ve en código?

### Aggregate Root: Order

```ts
export class Order {
  private items: OrderItem[] = [];
  private total: Money = Money.create(0);

  constructor(public readonly id: string) {}

  addItem(item: OrderItem) {
    const exists = this.items.find(i => i.productId === item.productId);
    if (exists) throw new Error('Item duplicado');

    this.items.push(item);
    this.total = this.total.add(item.getTotal());
  }

  getTotal(): number {
    return this.total.getValue();
  }

  getItems(): OrderItem[] {
    return this.items;
  }
}
```

> Esta clase actúa como Aggregate Root: contiene el estado completo y expone los métodos necesarios para modificarlo de forma válida.

---

## ¿Qué no deberías hacer?

Acceder directamente a las entidades internas o modificarlas sin pasar por el Root:

```ts
// ❌ Mala práctica
order.items.push(new OrderItem(...));

// ✅ Correcto
order.addItem(new OrderItem(...));
```

---

## ¿Cómo se diferencia de un Aggregate?

| Concepto         | Qué representa |
|------------------|----------------|
| Aggregate         | El conjunto completo: entidades, value objects, reglas |
| Aggregate Root    | La clase que gobierna ese conjunto y expone sus métodos |

---

## Ejemplo adicional: Company como Root

```ts
export class Company {
  private collaborators: User[] = [];

  constructor(public readonly id: CompanyId, private name: string) {}

  addCollaborator(user: User) {
    if (this.collaborators.find(u => u.id === user.id)) {
      throw new Error('Ya es colaborador');
    }

    this.collaborators.push(user);
  }

  removeCollaborator(userId: string) {
    this.collaborators = this.collaborators.filter(u => u.id !== userId);
  }

  getCollaborators(): User[] {
    return [...this.collaborators]; // proteger el array interno
  }
}
```

---

## 📦 Ubicación sugerida en tu monorepo

```bash
packages/
  domain/
    orders/
      order.aggregate.ts       ← contiene el Root
    companies/
      company.aggregate.ts     ← otro Root
    shared/
      value-objects/
        money.vo.ts
```

---

## Conclusión

El **Aggregate Root** es la pieza central del dominio en DDD:

- Es la interfaz pública del Aggregate
- Expone los métodos que el sistema puede usar
- Impone las reglas de negocio
- Protege el estado interno
- Y se convierte en el único punto de persistencia con el Repository

> Toda interacción con un Aggregate debe pasar por su Root.  
> Nunca modifiques objetos internos directamente.
