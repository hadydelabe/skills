# Naming Cheatsheet

## Domain Layer

| Artifact | Pattern | Example |
|---|---|---|
| Domain model | Noun, no suffix | `TimeLog`, `Employee`, `Device` |
| Value object | Noun, no suffix | `DomainId`, `SyncStatus` |
| Enum VO | Noun or adjective | `PunchType`, `TimeLogVerificationStatus` |
| Exception | Noun + `Exception` | `DuplicatePunchException` |
| Repository (interface) | Noun + `Repository` | `TimeLogRepository` |
| Use case | Action + Noun + `UseCase` | `PunchTimeUseCase`, `GetEmployeeTimeLogsUseCase` |
| Converter | Noun + `Converter` | `TimeLogVerificationStatusConverter` |

## Data Layer

| Artifact | Pattern | Example |
|---|---|---|
| DTO | Noun + `Dto` | `TimeLogDto`, `DeviceDto` |
| Floor entity | Noun + `Entity` | `TimeLogEntity`, `EmployeeEntity` |
| Mapper | Noun + `Mapper` | `TimeLogMapper`, `DeviceMapper` |
| DAO | Noun + `Dao` | `TimeLogDao`, `EmployeeDao` |
| Repository impl | Noun + `RepositoryImpl` | `TimeLogRepositoryImpl` |
| Remote datasource | Noun + `RemoteDatasource` | `TimeLogRemoteDatasource` |
| Remote impl | Noun + `RemoteDatasourceImpl` | `TimeLogRemoteDatasourceImpl` |
| Local datasource | Noun + `LocalDatasource` | `DeviceLocalDatasource` |
| Local impl | Noun + `LocalDatasourceImpl` | `DeviceLocalDatasourceImpl` |
| Binding | Noun + `DataBinding` | `TimeTrackingDataBinding` |

## Presentation Layer

| Artifact | Pattern | Example |
|---|---|---|
| Controller | Feature + `Controller` | `TimeLogController`, `DeviceRegistrationController` |
| Screen | Feature + `Screen` | `TimeLogScreen`, `DeviceRegistrationScreen` |
| Binding | Feature + `PresentationBinding` | `TimeTrackingPresentationBinding` |

## File Names

All file names use `snake_case.dart`:

| Class | File |
|---|---|
| `TimeLogMapper` | `time_log_mapper.dart` |
| `PunchTimeUseCase` | `punch_time_use_case.dart` |
| `TimeLogEntity` | `time_log_entity.dart` |
| `GetEmployeeTimeLogsUseCase` | `get_employee_time_logs_use_case.dart` |

## Method Names

| Action | Pattern | Example |
|---|---|---|
| Mapper (dto→domain) | `dtoToDomain()` | `TimeLogMapper.dtoToDomain(dto)` |
| Mapper (entity→domain) | `entityToDomain()` | `TimeLogMapper.entityToDomain(entity)` |
| Mapper (domain→entity) | `domainToEntity()` | `TimeLogMapper.domainToEntity(model)` |
| Mapper (dto→entity) | `dtoToEntity()` | `TimeLogMapper.dtoToEntity(dto)` |
| Domain predicate | `is` + adjective | `isSynced`, `isDeleted`, `isNew` |
| Domain intent | verb expressing intent | `markSynced()`, `markDeleted()` |
| Use case entry | `call()` or named action | `call()`, `execute()` |

## Forbidden Names

- `getData()` — too generic, name after the domain concept
- `setStatus()` — use intent-expressing method like `markSynced()`
- `handleXxx()` — use action verbs like `punchForEmployee()`
- `XxxHelper` — embed logic in the appropriate layer instead
