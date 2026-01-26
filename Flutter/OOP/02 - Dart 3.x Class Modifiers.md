Bu bölüm Dart 3 ile gelen class modifier'ları kapsar.

**Gereksinim:** Dart 3.0 veya üzeri. Projenin `pubspec.yaml` dosyasında:

```yaml
environment:
  sdk: '>=3.0.0 <4.0.0'
```

**Ön Bilgi:** [01 - OOP Temelleri](01%20-%20OOP%20Temelleri%20-%20Dart.md)
**Sonraki:** [03 - Constructor Tipleri](03%20-%20Constructor%20Tipleri.md)

---

## İçindekiler

1. [Neden Class Modifiers?](#neden-class-modifiers)
2. [abstract](#1-abstract)
3. [interface](#2-interface)
4. [base](#3-base)
5. [final](#4-final)
6. [sealed](#5-sealed)
7. [mixin](#6-mixin)
8. [mixin class](#7-mixin-class)
9. [Kombinasyonlar](#kombinasyonlar)
10. [Referans Tabloları](#referans-tabloları)

---

## Neden Class Modifiers?

Dart 3 öncesinde sadece `abstract` keyword'ü vardı. Bu, sınıfların nasıl kullanılacağını kontrol etmeyi zorlaştırıyordu.

### Problem

```dart
abstract class BaseCache<T> {
  final Box<T> _box;
  BaseCache(this._box);

  int save(T item) {
    _logOperation('save', item);
    return _box.put(item);
  }

  void _logOperation(String op, T item) {
    analytics.track('cache_$op', {'type': T.toString()});
  }
}
```

Başka bir developer `implements` ile bu sınıfı kullanıp loglama kodunu atlayabilir:

```dart
class HackedCache implements BaseCache<User> {
  @override
  int save(User item) {
    return someBox.put(item);  // loglama yok
  }
}
```

### Çözüm

Dart 3 modifier'ları ile sınıfların kullanım şeklini kontrol edebilirsin:

| Modifier | Etki |
|----------|------|
| `interface` | Sadece `implements` ile kullanılabilir |
| `base` | Sadece `extends` ile kullanılabilir |
| `final` | Ne `extends` ne `implements` yapılabilir |
| `sealed` | Alt sınıflar sadece aynı dosyada tanımlanabilir |

---

## 1. abstract

Doğrudan instance oluşturulamayan sınıf. Alt sınıflar tarafından tamamlanması gereken bir şablon tanımlar.

```dart
abstract class BaseRepository<T> {
  final Dio _dio;
  final String _endpoint;

  BaseRepository(this._dio, this._endpoint);

  Future<Options> get _authOptions async {
    final token = await SecureStorage.getToken();
    return Options(headers: {'Authorization': 'Bearer $token'});
  }

  Future<T> getById(int id) async {
    final response = await _dio.get(
      '$_endpoint/$id',
      options: await _authOptions,
    );
    return fromJson(response.data);
  }

  T fromJson(Map<String, dynamic> json);        // alt sınıf tamamlamalı
  Map<String, dynamic> toJson(T item);          // alt sınıf tamamlamalı
}
```

Kullanım:

```dart
final repo = BaseRepository<User>(...);  // HATA: Abstract class

class UserRepository extends BaseRepository<User> {
  UserRepository(Dio dio) : super(dio, '/users');

  @override
  User fromJson(Map<String, dynamic> json) => User.fromJson(json);

  @override
  Map<String, dynamic> toJson(User item) => item.toJson();
}
```

**Ne zaman:** Template sınıflar, ortak kod + alt sınıfın tamamlaması gereken metodlar.

---

## 2. interface

Sadece `implements` ile kullanılabilen sınıf. `extends` yasaktır.

### interface class vs abstract interface class

İki varyant var:

**`interface class`** — Instance oluşturulabilir, içinde kod olabilir:

```dart
interface class DefaultLogger {
  void log(String message) => print('[LOG] $message');
  void error(String message) => print('[ERROR] $message');
}

final logger = DefaultLogger();              // OK
class MyLogger extends DefaultLogger { }     // HATA: extends yasak
class FileLogger implements DefaultLogger {  // OK
  @override
  void log(String message) { ... }
  @override
  void error(String message) { ... }
}
```

**`abstract interface class`** — Saf kontrat, içinde kod yok:

```dart
abstract interface class ILogger {
  void log(String message);
  void error(String message);
}

final logger = ILogger();  // HATA: abstract
class ConsoleLogger implements ILogger {
  @override
  void log(String message) => print('[LOG] $message');
  @override
  void error(String message) => print('[ERROR] $message');
}
```

### Gerçek Senaryo

Farklı ödeme servisleri için ortak kontrat:

```dart
abstract interface class IPaymentService {
  Future<PaymentResult> processPayment(PaymentRequest request);
  Future<bool> refund(String transactionId);
  Future<List<PaymentMethod>> getSavedCards(String customerId);
  String get providerName;
}

class StripeService implements IPaymentService {
  @override
  String get providerName => 'Stripe';

  @override
  Future<PaymentResult> processPayment(PaymentRequest request) async {
    final paymentIntent = await Stripe.instance.confirmPayment(
      paymentIntentClientSecret: request.clientSecret,
    );
    return PaymentResult.success(paymentIntent.id);
  }
  // ...
}

class IyzicoService implements IPaymentService {
  @override
  String get providerName => 'iyzico';
  // tamamen farklı implementasyon
}
```

**Ne zaman:** Farklı implementasyonlar olacaksa (Stripe/iyzico, Firebase/Supabase), test için mock yazılacaksa.

---

## 3. base

Sadece `extends` ile kullanılabilen sınıf. `implements` yasaktır.

İçindeki kodun mutlaka kullanılmasını garanti eder. Kimse bypass edip kendi implementasyonunu yazamaz.

```dart
base class BaseAnalyticsService {
  final String _sessionId = const Uuid().v4();
  final String? _userId = AuthService.currentUserId;

  Future<void> trackEvent(String name, Map<String, dynamic> params) async {
    final enrichedParams = {
      ...params,
      'timestamp': DateTime.now().toIso8601String(),
      'session_id': _sessionId,
      'user_id': _userId,
      'app_version': PackageInfo.version,
      'platform': Platform.operatingSystem,
    };
    await _sendToBackend(name, enrichedParams);
  }

  Future<void> _sendToBackend(String name, Map<String, dynamic> params);
}
```

Kullanım:

```dart
class FakeAnalytics implements BaseAnalyticsService { }  // HATA: implements yasak

final class FirebaseAnalyticsService extends BaseAnalyticsService {
  @override
  Future<void> _sendToBackend(String name, Map<String, dynamic> params) async {
    await FirebaseAnalytics.instance.logEvent(
      name: name,
      parameters: params.cast<String, Object>(),
    );
  }
}
```

### Base Transitivity (Bulaşıcılık)

`base` class'tan türeyen sınıf da `base`, `final` veya `sealed` olmak zorunda:

```dart
base class BaseManager<T> { }

base class UserManager extends BaseManager<User> { }   // OK
final class UserManager extends BaseManager<User> { }  // OK
sealed class UserManager extends BaseManager<User> { } // OK
class UserManager extends BaseManager<User> { }        // HATA
```

Nedeni: Aksi halde UserManager'ı biri `implements` edip base'in korumasını kırabilir.

**Ne zaman:** Güvenlik, loglama, validasyon kodları atlanmamalı.

---

## 4. final

Ne `extends` ne `implements` yapılabilen sınıf (kütüphane dışından).

Sınıfın davranışını tamamen kilitler.

```dart
final class UserManager extends BaseManager<User> {
  UserManager(super.box);

  User? findByEmail(String email) {
    final query = box.query(User_.email.equals(email)).build();
    try {
      return query.findFirst();
    } finally {
      query.close();
    }
  }

  Future<void> deactivateUser(int userId) async {
    final user = getById(userId);
    if (user != null) {
      user.isActive = false;
      user.deactivatedAt = DateTime.now();
      save(user);
      await AuditService.log('user_deactivated', {'user_id': userId});
    }
  }
}
```

`final` olmasaydı:

```dart
class HackedUserManager extends UserManager {
  @override
  Future<void> deactivateUser(int userId) async {
    final user = getById(userId);
    user?.isActive = false;
    save(user!);
    // audit log atlandı
  }
}
```

### Utility Sınıfları

```dart
final class Validators {
  Validators._();

  static bool isEmail(String value) {
    return RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value);
  }

  static bool isStrongPassword(String value) {
    if (value.length < 8) return false;
    if (!value.contains(RegExp(r'[A-Z]'))) return false;
    if (!value.contains(RegExp(r'[a-z]'))) return false;
    if (!value.contains(RegExp(r'[0-9]'))) return false;
    return true;
  }
}
```

### Aynı Kütüphane İçinde

`final` sadece kütüphane dışından korur. Aynı dosya içinde extend edilebilir (genelde yapılmaz).

**Ne zaman:** Manager, Service, Repository implementasyonları, Configuration sınıfları, Singleton'lar.

---

## 5. sealed

Tüm alt sınıfları aynı dosyada olması gereken sınıf. Exhaustive pattern matching sağlar.

Temel özellikler:
- `sealed` class örtük olarak `abstract`tır, instance oluşturulamaz
- Tüm direct subclass'lar aynı dosyada tanımlanmalı
- Yeni alt tip eklemek API için **breaking change**

```dart
// product_state.dart

sealed class ProductState {}

final class ProductInitial extends ProductState {}

final class ProductLoading extends ProductState {}

final class ProductLoaded extends ProductState {
  final List<Product> products;
  final bool hasMore;
  final int currentPage;

  ProductLoaded({
    required this.products,
    required this.hasMore,
    this.currentPage = 1,
  });
}

final class ProductError extends ProductState {
  final String message;
  final bool canRetry;

  ProductError(this.message, {this.canRetry = true});
}
```

### Exhaustive Switch

Dart compiler tüm varyantları handle ettiğini kontrol eder:

```dart
Widget build(BuildContext context) {
  return BlocBuilder<ProductBloc, ProductState>(
    builder: (context, state) => switch (state) {
      ProductInitial() => const SizedBox.shrink(),
      ProductLoading() => const Center(child: CircularProgressIndicator()),
      ProductLoaded(:final products, :final hasMore) => ProductListView(
        products: products,
        hasMore: hasMore,
        onLoadMore: () => context.read<ProductBloc>().add(LoadMoreProducts()),
      ),
      ProductError(:final message, :final canRetry) => ErrorView(
        message: message,
        onRetry: canRetry ? () => context.read<ProductBloc>().add(LoadProducts()) : null,
      ),
    },
  );
}
```

Yeni bir state eklersen (örneğin `ProductRefreshing`), switch ifadesi derleme hatası verir.

### Indirect Subclass

Direct subclass'lar aynı dosyada olmalı, ama onların alt sınıfları başka dosyada olabilir:

```dart
// shapes.dart
sealed class Shape {}
final class Circle extends Shape {}
base class Polygon extends Shape {}

// other_file.dart
final class Triangle extends Polygon {}  // OK - Polygon'un alt sınıfı
```

### Sealed vs Freezed

| Özellik | Sealed | Freezed |
|---------|--------|---------|
| Code generation | Hayır | Evet |
| copyWith | Manuel | Otomatik |
| toJson/fromJson | Manuel | Otomatik |
| == ve hashCode | Manuel | Otomatik |
| Build süresi | Etkilemez | Artırır |

**Tavsiye:** Basit state'ler için Sealed, kompleks modeller için Freezed.

---

## 6. mixin

Birden fazla sınıfa eklenebilen davranış paketi.

Dart'ta bir sınıf sadece bir sınıfı extend edebilir. Birden fazla mixin'i `with` ile ekleyerek kod tekrarını önleyebilirsin.

```dart
mixin EmailValidationMixin {
  String? validateEmail(String? value) {
    if (value == null || value.isEmpty) return 'Email gerekli';
    if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value)) {
      return 'Geçersiz email formatı';
    }
    return null;
  }
}

mixin PasswordValidationMixin {
  String? validatePassword(String? value) {
    if (value == null || value.isEmpty) return 'Şifre gerekli';
    if (value.length < 8) return 'Şifre en az 8 karakter olmalı';
    if (!value.contains(RegExp(r'[A-Z]'))) return 'Şifre en az bir büyük harf içermeli';
    if (!value.contains(RegExp(r'[0-9]'))) return 'Şifre en az bir rakam içermeli';
    return null;
  }
}

class LoginFormController with EmailValidationMixin, PasswordValidationMixin {
  final emailController = TextEditingController();
  final passwordController = TextEditingController();

  bool validate() {
    final emailError = validateEmail(emailController.text);
    final passwordError = validatePassword(passwordController.text);
    return emailError == null && passwordError == null;
  }
}

class ProfileFormController with EmailValidationMixin {
  // sadece email validation
}
```

### mixin on Keyword

Mixin'in sadece belirli sınıflarla kullanılmasını sağlar:

```dart
mixin LoadingMixin<T> on Cubit<T> {
  bool _isLoading = false;
  bool get isLoading => _isLoading;

  Future<R> withLoading<R>(Future<R> Function() action) async {
    _isLoading = true;
    try {
      return await action();
    } finally {
      _isLoading = false;
    }
  }
}

class ProductCubit extends Cubit<ProductState> with LoadingMixin<ProductState> {
  // Cubit olduğu için LoadingMixin kullanılabilir
}

class SomeClass with LoadingMixin { }  // HATA: Cubit değil
```

### AfterLayout Örneği

```dart
mixin AfterLayoutMixin<T extends StatefulWidget> on State<T> {
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (mounted) afterFirstLayout(context);
    });
  }

  void afterFirstLayout(BuildContext context);
}

class _ProductDetailPageState extends State<ProductDetailPage>
    with AfterLayoutMixin<ProductDetailPage> {

  @override
  void afterFirstLayout(BuildContext context) {
    context.read<ProductBloc>().add(LoadProduct(widget.productId));
  }

  @override
  Widget build(BuildContext context) => Scaffold(...);
}
```

---

## 7. mixin class

Hem sınıf hem mixin olarak kullanılabilen yapı.

### Dart 3.0 Breaking Change

Dart 3.0 öncesinde herhangi bir class mixin olarak kullanılabilirdi:

```dart
// Dart 2.x
class Flyable {
  void fly() => print('Flying!');
}
class Bird with Flyable {}  // OK

// Dart 3.x
class Flyable {
  void fly() => print('Flying!');
}
class Bird with Flyable {}  // HATA: Flyable bir mixin değil
```

Çözüm: `mixin class` kullan:

```dart
mixin class Flyable {
  void fly() => print('Flying!');
}

class Bird with Flyable {}       // mixin olarak
class Airplane extends Flyable {} // class olarak
final flyer = Flyable();          // doğrudan instantiate
```

### Kısıtlamalar

`mixin class` mixin kurallarına uymalı:
- `extends` kullanılamaz (Object hariç)
- Generative constructor olamaz

```dart
mixin class ValidatedInput extends BaseInput {}  // HATA

mixin class Counter {
  int value;
  Counter(this.value);  // HATA: generative constructor
}

mixin class Counter {
  int value = 0;
  Counter();  // OK: parametresiz
}

mixin class Config {
  final String name;
  const Config(this.name);  // OK: const constructor
}
```

### Gerçek Senaryo

```dart
mixin class Disposable {
  bool _isDisposed = false;
  bool get isDisposed => _isDisposed;

  void dispose() {
    if (_isDisposed) return;
    _isDisposed = true;
    onDispose();
  }

  void onDispose() {}
}

class ResourceManager extends Disposable {
  final List<Resource> _resources = [];

  @override
  void onDispose() {
    for (final resource in _resources) {
      resource.release();
    }
    _resources.clear();
  }
}

class MyController extends BaseController with Disposable {
  @override
  void onDispose() { }
}
```

### base mixin class

`implements` yasağı ekler:

```dart
base mixin class SecureDisposable {
  void dispose() {
    _checkPermissions();
    _logDisposal();
    onDispose();
  }

  void _checkPermissions() { }
  void _logDisposal() { }
  void onDispose() { }
}

class MyResource extends SecureDisposable {}     // OK
class MyClass with SecureDisposable {}           // OK
class Fake implements SecureDisposable {}        // HATA
```

---

## Kombinasyonlar

### Modifier Sırası

```
abstract? → base/interface/final/sealed? → mixin? → class
```

Geçerli kombinasyonlar:

```dart
abstract class
base class
abstract base class
interface class
abstract interface class
final class
abstract final class
sealed class
mixin class
base mixin class
abstract mixin class
abstract base mixin class
```

### Geçersiz Kombinasyonlar

| Kombinasyon | Neden Geçersiz |
|-------------|----------------|
| `abstract sealed` | sealed zaten abstract |
| `interface base` | interface = sadece implements, base = sadece extends |
| `final sealed` | final = alt sınıf yok, sealed = aynı dosyada alt sınıf |
| `interface final` | interface = implements zorunlu, final = implements yasak |
| `base final` | base = extends edilebilir, final = extends yasak |
| `sealed mixin` | sealed class mixin olamaz |
| `interface mixin` | geçersiz kombinasyon |
| `final mixin` | final extends'i yasaklar |

---

## Referans Tabloları

### Temel Davranışlar

| Modifier | extends | implements | Kullanım |
|----------|---------|------------|----------|
| `abstract` | Evet | Evet | Template sınıf |
| `interface` | Hayır | Evet | Kontrat |
| `base` | Evet | Hayır | Kod koruması |
| `final` | Hayır | Hayır | Değiştirilemez |
| `sealed` | Aynı dosya | Hayır | Exhaustive switch |
| `mixin` | — | with | Ortak davranış |

### Tüm Kombinasyonlar

| Bildiri | Construct | Extend | Implement | Mix in | Exhaustive |
|---------|-----------|--------|-----------|--------|------------|
| `class` | Evet | Evet | Evet | Hayır | Hayır |
| `base class` | Evet | Evet | Hayır | Hayır | Hayır |
| `interface class` | Evet | Hayır | Evet | Hayır | Hayır |
| `final class` | Evet | Hayır | Hayır | Hayır | Hayır |
| `sealed class` | Hayır | Hayır | Hayır | Hayır | Evet |
| `abstract class` | Hayır | Evet | Evet | Hayır | Hayır |
| `abstract base class` | Hayır | Evet | Hayır | Hayır | Hayır |
| `abstract interface class` | Hayır | Hayır | Evet | Hayır | Hayır |
| `abstract final class` | Hayır | Hayır | Hayır | Hayır | Hayır |
| `mixin class` | Evet | Evet | Evet | Evet | Hayır |
| `base mixin class` | Evet | Evet | Hayır | Evet | Hayır |
| `abstract mixin class` | Hayır | Evet | Evet | Evet | Hayır |
| `abstract base mixin class` | Hayır | Evet | Hayır | Evet | Hayır |
| `mixin` | Hayır | Hayır | Evet | Evet | Hayır |
| `base mixin` | Hayır | Hayır | Hayır | Evet | Hayır |

### Karar Rehberi

| Senaryo | Modifier |
|---------|----------|
| API/Service kontratı | `abstract interface class` |
| Default implementasyonlu kontrat | `interface class` |
| Base repository/bloc | `abstract base class` |
| Manager/Service implementasyonu | `final class` |
| Bloc/Cubit state | `sealed class` |
| Validation helpers | `mixin` |
| Disposable pattern | `mixin class` |
| Utility (static metodlar) | `abstract final class` |
| Config/Settings | `final class` |

### Dart 2.x → 3.x Migration

```dart
class MyClass {}           →  final class MyClass {}
class Mixin {}             →  mixin class Mixin {}
abstract class IService {} →  abstract interface class IService {}
abstract class BaseRepo {} →  abstract base class BaseRepo {}
```
