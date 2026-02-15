# 📘 03 — OOP & Protocol-Oriented Programming (POP)

> **Phase 1 · Doküman 3/6**  
> **Konu:** OOP prensipleri, protocol-oriented programming, composition over inheritance  
> **Tahmini Süre:** 4–5 gün  
> **Ön Koşul:** [01 — Swift Temelleri](01%20-%20Swift%20Temelleri.md), [02 — Optionals](02%20-%20Optionals.md)

---

## 📑 İçindekiler

### Bölüm A: OOP
1. [OOP'nin Dört Temel Prensibi](#1-oopnin-dört-temel-prensibi)
2. [Inheritance (Kalıtım)](#2-inheritance-kalıtım)
3. [Polymorphism (Çok Biçimlilik)](#3-polymorphism-çok-biçimlilik)
4. [Encapsulation (Kapsülleme)](#4-encapsulation-kapsülleme)
5. [Abstraction (Soyutlama)](#5-abstraction-soyutlama)

### Bölüm B: POP
6. [POP'a Giriş: Swift'in DNA'sı](#6-popa-giriş-swiftin-dnası)
7. [Protocol Tanımlama ve Conformance](#7-protocol-tanımlama-ve-conformance)
8. [Protocol Extensions ve Default Implementations](#8-protocol-extensions-ve-default-implementations)
9. [Composition over Inheritance](#9-composition-over-inheritance)
10. [Protocol'ler ile Dependency Injection](#10-protokoller-ile-dependency-injection)
11. [Gerçek Hayat: Protocol-Based Service Layer](#11-gerçek-hayat-protocol-based-service-layer)

### Bölüm C: Karşılaştırma
12. [POP vs OOP — Mülakat Rehberi](#12-pop-vs-oop--mülakat-rehberi)

---

# BÖLÜM A: OBJECT-ORIENTED PROGRAMMING

## 1. OOP'nin Dört Temel Prensibi

| Prensip | Bir Cümlede | Swift Mekanizması |
|---|---|---|
| **Inheritance** | Bir sınıf, başka bir sınıfın özellik ve davranışlarını miras alır | `class Child: Parent` |
| **Polymorphism** | Farklı tipler, aynı arayüzü farklı şekillerde uygular | `override`, protocol conformance |
| **Encapsulation** | İç detayları gizle, kontrollü erişim sun | `private`, `private(set)` |
| **Abstraction** | Karmaşıklığı gizle, basit arayüz sun | Protocol'ler |

UIKit tamamen OOP tabanlıdır — `UIViewController`, `UIView`, `UITableViewCell` gibi sınıflar kalıtım hiyerarşisine dayanır. Bu yüzden iOS geliştirici olarak OOP'yi iyi bilmen gerekir.

---

## 2. Inheritance (Kalıtım)

Bir sınıfın (child/subclass) başka bir sınıftan (parent/superclass) özellik ve davranışları **miras almasıdır**. Swift'te yalnızca `class`'lar kalıtım destekler.

```swift
// ── Üst Sınıf ──
class Vehicle {
    let brand: String
    var speed: Double = 0
    
    init(brand: String) {
        self.brand = brand
    }
    
    func accelerate() {
        speed += 10
        print("[\(brand)] Hız: \(speed) km/h")
    }
    
    func describe() -> String {
        return "\(brand) — \(speed) km/h"
    }
}

// ── Alt Sınıf — miras alır ve genişletir ──
class ElectricCar: Vehicle {
    var batteryLevel: Double = 100
    
    // override — üst sınıfın davranışını değiştir
    override func accelerate() {
        guard batteryLevel > 0 else {
            print("[\(brand)] Batarya boş!")
            return
        }
        super.accelerate()  // Üst sınıfın orijinal davranışını çağır
        batteryLevel -= 2
    }
    
    // Yeni davranış ekle
    func charge() {
        batteryLevel = min(100, batteryLevel + 20)
        print("[\(brand)] Şarj: %\(batteryLevel)")
    }
}

let tesla = ElectricCar(brand: "Tesla")
tesla.accelerate()  // Hız: 10, Batarya: 98
tesla.charge()       // Şarj: %100
```

### final ile Kalıtımı Engelleme

```swift
// final class — bu sınıftan kalıtım yapılamaz
final class SingletonService {
    static let shared = SingletonService()
    private init() {}
}

// final method — belirli bir method override edilemez
class Animal {
    func makeSound() { print("...") }
    final func breathe() { print("Nefes alıyor") }  // Değiştirilemez
}
```

### Designated ve Convenience Initializer'lar

```swift
class User {
    let id: String
    let name: String
    let email: String
    let role: String
    
    // Designated init — tüm property'leri başlatır
    init(id: String, name: String, email: String, role: String) {
        self.id = id
        self.name = name
        self.email = email
        self.role = role
    }
    
    // Convenience init — daha az parametre, varsayılanlarla
    convenience init(name: String, email: String) {
        self.init(id: UUID().uuidString, name: name, email: email, role: "member")
    }
}

let user = User(name: "Zeynep", email: "zeynep@test.com")  // convenience
let admin = User(id: "A1", name: "Ahmet", email: "a@t.com", role: "admin")  // designated
```

---

## 3. Polymorphism (Çok Biçimlilik)

Farklı tiplerin **aynı arayüzü farklı şekillerde uygulamasıdır**.

### Runtime Polymorphism (Override ile)

```swift
class Shape {
    func area() -> Double { return 0 }
    func describe() -> String { return "Alan: \(String(format: "%.2f", area()))" }
}

class Circle: Shape {
    let radius: Double
    init(radius: Double) { self.radius = radius }
    override func area() -> Double { return .pi * radius * radius }
}

class Rectangle: Shape {
    let width, height: Double
    init(width: Double, height: Double) { self.width = width; self.height = height }
    override func area() -> Double { return width * height }
}

// Farklı tipler, aynı arayüz
let shapes: [Shape] = [Circle(radius: 5), Rectangle(width: 4, height: 6)]
for shape in shapes {
    print(shape.describe())  // Her biri kendi area()'sını çalıştırır
}
```

### Type Casting: is, as?, as!

```swift
for shape in shapes {
    if let circle = shape as? Circle {
        print("Daire, yarıçap: \(circle.radius)")
    } else if let rect = shape as? Rectangle {
        print("Dikdörtgen, \(rect.width)x\(rect.height)")
    }
}
```

---

## 4. Encapsulation (Kapsülleme)

İç state'i dış dünyadan gizleyip **kontrollü erişim** sunma.

```swift
class BankAccount {
    // Okunabilir ama dışarıdan değiştirilemez
    private(set) var balance: Double
    let ownerName: String
    
    // Tamamen gizli
    private let dailyLimit: Double = 5000
    private var dailyWithdrawn: Double = 0
    private var history: [(String, Double, Date)] = []
    
    init(ownerName: String, initialBalance: Double) {
        self.ownerName = ownerName
        self.balance = initialBalance
    }
    
    // Kontrollü API — iş kuralları burada uygulanır
    func withdraw(_ amount: Double) -> Result<Double, AccountError> {
        guard amount > 0 else {
            return .failure(.invalidAmount)
        }
        guard amount <= balance else {
            return .failure(.insufficientFunds)
        }
        guard dailyWithdrawn + amount <= dailyLimit else {
            return .failure(.dailyLimitExceeded)
        }
        
        balance -= amount
        dailyWithdrawn += amount
        history.append(("withdrawal", amount, Date()))
        return .success(balance)
    }
    
    func deposit(_ amount: Double) -> Result<Double, AccountError> {
        guard amount > 0 else { return .failure(.invalidAmount) }
        balance += amount
        history.append(("deposit", amount, Date()))
        return .success(balance)
    }
}

enum AccountError: Error {
    case invalidAmount, insufficientFunds, dailyLimitExceeded
}

let account = BankAccount(ownerName: "Ahmet", initialBalance: 10000)
print(account.balance)        // ✅ Okunabilir: 10000
// account.balance = 999999   // ❌ HATA: setter erişilemez
// account.dailyLimit         // ❌ HATA: private
```

---

## 5. Abstraction (Soyutlama)

Karmaşık implementasyon detaylarını gizleyip, sadece **"ne yapılacağını"** tanımlama. Swift'te abstract class yoktur — protocol'ler bu rolü üstlenir.

```swift
// Soyut arayüz — "NE" yapılacağını tanımlar
protocol DataStore {
    func save(key: String, value: String) throws
    func read(key: String) throws -> String?
    func delete(key: String) throws
}

// Somut implementasyon 1
class InMemoryStore: DataStore {
    private var storage: [String: String] = [:]
    func save(key: String, value: String) { storage[key] = value }
    func read(key: String) -> String? { return storage[key] }
    func delete(key: String) { storage.removeValue(forKey: key) }
}

// Somut implementasyon 2
class UserDefaultsStore: DataStore {
    func save(key: String, value: String) { UserDefaults.standard.set(value, forKey: key) }
    func read(key: String) -> String? { UserDefaults.standard.string(forKey: key) }
    func delete(key: String) { UserDefaults.standard.removeObject(forKey: key) }
}

// Kullanan taraf implementasyonu bilmiyor
class SettingsManager {
    private let store: DataStore  // Soyut tipe bağımlı
    
    init(store: DataStore) { self.store = store }
    
    func saveTheme(_ theme: String) throws { try store.save(key: "theme", value: theme) }
    func loadTheme() throws -> String { return try store.read(key: "theme") ?? "light" }
}
```

---

# BÖLÜM B: PROTOCOL-ORIENTED PROGRAMMING (POP)

## 6. POP'a Giriş: Swift'in DNA'sı

Apple, 2015 WWDC'de "Swift is a Protocol-Oriented Programming Language" diyerek POP'u tanıttı. Bu paradigma, OOP'nin kalıtım tabanlı yaklaşımı yerine **protocol composition** ile davranış oluşturmayı savunur.

Neden POP?

| OOP Sorunu | POP Çözümü |
|---|---|
| Tek kalıtım sınırlaması (Swift'te çoklu class kalıtımı yok) | Birden fazla protocol'e conform olabilirsin |
| Class-only kısıtlaması | struct, enum, class — hepsi protocol kullanabilir |
| Sıkı bağımlılık (tight coupling) | Gevşek bağımlılık (loose coupling) |
| "Massive Base Class" problemi | Küçük, odaklı protocol'ler |
| Test zorluğu (subclass gerekir) | Protocol ile mock oluştur |

---

## 7. Protocol Tanımlama ve Conformance

Protocol, bir tipin **ne yapabileceğini** tanımlayan bir sözleşmedir. Nasıl yapacağını belirtmez.

### 7.1 Property Gereksinimleri

```swift
protocol Identifiable {
    var id: String { get }           // Sadece okunabilir olması yeterli
}

protocol Configurable {
    var isEnabled: Bool { get set }  // Hem okunabilir hem yazılabilir olmalı
}

struct Feature: Identifiable, Configurable {
    let id: String                   // let ile tanımlayabilirsin (sadece get yeterli)
    var isEnabled: Bool              // var olmalı (get ve set gerekli)
}
```

### 7.2 Method Gereksinimleri

```swift
protocol Searchable {
    func search(query: String) -> [String]
}

protocol Sortable {
    mutating func sort(ascending: Bool)
    // mutating: struct'ların da conform olabilmesi için
}

protocol Cacheable {
    func cacheKey() -> String
    static func clearCache()  // Statik method gereksinimi
}
```

### 7.3 Initializer Gereksinimleri

```swift
protocol Decodable {
    init(from data: Data) throws
}

// Class'ta protocol init'i → required keyword gerekli
class APIResponse: Decodable {
    let statusCode: Int
    
    required init(from data: Data) throws {
        // Decoding mantığı
        self.statusCode = 200
    }
}
```

---

## 8. Protocol Extensions ve Default Implementations

Swift protocol'lerinin en güçlü özelliği: **extension ile varsayılan davranış** (default implementation) verebilirsin. Bu, protocol'e conform olan tiplerin her method'u tek tek yazma zorunluluğunu kaldırır.

```swift
protocol Loggable {
    var logPrefix: String { get }
    func log(_ message: String)
    func logError(_ error: Error)
    func logWarning(_ message: String)
}

// Varsayılan implementasyonlar — conform olan tipler bunları otomatik alır
extension Loggable {
    var logPrefix: String {
        return "[\(String(describing: type(of: self)))]"
    }
    
    func log(_ message: String) {
        print("\(logPrefix) ℹ️ \(message)")
    }
    
    func logError(_ error: Error) {
        print("\(logPrefix) ❌ ERROR: \(error.localizedDescription)")
    }
    
    func logWarning(_ message: String) {
        print("\(logPrefix) ⚠️ WARNING: \(message)")
    }
}

// Varsayılanları kullanan tip — hiçbir method yazmadan Loggable!
struct OrderService: Loggable {
    func processOrder(id: String) {
        log("Sipariş işleniyor: \(id)")
    }
}

// Varsayılanları override eden tip
struct PaymentService: Loggable {
    var logPrefix: String { return "[💳 PAYMENT]" }  // Özelleştirilmiş prefix
    
    func processPayment(amount: Double) {
        log("Ödeme: \(amount) TL")
    }
}

let orderService = OrderService()
orderService.processOrder(id: "ORD-001")
// [OrderService] ℹ️ Sipariş işleniyor: ORD-001

let paymentService = PaymentService()
paymentService.processPayment(amount: 150)
// [💳 PAYMENT] ℹ️ Ödeme: 150.0 TL
```

### Ek Fonksiyonellik Ekleme (Extension ile)

```swift
protocol Timestamped {
    var createdAt: Date { get }
}

// Extension ile ek fonksiyonellik — conform olan TÜM tipler otomatik alır
extension Timestamped {
    var timeAgo: String {
        let seconds = Date().timeIntervalSince(createdAt)
        if seconds < 60 { return "Az önce" }
        if seconds < 3600 { return "\(Int(seconds / 60)) dakika önce" }
        if seconds < 86400 { return "\(Int(seconds / 3600)) saat önce" }
        return "\(Int(seconds / 86400)) gün önce"
    }
    
    var isToday: Bool {
        return Calendar.current.isDateInToday(createdAt)
    }
}

struct Comment: Timestamped {
    let text: String
    let createdAt: Date
    // timeAgo ve isToday otomatik olarak gelir — yazmana gerek yok
}
```

---

## 9. Composition over Inheritance

POP'un temel felsefesi: Büyük kalıtım hiyerarşileri yerine, küçük ve odaklı protocol'leri **birleştirerek** (compose) davranış oluştur.

### Problem: Kalıtım ile Yapınca

```swift
// ❌ Kalıtım yoluyla — "Massive Base Class" problemi
class BaseModel {
    func save() { }
    func delete() { }
    func validate() -> Bool { return true }
    func log(_ msg: String) { }
    func cache() { }
    func track(_ event: String) { }
    // ... daha fazla sorumluluk ekleniyor, sınıf şişiyor
}

class UserModel: BaseModel {
    // Tüm özellikleri miras alıyor — kullanmasa bile
    // save(), delete(), validate(), log(), cache(), track() hepsi burada
}

class ReadOnlyReport: BaseModel {
    // save() ve delete() yeteneği var ama aslında kullanmaması gerekiyor!
    // Kalıtım gereksiz sorumluluklar dayatıyor
}
```

### Çözüm: Protocol Composition ile

```swift
// ✅ Küçük, odaklı protocol'ler
protocol Persistable {
    func save()
    func delete()
}

protocol Validatable {
    func validate() -> Bool
}

protocol Trackable {
    func track(_ event: String)
}

protocol Cacheable {
    func cache()
}

// Varsayılan davranışlar
extension Persistable {
    func save() { print("💾 Kaydediliyor...") }
    func delete() { print("🗑️ Siliniyor...") }
}

extension Trackable {
    func track(_ event: String) { print("📊 Event: \(event)") }
}

extension Cacheable {
    func cache() { print("📦 Cache'leniyor...") }
}

// ── Sadece ihtiyacın olan protocol'leri seç ──

// Tam donanımlı model
struct UserModel: Persistable, Validatable, Trackable, Cacheable {
    let name: String
    func validate() -> Bool { return !name.isEmpty }
}

// Salt okunur rapor — save/delete yok!
struct ReadOnlyReport: Cacheable {
    let title: String
    // Sadece cache yeteneği — gereksiz sorumluluk taşımıyor
}

// Analytics event — sadece tracking
struct AnalyticsEvent: Trackable {
    let eventName: String
}
```

### Protocol Composition Type (&)

Birden fazla protocol'ü tek bir tür olarak ifade edebilirsin:

```swift
// Bu fonksiyon hem Persistable hem Validatable olan bir tip kabul eder
func saveIfValid(_ item: Persistable & Validatable) {
    if item.validate() {
        item.save()
    } else {
        print("Validasyon başarısız — kaydedilmedi")
    }
}

let user = UserModel(name: "Ahmet")
saveIfValid(user)  // ✅ UserModel hem Persistable hem Validatable

let report = ReadOnlyReport(title: "Q4 Rapor")
// saveIfValid(report)  // ❌ DERLEME HATASI: ReadOnlyReport, Validatable değil
```

---

## 10. Protocol'ler ile Dependency Injection

Dependency Injection (DI), bir nesnenin bağımlılıklarını dışarıdan almasıdır. Protocol'ler ile birleştirildiğinde **testability** (test edilebilirlik) sağlar.

```swift
// ── 1. Protocol ile soyutla ──
protocol UserRepositoryProtocol {
    func fetchUser(id: String) async throws -> User
    func saveUser(_ user: User) async throws
}

struct User: Equatable {
    let id: String
    var name: String
    var email: String
}

// ── 2. Gerçek implementasyon ──
class UserRepository: UserRepositoryProtocol {
    func fetchUser(id: String) async throws -> User {
        // Gerçek API çağrısı
        print("🌐 API'den kullanıcı çekiliyor...")
        return User(id: id, name: "Ahmet", email: "ahmet@test.com")
    }
    
    func saveUser(_ user: User) async throws {
        print("🌐 Kullanıcı kaydediliyor...")
    }
}

// ── 3. Mock implementasyon (testler için) ──
class MockUserRepository: UserRepositoryProtocol {
    var stubbedUser: User?
    var saveCallCount = 0
    
    func fetchUser(id: String) async throws -> User {
        guard let user = stubbedUser else {
            throw NSError(domain: "mock", code: 404)
        }
        return user
    }
    
    func saveUser(_ user: User) async throws {
        saveCallCount += 1
    }
}

// ── 4. ViewModel — Protocol'e bağımlı, somut tipe DEĞİL ──
class UserProfileViewModel {
    private let repository: UserRepositoryProtocol  // ← Protocol tipi
    var user: User?
    var errorMessage: String?
    
    init(repository: UserRepositoryProtocol) {  // ← DI
        self.repository = repository
    }
    
    func loadUser(id: String) async {
        do {
            user = try await repository.fetchUser(id: id)
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}

// Gerçek kullanım:
let realVM = UserProfileViewModel(repository: UserRepository())

// Test kullanımı:
let mockRepo = MockUserRepository()
mockRepo.stubbedUser = User(id: "T1", name: "Test", email: "test@test.com")
let testVM = UserProfileViewModel(repository: mockRepo)
```

---

## 11. Gerçek Hayat: Protocol-Based Service Layer

Birden fazla analytics sağlayıcısını destekleyen, test edilebilir bir analytics sistemi:

```swift
// ═══════════════════════════════════════
//  PROTOCOL — Sözleşme
// ═══════════════════════════════════════

protocol AnalyticsProvider {
    var providerName: String { get }
    func logEvent(_ name: String, parameters: [String: Any])
    func setUserProperty(_ key: String, value: String)
}

// ═══════════════════════════════════════
//  SOMUT İMPLEMENTASYONLAR
// ═══════════════════════════════════════

struct FirebaseAnalytics: AnalyticsProvider {
    let providerName = "Firebase"
    
    func logEvent(_ name: String, parameters: [String: Any]) {
        print("[Firebase] Event: \(name) | Params: \(parameters)")
    }
    
    func setUserProperty(_ key: String, value: String) {
        print("[Firebase] User Property: \(key) = \(value)")
    }
}

struct MixpanelAnalytics: AnalyticsProvider {
    let providerName = "Mixpanel"
    
    func logEvent(_ name: String, parameters: [String: Any]) {
        print("[Mixpanel] Track: \(name) | Props: \(parameters)")
    }
    
    func setUserProperty(_ key: String, value: String) {
        print("[Mixpanel] People Set: \(key) = \(value)")
    }
}

struct ConsoleAnalytics: AnalyticsProvider {
    let providerName = "Console"
    
    func logEvent(_ name: String, parameters: [String: Any]) {
        print("[DEBUG] \(name): \(parameters)")
    }
    
    func setUserProperty(_ key: String, value: String) {
        print("[DEBUG] User: \(key) = \(value)")
    }
}

// ═══════════════════════════════════════
//  BİRLEŞTİRİCİ SERVİS — Tüm sağlayıcılara aynı anda gönder
// ═══════════════════════════════════════

class AnalyticsService {
    private var providers: [AnalyticsProvider]
    
    init(providers: [AnalyticsProvider]) {
        self.providers = providers
    }
    
    func track(_ event: String, parameters: [String: Any] = [:]) {
        providers.forEach { $0.logEvent(event, parameters: parameters) }
    }
    
    func identifyUser(_ userId: String) {
        providers.forEach { $0.setUserProperty("user_id", value: userId) }
    }
    
    func addProvider(_ provider: AnalyticsProvider) {
        providers.append(provider)
    }
}

// ═══════════════════════════════════════
//  KULLANIM
// ═══════════════════════════════════════

// Production — Firebase + Mixpanel
let prodAnalytics = AnalyticsService(providers: [
    FirebaseAnalytics(),
    MixpanelAnalytics()
])
prodAnalytics.track("purchase_completed", parameters: ["amount": 299.99, "currency": "TRY"])

// Debug — sadece console
let debugAnalytics = AnalyticsService(providers: [ConsoleAnalytics()])
debugAnalytics.track("button_tapped", parameters: ["button": "login"])

// Test — boş array ile
let testAnalytics = AnalyticsService(providers: [])
// Hiçbir şey gönderilmez — sessizce çalışır
```

---

# BÖLÜM C: POP vs OOP — MÜLAKAT REHBERİ

## 12. POP vs OOP — Karşılaştırma ve Mülakat Rehberi

### Tam Karşılaştırma Tablosu

| Kriter | OOP (Class Inheritance) | POP (Protocol Composition) |
|---|---|---|
| Temel mekanizma | Sınıf hiyerarşisi (is-a) | Protocol conformance (can-do) |
| Çoklu kalıtım | ❌ Swift'te yok | ✅ Birden fazla protocol |
| Value type desteği | ❌ Sadece class | ✅ struct, enum, class |
| Varsayılan davranış | Üst sınıfın method'u | Protocol extension |
| Bağımlılık derecesi | Sıkı (tight coupling) | Gevşek (loose coupling) |
| Test edilebilirlik | Zor (mock için subclass) | Kolay (mock protocol conform eder) |
| Kod paylaşımı | Vertical (yukarıdan aşağı) | Horizontal (yatay, ihtiyaca göre) |
| Yeni davranış ekleme | Base class'ı değiştirmek gerekebilir | Yeni protocol ekle, mevcut koda dokunma |
| Swift topluluğu | Gerektiğinde | **Varsayılan tercih** |

### Ne Zaman OOP, Ne Zaman POP?

**OOP (Class + Inheritance) tercih et:**
- UIKit ile çalışırken (UIKit zaten OOP tabanlı)
- Shared mutable state gerektiğinde (reference type gerekli)
- Gerçek bir "is-a" ilişkisi olduğunda (ElectricCar IS-A Vehicle)
- Deinit (cleanup) davranışı gerektiğinde

**POP (Protocol + Composition) tercih et:**
- Yeni bir API/service layer tasarlarken
- Dependency injection ve testability istediğinde
- Birden fazla bağımsız yeteneği birleştirmek istediğinde
- Value type'lar (struct/enum) ile çalışırken
- Yeni bir modül veya framework tasarlarken

### Mülakat Cevabı Şablonu

**Soru:** "POP ile OOP arasındaki farkı açıklar mısın? Swift'te hangisini tercih edersin?"

**Örnek Cevap:**

> "Swift'te genellikle POP'u tercih ederim, üç temel nedenden dolayı:
>
> **Birincisi**, POP struct ve enum'larla da çalışır — sadece class'larla sınırlı değildir. Bu, value type'ların avantajlarından (thread safety, predictable behavior) yararlanmamı sağlar.
>
> **İkincisi**, birden fazla protocol'e conform olabilirsiniz — Swift'te çoklu class kalıtımı yoktur ama birden fazla protocol'ü compose edebilirsiniz. Bu, SOLID'deki Interface Segregation prensibine doğrudan hizmet eder.
>
> **Üçüncüsü**, test edilebilirliği büyük ölçüde artırır. Service layer'ımı protocol ile soyutlayıp, testlerde mock implementasyon geçirebilirim.
>
> Ancak OOP'yi tamamen reddetmiyorum. UIKit class hiyerarşisi OOP tabanlıdır ve bu yapıyı kullanmak doğaldır. Ayrıca shared mutable state gerektiğinde class (reference type) kullanmak doğru tercihtir. İkisi birbirini tamamlar — projenin ihtiyacına göre doğru aracı seçmek önemli."

### Pratik Kurallar

```
Protocol-first düşün:
1. Önce "bu nesne NE yapabilmeli?" diye sor → Protocol tanımla
2. Sonra "varsayılan davranışı ne olmalı?" → Protocol extension yaz
3. Son olarak "somut implementasyon ne?" → struct veya class yaz
4. Test yazarken → Mock protocol'ü conform etsin
```

---

## 📝 Bölüm Sonu Notları

Bu dokümanı tamamladıktan sonra şunları yapabilmelisin:

1. OOP'nin 4 prensibini Swift örnekleriyle açıklamak
2. `override`, `super`, `final` kullanımını bilmek
3. Designated vs convenience initializer farkını bilmek
4. Type casting (`is`, `as?`, `as!`) kullanmak
5. Protocol tanımlamak, conform olmak, extension yazmak
6. Default implementation'lar ile kod tekrarını önlemek
7. Protocol composition ile küçük, odaklı protocol'leri birleştirmek
8. Protocol-based dependency injection uygulamak
9. POP vs OOP karşılaştırmasını mülakatta açıklamak

---

> **Sonraki Doküman:** [04 — Closures & Memory](04%20-%20Closures%20ve%20Memory.md)

---

*Phase 1 · Doküman 3/6 · Şubat 2026*
