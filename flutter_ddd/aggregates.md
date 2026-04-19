# Domain Models

## Rules

- **Extends Equatable** — value-based equality for reactive state management
- **Immutable with copyWith** — no setters, new instances via `copyWith()`
- **Intent-expressing methods** — `markSynced()`, `markDeleted()` — not `setStatus()`
- **No Flutter/data imports** — pure Dart + Equatable only
- **Dual ID for syncable models** — `DomainId` wrapping localId + remoteId

---

## Domain Model Template

```dart
// time_log.dart — features/time_tracking/domain/models/
import 'package:equatable/equatable.dart';

class TimeLog extends Equatable {
  final DomainId? id;
  final int? employeeId;
  final DateTime punchTime;
  final PunchType punchType;
  final TimeLogVerificationStatus verificationStatus;
  final SyncStatus? syncStatus;
  final bool? deleted;

  const TimeLog({
    this.id,
    this.employeeId,
    required this.punchTime,
    required this.punchType,
    this.verificationStatus = TimeLogVerificationStatus.notVerified,
    this.syncStatus,
    this.deleted,
  });

  // Domain predicates
  bool get isSynced => syncStatus == SyncStatus.synced;
  bool get isDeleted => deleted == true;
  bool get isNew => id?.remoteId == null && !isDeleted;
  int? get localId => id?.localId;
  int? get remoteId => id?.remoteId;

  // Intent-expressing methods (return new instance)
  TimeLog markSynced() => copyWith(syncStatus: SyncStatus.synced);
  TimeLog markDeleted() => copyWith(deleted: true);

  TimeLog copyWith({
    DomainId? id,
    int? employeeId,
    DateTime? punchTime,
    PunchType? punchType,
    TimeLogVerificationStatus? verificationStatus,
    SyncStatus? syncStatus,
    bool? deleted,
  }) {
    return TimeLog(
      id: id ?? this.id,
      employeeId: employeeId ?? this.employeeId,
      punchTime: punchTime ?? this.punchTime,
      punchType: punchType ?? this.punchType,
      verificationStatus: verificationStatus ?? this.verificationStatus,
      syncStatus: syncStatus ?? this.syncStatus,
      deleted: deleted ?? this.deleted,
    );
  }

  @override
  List<Object?> get props => [id, employeeId, punchTime, punchType, verificationStatus, syncStatus, deleted];
}
```

---

## DomainId — Dual Identity

For offline-first sync, every syncable model uses `DomainId`:

```dart
// domain_id.dart — shared/sync/value_objects/
class DomainId extends Equatable {
  final int? localId;
  final int? remoteId;

  const DomainId({this.localId, this.remoteId});

  @override
  List<Object?> get props => [localId, remoteId];
}
```

- `localId` = auto-generated Floor primary key (available immediately)
- `remoteId` = server-assigned ID (populated after sync)

---

## Hierarchy

```
domain/
  models/
    time_log.dart               <- domain model
    time_log_detail.dart        <- read model (if different shape)
  enums/
    punch_type.dart             <- enum VO
  value_objects/
    domain_id.dart              <- dual identity (in shared/)
  exceptions/
    duplicate_punch_exception.dart
```
