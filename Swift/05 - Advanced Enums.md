# 📘 05 — Advanced Enums

> **Phase 1 · Doküman 5/6**  
> **Konu:** Raw values, associated values, pattern matching, enum ile state management, CaseIterable, recursive enums  
> **Tahmini Süre:** 2–3 gün  
> **Ön Koşul:** [01](01%20-%20Swift%20Temelleri.md)–[04](04%20-%20Closures%20ve%20Memory.md)

---

## 📑 İçindekiler

1. [Swift Enum'ları Neden Özel?](#1-swift-enumları-neden-özel)
2. [Raw Values](#2-raw-values)
3. [Associated Values](#3-associated-values)
4. [Pattern Matching (switch)](#4-pattern-matching-switch)
5. [Enum'larda Computed Properties ve Methods](#5-enumlarda-computed-properties-ve-methods)
6. [CaseIterable](#6-caseiterable)
7. [Enum ile State Management](#7-enum-ile-state-management)
8. [Recursive Enums (indirect)](#8-recursive-enums-indirect)
9. [Gerçek Hayat: Network Layer Enum Tasarımı](#9-gerçek-hayat-network-layer-enum-tasarımı)

---

## 1. Swift Enum'ları Neden Özel?

Swift'teki enum'lar, çoğu dilin enum'larından çok daha güçlüdür. Sadece bir sabit listesi değil, **tam teşekküllü veri modelleri** olabilirler: associated values taşıyabilir, computed property ve method'ları olabilir, protocol'lere conform olabilirler.

| Özellik | Dart enum | Swift enum |
|---|---|---|
| Sabit değerler | ✅ | ✅ |
| Computed properties | ✅ (Dart 2.17+) | ✅ |
| Methods | ✅ (Dart 2.17+) | ✅ |
| Associated values (case başına farklı veri) | ❌ | ✅ |
| Pattern matching (where, binding) | ❌ | ✅ |
| Recursive enums | ❌ | ✅ |

---

## 2. Raw Values

Her case'e **sabit bir değer** atanmasıdır. Tüm case'ler aynı türde raw value taşır.

```swift
// ── String raw values ──
enum APIEndpoint: String {
    case users = "/api/v1/users"
    case products = "/api/v1/products"
    case orders = "/api/v1/orders"
    case auth = "/api/v1/auth"
}

let endpoint = APIEndpoint.users
print(endpoint.rawValue)  // "/api/v1/users"

// Raw value'dan enum oluşturma — Optional döner (geçersiz değer olabilir)
let parsed = APIEndpoint(rawValue: "/api/v1/products")  // Optional(.products)
let invalid = APIEndpoint(rawValue: "/invalid")           // nil

// ── Int raw values — otomatik artar ──
enum Priority: Int {
    case low = 0
    case medium = 1
    case high = 2
    case critical = 3
}

// ── String raw values — case ismi otomatik değer olur ──
enum Direction: String {
    case north    // rawValue: "north" (otomatik)
    case south    // rawValue: "south"
    case east     // rawValue: "east"
    case west     // rawValue: "west"
}
```

---

## 3. Associated Values

Her case'in **farklı tipte ve sayıda veri** taşıyabilmesi. Bu, Swift enum'larını diğer dillerden ayıran en güçlü özelliktir.

```swift
enum PaymentMethod {
    case cash
    case creditCard(number: String, expiryMonth: Int, expiryYear: Int)
    case bankTransfer(iban: String, bankName: String)
    case wallet(provider: String, balance: Double)
}

// Kullanım
let payment1 = PaymentMethod.cash
let payment2 = PaymentMethod.creditCard(number: "4111111111111111", expiryMonth: 12, expiryYear: 2027)
let payment3 = PaymentMethod.wallet(provider: "Apple Pay", balance: 500)

// Her case farklı veri taşıyor — tek bir enum ile ifade ediliyor
```

### Raw Values vs Associated Values

| | Raw Values | Associated Values |
|---|---|---|
| Değer tipi | Tüm case'ler aynı tür | Her case farklı tür olabilir |
| Değer atama | Derleme zamanında sabit | Runtime'da atanır |
| Kullanım | Sabit eşleşme (API path, status code) | Veri taşıma (ödeme detayları, hata bilgisi) |
| Aynı anda kullanım | ❌ Birlikte kullanılamaz | ❌ Birlikte kullanılamaz |

---

## 4. Pattern Matching (switch)

Swift'te `switch` statement'i, enum'larla birlikte son derece güçlü pattern matching sağlar.

### Temel Pattern Matching

```swift
func describePayment(_ method: PaymentMethod) -> String {
    switch method {
    case .cash:
        return "Nakit ödeme"
        
    case .creditCard(let number, let month, let year):
        let last4 = String(number.suffix(4))
        return "Kredi kartı: ***\(last4), Son kullanma: \(month)/\(year)"
        
    case .bankTransfer(let iban, let bank):
        return "Havale: \(bank) — \(iban)"
        
    case .wallet(let provider, let balance):
        return "\(provider) — Bakiye: \(balance) TL"
    }
    // ⚠️ default gerekmiyor — tüm case'ler karşılandı (exhaustive)
}
```

### where ile Koşullu Pattern Matching

```swift
enum DeliveryStatus {
    case pending
    case shipped(trackingNumber: String, carrier: String)
    case delivered(date: Date)
    case returned(reason: String)
}

func handleDelivery(_ status: DeliveryStatus) {
    switch status {
    case .shipped(let tracking, let carrier) where carrier == "Yurtiçi Kargo":
        print("🚚 Yurtiçi Kargo takip: \(tracking)")
        
    case .shipped(let tracking, let carrier) where carrier == "DHL":
        print("✈️ DHL uluslararası takip: \(tracking)")
        
    case .shipped(let tracking, _):
        print("📦 Kargoda: \(tracking)")
        
    case .returned(let reason) where reason.contains("hasarlı"):
        print("⚠️ Hasarlı ürün iadesi!")
        
    case .returned(let reason):
        print("↩️ İade: \(reason)")
        
    case .delivered(let date):
        print("✅ Teslim: \(date)")
        
    case .pending:
        print("⏳ Hazırlanıyor...")
    }
}
```

### if case let — Tekil Case Kontrolü

Sadece bir case'i kontrol etmek istiyorsan `switch` yerine `if case let` kullan:

```swift
let status = DeliveryStatus.shipped(trackingNumber: "TR123", carrier: "Yurtiçi Kargo")

if case .shipped(let tracking, _) = status {
    print("Takip: \(tracking)")
}

// guard case let — erken çıkış versiyonu
func getTrackingNumber(from status: DeliveryStatus) -> String? {
    guard case .shipped(let tracking, _) = status else {
        return nil
    }
    return tracking
}
```

---

## 5. Enum'larda Computed Properties ve Methods

```swift
enum Membership {
    case free
    case basic(monthlyPrice: Double)
    case premium(monthlyPrice: Double, features: [String])
    case enterprise(customPrice: Double, seatCount: Int)
    
    var displayName: String {
        switch self {
        case .free: return "Ücretsiz"
        case .basic: return "Basic"
        case .premium: return "Premium"
        case .enterprise: return "Enterprise"
        }
    }
    
    var monthlyPrice: Double {
        switch self {
        case .free: return 0
        case .basic(let price): return price
        case .premium(let price, _): return price
        case .enterprise(let price, _): return price
        }
    }
    
    var annualPrice: Double {
        return monthlyPrice * 12 * 0.8  // %20 yıllık indirim
    }
    
    func canAccess(feature: String) -> Bool {
        switch self {
        case .free:
            return false
        case .basic:
            return ["basic_support", "5gb_storage"].contains(feature)
        case .premium(_, let features):
            return features.contains(feature)
        case .enterprise:
            return true  // Tüm özelliklere erişim
        }
    }
}

let myPlan = Membership.premium(monthlyPrice: 29.99, features: ["analytics", "api_access", "priority_support"])
print("\(myPlan.displayName): \(myPlan.monthlyPrice)/ay")
print("Analytics erişimi: \(myPlan.canAccess(feature: "analytics"))")
```

---

## 6. CaseIterable

`CaseIterable` protocol'üne conform olan enum'lar otomatik olarak bir `allCases` koleksiyonu alır. Menüler, picker'lar ve form'larda çok kullanışlıdır.

```swift
enum AppTheme: String, CaseIterable {
    case light = "Açık"
    case dark = "Koyu"
    case system = "Sistem"
    case highContrast = "Yüksek Kontrast"
}

// Tüm case'lere erişim
for theme in AppTheme.allCases {
    print("\(theme.rawValue)")
}

// Picker / segmented control için ideal
let themeOptions = AppTheme.allCases.map { $0.rawValue }
// ["Açık", "Koyu", "Sistem", "Yüksek Kontrast"]

// ⚠️ Associated values olan enum'lar CaseIterable olamaz (otomatik olarak)
// Çünkü sonsuz sayıda olası değer var
```

---

## 7. Enum ile State Management

Enum'lar, uygulama state'ini yönetmek için mükemmeldir. Bir anda sadece **bir state'te** olabilirsin — geçersiz state kombinasyonları imkansız hale gelir.

```swift
enum ViewState<T> {
    case idle
    case loading
    case loaded(T)
    case error(AppError)
    case empty(message: String)
    
    var isLoading: Bool {
        if case .loading = self { return true }
        return false
    }
    
    var data: T? {
        if case .loaded(let data) = self { return data }
        return nil
    }
    
    var errorMessage: String? {
        if case .error(let error) = self { return error.localizedDescription }
        return nil
    }
}

enum AppError: Error, LocalizedError {
    case network, server, unauthorized
    
    var errorDescription: String? {
        switch self {
        case .network: return "İnternet bağlantısı yok"
        case .server: return "Sunucu hatası"
        case .unauthorized: return "Oturumunuz sona erdi"
        }
    }
}

// ── ViewModel'de kullanım ──
class ProductListViewModel {
    var state: ViewState<[String]> = .idle {
        didSet { onStateChanged?(state) }
    }
    
    var onStateChanged: ((ViewState<[String]>) -> Void)?
    
    func loadProducts() {
        state = .loading
        
        // Simülasyon
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) { [weak self] in
            let products = ["MacBook", "iPhone", "iPad"]
            if products.isEmpty {
                self?.state = .empty(message: "Henüz ürün eklenmemiş")
            } else {
                self?.state = .loaded(products)
            }
        }
    }
}

// ── UI tarafında state'e göre render ──
func renderUI(for state: ViewState<[String]>) {
    switch state {
    case .idle:
        print("Henüz veri yüklenmedi")
    case .loading:
        print("Yükleniyor...")
    case .loaded(let products):
        print("Ürünler: \(products)")
    case .error(let error):
        print("Hata: \(error.localizedDescription)")
    case .empty(let message):
        print("Boş: \(message)")
    }
}
```

**Neden Boolean yerine Enum?**

```swift
// ❌ Boolean'lar ile — geçersiz state kombinasyonları mümkün
var isLoading = true
var hasError = true
var data: [String]? = ["item"]
// isLoading=true VE hasError=true VE data != nil → Bu durum ne anlama geliyor?!

// ✅ Enum ile — aynı anda sadece bir state
var state: ViewState<[String]> = .loading
// .loading iken .error veya .loaded olmak İMKANSIZ
```

---

## 8. Recursive Enums (indirect)

Bir enum case'i **kendi tipini** associated value olarak taşıdığında `indirect` keyword'ü kullanılır. Ağaç yapıları ve matematiksel ifadelerde kullanışlıdır.

```swift
// Aritmetik ifade ağacı
indirect enum ArithmeticExpression {
    case number(Double)
    case addition(ArithmeticExpression, ArithmeticExpression)
    case subtraction(ArithmeticExpression, ArithmeticExpression)
    case multiplication(ArithmeticExpression, ArithmeticExpression)
}

func evaluate(_ expression: ArithmeticExpression) -> Double {
    switch expression {
    case .number(let value):
        return value
    case .addition(let left, let right):
        return evaluate(left) + evaluate(right)
    case .subtraction(let left, let right):
        return evaluate(left) - evaluate(right)
    case .multiplication(let left, let right):
        return evaluate(left) * evaluate(right)
    }
}

// (5 + 3) * 2
let expr = ArithmeticExpression.multiplication(
    .addition(.number(5), .number(3)),
    .number(2)
)

print(evaluate(expr))  // 16.0
```

---

## 9. Gerçek Hayat: Network Layer Enum Tasarımı

Profesyonel iOS projelerinde enum'ların kullanıldığı en yaygın yer: network layer.

```swift
// ═══════════════════════════════════════
//  HTTP METHOD
// ═══════════════════════════════════════
enum HTTPMethod: String {
    case get = "GET"
    case post = "POST"
    case put = "PUT"
    case patch = "PATCH"
    case delete = "DELETE"
}

// ═══════════════════════════════════════
//  API ENDPOINT — Her endpoint kendi konfigürasyonunu taşır
// ═══════════════════════════════════════
enum Endpoint {
    case getProducts
    case getProductDetail(id: String)
    case createOrder(productId: String, quantity: Int)
    case login(email: String, password: String)
    case updateProfile(name: String, avatarURL: String?)
    
    var path: String {
        switch self {
        case .getProducts: return "/products"
        case .getProductDetail(let id): return "/products/\(id)"
        case .createOrder: return "/orders"
        case .login: return "/auth/login"
        case .updateProfile: return "/profile"
        }
    }
    
    var method: HTTPMethod {
        switch self {
        case .getProducts, .getProductDetail: return .get
        case .createOrder, .login: return .post
        case .updateProfile: return .put
        }
    }
    
    var requiresAuth: Bool {
        switch self {
        case .login: return false
        default: return true
        }
    }
    
    var body: [String: Any]? {
        switch self {
        case .createOrder(let productId, let quantity):
            return ["product_id": productId, "quantity": quantity]
        case .login(let email, let password):
            return ["email": email, "password": password]
        case .updateProfile(let name, let avatarURL):
            var dict: [String: Any] = ["name": name]
            if let url = avatarURL { dict["avatar_url"] = url }
            return dict
        default:
            return nil
        }
    }
}

// ═══════════════════════════════════════
//  NETWORK ERROR
// ═══════════════════════════════════════
enum NetworkError: Error, LocalizedError {
    case invalidURL
    case noConnection
    case timeout
    case serverError(statusCode: Int, message: String?)
    case decodingFailed(underlyingError: Error)
    case unauthorized
    case unknown(Error)
    
    var errorDescription: String? {
        switch self {
        case .invalidURL: return "Geçersiz URL"
        case .noConnection: return "İnternet bağlantınızı kontrol edin"
        case .timeout: return "İstek zaman aşımına uğradı"
        case .serverError(let code, let msg): return "Sunucu hatası (\(code)): \(msg ?? "Bilinmeyen")"
        case .decodingFailed: return "Veri işleme hatası"
        case .unauthorized: return "Oturumunuz sona erdi, lütfen tekrar giriş yapın"
        case .unknown(let error): return "Beklenmeyen hata: \(error.localizedDescription)"
        }
    }
    
    var isRetryable: Bool {
        switch self {
        case .noConnection, .timeout, .serverError(let code, _) where code >= 500:
            return true
        default:
            return false
        }
    }
}

// ═══════════════════════════════════════
//  KULLANIM
// ═══════════════════════════════════════

func buildRequest(for endpoint: Endpoint, baseURL: String) {
    let url = baseURL + endpoint.path
    let method = endpoint.method.rawValue
    let needsAuth = endpoint.requiresAuth
    
    print("[\(method)] \(url) | Auth: \(needsAuth)")
    
    if let body = endpoint.body {
        print("  Body: \(body)")
    }
}

buildRequest(for: .getProducts, baseURL: "https://api.example.com")
// [GET] https://api.example.com/products | Auth: true

buildRequest(for: .login(email: "test@test.com", password: "123456"), baseURL: "https://api.example.com")
// [POST] https://api.example.com/auth/login | Auth: false
//   Body: ["email": "test@test.com", "password": "123456"]

buildRequest(for: .getProductDetail(id: "P001"), baseURL: "https://api.example.com")
// [GET] https://api.example.com/products/P001 | Auth: true
```

---

## 📝 Bölüm Sonu Notları

Bu dokümanı tamamladıktan sonra şunları yapabilmelisin:

1. Raw values ve associated values farkını bilmek
2. `switch` ile gelişmiş pattern matching yapmak (where, binding)
3. `if case let` ve `guard case let` kullanmak
4. Enum'lara computed property ve method eklemek
5. CaseIterable ile tüm case'lere erişmek
6. Enum ile state management uygulamak (ViewState pattern)
7. Boolean yerine enum kullanmanın avantajlarını açıklamak
8. Network layer'da enum kullanmak (endpoint, error tanımlama)

---

> **Sonraki Doküman:** [06 — Error Handling, Generics & Access Control](06%20-%20Error%20Handling%20-%20Generics%20-%20Access%20Control.md)

---

*Phase 1 · Doküman 5/6 · Şubat 2026*
