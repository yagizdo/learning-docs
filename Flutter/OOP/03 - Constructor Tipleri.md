Bu bölümde Dart'taki farklı constructor tiplerini ve factory constructor'ın ne zaman kullanılacağını öğreneceksin.

**Ön Bilgi:** [02 - Class Modifiers](02%20-%20Dart%203.x%20Class%20Modifiers.md)
**Sonraki:** [04 - Design Patterns](04%20-%20Design%20Patterns.md)

---

## İçindekiler

1. [Constructor Tipleri Karşılaştırması](#constructor-tipleri-karşılaştırması)
2. [Initializer List](#initializer-list)
3. [Super Initializer](#super-initializer)
4. [Super Parameters](#super-parameters)
5. [Default Constructor](#1-default-constructor)
6. [Named Constructor](#2-named-constructor)
7. [Factory Constructor](#3-factory-constructor)
8. [Karar Şeması](#karar-şeması)
9. [Dikkat Edilecekler](#dikkat-edilecekler)

---

## Constructor Tipleri Karşılaştırması

| Tip | Yeni Instance | Cache/Singleton | Subclass Dönebilir |
|-----|---------------|-----------------|-------------------|
| Default | Her zaman | Hayır | Hayır |
| Named | Her zaman | Hayır | Hayır |
| Factory | Opsiyonel | Evet | Evet |

---

## Initializer List

Constructor body'den önce field'ları başlatmak için kullanılır. `:` ile başlar ve virgülle ayrılır.

### Temel Syntax

```dart
class Point {
  final double x;
  final double y;
  final double distanceFromOrigin;

  Point(this.x, this.y)
    : distanceFromOrigin = sqrt(x * x + y * y);
}
```

### Kurallar

- Initializer list constructor body'den **önce** çalışır
- Sağ tarafta `this` erişimi **yok**
- Final field'lar burada başlatılabilir

```dart
class Rectangle {
  final double width;
  final double height;
  final double area;

  Rectangle(this.width, this.height)
    : area = width * height;  // OK: parametrelere erişim var

  // HATA: this.width kullanılamaz
  Rectangle.wrong(this.width, this.height)
    : area = this.width * this.height;  // Compile error
}
```

### Assert ile Validasyon

```dart
class PositivePoint {
  final double x;
  final double y;

  PositivePoint(this.x, this.y)
    : assert(x >= 0, 'x must be positive'),
      assert(y >= 0, 'y must be positive');
}
```

---

## Super Initializer

Üst sınıfın constructor'ını çağırmak için kullanılır.

### Temel Kullanım

```dart
class Person {
  final String name;
  Person(this.name);
}

class Employee extends Person {
  final String department;

  Employee(String name, this.department)
    : super(name);  // Person constructor'ını çağır
}
```

### Named Constructor ile

```dart
class Person {
  final String name;
  Person(this.name);
  Person.anonymous() : name = 'Anonymous';
}

class Employee extends Person {
  final String department;

  Employee.temp(this.department)
    : super.anonymous();  // Person.anonymous() çağır
}
```

### Gerçek Senaryo: Theme Sınıfı

```dart
final class CustomFilledButtonTheme extends FilledButtonThemeData {
  CustomFilledButtonTheme({
    required AppColorScheme colorScheme,
    required TextTheme textTheme,
  }) : super(
         style: FilledButtonStyle(colorScheme, textTheme),
       );
}
```

### Çalışma Sırası

```dart
class Parent {
  Parent() {
    print('2. Parent constructor body');
  }
}

class Child extends Parent {
  final int value;

  Child(int v)
    : value = v,      // 1. Initializer list
      super() {       // 2. Parent constructor
    print('3. Child constructor body');
  }
}

// Çıktı:
// 2. Parent constructor body
// 3. Child constructor body
```

**Best Practice:** `super()` initializer list'te **en sona** yazılmalı.

---

## Super Parameters

Dart 2.17+ ile gelen shorthand syntax. Parametreleri üst sınıfa otomatik iletir.

### Geleneksel Yöntem

```dart
class Vector2d {
  final double x;
  final double y;
  Vector2d(this.x, this.y);
}

// Eski yöntem - tekrar var
class Vector3d extends Vector2d {
  final double z;
  Vector3d(double x, double y, this.z) : super(x, y);
}
```

### Super Parameters ile

```dart
class Vector3d extends Vector2d {
  final double z;
  Vector3d(super.x, super.y, this.z);  // Otomatik forward
}
```

### Named Parameters

```dart
class Parent {
  final String name;
  final int age;
  Parent({required this.name, required this.age});
}

class Child extends Parent {
  final String school;

  Child({
    required super.name,   // Parent'a forward
    required super.age,    // Parent'a forward
    required this.school,
  });
}

final child = Child(name: 'Ali', age: 10, school: 'ABC');
```

### Ne Zaman Kullanılır?

| Durum | Yöntem |
|-------|--------|
| Parametreler doğrudan parent'a gidecek | `super.param` |
| Parametre işlendikten sonra parent'a gidecek | `super(processed)` |
| Parent'ta default constructor yok | `super.namedConstructor()` |

```dart
// super.param - doğrudan forward
class Dog extends Animal {
  Dog(super.name);
}

// super() - işlenmiş değer
class UppercaseDog extends Animal {
  UppercaseDog(String name) : super(name.toUpperCase());
}
```

---

## 1. Default Constructor

En basit constructor tipi.

```dart
class User {
  final String name;
  final String email;

  User(this.name, this.email);
}

final user = User('Ahmet', 'ahmet@test.com');
```

Her çağrıda yeni instance oluşturur.

---

## 2. Named Constructor

Farklı oluşturma yolları için kullanılır.

```dart
class User {
  final String name;
  final String email;
  final bool isGuest;

  User(this.name, this.email) : isGuest = false;

  User.guest()
      : name = 'Guest',
        email = '',
        isGuest = true;

  User.fromJson(Map<String, dynamic> json)
      : name = json['name'] as String,
        email = json['email'] as String,
        isGuest = false;
}

final user1 = User('Ahmet', 'ahmet@test.com');
final user2 = User.guest();
final user3 = User.fromJson({'name': 'Ali', 'email': 'ali@test.com'});
```

Her çağrıda yeni instance oluşturur. `this` keyword'üne erişimi vardır.

---

## 3. Factory Constructor

`return` ile ne döneceğine sen karar verirsin.

### 3.1 Singleton Pattern

```dart
class DatabaseService {
  static DatabaseService? _instance;

  DatabaseService._internal();

  factory DatabaseService() {
    _instance ??= DatabaseService._internal();
    return _instance!;
  }
}

final db1 = DatabaseService();
final db2 = DatabaseService();
print(db1 == db2);  // true - aynı instance
```

### 3.2 Cache

```dart
class ImageLoader {
  static final Map<String, ImageLoader> _cache = {};

  final String url;
  final Uint8List data;

  ImageLoader._internal(this.url, this.data);

  factory ImageLoader(String url) {
    if (_cache.containsKey(url)) {
      return _cache[url]!;
    }

    final data = _loadFromNetwork(url);
    final loader = ImageLoader._internal(url, data);
    _cache[url] = loader;
    return loader;
  }

  static Uint8List _loadFromNetwork(String url) {
    // Network call
    return Uint8List(0);
  }
}
```

### 3.3 Subclass Dönme

```dart
abstract class Animal {
  String speak();

  factory Animal(String type) {
    return switch (type) {
      'dog' => Dog(),
      'cat' => Cat(),
      _ => throw ArgumentError('Unknown animal: $type'),
    };
  }
}

class Dog implements Animal {
  @override
  String speak() => 'Hav hav';
}

class Cat implements Animal {
  @override
  String speak() => 'Miyav';
}

final animal = Animal('dog');  // Dog instance döner
print(animal.speak());  // Hav hav
```

---

## Karar Şeması

```
Her zaman yeni instance mı?
├─ Evet → Farklı oluşturma yolları var mı?
│         ├─ Evet → Named Constructor
│         └─ Hayır → Default Constructor
│
└─ Hayır → Factory Constructor
           ├─ Singleton → factory ClassName()
           ├─ Cache → factory ClassName(param)
           └─ Subclass → factory ClassName.create(type)
```

---

## ObjectBox CacheFactory Örneği

```dart
final class CacheFactory {
  static CacheFactory? _instance;
  final Store _store;

  CacheFactory._(this._store);

  factory CacheFactory() {
    if (_instance == null) {
      throw StateError('CacheFactory not initialized. Call init() first.');
    }
    return _instance!;
  }

  static Future<void> init() async {
    if (_instance != null) return;
    final store = await openStore();
    _instance = CacheFactory._(store);
  }

  late final UserManager userManager = UserManager(_store.box<User>());
  late final OrderManager orderManager = OrderManager(_store.box<Order>());

  void dispose() {
    _store.close();
    _instance = null;
  }
}

// Kullanım
await CacheFactory.init();
final factory = CacheFactory();
factory.userManager.save(user);
```

---

## Dikkat Edilecekler

### Factory Constructor'da this Yok

```dart
class User {
  final String name;

  // HATA
  factory User.create() {
    this.name = 'Test';  // Compile error
    return User('Test');
  }

  // OK
  factory User.create() {
    return User('Test');
  }

  User(this.name);
}
```

### Factory Constructor Async Olamaz

```dart
class Service {
  // HATA
  factory Service() async {
    await init();
    return Service._();
  }

  // OK - static async method kullan
  static Future<Service> create() async {
    await init();
    return Service._();
  }

  Service._();
}
```

---

## Özet

| Constructor | this erişimi | return zorunlu | Yeni instance |
|-------------|--------------|----------------|---------------|
| Default | Evet | Hayır | Her zaman |
| Named | Evet | Hayır | Her zaman |
| Factory | Hayır | Evet | Opsiyonel |
