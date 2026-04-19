# Layer Boundaries

## Feature Structure

```
lib/features/{context}/
  domain/
    models/                    <- Domain models (Equatable)
    enums/                     <- Enum value objects
    value_objects/             <- Validated VOs
    exceptions/                <- Feature-specific exceptions
    converters/                <- Custom JSON converters
    repositories/              <- Abstract repository interfaces
    use_cases/                 <- One class per business action
    services/                  <- Domain services (orchestration)
    bindings/                  <- Domain-level GetX binding

  data/
    datasources/
      local/
        daos/                  <- Floor DAO interfaces
        {xxx}_local_datasource.dart
      remote/
        {xxx}_remote_datasource.dart
    dtos/                      <- API data transfer objects
    entities/                  <- Floor database entities
    mappers/                   <- Static DTO <-> Entity <-> Domain converters
    repositories/              <- Repository implementations
    bindings/                  <- Data-level GetX binding

  presentation/
    controllers/               <- GetX controllers (extend BaseController)
    screens/                   <- Full-page widgets
    widgets/                   <- Feature-specific widgets
    bindings/                  <- Presentation-level GetX binding
```

---

## Forbidden Imports

| Layer | May NOT import |
|---|---|
| `domain/models`, `domain/use_cases` | `data/*`, `presentation/*`, `package:get/*`, `package:floor/*` |
| `data/repositories` | `presentation/*` |
| `presentation/controllers` | `data/datasources/*`, `data/entities/*`, `data/dtos/*` |

---

## Allowed Import Directions

```
Presentation ──> Domain (use cases, repositories interfaces, models)
Data ──────────> Domain (repositories interfaces, models)
Domain ────────> (nothing outside domain, only dart:core + equatable)
```

Domain has zero dependencies on outer layers. Always.

---

## Bounded Contexts

| Context key | Feature root |
|---|---|
| `user_management` | `lib/features/user_management/` |
| `device_management` | `lib/features/device_management/` |
| `time_tracking` | `lib/features/time_tracking/` |
| `sync` | `lib/features/sync/` |
| `face_authentication` | `lib/features/face_authentication/` |
| `data_management` | `lib/features/data_management/` |
| `app_update` | `lib/features/app_update/` |

---

## Cross-Cutting Code

| Directory | Purpose |
|---|---|
| `lib/core/` | Shared infra: HttpClient, AppLogger, BaseController, EventBus, Floor DB |
| `lib/shared/` | Shared domain: SyncableEntity mixin, SyncStatus, DomainId |
| `lib/utils/` | UI utilities: components, extensions, constants |

---

## DB Table Naming

Floor entities use snake_case table names:

| Context | Example table |
|---|---|
| user_management | `users`, `employees` |
| device_management | `devices` |
| time_tracking | `time_logs` |
| sync | `sync_metadata` |
