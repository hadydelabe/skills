# GetX 3-Layer Bindings

## Principle

Each feature uses a **3-layer binding chain** that ensures dependencies are registered in the correct order. Each layer calls the previous layer before registering its own dependencies.

```
FeatureBinding (root)
  └── PresentationBinding
        └── DomainBinding
              └── DataBinding
```

---

## Data Binding

Registers datasources (local + remote):

```dart
// time_tracking_data_binding.dart — data/bindings/
class TimeTrackingDataBinding extends Bindings {
  @override
  void dependencies() {
    // Local datasources (using Floor DAO from core database)
    Get.lazyPut<TimeLogLocalDatasource>(
      () => TimeLogLocalDatasourceImpl(
        Get.find<AppDatabase>().timeLogDao,
      ),
    );

    // Remote datasources
    Get.lazyPut<TimeLogRemoteDatasource>(
      () => TimeLogRemoteDatasourceImpl(
        httpClient: Get.find<HttpClient>(),
        deviceLocalDatasource: Get.find<DeviceLocalDatasource>(),
      ),
    );

    // Repository implementation
    Get.lazyPut<TimeLogRepository>(
      () => TimeLogRepositoryImpl(
        localDatasource: Get.find(),
        remoteDatasource: Get.find(),
      ),
    );
  }
}
```

---

## Domain Binding

Ensures data layer is loaded, then registers use cases:

```dart
// time_tracking_domain_binding.dart — domain/bindings/
class TimeTrackingDomainBinding extends Bindings {
  @override
  void dependencies() {
    // Ensure data layer is loaded
    TimeTrackingDataBinding().dependencies();

    // Use cases
    Get.lazyPut(() => PunchTimeUseCase(
      timeLogRepository: Get.find(),
      getLastPunchUseCase: Get.find(),
    ));

    Get.lazyPut(() => GetLastPunchUseCase(
      timeLogRepository: Get.find(),
    ));

    Get.lazyPut(() => GetEmployeeTimeLogsUseCase(
      repository: Get.find(),
    ));
  }
}
```

---

## Presentation Binding

Ensures domain layer is loaded, then registers controllers:

```dart
// time_tracking_presentation_binding.dart — presentation/bindings/
class TimeTrackingPresentationBinding extends Bindings {
  @override
  void dependencies() {
    // Ensure domain layer is loaded
    TimeTrackingDomainBinding().dependencies();

    // Controllers
    Get.lazyPut(() => TimeLogController(
      punchTimeUseCase: Get.find(),
      getEmployeeTimeLogsUseCase: Get.find(),
    ));
  }
}
```

---

## Root Feature Binding

Entry point wired into `AppBinding`:

```dart
// time_tracking_binding.dart — presentation/bindings/
class TimeTrackingBinding extends Bindings {
  @override
  void dependencies() {
    TimeTrackingPresentationBinding().dependencies();
  }
}
```

---

## Registration in AppBinding

All feature bindings are orchestrated in the root `AppBinding`:

```dart
// app_binding.dart — lib/bindings/
class AppBinding extends Bindings {
  @override
  void dependencies() {
    CoreBinding().dependencies();
    UserBinding().dependencies();
    DeviceBinding().dependencies();
    TimeTrackingBinding().dependencies();
    SyncBinding().dependencies();
    // ... other features
  }
}
```

---

## Rules

- **Chain order**: Data → Domain → Presentation (never skip)
- **`Get.lazyPut`** for feature-scoped deps (created on first access)
- **`Get.put`** with `permanent: true` for core services only (HttpClient, AppLogger, Database)
- **Each layer ensures its dependency layer is initialized first**
- **Never use `Get.find()` in domain layer code** — inject via constructor parameters
