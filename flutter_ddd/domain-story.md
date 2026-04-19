# Domain Story — Narration & Extraction

## How to Narrate

Ask the user for a scenario in plain language:

> "Tell me what happens. Who does what, and what are the business rules?"

Good stories mention:
- An **actor** (Admin, Employee, Device, System)
- An **intent** (enroll face, punch in, sync data, register device)
- A **constraint** (only if enrolled, only SUPER_ADMIN, unless already synced)

---

## Extraction Template

From the story, extract these artifacts:

### Actors
Who initiates? Who is affected?
- Initiator → maps to a caller role (SUPER_ADMIN, ADMIN, or system)
- Affected entity → likely a **Domain Model**

### Commands
Verb phrases representing state-changing intent.
- "Enroll employee face" → `EnrollFaceUseCase`
- "Punch in via fingerprint" → `PunchTimeUseCase`
- One command = one use case

### Queries
Read-only requests.
- "Get employee time logs" → `GetEmployeeTimeLogsUseCase`
- No side effects

### Aggregates / Domain Models
Nouns that:
- Own state and lifecycle
- Enforce invariants
- Have a unique identity (often dual: localId + remoteId)

Example: `TimeLog`, `Employee`, `Device`

Every domain model needs:
- Extends `Equatable` with `props` list
- `copyWith()` for immutable updates
- Domain methods expressing intent (not setters)

### Policies
Business rules extracted from "unless", "only if", "cannot":
- "Cannot punch twice within 1 minute" → `DuplicatePunchException`
- Enforced in use case, not in controller

### Value Objects
Descriptive data with no lifecycle:
- `DomainId` (wraps localId + remoteId)
- `SyncStatus` (pending, inProgress, synced, failed)
- `TimeLogVerificationStatus` (notVerified, approved, rejected)
- `DeviceValidationResult` (success, noInternet, notRegistered)

---

## Extraction Output Format

Present to user before proceeding:

```
STORY: [one sentence summary]

Actors:        Admin (initiator), Employee (affected)
Commands:      EnrollFingerprint
Queries:       GetEnrollmentStatus
Domain Model:  Enrollment
Policies:      Cannot enroll if already enrolled for this biometric type
Value Objects: BiometricType (face, fingerprint, rfid), EnrollmentStatus

Bounded context: device_management
New model or extending: new Enrollment model
Sync required: yes (offline-first)
```

Wait for confirmation before Phase 3.
