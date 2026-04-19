---
name: flutter_ddd
description: Domain-Driven Design guided generation for Gestime Flutter kiosk app. Use when implementing a new feature, aggregate, use case, or bounded context slice in the Flutter mobile project. Drives domain story → layer-by-layer code generation with confirmation at each step. Targets Clean Architecture + GetX + Floor ORM.
---

# Flutter DDD Skill — Gestime Mobile

## Philosophy

DDD is about **capturing the domain language in code**. Every class name, method name, and file name should be something a domain expert could recognize. Infrastructure details live in the data layer; intent lives in the domain.

This skill drives a structured loop:
1. **Narrate** the domain story
2. **Extract** DDD artifacts
3. **Generate** one layer at a time — confirm before each
4. **Wire** bindings for DI
5. **Audit** the result

---

## Phase 1: STORY

Ask the user to narrate a scenario. See [domain-story.md](domain-story.md).

From the story, extract and present:

| Artifact | Extracted |
|---|---|
| **Actors** | Who initiates the action |
| **Commands** | What they want to do (verb phrase) |
| **Queries** | What they want to know (read-only) |
| **Aggregates** | Nouns that own state and invariants |
| **Policies** | Business rules that must hold |
| **Value Objects** | Descriptive attributes with no identity |

Present extraction to user. Confirm before proceeding.

---

## Phase 2: PLACEMENT

Ask:
1. Which bounded context? → `user_management` / `device_management` / `time_tracking` / `sync` / `face_authentication` / `data_management` / `app_update`
2. New aggregate or extending an existing one?

Confirm the feature root: `lib/features/{context}/`

---

## Phase 3: GUIDED GENERATION

Generate **one layer at a time**. After each step, confirm with user before proceeding.

```
Step 1: Domain model    → Domain model (Equatable) + value objects + enums
Step 2: Repository      → Abstract repository interface in domain/repositories/
Step 3: Use cases       → One class per action in domain/use_cases/
Step 4: Floor entity    → Database entity + DAO in data/entities/ and data/datasources/local/
Step 5: DTO             → API data transfer object in data/dtos/
Step 6: Mapper          → Static mapper class in data/mappers/
Step 7: Datasources     → Local + remote datasource interfaces & impls
Step 8: Repository impl → Concrete repository in data/repositories/
Step 9: Controller      → GetX controller (extends BaseController) in presentation/
Step 10: Bindings       → 3-layer GetX bindings (data → domain → presentation)
```

See [layers.md](layers.md) for file paths. See [naming.md](naming.md) for all naming rules.

---

## Phase 4: REFACTOR CHECK

After all layers are generated, audit:

- [ ] Any method > 20 lines? → Extract helpers
- [ ] Missing guard clauses? → Add null/existence checks at top
- [ ] Repository has too many methods? → Consider splitting
- [ ] Primitives used where value objects should be? → Extract VO
- [ ] Flutter/GetX imports leaking into domain model? → Remove
- [ ] Domain importing from `data/`? → Fix (see [layers.md](layers.md))
- [ ] Entity missing sync fields? → Add SyncableEntity mixin if needed

---

## Quick Reference

- Domain models → [aggregates.md](aggregates.md)
- Value objects → [value-objects.md](value-objects.md)
- Repository pattern → [repositories.md](repositories.md)
- Use case pattern → [use-cases.md](use-cases.md)
- Floor persistence → [persistence.md](persistence.md)
- GetX bindings → [bindings.md](bindings.md)
- Layer boundaries → [layers.md](layers.md)
- Naming cheatsheet → [naming.md](naming.md)
- Domain story extraction → [domain-story.md](domain-story.md)
