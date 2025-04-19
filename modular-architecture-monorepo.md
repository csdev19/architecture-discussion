# 🧱 Arquitectura Modular en Monorepo – NestJS + DDD + Turbo + Drizzle

Este documento resume cómo organizar tu monorepo usando principios de arquitectura limpia y Domain-Driven Design, aprovechando las ventajas de NestJS, Drizzle y Turborepo.

---

## 📁 Estructura general del monorepo

```bash
apps/
  api/                  ← NestJS app principal
  cron/                 ← Jobs, CLI, workers, etc.
  admin-web/            ← Web pública o dashboard

packages/
  domain/               ← Entidades, Value Objects, interfaces
  application/          ← Casos de uso (use-cases)
  database/             ← Drizzle + repositorios concretos
  services/             ← Adaptadores a servicios externos (email, S3, etc.)
  shared/               ← Tipos globales, validadores, helpers

```

## 🧠 ¿Qué contiene cada capa?

| Capa | Contenido principal | Ubicación recomendada | 
|---|---|---|
| domain/ | Entidades, Value Objects, interfaces de repositorio | packages/domain | 
| application/ | Casos de uso, lógica orquestada que usa el dominio | packages/application |
| database/ | Implementación concreta de repositorios, usa Drizzle o Prisma | packages/database | 
| services/ | Adaptadores a servicios externos (Email, S3, Notificaciones, etc.) | packages/services | 
| infrastructure/ | Módulos de NestJS, controllers, DI de servicios | apps/api u otras apps | 


## 🧪 Ejemplo: CreateAccount como caso de uso puro y desacoplado

packages/domain/accounts/account.entity.ts

```
export class Account {
  constructor(public readonly id: string, public readonly email: Email) {}

  activate() {
    // lógica de dominio
  }
}
```

packages/application/accounts/create-account.ts

```
export class CreateAccount {
  constructor(
    private readonly accountRepo: IAccountRepository,
    private readonly emailService: EmailService
  ) {}

  async execute(input: CreateAccountInput) {
    const account = Account.create(input.email);
    await this.accountRepo.save(account);
    await this.emailService.sendWelcome(account.email);
  }
}

```

packages/database/account.repository.ts

```
export class AccountRepository implements IAccountRepository {
  async save(account: Account): Promise<void> {
    await db.insert(accounts).values({ ... });
  }
}

```


packages/services/email.service.ts

```
export class EmailService {
  async sendWelcome(email: string) {
    // lógica de envío real
  }
}

```

## 🧩 En NestJS (apps/api)

apps/api/src/modules/accounts/accounts.module.ts

```
@Module({
  controllers: [AccountsController],
  providers: [
    CreateAccountProvider,      // viene de application
    AccountRepositoryProvider,  // implementado en database
    EmailServiceProvider        // implementado en services
  ],
})
export class AccountsModule {}

```

apps/api/src/modules/accounts/accounts.controller.ts

```
@Controller('accounts')
export class AccountsController {
  constructor(private readonly createAccount: CreateAccount) {}

  @Post()
  async create(@Body() dto: CreateAccountInput) {
    await this.createAccount.execute(dto);
    return { message: 'Cuenta creada' };
  }
}

```


## 🎁 Beneficios de esta arquitectura

| Beneficio | ¿Qué logras? |
|--|---|
| 🔁 Reutilización | Puedes usar los casos de uso en múltiples apps (API, workers, CLI) |
| 🧼 Separación | Cada capa tiene su responsabilidad clara |
| 🧪 Testeabilidad | Puedes testear el dominio y los use-cases sin NestJS ni base de datos |
| 🔄 Sustituibilidad | Puedes cambiar Drizzle por Prisma, o EmailService por Sendgrid |
| 📦 Escalabilidad | Puedes dividir el proyecto en equipos por contexto o feature |



## 🧠 Convención de uso recomendada

-	Los casos de uso (application) orquestan lógica de negocio.
-	Los repositorios (database) implementan contratos definidos en domain.
-	Los servicios (services) son adaptadores externos a cosas como email, sms, etc.
-	Todo el código en packages/ es puro y sin dependencias de NestJS.
-	NestJS solo hace de coordinador (DI, HTTP, módulos).


## 🛠 Bonus con Turborepo + pnpm

En el pnpm-workspace.yaml:

```
packages:
  - "apps/*"
  - "packages/*"
```

Y en turbo.json::

```
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "dev": {
      "cache": false
    }
  }
}
```



