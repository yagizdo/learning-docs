# 📘 06 — Error Handling, Generics & Access Control

> **Phase 1 · Doküman 6/6**  
> **Konu:** `do-try-catch`, `Result` type, generic fonksiyonlar/tipler, protocol constraints, erişim seviyeleri  
> **Tahmini Süre:** 3–4 gün  
> **Ön Koşul:** [01](01%20-%20Swift%20Temelleri.md)–[05](05%20Advanced%20Enums.md)

---

## 📑 İçindekiler

### A: Error Handling
1. [Swift'te Hata Yönetimi Felsefesi](#1-swiftte-hata-yönetimi-felsefesi)
2. [Error Protocol ve Özel Hata Tipleri](#2-error-protocol-ve-özel-hata-tipleri)
3. [throws ve Hata Fırlatma](#3-throws-ve-hata-fırlatma)
4. [do-try-catch Mekanizması](#4-do-try-catch-mekanizması)
5. [try? ve try!](#5-try-ve-try)
6. [Result Type](#6-result-type)
7. [Gerçek Hayat: Katmanlı Error Handling](#7-gerçek-hayat-katmanlı-error-handling)

### B: Generics
8. [Generics'e Giriş](#8-genericse-giriş)
9. [Generic Fonksiyonlar](#9-generic-fonksiyonlar)
10. [Generic Tipler](#10-generic-tipler)
11. [Protocol Constraints (where)](#11-protocol-constraints-where)
12. [Associated Types](#12-associated-types)
13. [Gerçek Hayat: Generic Repository Pattern](#13-gerçek-hayat-generic-repository-pattern)

### C: Access Control
14. [5 Erişim Seviyesi](#14-5-erişim-seviyesi)
15. [private(set) ve Setter Kısıtlama](#15-privateset-ve-setter-kısıtlama)
16. [Ne Zaman Hangisi?](#16-ne-zaman-hangisi)
17. [Gerçek Hayat: Modüler Mimari](#17-gerçek-hayat-modüler-mimari)

---

# A: ERROR HANDLING

## 1. Swift'te Hata Yönetimi Felsefesi

Swift, hataları **tür sistemi** üzerinden yönetir. Bir fonksiyon hata fırlatabiliyorsa, bunu imzasında (`throws`) belirtmek **zorundadır** ve çağıran taraf bunu **handle etmek zorundadır** (`try`). Derleyici bunu zorlar — bir hatayı yanlışlıkla göz ardı edemezsin.

---

## 2. Error Protocol ve Özel Hata Tipleri

Swift'te hata fırlatmak için tipinin `Error` protocol'üne conform olması gerekir. Pratikte enum'lar en yaygın tercih olur çünkü hata türlerini güzel şekilde gruplar.

```swift
// ── Basit hata tanımı ──
enum ValidationError: Error {
    case emptyField(fieldName: String)
    case tooShort(fieldName: String, minLength: Int)
    case invalidFormat(fieldName: String, reason: String)
    case outOfRange(fieldName: String, min: Double, max: Double)
}

// ── LocalizedError ile kullanıcı-dostu mesajlar ──
enum PaymentError: Error, LocalizedError {
    case insufficientFunds(required: Double, available: Double)
    case cardExpired
    case cardDeclined(reason: String)
    case networkFailure
    
    var errorDescription: String? {
        switch self {
        case .insufficientFunds(let required, let available):
            return "Yetersiz bakiye. Gerekli: \(required) TL, Mevcut: \(available) TL"
        case .cardExpired:
            return "Kartınızın süresi dolmuş"
        case .cardDeclined(let reason):
            return "Kart reddedildi: \(reason)"
        case .networkFailure:
            return "Bağlantı hatası. Lütfen tekrar deneyin."
        }
    }
}
```

---

## 3. throws ve Hata Fırlatma

```swift
func validateEmail(_ email: String) throws -> String {
    guard !email.isEmpty else {
        throw ValidationError.emptyField(fieldName: "E-posta")
    }
    
    guard email.count >= 5 else {
        throw ValidationError.tooShort(fieldName: "E-posta", minLength: 5)
    }
    
    guard email.contains("@") && email.contains(".") else {
        throw ValidationError.invalidFormat(fieldName: "E-posta", reason: "@ ve . içermelidir")
    }
    
    return email.lowercased().trimmingCharacters(in: .whitespaces)
}

func validateAge(_ age: Int) throws -> Int {
    guard age >= 18 && age <= 120 else {
        throw ValidationError.outOfRange(fieldName: "Yaş", min: 18, max: 120)
    }
    return age
}

// ── throws propagation — hata otomatik yayılır ──
func registerUser(email: String, age: Int) throws -> String {
    let validEmail = try validateEmail(email)  // Hata varsa otomatik fırlatılır
    let validAge = try validateAge(age)         // Hata varsa otomatik fırlatılır
    
    return "Kullanıcı kaydedildi: \(validEmail), yaş: \(validAge)"
}
```

---

## 4. do-try-catch Mekanizması

```swift
// ── Spesifik hata yakalama ──
do {
    let result = try registerUser(email: "test@test.com", age: 25)
    print("✅ \(result)")
} catch ValidationError.emptyField(let field) {
    print("❌ \(field) boş olamaz")
} catch ValidationError.tooShort(let field, let min) {
    print("❌ \(field) en az \(min) karakter olmalı")
} catch ValidationError.invalidFormat(let field, let reason) {
    print("❌ \(field) geçersiz: \(reason)")
} catch ValidationError.outOfRange(let field, let min, let max) {
    print("❌ \(field) \(Int(min))-\(Int(max)) arasında olmalı")
} catch {
    // Genel catch — tüm diğer hatalar
    print("❌ Beklenmeyen hata: \(error.localizedDescription)")
}
```

---

## 5. try? ve try!

```swift
// ── try? — Hata olursa nil döner, hata detayı kaybolur ──
let result1 = try? validateEmail("test@test.com")  // Optional("test@test.com")
let result2 = try? validateEmail("")                 // nil (hata detayı yok)

// Nil-coalescing ile birlikte
let email = (try? validateEmail(userInput)) ?? "default@example.com"

// ── try! — Hata olursa CRASH! Production'da KULLANMA ──
let guaranteed = try! validateEmail("valid@email.com")  // Risk alıyorsun
// try! validateEmail("")  // 💥 CRASH
```

---

## 6. Result Type

`Result<Success, Failure>` bir operasyonun başarı veya başarısızlığını **tek bir değer** olarak ifade eder. Özellikle completion handler'larda kullanışlıdır.

```swift
func fetchUser(id: String, completion: @escaping (Result<User, NetworkError>) -> Void) {
    guard !id.isEmpty else {
        completion(.failure(.invalidURL))
        return
    }
    
    // Simülasyon
    DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
        let user = User(id: id, name: "Ahmet", email: "ahmet@test.com")
        completion(.success(user))
    }
}

struct User { let id, name, email: String }
enum NetworkError: Error { case invalidURL, serverError, decodingFailed }

// Kullanım
fetchUser(id: "U001") { result in
    switch result {
    case .success(let user):
        print("✅ \(user.name)")
    case .failure(let error):
        print("❌ \(error)")
    }
}

// ── Result'ın map ve flatMap'i ──
let userResult: Result<User, NetworkError> = .success(User(id: "1", name: "Test", email: "t@t.com"))

let nameResult = userResult.map { $0.name }      // Result<String, NetworkError>.success("Test")
let emailResult = userResult.map { $0.email }     // Result<String, NetworkError>.success("t@t.com")

// Result → throws dönüşümü
do {
    let user = try userResult.get()  // Başarılıysa değer, başarısızsa throw
    print(user.name)
} catch {
    print(error)
}
```

---

## 7. Gerçek Hayat: Katmanlı Error Handling

```swift
// ── Katmanlı hata hiyerarşisi ──
enum AppError: Error, LocalizedError {
    case api(APIError)
    case validation(ValidationError)
    case storage(String)
    
    var errorDescription: String? {
        switch self {
        case .api(let e): return "Ağ Hatası: \(e.localizedDescription)"
        case .validation(let e): return "Doğrulama: \(e.localizedDescription)"
        case .storage(let msg): return "Depolama: \(msg)"
        }
    }
    
    var isRetryable: Bool {
        if case .api(let apiError) = self { return apiError.isRetryable }
        return false
    }
}

enum APIError: Error, LocalizedError {
    case noConnection, timeout, serverError(Int), unauthorized
    
    var isRetryable: Bool {
        switch self {
        case .noConnection, .timeout: return true
        case .serverError(let code): return code >= 500
        case .unauthorized: return false
        }
    }
    
    var errorDescription: String? {
        switch self {
        case .noConnection: return "İnternet bağlantısı yok"
        case .timeout: return "Zaman aşımı"
        case .serverError(let code): return "Sunucu hatası (\(code))"
        case .unauthorized: return "Oturum süresi doldu"
        }
    }
}

// ── Hata işleyici ──
func handleError(_ error: AppError) {
    print("❌ \(error.localizedDescription)")
    
    if error.isRetryable {
        print("🔄 Tekrar denenebilir")
    }
    
    if case .api(.unauthorized) = error {
        print("🔐 Login ekranına yönlendiriliyor...")
    }
}
```

---

# B: GENERICS

## 8. Generics'e Giriş

Generic'ler, **farklı tipler için aynı mantığı tekrar tekrar yazmadan** kullanmanı sağlar. Swift standard library'nin büyük çoğunluğu generic'ler üzerine kuruludur — `Array<Element>`, `Dictionary<Key, Value>`, `Optional<Wrapped>`, `Result<Success, Failure>` hepsi generic tiplerdir.

---

## 9. Generic Fonksiyonlar

```swift
// ── Generic olmadan — her tip için ayrı fonksiyon ──
func printIntArray(_ array: [Int]) { array.forEach { print($0) } }
func printStringArray(_ array: [String]) { array.forEach { print($0) } }
// ... her tip için tekrar? 😩

// ── Generic ile — tek fonksiyon, tüm tipler ──
func printArray<T>(_ array: [T]) {
    array.forEach { print($0) }
}

printArray([1, 2, 3])            // Int
printArray(["a", "b", "c"])      // String
printArray([true, false, true])  // Bool

// ── Birden fazla generic parametre ──
func makePair<A, B>(_ first: A, _ second: B) -> (A, B) {
    return (first, second)
}

let pair = makePair("Ahmet", 28)  // (String, Int)
```

---

## 10. Generic Tipler

```swift
// ── Generic Stack (Yığın) veri yapısı ──
struct Stack<Element> {
    private var items: [Element] = []
    
    var isEmpty: Bool { items.isEmpty }
    var count: Int { items.count }
    var peek: Element? { items.last }
    
    mutating func push(_ item: Element) {
        items.append(item)
    }
    
    @discardableResult
    mutating func pop() -> Element? {
        return items.isEmpty ? nil : items.removeLast()
    }
}

var intStack = Stack<Int>()
intStack.push(10)
intStack.push(20)
print(intStack.pop()!)  // 20

var stringStack = Stack<String>()
stringStack.push("Merhaba")
```

---

## 11. Protocol Constraints (where)

Generic parametrelere **kısıtlama** ekleyerek, sadece belirli protocol'lere uyan tipleri kabul edebilirsin.

```swift
// ── Temel constraint ──
func findIndex<T: Equatable>(of target: T, in array: [T]) -> Int? {
    for (index, item) in array.enumerated() {
        if item == target { return index }
    }
    return nil
}

findIndex(of: "Swift", in: ["Dart", "Swift", "Kotlin"])  // Optional(1)

// ── Birden fazla constraint ──
func sortAndDisplay<T: Comparable & CustomStringConvertible>(_ items: [T]) {
    let sorted = items.sorted()
    sorted.forEach { print($0.description) }
}

// ── where clause ile karmaşık constraint'ler ──
func areAllEqual<T: Equatable>(_ array: [T]) -> Bool where T: Hashable {
    return Set(array).count <= 1
}

areAllEqual([5, 5, 5])     // true
areAllEqual([1, 2, 3])     // false

// ── Extension'larda where ──
extension Stack where Element: Equatable {
    func contains(_ item: Element) -> Bool {
        return items.contains(item)
    }
}

extension Stack where Element: Numeric {
    func sum() -> Element {
        return items.reduce(0, +)
    }
}

var numStack = Stack<Int>()
numStack.push(10)
numStack.push(20)
print(numStack.sum())        // 30
print(numStack.contains(10)) // true
```

---

## 12. Associated Types

Protocol'lerde generic davranış tanımlamak için `associatedtype` kullanılır.

```swift
protocol Container {
    associatedtype Item  // Conform eden tip, Item'ın ne olduğunu belirler
    
    var count: Int { get }
    mutating func add(_ item: Item)
    func item(at index: Int) -> Item?
}

struct StringBag: Container {
    typealias Item = String  // Item = String olarak belirlendi
    
    private var items: [String] = []
    var count: Int { items.count }
    
    mutating func add(_ item: String) { items.append(item) }
    func item(at index: Int) -> String? {
        guard items.indices.contains(index) else { return nil }
        return items[index]
    }
}

// associatedtype constraint ile
protocol Repository {
    associatedtype Entity: Identifiable  // Entity, Identifiable olmalı
    
    func getAll() async throws -> [Entity]
    func getById(_ id: Entity.ID) async throws -> Entity?
    func save(_ entity: Entity) async throws
}
```

---

## 13. Gerçek Hayat: Generic Repository Pattern

```swift
protocol Storable: Identifiable, Codable { }

class InMemoryRepository<T: Storable>: Repository where T.ID == String {
    typealias Entity = T
    
    private var storage: [String: T] = [:]
    
    func getAll() async throws -> [T] {
        return Array(storage.values)
    }
    
    func getById(_ id: String) async throws -> T? {
        return storage[id]
    }
    
    func save(_ entity: T) async throws {
        storage[entity.id] = entity
    }
    
    func delete(_ id: String) async throws {
        storage.removeValue(forKey: id)
    }
}

// ── Modeller ──
struct TodoItem: Storable {
    let id: String
    var title: String
    var isCompleted: Bool
}

struct Note: Storable {
    let id: String
    var content: String
    var createdAt: Date
}

// ── Aynı repository, farklı tipler ──
let todoRepo = InMemoryRepository<TodoItem>()
let noteRepo = InMemoryRepository<Note>()

// TodoItem'a özel repository yazmana gerek yok!
```

---

# C: ACCESS CONTROL

## 14. 5 Erişim Seviyesi

Swift'te en açıktan en kısıtlıya doğru 5 erişim seviyesi vardır:

```
open > public > internal > fileprivate > private
```

```swift
// ── open ── Modül dışından erişilebilir VE override edilebilir
open class BaseTheme {
    open func primaryColor() -> String { "Blue" }  // Override edilebilir
}

// ── public ── Modül dışından erişilebilir AMA override edilemez
public struct APIConfig {
    public let baseURL: String
    public init(baseURL: String) { self.baseURL = baseURL }
}

// ── internal (VARSAYILAN) ── Aynı modül içinde her yerden erişilebilir
class UserManager {           // internal (belirtilmedi = internal)
    func fetchUsers() { }     // internal
}

// ── fileprivate ── Sadece aynı dosya içinden erişilebilir
fileprivate struct CacheEntry {
    let key: String
    let value: Any
}

// ── private ── Sadece tanımlandığı scope (kapsam) içinden erişilebilir
class OrderService {
    private var pendingOrders: [String] = []  // Dışarıdan ERİŞİLEMEZ
    private let maxRetries = 3
    
    func submitOrder(_ orderId: String) {
        pendingOrders.append(orderId)  // Sınıf içinde erişilebilir
    }
}
```

---

## 15. private(set) ve Setter Kısıtlama

Bir property'nin okunmasına izin verip yazılmasını kısıtlama — encapsulation'ın en yaygın aracı.

```swift
class ScoreTracker {
    // Dışarıdan okunabilir, sadece bu sınıf değiştirebilir
    private(set) var highScore: Int = 0
    private(set) var currentScore: Int = 0
    private(set) var gamesPlayed: Int = 0
    
    func addPoints(_ points: Int) {
        currentScore += points
        if currentScore > highScore {
            highScore = currentScore
        }
    }
    
    func endGame() {
        gamesPlayed += 1
        currentScore = 0
    }
}

let tracker = ScoreTracker()
tracker.addPoints(50)
print(tracker.currentScore)   // ✅ 50 — okuma izni var
// tracker.currentScore = 999  // ❌ HATA — yazma izni yok
```

---

## 16. Ne Zaman Hangisi?

| Seviye | Kullanım | Ne Zaman |
|---|---|---|
| `open` | Framework: override edilecek class/method | Framework yazıyorsan |
| `public` | Framework: dışarıya açık ama değiştirilemeyen API | Framework API'si |
| `internal` | Uygulama içi — varsayılan | Çoğu kod |
| `fileprivate` | Aynı dosyadaki extension'lar arası paylaşım | Nadir |
| `private` | İç implementasyon detayları | **Mümkün olduğunca çok kullan** |

**Altın Kural:** En kısıtlı seviyeyle başla (`private`), gerektiğinde gevşet. Bu, encapsulation'ın temelidir ve kodun bakımını kolaylaştırır.

---

## 17. Gerçek Hayat: Modüler Mimari

```swift
// ═══════════════════════════════════════
//  "NetworkKit" MODÜLÜ — Dışarıya açık API
// ═══════════════════════════════════════

// PUBLIC — kullanıcılar görecek
public protocol NetworkServiceProtocol {
    func fetch<T: Decodable>(endpoint: String) async throws -> T
}

public enum NetworkError: Error {
    case invalidURL, serverError(Int), decodingError
}

// PUBLIC class — oluşturulabilir, kullanılabilir
public class NetworkService: NetworkServiceProtocol {
    private let baseURL: String         // PRIVATE — dış dünya bilmemeli
    private let session: URLSession     // PRIVATE
    private let decoder: JSONDecoder    // PRIVATE
    
    public init(baseURL: String) {
        self.baseURL = baseURL
        self.session = .shared
        self.decoder = JSONDecoder()
        self.decoder.keyDecodingStrategy = .convertFromSnakeCase
    }
    
    // PUBLIC — protocol'den gelen
    public func fetch<T: Decodable>(endpoint: String) async throws -> T {
        let url = try buildURL(for: endpoint)
        let data = try await performRequest(url)
        return try decode(data)
    }
    
    // PRIVATE — iç implementasyon
    private func buildURL(for endpoint: String) throws -> URL {
        guard let url = URL(string: "\(baseURL)/\(endpoint)") else {
            throw NetworkError.invalidURL
        }
        return url
    }
    
    private func performRequest(_ url: URL) async throws -> Data {
        let (data, _) = try await session.data(from: url)
        return data
    }
    
    private func decode<T: Decodable>(_ data: Data) throws -> T {
        do { return try decoder.decode(T.self, from: data) }
        catch { throw NetworkError.decodingError }
    }
}

// ═══════════════════════════════════════
//  KULLANICI PERSPEKTİFİ (başka modülden)
// ═══════════════════════════════════════

// Görünenler: NetworkServiceProtocol, NetworkService (public init + public methods), NetworkError
// Görünmeyenler: buildURL, performRequest, decode, baseURL, session, decoder
```

---

## 📝 Phase 1 Genel Özet

6 dokümanlık bu seriyi tamamlayarak Swift dilinin temellerini kapsamlı şekilde öğrendin. İşte öğrendiğin konuların özeti:

| Doküman | Ana Konular | Mülakat Değeri |
|---|---|---|
| 01 - Swift Temelleri | Syntax, veri tipleri, struct vs class | ⭐⭐ |
| 02 - Optionals | if let, guard let, optional chaining | ⭐⭐⭐ |
| 03 - OOP & POP | 4 OOP prensibi, protocol composition, DI | ⭐⭐⭐⭐⭐ |
| 04 - Closures & Memory | @escaping, [weak self], retain cycles | ⭐⭐⭐⭐⭐ |
| 05 - Advanced Enums | Associated values, state management | ⭐⭐⭐ |
| 06 - Error, Generics, Access | Result type, generic repository, modüler mimari | ⭐⭐⭐ |

**Bir sonraki adımın:** [Phase 1 Index](./00_Phase1_Index.md)'teki checklist'i kontrol et, pratik ödevleri yap ve Phase 2'ye geç.

---

> **Sonraki Phase:** [Phase 2 — UIKit Temelleri: Programmatic UI](../phase2/00_Phase2_Index.md)

---

*Phase 1 · Doküman 6/6 · Şubat 2026*
