# Floor Persistence

## Floor Entity

```dart
// time_log_entity.dart — data/entities/
@Entity(tableName: 'time_logs')
class TimeLogEntity extends Equatable {
  @PrimaryKey(autoGenerate: true)
  final int? id;

  @ColumnInfo(name: 'remote_id')
  final int? remoteId;

  @ColumnInfo(name: 'employee_id')
  final int employeeId;

  @ColumnInfo(name: 'punch_time')
  final String punchTime;  // ISO8601 string for Floor

  @ColumnInfo(name: 'punch_type')
  final String punchType;

  @ColumnInfo(name: 'sync_status')
  final String? syncStatus;

  @ColumnInfo(name: 'deleted')
  final bool deleted;

  const TimeLogEntity({
    this.id,
    this.remoteId,
    required this.employeeId,
    required this.punchTime,
    required this.punchType,
    this.syncStatus,
    this.deleted = false,
  });

  @override
  List<Object?> get props => [id, remoteId, employeeId, punchTime, punchType, syncStatus, deleted];
}
```

Rules:
- `@Entity(tableName: 'snake_case')` — always explicit table name
- `@ColumnInfo(name: 'snake_case')` — always explicit column name
- `@PrimaryKey(autoGenerate: true)` — local ID, auto-generated
- `remoteId` — nullable, populated after sync
- `syncStatus` — string representation of SyncStatus enum
- `deleted` — soft delete flag for sync
- No domain logic, no business methods

---

## SyncableEntity Mixin

For entities that sync to server:

```dart
// syncable_entity.dart — shared/sync/mixins/
mixin SyncableEntity {
  String? get syncStatus;
  int? get remoteId;
  bool get deleted;

  bool get isNew => remoteId == null && !deleted;
  bool get needsSync => syncStatus != 'synced';
  bool get isSynced => syncStatus == 'synced';
}
```

---

## Floor DAO

```dart
// time_log_dao.dart — data/datasources/local/daos/
@dao
abstract class TimeLogDao {
  @Query('SELECT * FROM time_logs WHERE id = :id')
  Future<TimeLogEntity?> getById(int id);

  @Query('SELECT * FROM time_logs WHERE employee_id = :employeeId AND deleted = 0 ORDER BY punch_time DESC LIMIT :limit OFFSET :offset')
  Future<List<TimeLogEntity>> getByEmployee(int employeeId, int limit, int offset);

  @Query('SELECT * FROM time_logs WHERE sync_status != "synced" AND deleted = 0')
  Future<List<TimeLogEntity>> getPendingSync();

  @insert
  Future<int> insertTimeLog(TimeLogEntity entity);

  @update
  Future<void> updateTimeLog(TimeLogEntity entity);

  @Query('UPDATE time_logs SET sync_status = :status WHERE id = :id')
  Future<void> updateSyncStatus(int id, String status);
}
```

---

## Database Registration

Floor database is registered globally in `core/database/`:

```dart
// app_database.dart
@Database(
  version: 3,
  entities: [EmployeeEntity, TimeLogEntity, UserEntity, SyncMetadataEntity],
  views: [PendingCountView],
)
abstract class AppDatabase extends FloorDatabase {
  EmployeesDao get employeeDao;
  TimeLogDao get timeLogDao;
  UserDao get userDao;
  SyncMetadataDao get syncMetadataDao;
}
```

When adding a new entity:
1. Create the entity class
2. Create the DAO
3. Add entity to `@Database(entities: [...])`
4. Add DAO getter to `AppDatabase`
5. Increment database version + add migration
6. Run `dart run build_runner build`
