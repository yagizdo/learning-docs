# POC - ObjectBox Cache System

## İçindekiler

1. [ObjectBox Nedir?](#objectbox-nedir)
2. [Temel Özellikler](#temel-özellikler)
3. [Kurulum](#kurulum)
4. [Implementation Patterns](#implementation-patterns)
5. [Gerçek Dünya Örneği](#gerçek-dünya-örneği)
6. [Hızlı Referans](#hızlı-referans)

---

## ObjectBox Nedir?

ObjectBox, Flutter/Dart için yüksek performanslı, ACID uyumlu NoSQL veritabanıdır.

**Neden ObjectBox?**

- ORM overhead yok - doğrudan nesne persist
- SQLite'dan 10x daha hızlı
- Cross-platform (iOS, Android, Desktop, Web)
- Reactive streams desteği
- İlişkisel veri desteği (ToOne, ToMany)

---

## Temel Özellikler

### Entity & Annotations

| Annotation | Açıklama | Örnek |
|------------|----------|-------|
| `@Entity()` | Sınıfı veritabanı nesnesi yapar | `@Entity() class User {}` |
| `@Id()` | Auto-increment primary key | `@Id() int id = 0;` |
| `@Index` | Sorgu performansını artırır | `@Index() String email;` |
| `@Unique` | Benzersizlik zorunlu | `@Unique() String username;` |
| `@Property(type:)` | Özel tip tanımı | `@Property(type: PropertyType.date)` |
| `@Transient` | Veritabanına kaydedilmez | `@Transient() String tempData;` |

### Relations

**ToOne - Bire-bir ilişki:**

```dart
@Entity()
class Order {
  @Id() int id = 0;
  final customer = ToOne<Customer>();
}
```

**ToMany - Bire-çok ilişki:**

```dart
@Entity()
class Customer {
  @Id() int id = 0;
  @Backlink(to: 'customer')
  final orders = ToMany<Order>();
}
```

### Queries

**Temel sorgu:**

```dart
final query = box.query(User_.email.equals('test@mail.com')).build();
final user = query.findFirst();
query.close();
```

**Koşul operatörleri:**

| Operatör | Kullanım |
|----------|----------|
| `equals()` | Eşitlik kontrolü |
| `notEquals()` | Eşit değil |
| `contains()` | String içerir |
| `startsWith()` | İle başlar |
| `endsWith()` | İle biter |
| `between()` | Aralık sorgusu |
| `greaterThan()` | Büyüktür |
| `lessThan()` | Küçüktür |
| `greaterOrEqual()` | Büyük veya eşit |
| `lessOrEqual()` | Küçük veya eşit |
| `isNull()` | Null kontrolü |
| `oneOf()` | Liste içinde var mı |
| `&` | VE (AND) |
| `|` | VEYA (OR) |

**Sıralama ve limit:**

```dart
final query = box.query(condition)
    .order(User_.name)
    .build();
query.limit = 10;
query.offset = 0;
final results = query.find();
query.close();
```

### Reactive Streams

**Query watch:**

```dart
final stream = box.query(condition)
    .watch(triggerImmediately: true)
    .map((q) => q.find());
```

**StreamBuilder ile kullanım:**

```dart
StreamBuilder<List<User>>(
  stream: userStream,
  builder: (context, snapshot) {
    if (!snapshot.hasData) return CircularProgressIndicator();
    return ListView.builder(
      itemCount: snapshot.data!.length,
      itemBuilder: (ctx, i) => Text(snapshot.data![i].name),
    );
  },
)
```

### Transactions

**Atomic işlemler:**

```dart
store.runInTransaction(TxMode.write, () {
  box.put(user1);
  box.put(user2);
  // Ya hepsi başarılı ya hiçbiri
});
```

**Async transaction (UI bloklama yok):**

```dart
await store.runInTransactionAsync(TxMode.write, (Store store, tx) {
  store.box<User>().putMany(users);
});
```

**MVCC (Multi-Version Concurrency Control):**

- Okuma işlemleri yazma işlemlerini bloklamaz
- Paralel okuma güvenli
- Yazma işlemleri sıralı çalışır

---

## Kurulum

**pubspec.yaml:**

```yaml
dependencies:
  objectbox: ^4.0.0

dev_dependencies:
  build_runner: ^2.4.0
  objectbox_generator: ^4.0.0
```

**Code generation:**

```bash
dart run build_runner build
```

**iOS için (Podfile):**

```ruby
platform :ios, '12.0'
```

---

## Implementation Patterns

### Pattern 1: Factory Pattern (Singleton + Generic Manager)

Merkezi Store yönetimi ve generic CRUD operasyonları için.

**Entity Tanımları:**

```dart
@Entity()
class User {
  @Id() int id = 0;
  String name;
  String email;
  String? avatarUrl;

  @Property(type: PropertyType.date)
  DateTime createdAt;

  User({required this.name, required this.email, this.avatarUrl, DateTime? createdAt})
      : createdAt = createdAt ?? DateTime.now();
}

@Entity()
class Product {
  @Id() int id = 0;
  String name;
  String description;
  double price;
  String imageUrl;
  bool isFavorite;

  Product({
    required this.name,
    required this.description,
    required this.price,
    required this.imageUrl,
    this.isFavorite = false,
  });
}

@Entity()
class Order {
  @Id() int id = 0;
  String status;
  double totalAmount;

  @Property(type: PropertyType.date)
  DateTime orderDate;

  final user = ToOne<User>();

  Order({required this.status, required this.totalAmount, DateTime? orderDate})
      : orderDate = orderDate ?? DateTime.now();
}
```

**Interface (Kontrat):**

```dart
abstract interface class IBaseManager<T> {
  int save(T entity);
  List<int> saveAll(List<T> entities);
  T? getById(int id);
  List<T> getAll();
  bool deleteById(int id);
  int deleteAll();
  int get count;
}
```

**BaseManager (Generic CRUD):**

```dart
base class BaseManager<T> implements IBaseManager<T> {
  @protected
  final Box<T> box;

  BaseManager(this.box);

  @override
  int save(T entity) => box.put(entity);

  @override
  List<int> saveAll(List<T> entities) => box.putMany(entities);

  @override
  T? getById(int id) => box.get(id);

  @override
  List<T> getAll() => box.getAll();

  @override
  bool deleteById(int id) => box.remove(id);

  @override
  int deleteAll() => box.removeAll();

  @override
  int get count => box.count();

  @protected
  List<T> executeQuery(Query<T> query) {
    try {
      return query.find();
    } finally {
      query.close();
    }
  }

  @protected
  T? executeQueryFirst(Query<T> query) {
    try {
      return query.findFirst();
    } finally {
      query.close();
    }
  }
}
```

**Özel Manager'lar:**

```dart
final class UserManager extends BaseManager<User> {
  UserManager(super.box);

  User? findByEmail(String email) {
    final query = box.query(User_.email.equals(email)).build();
    return executeQueryFirst(query);
  }

  List<User> searchByName(String name) {
    final query = box.query(User_.name.contains(name, caseSensitive: false)).build();
    return executeQuery(query);
  }
}

final class ProductManager extends BaseManager<Product> {
  ProductManager(super.box);

  List<Product> getFavorites() {
    final query = box.query(Product_.isFavorite.equals(true)).build();
    return executeQuery(query);
  }

  List<Product> filterByPriceRange(double min, double max) {
    final query = box.query(
      Product_.price.greaterOrEqual(min) & Product_.price.lessOrEqual(max)
    ).build();
    return executeQuery(query);
  }

  void toggleFavorite(int productId) {
    final product = getById(productId);
    if (product != null) {
      product.isFavorite = !product.isFavorite;
      save(product);
    }
  }
}

final class OrderManager extends BaseManager<Order> {
  OrderManager(super.box);

  List<Order> findByStatus(String status) {
    final query = box.query(Order_.status.equals(status)).build();
    return executeQuery(query);
  }

  List<Order> getPendingOrders() => findByStatus('pending');
  List<Order> getDeliveredOrders() => findByStatus('delivered');

  double getTotalSpending() {
    return getAll().fold(0.0, (sum, order) => sum + order.totalAmount);
  }
}
```

**CacheFactory (Singleton):**

```dart
final class CacheFactory {
  static CacheFactory? _instance;
  final Store _store;

  CacheFactory._(this._store);

  static CacheFactory get instance {
    final i = _instance;
    if (i == null) {
      throw StateError('CacheFactory not initialized. Call init() first.');
    }
    return i;
  }

  static Future<void> init() async {
    if (_instance != null) return;
    final store = await openStore();
    _instance = CacheFactory._(store);
  }

  late final UserManager userManager = UserManager(_store.box<User>());
  late final ProductManager productManager = ProductManager(_store.box<Product>());
  late final OrderManager orderManager = OrderManager(_store.box<Order>());

  void dispose() {
    _store.close();
    _instance = null;
  }

  void clearAll() {
    userManager.deleteAll();
    productManager.deleteAll();
    orderManager.deleteAll();
  }
}
```

**Ne zaman kullan:**

- Merkezi cache yönetimi gerektiğinde
- Tek Store instance zorunlu olduğunda
- Manager'lar arası koordinasyon gerektiğinde
- Basit, hızlı kurulum istendiğinde

---

### Pattern 2: Repository Pattern

Dependency Injection ve test edilebilirlik öncelikli projeler için.

**Repository Interface:**

```dart
abstract interface class IRepository<T> {
  Future<int> save(T entity);
  Future<T?> getById(int id);
  Future<List<T>> getAll();
  Future<bool> delete(int id);
  Future<int> deleteAll();
}
```

**Concrete Repository:**

```dart
class UserRepository implements IRepository<User> {
  final Box<User> _box;

  UserRepository(Store store) : _box = store.box<User>();

  @override
  Future<int> save(User entity) async => _box.put(entity);

  @override
  Future<User?> getById(int id) async => _box.get(id);

  @override
  Future<List<User>> getAll() async => _box.getAll();

  @override
  Future<bool> delete(int id) async => _box.remove(id);

  @override
  Future<int> deleteAll() async => _box.removeAll();

  // Domain-specific queries
  Future<User?> findByEmail(String email) async {
    final query = _box.query(User_.email.equals(email)).build();
    try {
      return query.findFirst();
    } finally {
      query.close();
    }
  }
}
```

**DI ile kullanım (get_it örneği):**

```dart
final getIt = GetIt.instance;

Future<void> setupDependencies() async {
  final store = await openStore();

  getIt.registerSingleton<Store>(store);
  getIt.registerLazySingleton<IRepository<User>>(() => UserRepository(getIt<Store>()));
  getIt.registerLazySingleton<IRepository<Product>>(() => ProductRepository(getIt<Store>()));
}
```

**Ne zaman kullan:**

- Dependency Injection kullanıyorsan
- Test edilebilirlik öncelikse
- Her repository bağımsız olabilir
- Clean Architecture uyguluyorsan

---

### Pattern Karşılaştırması

| Özellik | Factory Pattern | Repository Pattern |
|---------|-----------------|-------------------|
| Store yönetimi | Merkezi | Dağıtık (DI container) |
| Test kolaylığı | Orta | Yüksek |
| DI uyumu | Düşük | Yüksek |
| Kurulum basitliği | Yüksek | Orta |
| Bağımlılık yönetimi | Implicit | Explicit |
| Kullanım senaryosu | Basit uygulamalar | Enterprise/Clean Arch |

---

## Gerçek Dünya Örneği

### E-ticaret Cache Sistemi

**main.dart:**

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await CacheFactory.init();
  runApp(const MyApp());
}
```

**ProductCubit:**

```dart
class ProductCubit extends Cubit<ProductState> {
  final _productManager = CacheFactory.instance.productManager;

  ProductCubit() : super(ProductInitial());

  void loadFavorites() {
    emit(ProductLoading());
    final favorites = _productManager.getFavorites();

    if (favorites.isEmpty) {
      emit(ProductEmpty('Henüz favori ürününüz yok'));
    } else {
      emit(ProductLoaded(favorites));
    }
  }

  void toggleFavorite(int productId) {
    _productManager.toggleFavorite(productId);
    loadFavorites();
  }

  void filterByPrice(double min, double max) {
    emit(ProductLoading());
    final filtered = _productManager.filterByPriceRange(min, max);
    emit(ProductLoaded(filtered));
  }
}
```

**FavoritesView:**

```dart
class FavoritesView extends StatelessWidget {
  const FavoritesView({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => ProductCubit()..loadFavorites(),
      child: Scaffold(
        appBar: AppBar(title: const Text('Favorilerim')),
        body: BlocBuilder<ProductCubit, ProductState>(
          builder: (context, state) => switch (state) {
            ProductInitial() => const SizedBox.shrink(),
            ProductLoading() => const Center(child: CircularProgressIndicator()),
            ProductEmpty(:final message) => Center(child: Text(message)),
            ProductLoaded(:final products) => ListView.builder(
              itemCount: products.length,
              itemBuilder: (context, index) {
                final product = products[index];
                return ListTile(
                  title: Text(product.name),
                  subtitle: Text('${product.price} TL'),
                  trailing: IconButton(
                    icon: Icon(
                      product.isFavorite ? Icons.favorite : Icons.favorite_border,
                      color: product.isFavorite ? Colors.red : null,
                    ),
                    onPressed: () => context.read<ProductCubit>().toggleFavorite(product.id),
                  ),
                );
              },
            ),
            ProductError(:final message) => Center(child: Text(message)),
          },
        ),
      ),
    );
  }
}
```

---

## Hızlı Referans

### Box Operasyonları

| Metod | Açıklama |
|-------|----------|
| `put(entity)` | Kaydet/güncelle (id varsa update) |
| `putMany(list)` | Toplu kaydet |
| `putQueued(entity)` | Async kaydet (fire-and-forget) |
| `get(id)` | ID ile getir |
| `getMany(ids)` | Birden fazla ID ile getir |
| `getAll()` | Tümünü getir |
| `remove(id)` | ID ile sil |
| `removeMany(ids)` | Toplu sil |
| `removeAll()` | Tümünü sil |
| `count()` | Toplam kayıt sayısı |
| `isEmpty` | Boş mu kontrolü |
| `contains(id)` | ID var mı kontrolü |

### Query Builder Cheatsheet

```dart
// Temel sorgu
box.query(User_.name.equals('Ali')).build()

// Çoklu koşul (AND)
box.query(User_.age.greaterThan(18) & User_.isActive.equals(true)).build()

// Çoklu koşul (OR)
box.query(User_.role.equals('admin') | User_.role.equals('moderator')).build()

// String arama
box.query(User_.name.contains('ali', caseSensitive: false)).build()

// Null kontrolü
box.query(User_.email.isNull()).build()

// Sıralama
box.query(condition).order(User_.name, flags: Order.descending).build()

// Birden fazla sıralama
box.query(condition).order(User_.lastName).order(User_.firstName).build()

// Limit ve offset
query.limit = 20;
query.offset = 40; // sayfa 3

// Property query (tek alan)
box.query(condition).build().property(User_.email).find() // List<String>

// Distinct
box.query(condition).build().property(User_.city).distinct().find()
```

### Performans İpuçları

| Durum | Önerilen |
|-------|----------|
| Toplu kayıt | `putMany()` kullan, döngüde `put()` değil |
| Sık sorgulanan alan | `@Index` annotation ekle |
| Unique alan | `@Unique` kullan, manuel kontrol yapma |
| Query sonrası | `query.close()` çağırmayı unutma |
| Transaction | Ağır iş yapma, kısa tut |
| Büyük veri | `putQueued()` ile async kaydet |
| Sayfalama | `limit` ve `offset` kullan |
| Tek sonuç | `findFirst()` kullan, `find()[0]` değil |

### Yapı Özeti

```
CacheFactory (Singleton)
├── userManager → UserManager
├── productManager → ProductManager
└── orderManager → OrderManager

Manager Hierarchy:
IBaseManager<T> (interface)
    ↑
BaseManager<T> (base class)
    ↑
UserManager, ProductManager, OrderManager (final classes)
```
