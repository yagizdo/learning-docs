Bu bölümde Dart'ta sık kullanılan design pattern'leri ve bunların neden/nasıl kullanıldığını öğreneceksin.

**Ön Bilgi:** [03 - Constructor Tipleri](03%20-%20Constructor%20Tipleri.md)
**Sonraki:** [05 - Factory Pattern ObjectBox](POC%20-%20Factory%20Pattern%20Objectbox.md)

---

## İçindekiler

1. [Singleton Pattern](#singleton-pattern)
2. [Factory Pattern](#factory-pattern)
3. [Ne Zaman Kullanılmalı?](#ne-zaman-kullanılmalı)
4. [Özet](#özet)

---

## Singleton Pattern

### Problem

Bir Flutter uygulamasında şu senaryoyu düşün:

```dart
// Her ekranda yeni API client oluşturuluyor
class HomeScreen extends StatelessWidget {
  final apiClient = ApiClient();  // instance 1
}

class ProfileScreen extends StatelessWidget {
  final apiClient = ApiClient();  // instance 2 - FARKLI!
}

class SettingsScreen extends StatelessWidget {
  final apiClient = ApiClient();  // instance 3 - YİNE FARKLI!
}
```

**Sorunlar:**
- Her seferinde yeni instance = gereksiz memory kullanımı
- Her instance farklı state tutuyor (token, headers)
- Logout yapınca hepsini temizlemen lazım

### Çözüm: Private Constructor + Factory

```dart
class ApiClient {
  static ApiClient? _instance;

  String? _authToken;
  final String baseUrl = 'https://api.myapp.com';

  ApiClient._internal();  // Private constructor

  factory ApiClient() {
    _instance ??= ApiClient._internal();
    return _instance!;
  }

  void setToken(String token) => _authToken = token;
  void clearToken() => _authToken = null;
}

// Artık her yerde aynı instance
final api1 = ApiClient();
final api2 = ApiClient();
print(api1 == api2);  // true - AYNI instance!
```

---

### Private Constructor Neden Gerekli?

Normal constructor public olduğunda herkes yeni instance oluşturabilir:

```dart
class ApiClient {
  ApiClient();  // Public - herkes çağırabilir
}

final api1 = ApiClient();  // instance 1
final api2 = ApiClient();  // instance 2 - FARKLI!
```

Private constructor ile sadece sınıf içinden instance oluşturulabilir:

```dart
class ApiClient {
  ApiClient._internal();  // Private - dışarıdan çağrılamaz!
}

// HATA: The constructor 'ApiClient._internal' isn't defined
final api = ApiClient._internal();  // Compile error
```

---

### Neden `._internal()` İsmi?

Bu bir **convention** (gelenek), zorunluluk değil:

| İsim | Açıklama | Kullanım |
|------|----------|----------|
| `._internal()` | En yaygın, "içsel" anlamında | Dart community standardı |
| `._()` | En kısa, minimal | Hızlı yazım için |
| `._private()` | Açıkça "private" belirtir | Okunabilirlik için |
| `._singleton()` | Pattern'i belirtir | Self-documenting |

**Hepsi aynı işi yapar!** Underscore (`_`) prefix'i Dart'ta private yapar.

```dart
// Hepsi geçerli ve aynı işlevi görür
class A { A._internal(); }
class B { B._(); }
class C { C._private(); }
class D { D._singleton(); }
```

**Önemli:** `_` prefix'i Dart'ta **library-private** yapar. Aynı dosyadaki kodlar erişebilir, farklı dosyalar erişemez.

---

### Gerçek Hayat Örnekleri

#### Örnek 1: Authentication Service

```dart
class AuthService {
  static final AuthService _instance = AuthService._internal();
  factory AuthService() => _instance;
  AuthService._internal();

  User? _currentUser;
  String? _accessToken;

  User? get currentUser => _currentUser;
  bool get isLoggedIn => _currentUser != null;

  Future<void> login(String email, String password) async {
    final response = await _apiCall(email, password);
    _currentUser = User.fromJson(response['user']);
    _accessToken = response['token'];
  }

  Future<void> logout() async {
    _currentUser = null;
    _accessToken = null;
  }
}

// Kullanım - her yerde aynı user state'e erişim
class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final user = AuthService().currentUser;  // Aynı instance
    return Text('Merhaba ${user?.name}');
  }
}

class ProfileScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final isLoggedIn = AuthService().isLoggedIn;  // Aynı instance
    return isLoggedIn ? ProfileView() : LoginView();
  }
}
```

#### Örnek 2: App Configuration

```dart
class AppConfig {
  AppConfig._();
  static final AppConfig instance = AppConfig._();

  late String apiBaseUrl;
  late String appVersion;
  late bool isProduction;

  void initialize({
    required String apiBaseUrl,
    required String appVersion,
    required bool isProduction,
  }) {
    this.apiBaseUrl = apiBaseUrl;
    this.appVersion = appVersion;
    this.isProduction = isProduction;
  }
}

// main.dart'ta bir kere initialize et
void main() {
  AppConfig.instance.initialize(
    apiBaseUrl: 'https://api.myapp.com',
    appVersion: '1.0.0',
    isProduction: true,
  );
  runApp(MyApp());
}

// Her yerde kullan
class ApiService {
  final baseUrl = AppConfig.instance.apiBaseUrl;
}
```

#### Örnek 3: Logger Service

```dart
class Logger {
  static final Logger _instance = Logger._internal();
  factory Logger() => _instance;
  Logger._internal();

  final List<String> _logs = [];

  void debug(String message, {String? tag}) {
    final log = '[DEBUG]${tag != null ? '[$tag]' : ''} $message';
    _logs.add(log);
    if (!AppConfig.instance.isProduction) {
      print(log);
    }
  }

  void error(String message, {Object? error, StackTrace? stackTrace}) {
    final log = '[ERROR] $message ${error ?? ''}';
    _logs.add(log);
    print(log);
    // Production'da crash reporting'e gönder
    if (AppConfig.instance.isProduction) {
      _sendToCrashlytics(message, error, stackTrace);
    }
  }

  List<String> getLogs() => List.unmodifiable(_logs);
}

// Kullanım
Logger().debug('User tapped login button', tag: 'AuthScreen');
Logger().error('API call failed', error: e, stackTrace: st);
```

#### Örnek 4: Database Manager (ObjectBox)

```dart
class DatabaseManager {
  static DatabaseManager? _instance;
  late final Store _store;

  DatabaseManager._internal();

  factory DatabaseManager() {
    if (_instance == null) {
      throw StateError('DatabaseManager not initialized. Call init() first.');
    }
    return _instance!;
  }

  static Future<void> init() async {
    if (_instance != null) return;

    final store = await openStore();
    _instance = DatabaseManager._internal().._store = store;
  }

  Box<T> box<T>() => _store.box<T>();

  void dispose() {
    _store.close();
    _instance = null;
  }
}

// main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await DatabaseManager.init();
  runApp(MyApp());
}

// Her yerde kullan
class UserRepository {
  final _box = DatabaseManager().box<User>();

  List<User> getAll() => _box.getAll();
  void save(User user) => _box.put(user);
}
```

---

### Çalışma Prensibi

```
İlk Çağrı:                    Sonraki Çağrılar:
┌─────────────────┐          ┌─────────────────┐
│ AuthService()   │          │ AuthService()   │
│   ↓             │          │   ↓             │
│ factory         │          │ factory         │
│   ↓             │          │   ↓             │
│ _instance null? │          │ _instance null? │
│   ↓ EVET        │          │   ↓ HAYIR       │
│ ._internal()    │          │ return _instance│
│   ↓             │          └─────────────────┘
│ _instance = new │
│   ↓             │
│ return _instance│
└─────────────────┘
```

---

## Singleton: Ne Zaman Kullanılmalı?

### Uygun Senaryolar

| Senaryo | Neden Singleton? | Örnek |
|---------|------------------|-------|
| Authentication | Tek user state | AuthService |
| API Client | Tek base URL, headers | ApiClient |
| App Configuration | Tek config source | AppConfig |
| Database | Tek connection pool | DatabaseManager |
| Logger | Merkezi log toplama | Logger |
| Analytics | Tek tracking instance | AnalyticsService |

### Dikkat Edilmesi Gerekenler

**Test zorluğu:**

```dart
// KÖTÜ: Test edilemez - mock yapamıyorsun
class UserRepository {
  Future<void> saveUser(User user) async {
    await DatabaseManager().box<User>().put(user);  // Gerçek DB kullanılıyor
  }
}

// İYİ: Dependency Injection ile test edilebilir
class UserRepository {
  final Box<User> _userBox;

  UserRepository(this._userBox);  // Inject edilebilir

  Future<void> saveUser(User user) async {
    await _userBox.put(user);  // Mock box inject edilebilir
  }
}

// Test'te
final mockBox = MockBox<User>();
final repo = UserRepository(mockBox);
```

**Önerilen yaklaşım:** GetIt ile DI kullan:

```dart
// service_locator.dart
final getIt = GetIt.instance;

void setupLocator() {
  // Singleton olarak register et
  getIt.registerSingleton<AuthService>(AuthService());
  getIt.registerSingleton<ApiClient>(ApiClient());

  // Lazy singleton - ilk kullanımda oluştur
  getIt.registerLazySingleton<Logger>(() => Logger());
}

// Kullanım
class LoginController {
  final _authService = getIt<AuthService>();
  final _logger = getIt<Logger>();

  Future<void> login(String email, String password) async {
    _logger.debug('Login attempt', tag: 'LoginController');
    await _authService.login(email, password);
  }
}

// Test'te mock kullanabilirsin
getIt.registerSingleton<AuthService>(MockAuthService());
```

---

## Factory Pattern

Factory pattern, singleton'dan farklı olarak **farklı tipte** objeler döndürebilir.

### Gerçek Hayat Örneği: Payment Processor

```dart
abstract class PaymentProcessor {
  Future<PaymentResult> process(double amount);

  factory PaymentProcessor(String method) {
    return switch (method) {
      'credit_card' => CreditCardProcessor(),
      'paypal' => PayPalProcessor(),
      'apple_pay' => ApplePayProcessor(),
      _ => throw ArgumentError('Unknown payment method: $method'),
    };
  }
}

class CreditCardProcessor implements PaymentProcessor {
  @override
  Future<PaymentResult> process(double amount) async {
    // Stripe API call
    return PaymentResult.success();
  }
}

class PayPalProcessor implements PaymentProcessor {
  @override
  Future<PaymentResult> process(double amount) async {
    // PayPal SDK call
    return PaymentResult.success();
  }
}

class ApplePayProcessor implements PaymentProcessor {
  @override
  Future<PaymentResult> process(double amount) async {
    // Apple Pay integration
    return PaymentResult.success();
  }
}

// Kullanım
class CheckoutController {
  Future<void> checkout(String paymentMethod, double total) async {
    final processor = PaymentProcessor(paymentMethod);
    final result = await processor.process(total);

    if (result.isSuccess) {
      // Order complete
    }
  }
}
```

### Singleton vs Factory

| Özellik | Singleton | Factory |
|---------|-----------|---------|
| Dönen instance | Her zaman aynı | Farklı olabilir |
| Subclass dönebilir | Hayır | Evet |
| Use case | Shared state | Polymorphism |
| Örnek | AuthService | PaymentProcessor |

Detaylı bilgi: [03 - Constructor Tipleri](03%20-%20Constructor%20Tipleri.md#3-factory-constructor)

---

## Özet

| Pattern | Amaç | Private Constructor | Örnek |
|---------|------|---------------------|-------|
| Singleton | Tek instance, shared state | `._internal()`, `._()` | AuthService, Logger |
| Factory | Farklı tipler dönme | Gerekmez | PaymentProcessor |
| Cache | Aynı params = aynı instance | `._internal()` | ImageLoader |

### Karar Şeması

```
Tek instance mi lazım?
├─ Evet → Singleton Pattern
│         ├─ Test edilecek mi? → GetIt/DI kullan
│         └─ Basit kullanım → Factory constructor
│
└─ Hayır → Farklı tipler mi?
          ├─ Evet → Factory Pattern
          └─ Hayır → Normal/Named Constructor
```

---

## Kaynaklar

- [Dart Constructors](https://dart.dev/language/constructors)
- [Effective Dart: Design](https://dart.dev/effective-dart/design)
- [Design Patterns in Dart](https://scottt2.github.io/design-patterns-in-dart/singleton/)
- [Singletons in Flutter: How to Avoid Them](https://codewithandrea.com/articles/flutter-singletons/)
