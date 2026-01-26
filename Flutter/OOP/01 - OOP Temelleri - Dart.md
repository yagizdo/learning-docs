Bu bölümde nesne yönelimli programlamanın dört temel kavramını öğreneceksin.

**Ön Bilgi:** Dart syntax bilgisi
**Sonraki:** [02 - Class Modifiers](02%20-%20Dart%203.x%20Class%20Modifiers.md)

---

## İçindekiler

1. [Problem: Tekrar Eden Kod](#problem-tekrar-eden-kod)
2. [Abstraction](#1-abstraction-soyutlama)
3. [Encapsulation](#2-encapsulation-kapsülleme)
4. [Inheritance](#3-inheritance-miras-alma)
5. [Polymorphism](#4-polymorphism-çok-biçimlilik)
6. [extends vs implements](#extends-vs-implements)

---

## Problem: Tekrar Eden Kod

Aşağıdaki kodu incele:

```dart
class UserScreen {
  Future<User> fetchUser() async {
    final dio = Dio()..options.baseUrl = 'https://api.com';
    final token = await prefs.getString('token');
    final response = await dio.get('/user',
      options: Options(headers: {'Authorization': 'Bearer $token'}));
    return User.fromJson(response.data);
  }
}

class ProductScreen {
  Future<List<Product>> fetchProducts() async {
    final dio = Dio()..options.baseUrl = 'https://api.com';
    final token = await prefs.getString('token');
    final response = await dio.get('/products',
      options: Options(headers: {'Authorization': 'Bearer $token'}));
    return (response.data as List).map(Product.fromJson).toList();
  }
}
```

**Problem:** Dio setup, token ekleme ve base URL her ekranda tekrarlanıyor. 50 ekranda aynı kod olacak. Base URL değişirse 50 yeri güncellemen gerekecek.

OOP bu problemi çözer.

---

## 1. Abstraction (Soyutlama)

**Tanım:** "Ne yapılacağını" tanımla, "nasıl yapılacağını" tanımlama.

### Kavram

Araba kullanırken "motoru çalıştır" diyorsun. Motorun içinde ne olduğunu bilmene gerek yok. Bu soyutlamadır.

### Dart'ta Uygulama

```dart
abstract class IUserRepository {
  Future<User> getUser(int id);
  Future<void> saveUser(User user);
}
```

Bu bir **kontrat**. "Bu interface'i implement eden her sınıf bu metodları sağlamalı" diyor.

### Neden Kullanılır

```dart
class UserCubit extends Cubit<UserState> {
  final IUserRepository _repository;

  UserCubit(this._repository) : super(UserInitial());

  Future<void> loadUser(int id) async {
    emit(UserLoading());
    final user = await _repository.getUser(id);
    emit(UserLoaded(user));
  }
}
```

Cubit, repository'nin nasıl çalıştığını bilmiyor. Sadece `getUser` metodunun varlığını biliyor.

**Fayda:** Repository implementasyonu değişse bile (SharedPreferences → Firebase), Cubit'e dokunmuyorsun.

---

## 2. Encapsulation (Kapsülleme)

**Tanım:** İç detayları gizle, sadece gerekli olanı dışarı aç.

### Kavram

ATM'de sadece "Para Çek" butonu var. Kasanın nasıl açıldığını görmüyorsun.

### Dart'ta Uygulama

```dart
class AuthService {
  String? _accessToken;   // Private
  String? _refreshToken;  // Private

  bool get isLoggedIn => _accessToken != null;  // Public getter

  Future<void> login(String email, String password) async {
    final response = await _authenticate(email, password);
    _accessToken = response.accessToken;
    _refreshToken = response.refreshToken;
    await _saveTokens();
  }

  Future<AuthResponse> _authenticate(String email, String password) async {
    // API call
  }

  Future<void> _saveTokens() async {
    // Secure storage'a kaydet
  }
}
```

### Kullanım

```dart
final auth = AuthService();
await auth.login('test@test.com', '123456');

print(auth.isLoggedIn);    // true
print(auth._accessToken);  // HATA - Private'a erişilemez
```

**Fayda:** Token yönetimi değişse bile dışarıdaki kodlar etkilenmez.

---

## 3. Inheritance (Miras Alma)

**Tanım:** Ortak özellikleri bir üst sınıftan al.

### Kavram

"Köpek" ve "Kedi" ikisi de "Hayvan"dan türer. İkisi de yemek yer, uyur. Bu ortak davranışlar Hayvan sınıfında tanımlanır.

### Dart'ta Uygulama

```dart
abstract class BaseRepository {
  final Dio dio;

  BaseRepository(this.dio);

  Future<Options> get authOptions async {
    final token = await storage.getString('token');
    return Options(headers: {'Authorization': 'Bearer $token'});
  }

  Exception handleError(DioException e) => switch (e.response?.statusCode) {
    401 => UnauthorizedException(),
    404 => NotFoundException(),
    _ => NetworkException(e.message ?? ''),
  };
}

class UserRepository extends BaseRepository {
  UserRepository(super.dio);

  Future<User> getUser(int id) async {
    final response = await dio.get('/users/$id', options: await authOptions);
    return User.fromJson(response.data);
  }
}

class ProductRepository extends BaseRepository {
  ProductRepository(super.dio);

  Future<List<Product>> getProducts() async {
    final response = await dio.get('/products', options: await authOptions);
    return (response.data as List).map(Product.fromJson).toList();
  }
}
```

**Fayda:** `authOptions` ve `handleError` bir kere yazıldı, her repository'de kullanılıyor.

---

## 4. Polymorphism (Çok Biçimlilik)

**Tanım:** Aynı interface, farklı davranışlar.

### Kavram

"Ödeme yap" butonu. Kart, nakit veya kripto ile ödeme yapabilirsin. Buton aynı, arkadaki işlem farklı.

### Dart'ta Uygulama

```dart
abstract class IPaymentService {
  Future<PaymentResult> pay(double amount);
}

class StripePaymentService implements IPaymentService {
  @override
  Future<PaymentResult> pay(double amount) async {
    // Stripe API call
    return PaymentResult.success('stripe_123');
  }
}

class IyzicoPaymentService implements IPaymentService {
  @override
  Future<PaymentResult> pay(double amount) async {
    // iyzico API call
    return PaymentResult.success('iyzico_456');
  }
}

class CheckoutCubit extends Cubit<CheckoutState> {
  final IPaymentService _paymentService;

  CheckoutCubit(this._paymentService) : super(CheckoutInitial());

  Future<void> checkout(double amount) async {
    emit(CheckoutLoading());
    final result = await _paymentService.pay(amount);
    emit(CheckoutSuccess(result.transactionId));
  }
}
```

### Kullanım

```dart
// Türkiye
final cubit = CheckoutCubit(IyzicoPaymentService());

// Yurtdışı
final cubit = CheckoutCubit(StripePaymentService());

// Test
final cubit = CheckoutCubit(MockPaymentService());
```

**Fayda:** Aynı Cubit farklı ödeme sistemleriyle çalışıyor.

---

## extends vs implements

| Özellik | extends | implements |
|---------|---------|------------|
| Kod miras alır | Evet | Hayır |
| Kaç tane | 1 | Birden fazla |
| Kullanım | Ortak kod paylaşımı | Kontrat tanımlama |

### Karar Verme

```dart
// Üst sınıfta { } içinde kod var mı?
abstract class BaseRepository {
  Future<Options> get authOptions async {  // Kod var
    final token = await storage.getString('token');
    return Options(headers: {'Authorization': 'Bearer $token'});
  }
}
// extends kullan

// Üst sınıfta sadece ; var mı?
abstract class IUserRepository {
  Future<User> getUser(int id);  // Sadece imza
  Future<void> saveUser(User user);
}
// implements kullan
```

### Yaygın Pattern

```dart
class UserRepository extends BaseRepository implements IUserRepository {
  UserRepository(super.dio);

  @override
  Future<User> getUser(int id) async {
    final response = await dio.get('/users/$id', options: await authOptions);
    return User.fromJson(response.data);
  }

  @override
  Future<void> saveUser(User user) async {
    await dio.post('/users', data: user.toJson(), options: await authOptions);
  }
}
```

- `BaseRepository`'den: authOptions, handleError (kod)
- `IUserRepository`'den: getUser, saveUser (kontrat)

---

## Özet

| Kavram | Tanım | Örnek |
|--------|-------|-------|
| Abstraction | Ne yapılacağını tanımla | `abstract class IRepository` |
| Encapsulation | İç detayları gizle | `_privateField` |
| Inheritance | Ortak kodu miras al | `extends BaseRepository` |
| Polymorphism | Aynı interface, farklı davranış | `implements IPaymentService` |
