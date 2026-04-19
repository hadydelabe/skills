# Use Cases

## Principle

One use case = one business action. Use cases orchestrate domain logic: **Guard → Load → Enforce → Execute → Return**.

---

## Use Case Template

```dart
// punch_time_use_case.dart — domain/use_cases/
class PunchTimeUseCase {
  final TimeLogRepository _timeLogRepository;
  final GetLastPunchUseCase _getLastPunchUseCase;

  PunchTimeUseCase({
    required TimeLogRepository timeLogRepository,
    required GetLastPunchUseCase getLastPunchUseCase,
  })  : _timeLogRepository = timeLogRepository,
        _getLastPunchUseCase = getLastPunchUseCase;

  Future<TimeLog> call({
    required int employeeId,
    required PunchType punchType,
  }) async {
    // 1. Guard
    if (employeeId <= 0) throw ArgumentError('Invalid employee ID');

    // 2. Load (check last punch)
    final lastPunch = await _getLastPunchUseCase.call(employeeId);

    // 3. Enforce policy (no duplicate within 1 min)
    if (lastPunch != null && _isDuplicate(lastPunch)) {
      throw DuplicatePunchException();
    }

    // 4. Execute
    return _timeLogRepository.punchForEmployee(
      employeeId: employeeId,
      punchType: punchType,
    );
  }

  bool _isDuplicate(TimeLog lastPunch) {
    return DateTime.now().difference(lastPunch.punchTime).inMinutes < 1;
  }
}
```

---

## Rules

- **Single responsibility** — one public method (`call()` or named action)
- **Depends on repositories and other use cases** — never on datasources or controllers
- **Guard → Load → Enforce → Execute → Return** — consistent flow
- **< 20 lines** per method — extract helpers if needed
- **No Flutter imports** — pure Dart + domain types
- **Composition over inheritance** — use cases depend on other use cases, not a base class

---

## Query Use Case

```dart
// get_employee_time_logs_use_case.dart — domain/use_cases/
class GetEmployeeTimeLogsUseCase {
  final TimeLogDetailRepository _repository;

  GetEmployeeTimeLogsUseCase({required TimeLogDetailRepository repository})
      : _repository = repository;

  Future<List<TimeLogDetail>> call(int employeeId, {int offset = 0, int limit = 20}) {
    return _repository.getByEmployee(employeeId, offset: offset, limit: limit);
  }
}
```

---

## Naming Convention

| Type | Pattern | Example |
|---|---|---|
| Command | `VerbNounUseCase` | `PunchTimeUseCase`, `RegisterDeviceUseCase` |
| Query | `GetNounUseCase` | `GetEmployeeTimeLogsUseCase`, `GetLastPunchUseCase` |
| Validation | `ValidateNounUseCase` | `ValidateDeviceUseCase` |
| Check | `IsNounUseCase` | `IsDeviceRegisteredUseCase` |

---

## Hierarchy

```
domain/use_cases/
  punch_time_use_case.dart
  get_employee_time_logs_use_case.dart
  get_last_punch_use_case.dart
  update_time_log_verification_status_use_case.dart
```
