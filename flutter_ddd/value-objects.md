# Value Objects

## Decision Rule

| Use `enum` when | Use `class` when |
|---|---|
| Closed set of values | Validated wrapper around primitives |
| Domain status/type | Composite of multiple fields |
| Maps to int/string in persistence | Needs factory constructors for named states |

---

## Enum — Domain Status/Type

```dart
// punch_type.dart — domain/enums/
enum PunchType { clockIn, clockOut }
```

With persistence mapping:

```dart
// time_log_verification_status.dart — domain/enums/
enum TimeLogVerificationStatus {
  notVerified(0),
  approved(1),
  rejected(2);

  const TimeLogVerificationStatus(this.value);
  final int value;

  static TimeLogVerificationStatus fromValue(int value) {
    return TimeLogVerificationStatus.values.firstWhere(
      (e) => e.value == value,
      orElse: () => TimeLogVerificationStatus.notVerified,
    );
  }
}
```

---

## Class — Result Value Object

For operations that return multiple states:

```dart
// device_validation_result.dart — domain/value_objects/
class DeviceValidationResult extends Equatable {
  final bool isValid;
  final String? errorMessage;
  final DeviceValidationError? error;

  const DeviceValidationResult._({
    required this.isValid,
    this.errorMessage,
    this.error,
  });

  factory DeviceValidationResult.success() =>
    const DeviceValidationResult._(isValid: true);

  factory DeviceValidationResult.noInternet() =>
    const DeviceValidationResult._(
      isValid: false,
      error: DeviceValidationError.noInternet,
      errorMessage: 'No internet connection',
    );

  factory DeviceValidationResult.notRegistered() =>
    const DeviceValidationResult._(
      isValid: false,
      error: DeviceValidationError.notRegistered,
      errorMessage: 'Device not registered',
    );

  @override
  List<Object?> get props => [isValid, errorMessage, error];
}
```

---

## Shared Value Objects (lib/shared/)

Cross-feature value objects live in `lib/shared/`:

```dart
// sync_status.dart — shared/sync/value_objects/
enum SyncStatus { pending, inProgress, synced, failed }

// domain_id.dart — shared/sync/value_objects/
class DomainId extends Equatable {
  final int? localId;
  final int? remoteId;
  const DomainId({this.localId, this.remoteId});
  @override
  List<Object?> get props => [localId, remoteId];
}
```

---

## Placement

```
domain/enums/                  <- PunchType, TimeLogVerificationStatus
domain/value_objects/          <- DeviceValidationResult, feature-specific VOs
shared/sync/value_objects/     <- DomainId, SyncStatus (cross-feature)
```
