# Repository Pattern

## Principle

Repository interfaces live in **domain** — they define what data the domain needs. Implementations live in **data** — they decide how to fetch/store it.

---

## Domain Repository Interface

```dart
// time_log_repository.dart — domain/repositories/
abstract class TimeLogRepository {
  Future<TimeLog?> getById(int id);
  Future<List<TimeLog>> getByEmployee(int employeeId);
  Future<TimeLog> punchForEmployee({
    required int employeeId,
    required PunchType punchType,
  });
  Future<void> updateVerificationStatus(int id, TimeLogVerificationStatus status);
}
```

Rules:
- Returns **domain models**, never DTOs or entities
- Method names express domain intent
- No persistence details (no SQL, no HTTP)
- Parameters are domain types or primitives

---

## Data Repository Implementation

```dart
// time_log_repository_impl.dart — data/repositories/
class TimeLogRepositoryImpl implements TimeLogRepository {
  final TimeLogLocalDatasource _localDatasource;
  final TimeLogRemoteDatasource _remoteDatasource;

  TimeLogRepositoryImpl({
    required TimeLogLocalDatasource localDatasource,
    required TimeLogRemoteDatasource remoteDatasource,
  })  : _localDatasource = localDatasource,
        _remoteDatasource = remoteDatasource;

  @override
  Future<TimeLog> punchForEmployee({
    required int employeeId,
    required PunchType punchType,
  }) async {
    // 1. Create domain model
    final timeLog = TimeLog(
      employeeId: employeeId,
      punchTime: DateTime.now(),
      punchType: punchType,
      syncStatus: SyncStatus.pending,
    );

    // 2. Map to entity and persist locally
    final entity = TimeLogMapper.domainToEntity(timeLog);
    final savedEntity = await _localDatasource.insert(entity);

    // 3. Publish sync event
    EventBus.publish(TimeLogSyncRequiredEvent(localEntityId: savedEntity.id));

    // 4. Return domain model
    return TimeLogMapper.entityToDomain(savedEntity);
  }
}
```

Responsibilities:
- **Implement** domain repository interface
- **Decide** source priority (local vs remote)
- **Coordinate** between datasources
- **Transform** data using mappers
- **Publish** domain events when needed

---

## Splitting Repositories

When a repository grows beyond 7-8 methods, consider splitting by read/write concern:

```
domain/repositories/
  time_log_repository.dart          <- write operations (punch, update)
  time_log_detail_repository.dart   <- read operations (queries, pagination)
```
