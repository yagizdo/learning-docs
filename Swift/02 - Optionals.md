# 📘 02 — Optionals

> **Phase 1 · Doküman 2/6**  
> **Konu:** Optional kavramı, unwrapping yöntemleri, optional chaining, pattern matching  
> **Tahmini Süre:** 2–3 gün  
> **Ön Koşul:** [01 — Swift Temelleri](01%20-%20Swift%20Temelleri.md)

---

## 📑 İçindekiler

1. [Optional Nedir ve Neden Var?](#1-optional-nedir-ve-neden-var)
2. [Optional Tanımlama ve Temel Kullanım](#2-optional-tanımlama-ve-temel-kullanım)
3. [if let — Optional Binding](#3-if-let--optional-binding)
4. [guard let — Early Exit](#4-guard-let--early-exit)
5. [if let vs guard let — Karar Rehberi](#5-if-let-vs-guard-let--karar-rehberi)
6. [Nil-Coalescing Operator (??)](#6-nil-coalescing-operator-)
7. [Optional Chaining](#7-optional-chaining)
8. [Force Unwrapping (!) — ve Neden Tehlikeli](#8-force-unwrapping----ve-neden-tehlikeli)
9. [Implicitly Unwrapped Optionals (IUO)](#9-implicitly-unwrapped-optionals-iuo)
10. [Optional'larda Pattern Matching](#10-optionallarda-pattern-matching)
11. [map ve flatMap ile Optional Dönüşümleri](#11-map-ve-flatmap-ile-optional-dönüşümleri)
12. [Gerçek Hayat: Katmanlı Optional Yönetimi](#12-gerçek-hayat-katmanlı-optional-yönetimi)

---

## 1. Optional Nedir ve Neden Var?

Optional, Swift'in en temel güvenlik mekanizmasıdır. Bir değerin **var olabileceğini VEYA hiç olmayabileceğini (nil)** ifade eden bir sarmalayıcıdır (wrapper).

### Temel Kavram

Bunu bir kargo kutusu olarak düşün:
- Kutunun içinde bir ürün olabilir → `Optional("MacBook")`
- Kutu boş olabilir → `nil`

Önemli olan şu: **kutuyu açmadan (unwrap etmeden) içindekilere ulaşamazsın.** Swift derleyicisi bunu zorlar.

### Neden Var?

Programlama dünyasının en yaygın hatalarından biri **null reference** (null pointer exception) hatalarıdır. Tony Hoare — null referansı icat eden kişi — bunu "billion dollar mistake" (milyar dolarlık hata) olarak tanımlar.

Swift bu sorunu dilin tasarımına entegre ederek çözer: Bir değer nil olabiliyorsa, **derleyici seni bunu açıkça belirtmeye ve güvenli şekilde kullanmaya zorlar.**

```swift
// Non-optional — ASLA nil olamaz
var appName: String = "MyApp"
// appName = nil  // ❌ DERLEME HATASI — String tipi nil kabul etmez

// Optional — nil olabilir
var currentUser: String? = "Ahmet"
currentUser = nil  // ✅ Sorun yok — String? tipi nil kabul eder
```

---

## 2. Optional Tanımlama ve Temel Kullanım

```swift
// ── Optional tanımlama yöntemleri ──
var name: String? = "Zeynep"     // Değer var
var middleName: String? = nil     // Değer yok
var age: Int? = nil               // Henüz belirlenmemiş

// Optional tanımlandığında varsayılan değer nil'dir
var nickname: String?  // Otomatik olarak nil

// ── Optional bir değeri doğrudan kullanamazsın ──
let greeting = "Merhaba, " + name  // ❌ DERLEME HATASI!
// Error: Value of optional type 'String?' must be unwrapped to a value of type 'String'

// Optional kullanmak için UNWRAP etmelisin (sonraki bölümlerde)
```

### Optional Perde Arkası

Optional aslında bir **enum**'dur. Swift standart kütüphanesindeki tanımı şöyledir:

```swift
// Swift'in Optional tanımı (basitleştirilmiş)
enum Optional<Wrapped> {
    case none          // nil
    case some(Wrapped)  // değer var
}

// Yani String? aslında Optional<String> demek
let explicitOptional: Optional<String> = .some("Ahmet")
let shorthandOptional: String? = "Ahmet"
// İkisi tamamen aynı şey — kısa yazım (syntactic sugar)
```

Bu bilgi neden önemli? Çünkü Optional'ın bir enum olduğunu bilmek, `switch` ile pattern matching yapabileceğini ve `map`/`flatMap` gibi fonksiyonel dönüşümler uygulayabileceğini anlamana yardımcı olur (bkz. Bölüm 10 ve 11).

---

## 3. if let — Optional Binding

`if let`, Optional'ın içindeki değere güvenli bir şekilde ulaşmanın en yaygın yoludur. Değer varsa unwrap eder ve blok içinde kullanılabilir hale getirir; yoksa `else` bloğuna düşer.

```swift
// ── Temel kullanım ──
func displayProfile(name: String?, email: String?) {
    if let name = name {
        // "name" artık String (Optional DEĞİL) — güvenle kullanabilirsin
        print("İsim: \(name)")
    } else {
        print("İsim bilgisi yok")
    }
    
    if let email = email {
        print("E-posta: \(email)")
    }
}

displayProfile(name: "Ahmet", email: nil)
// İsim: Ahmet
// (email satırı basılmadı — nil olduğu için if bloğuna girmedi)
```

### Swift 5.7+ Kısa Syntax (if let shorthand)

Swift 5.7'den itibaren, aynı isimle unwrap yaparken kısa yazım kullanabilirsin:

```swift
func processUser(name: String?, age: Int?) {
    // Eski syntax
    if let name = name {
        print(name)
    }
    
    // Yeni syntax (Swift 5.7+) — aynı şey, daha temiz
    if let name {
        print(name)
    }
    
    if let age {
        print("Yaş: \(age)")
    }
}
```

### Birden Fazla Optional'ı Aynı Anda Unwrap Etme

```swift
func createOrder(productId: String?, quantity: Int?, userId: String?) {
    // Virgülle ayırarak birden fazla optional'ı aynı anda kontrol et
    if let productId, let quantity, let userId {
        print("Sipariş: \(productId) x\(quantity), Kullanıcı: \(userId)")
    } else {
        print("Eksik bilgi — sipariş oluşturulamadı")
    }
}

createOrder(productId: "P001", quantity: 3, userId: "U123")
// "Sipariş: P001 x3, Kullanıcı: U123"

createOrder(productId: "P001", quantity: nil, userId: "U123")
// "Eksik bilgi — sipariş oluşturulamadı"
```

### if let ile Ek Koşul (where yerine virgül)

```swift
func processPayment(amount: Double?, cardNumber: String?) {
    if let amount, amount > 0, let cardNumber, cardNumber.count == 16 {
        print("✅ Ödeme işleniyor: \(amount) TL, Kart: ***\(cardNumber.suffix(4))")
    } else {
        print("❌ Geçersiz ödeme bilgileri")
    }
}
```

---

## 4. guard let — Early Exit

`guard let`, fonksiyonun başında ön koşulları kontrol edip, sağlanmıyorsa **erken çıkış** yapma mekanizmasıdır. Koşul sağlanmazsa `return`, `throw`, `break` veya `continue` ile bloktan çıkmak **zorunludur**.

En büyük avantajı: unwrap edilen değer, `guard` bloğundan sonra **fonksiyonun geri kalanında** kullanılabilir hale gelir.

```swift
func processRegistration(
    name: String?,
    email: String?,
    password: String?,
    age: Int?
) -> String {
    // Her guard, bir "kapı" gibi düşün
    // Koşul sağlanmazsa fonksiyondan çıkılır
    
    guard let name, !name.isEmpty else {
        return "❌ İsim boş olamaz"
    }
    
    guard let email, email.contains("@") else {
        return "❌ Geçersiz e-posta"
    }
    
    guard let password, password.count >= 8 else {
        return "❌ Şifre en az 8 karakter olmalı"
    }
    
    guard let age, age >= 18 else {
        return "❌ 18 yaşından küçükler kayıt olamaz"
    }
    
    // ✅ Buraya ulaştıysan TÜM değerler güvenli
    // name, email, password, age — hepsi non-optional
    return "✅ Kayıt başarılı: \(name), \(email), yaş: \(age)"
}

print(processRegistration(name: "Ahmet", email: "ahmet@test.com", password: "12345678", age: 25))
// "✅ Kayıt başarılı: Ahmet, ahmet@test.com, yaş: 25"

print(processRegistration(name: nil, email: "test@test.com", password: "12345678", age: 25))
// "❌ İsim boş olamaz"
```

### guard let'in Scope Avantajı

`guard let` ile unwrap edilen değişken, fonksiyonun **geri kalan tamamında** geçerlidir. `if let` ile unwrap edilen değişken ise sadece `if` bloğunun **içinde** geçerlidir.

```swift
func processWithGuard(value: String?) {
    guard let value else { return }
    
    // "value" artık tüm fonksiyon boyunca non-optional
    print(value)
    doSomethingWith(value)
    doAnotherThingWith(value)
}

func processWithIfLet(value: String?) {
    if let value {
        // "value" sadece bu blok içinde non-optional
        print(value)
    }
    // Burada "value" tekrar Optional — kullanamıyorsun (unwrap edilmemiş)
}
```

---

## 5. if let vs guard let — Karar Rehberi

Bu iki yapıyı ne zaman kullanacağını bilmek, profesyonel Swift kodu yazmanın temel taşıdır.

| Durum | Tercih | Neden |
|---|---|---|
| Fonksiyon başında validasyon | `guard let` | Erken çıkış, "happy path" sol marjinde kalır |
| Birden fazla optional'ı sırayla kontrol | `guard let` | Her biri ayrı hata mesajı döndürebilir |
| İki yollu mantık (varsa X yap, yoksa Y yap) | `if let` | Her iki yolda da anlamlı iş yapılıyorsa |
| Kısa, basit kontrol | `if let` | Gereksiz guard ceremonisi yok |
| Optional'a fonksiyon boyunca erişim gerekli | `guard let` | Scope avantajı |
| Döngü içinde atlama | `guard` + `continue` | Geçersiz elemanları atla |

### Gerçek Hayat Karşılaştırması

```swift
// ── guard let ile (önerilen stil) ──
func fetchUserProfile(userId: String?) -> UserProfile? {
    guard let userId, !userId.isEmpty else {
        print("Geçersiz kullanıcı ID")
        return nil
    }
    
    guard let cachedProfile = cache.get(userId) else {
        print("Cache'te bulunamadı, API'ye gidiliyor")
        return fetchFromAPI(userId)
    }
    
    return cachedProfile
}

// ── if let ile aynı fonksiyon (daha az okunabilir) ──
func fetchUserProfileNested(userId: String?) -> UserProfile? {
    if let userId, !userId.isEmpty {
        if let cachedProfile = cache.get(userId) {
            return cachedProfile
        } else {
            print("Cache'te bulunamadı, API'ye gidiliyor")
            return fetchFromAPI(userId)
        }
    } else {
        print("Geçersiz kullanıcı ID")
        return nil
    }
}
// İç içe geçmiş if-else yapısı ("pyramid of doom") okunabilirliği düşürür
```

**Geliştirici Bakış Açısı:** Profesyonel iOS kodlarında `guard let`, fonksiyon girişlerinde standart pratiktir. Code review'da "Bu neden `guard` yerine `if let` ile yazılmış?" sorusuyla karşılaşabilirsin. Genel kural: **koşul sağlanmadığında fonksiyondan çıkacaksan → guard, her iki yolda da iş yapacaksan → if let.**

---

## 6. Nil-Coalescing Operator (??)

Bir Optional nil ise **varsayılan bir değer** sağlar. Basit, güçlü ve günlük iOS geliştirmede çok sık kullanılır.

```swift
// Temel kullanım
let username: String? = nil
let displayName = username ?? "Anonim Kullanıcı"
// displayName = "Anonim Kullanıcı" (String — Optional DEĞİL)

// Zincirleme kullanım — soldan sağa kontrol edilir
let primaryPhone: String? = nil
let secondaryPhone: String? = nil
let officePhone: String? = "+90 212 555 1234"
let emergencyPhone = "112"

let contactNumber = primaryPhone ?? secondaryPhone ?? officePhone ?? emergencyPhone
// "+90 212 555 1234" (ilk nil olmayan değer)
```

### Gerçek Hayat Kullanımları

```swift
// Kullanıcı ayarlarında varsayılan değerler
struct AppSettings {
    static func fontSize() -> CGFloat {
        return UserDefaults.standard.object(forKey: "fontSize") as? CGFloat ?? 16.0
    }
    
    static func themeName() -> String {
        return UserDefaults.standard.string(forKey: "theme") ?? "light"
    }
    
    static func maxCacheSize() -> Int {
        return UserDefaults.standard.object(forKey: "maxCache") as? Int ?? 50
    }
}

// API yanıtlarında eksik alanlar
struct UserResponse: Decodable {
    let name: String?
    let avatarURL: String?
    let bio: String?
}

func displayUser(_ response: UserResponse) {
    let name = response.name ?? "İsimsiz Kullanıcı"
    let avatar = response.avatarURL ?? "default_avatar.png"
    let bio = response.bio ?? "Henüz bir biyografi eklenmemiş."
    
    print("\(name) — \(bio)")
}
```

---

## 7. Optional Chaining

Bir Optional'ın property'sine, method'una veya subscript'ine güvenli şekilde ulaşma mekanizmasıdır. Zincirdeki herhangi bir nokta nil ise, **tüm ifade nil döner** ve crash olmaz.

```swift
struct Address {
    let city: String
    let district: String?
    let postalCode: String?
}

struct Company {
    let name: String
    let address: Address?
}

struct Employee {
    let fullName: String
    let company: Company?
}

let employee: Employee? = Employee(
    fullName: "Zeynep Yılmaz",
    company: Company(
        name: "Tech Corp",
        address: Address(city: "İstanbul", district: "Kadıköy", postalCode: nil)
    )
)

// Optional chaining — her ? bir "güvenlik kapısı"
let city = employee?.company?.address?.city
// Optional("İstanbul") — tüm zincir başarılı

let postalCode = employee?.company?.address?.postalCode
// nil — postalCode nil olduğu için

let uppercasedCity = employee?.company?.address?.city.uppercased()
// Optional("İSTANBUL") — city non-optional olduğu için .uppercased() güvenle çağrılabilir

// Method çağrısı ile
let cityLength = employee?.company?.address?.city.count
// Optional(8) — "İstanbul".count = 8
```

### Optional Chaining ile Nil-Coalescing Kombinasyonu

```swift
// Pratik ve temiz syntax
let displayCity = employee?.company?.address?.city ?? "Bilinmeyen Şehir"
// "İstanbul" (String — Optional DEĞİL)

let displayPostal = employee?.company?.address?.postalCode ?? "Kod yok"
// "Kod yok"
```

### Optional Chaining ile Atama

```swift
struct Settings {
    var theme: String?
}

class AppConfig {
    var settings: Settings?
}

var config: AppConfig? = AppConfig()
config?.settings = Settings(theme: "dark")

// Optional chaining ile atama — config nil ise hiçbir şey olmaz
config?.settings?.theme = "light"

// config nil yapılırsa
config = nil
config?.settings?.theme = "dark"  // Hiçbir şey olmaz, crash da olmaz
```

---

## 8. Force Unwrapping (!) — ve Neden Tehlikeli

Force unwrapping, Optional'ın içindeki değere **sormadan, kontrolsüz** erişmektir. Değer nil ise uygulama **anında çöker** (fatal error — runtime crash).

```swift
let validOptional: String? = "Merhaba"
let value = validOptional!  // ✅ Bu sefer çalışır — değer var

let emptyOptional: String? = nil
// let crash = emptyOptional!
// 💥 Fatal error: Unexpectedly found nil while unwrapping an Optional value
```

### Ne Zaman Kabul Edilebilir?

Force unwrapping'in kabul edildiği **çok nadir** durumlar:

```swift
// 1. IBOutlet'ler — Interface Builder bağlantıları
// Storyboard/XIB bağlantısı doğru yapıldıysa sistem değeri garanti eder
// @IBOutlet weak var titleLabel: UILabel!

// 2. Hemen sonra atanan değerler
let image = UIImage(named: "app_icon")!
// Uygulamanın kendi asset'i — asset catalog'da olduğundan eminsin
// Yine de "UIImage(named:)" nil dönebilir, dikkatli ol

// 3. Test kodunda — kontrollü ortam
func testUserCreation() {
    let user = createTestUser()
    XCTAssertNotNil(user)
    let unwrappedUser = user!  // Test'te crash istiyorsun — hatayı hemen görmek için
}
```

### Force Unwrapping Yerine Ne Kullanmalı?

```swift
// ❌ Tehlikeli
let data = fetchData()!

// ✅ Güvenli alternatifler
// 1. guard let
guard let data = fetchData() else {
    print("Veri alınamadı")
    return
}

// 2. if let
if let data = fetchData() {
    process(data)
}

// 3. Nil-coalescing
let data = fetchData() ?? defaultData
```

**Geliştirici Bakış Açısı:** Code review'da `!` gören bir senior geliştirici sana **kesinlikle** "Neden force unwrap kullandın? Bu nil olabilir mi?" diye soracaktır. Her `!` için savunulabilir bir gerekçen olmalı. Genel kural: **gerçek projelerde `!` kullanmaktan kaçın.**

---

## 9. Implicitly Unwrapped Optionals (IUO)

Tanımlarken optional, kullanırken otomatik unwrap olan özel bir türdür. `!` ile tanımlanır.

```swift
var serverURL: String!  // Implicitly Unwrapped Optional

// Uygulama başlangıcında konfigürasyon okunuyor
func configure() {
    serverURL = "https://api.example.com"
}

func makeRequest() {
    // serverURL'ü unwrap etmeden kullanabilirsin
    print("İstek gönderiliyor: \(serverURL)")
    // ⚠️ AMA configure() çağrılmamışsa → 💥 CRASH
}
```

### IUO Ne Zaman Kullanılır?

| Kullanım | Açıklama |
|---|---|
| IBOutlet'ler | `@IBOutlet weak var label: UILabel!` — Storyboard bağlantıları |
| İki aşamalı init | Değer init'te atanamıyor ama ilk kullanımdan önce kesinlikle atanacak |
| Legacy Obj-C API | Bazı eski API'ler IUO döner |

**Geliştirici Bakış Açısı:** Yeni kodda IUO kullanımını minimize et. IBOutlet'ler dışında IUO kullanıyorsan, büyük ihtimalle daha iyi bir tasarım mümkündür.

---

## 10. Optional'larda Pattern Matching

Optional bir enum olduğu için, `switch` ile güçlü pattern matching yapabilirsin.

```swift
// ── switch ile Optional pattern matching ──
let statusMessage: String? = "Sipariş kargoya verildi"

switch statusMessage {
case .some(let message) where message.contains("kargo"):
    print("🚚 Kargo bildirimi: \(message)")
case .some(let message):
    print("📩 Mesaj: \(message)")
case .none:
    print("📭 Mesaj yok")
}

// ── Daha kısa syntax ──
switch statusMessage {
case let message? where message.contains("kargo"):  // .some(let message) yerine let message?
    print("🚚 \(message)")
case let message?:
    print("📩 \(message)")
case nil:
    print("📭 Mesaj yok")
}
```

### for-case-let ile Koleksiyonlarda Optional Filtreleme

```swift
let responses: [String?] = ["Evet", nil, "Hayır", nil, "Belki", nil]

// Sadece nil olmayanları işle
for case let response? in responses {
    print("Yanıt: \(response)")
}
// Yanıt: Evet
// Yanıt: Hayır
// Yanıt: Belki

// Alternatif: compactMap
let validResponses = responses.compactMap { $0 }
// ["Evet", "Hayır", "Belki"]
```

### if case let ile Tekil Kontrol

```swift
let errorCode: Int? = 404

if case let code? = errorCode, code >= 400 {
    print("Client hatası: \(code)")
}
// "Client hatası: 404"
```

---

## 11. map ve flatMap ile Optional Dönüşümleri

Optional bir enum (container) olduğu için, `map` ve `flatMap` ile fonksiyonel dönüşümler yapabilirsin. Bu, gereksiz `if let` / `guard let` ceremonisinden kaçınmanı sağlar.

### Optional.map

Değer varsa dönüşüm uygular, nil ise nil döner.

```swift
let optionalNumber: Int? = 42

// if let ile (daha uzun)
let doubled1: Int?
if let number = optionalNumber {
    doubled1 = number * 2
} else {
    doubled1 = nil
}

// map ile (daha kısa ve fonksiyonel)
let doubled2 = optionalNumber.map { $0 * 2 }
// Optional(84)

let nilNumber: Int? = nil
let doubled3 = nilNumber.map { $0 * 2 }
// nil

// Gerçek hayat: String dönüşümü
let userId: String? = "usr_123"
let uppercasedId = userId.map { $0.uppercased() }
// Optional("USR_123")
```

### Optional.flatMap

Dönüşüm fonksiyonu da Optional dönüyorsa, iç içe Optional'ı düzleştirir.

```swift
let stringNumber: String? = "42"

// map kullanınca → Optional<Optional<Int>> (iç içe!)
let nestedResult = stringNumber.map { Int($0) }
// Optional(Optional(42)) — 🤢 İç içe optional

// flatMap kullanınca → Optional<Int> (düzleştirilmiş)
let flatResult = stringNumber.flatMap { Int($0) }
// Optional(42) — ✅ Temiz

// nil senaryosu
let invalidString: String? = "abc"
let parsed = invalidString.flatMap { Int($0) }
// nil — "abc" Int'e dönüştürülemez
```

### Zincirleme Dönüşümler

```swift
struct UserDTO {
    let name: String?
    let birthYear: String?
}

let dto = UserDTO(name: "  Ahmet Yılmaz  ", birthYear: "1996")

// Zincirleme map/flatMap ile temiz veri dönüşümü
let cleanedName = dto.name
    .map { $0.trimmingCharacters(in: .whitespaces) }  // Boşlukları temizle
    .map { $0.uppercased() }                           // Büyük harfe çevir

let age = dto.birthYear
    .flatMap { Int($0) }           // String → Int? (flatMap çünkü Int() Optional döner)
    .map { 2026 - $0 }             // Yaş hesapla

print(cleanedName ?? "İsimsiz")  // "AHMET YILMAZ"
print(age ?? 0)                   // 30
```

---

## 12. Gerçek Hayat: Katmanlı Optional Yönetimi

Aşağıdaki örnek, bir kullanıcı profili sayfasında karşılaşacağın tüm Optional yönetim tekniklerini birleştirir:

```swift
// ══════════════════════════════════════
//  API YANIT MODELLERİ
// ══════════════════════════════════════

struct APIUserResponse {
    let id: String?
    let firstName: String?
    let lastName: String?
    let email: String?
    let avatarURL: String?
    let address: APIAddressResponse?
    let preferences: APIPreferencesResponse?
}

struct APIAddressResponse {
    let street: String?
    let city: String?
    let country: String?
    let postalCode: String?
}

struct APIPreferencesResponse {
    let language: String?
    let theme: String?
    let notificationsEnabled: Bool?
}


// ══════════════════════════════════════
//  UYGULAMA MODEL — Temiz, non-optional alanlar
// ══════════════════════════════════════

struct UserProfile {
    let id: String
    let displayName: String
    let email: String
    let avatarURL: String
    let city: String
    let country: String
    let language: String
    let theme: String
    let notificationsEnabled: Bool
}


// ══════════════════════════════════════
//  MAPPER — Optional'ları güvenle dönüştür
// ══════════════════════════════════════

enum MappingError: Error {
    case missingRequiredField(String)
}

struct UserProfileMapper {
    
    /// API yanıtını uygulama modeline dönüştürür.
    /// Zorunlu alanlar yoksa hata fırlatır, opsiyonel alanlar için varsayılan değerler kullanılır.
    static func map(from response: APIUserResponse) throws -> UserProfile {
        
        // ── Zorunlu alanlar: guard let ile erken çıkış ──
        guard let id = response.id, !id.isEmpty else {
            throw MappingError.missingRequiredField("id")
        }
        
        guard let email = response.email, email.contains("@") else {
            throw MappingError.missingRequiredField("email")
        }
        
        // ── Display name: map ile fonksiyonel dönüşüm ──
        let firstName = response.firstName
            .map { $0.trimmingCharacters(in: .whitespaces) }
            ?? ""
        
        let lastName = response.lastName
            .map { $0.trimmingCharacters(in: .whitespaces) }
            ?? ""
        
        let displayName: String
        if !firstName.isEmpty && !lastName.isEmpty {
            displayName = "\(firstName) \(lastName)"
        } else if !firstName.isEmpty {
            displayName = firstName
        } else {
            displayName = email.split(separator: "@").first.map(String.init) ?? "Kullanıcı"
        }
        
        // ── Opsiyonel alanlar: nil-coalescing ile varsayılan değerler ──
        let avatarURL = response.avatarURL ?? "https://cdn.example.com/default-avatar.png"
        
        // ── Optional chaining + nil-coalescing kombine ──
        let city = response.address?.city ?? "Belirtilmemiş"
        let country = response.address?.country ?? "Belirtilmemiş"
        
        // ── İç içe optional chaining ──
        let language = response.preferences?.language ?? "tr"
        let theme = response.preferences?.theme ?? "system"
        let notificationsEnabled = response.preferences?.notificationsEnabled ?? true
        
        return UserProfile(
            id: id,
            displayName: displayName,
            email: email,
            avatarURL: avatarURL,
            city: city,
            country: country,
            language: language,
            theme: theme,
            notificationsEnabled: notificationsEnabled
        )
    }
}


// ══════════════════════════════════════
//  KULLANIM
// ══════════════════════════════════════

// Senaryo 1: Tam veri
let fullResponse = APIUserResponse(
    id: "U001",
    firstName: "Zeynep",
    lastName: "Aydın",
    email: "zeynep@example.com",
    avatarURL: "https://cdn.example.com/zeynep.jpg",
    address: APIAddressResponse(street: "Bağdat Caddesi", city: "İstanbul", country: "Türkiye", postalCode: "34710"),
    preferences: APIPreferencesResponse(language: "tr", theme: "dark", notificationsEnabled: false)
)

if let profile = try? UserProfileMapper.map(from: fullResponse) {
    print("✅ \(profile.displayName) — \(profile.city), \(profile.country)")
    // "✅ Zeynep Aydın — İstanbul, Türkiye"
}

// Senaryo 2: Eksik verilerle (gerçek API'lerden gelen tipik yanıt)
let partialResponse = APIUserResponse(
    id: "U002",
    firstName: nil,
    lastName: nil,
    email: "mystery@test.com",
    avatarURL: nil,
    address: nil,
    preferences: nil
)

if let profile = try? UserProfileMapper.map(from: partialResponse) {
    print("✅ \(profile.displayName) — \(profile.city)")
    // "✅ mystery — Belirtilmemiş"
    // displayName: email'den türetildi
    // city: varsayılan değer kullanıldı
    // avatar, language, theme, notifications: hepsi varsayılan değer
}

// Senaryo 3: Zorunlu alan eksik — hata fırlatılır
let invalidResponse = APIUserResponse(
    id: nil,  // ⚠️ Zorunlu alan eksik
    firstName: "Test",
    lastName: nil,
    email: "test@test.com",
    avatarURL: nil,
    address: nil,
    preferences: nil
)

do {
    let profile = try UserProfileMapper.map(from: invalidResponse)
} catch MappingError.missingRequiredField(let field) {
    print("❌ Zorunlu alan eksik: \(field)")
    // "❌ Zorunlu alan eksik: id"
} catch {
    print("❌ Beklenmeyen hata: \(error)")
}
```

Bu örnek, gerçek bir iOS uygulamasında karşılaşacağın tipik bir senaryoyu gösterir: API yanıtları her zaman tam gelmez. Bazı alanlar zorunludur (yoksa hata), bazıları opsiyoneldir (varsayılan değer kullanılır). Bu yapıyı kurarak uygulamanın **asla crash olmadan** eksik verileri zarif şekilde yönetmesini sağlarsın.

---

## 📝 Bölüm Sonu Notları

Bu dokümanı tamamladıktan sonra şunları yapabilmelisin:

1. Optional'ın ne olduğunu ve Swift'te neden var olduğunu açıklamak
2. `if let` ve `guard let` arasındaki farkı bilmek ve doğru senaryoda kullanmak
3. Nil-coalescing (`??`) ile varsayılan değer sağlamak
4. Optional chaining ile derin iç içe yapılara güvenle erişmek
5. Force unwrapping'in risklerini bilmek ve neredeyse hiç kullanmamak
6. `map` ve `flatMap` ile Optional dönüşümleri yapmak
7. Gerçek bir API yanıtındaki Optional'ları profesyonel şekilde yönetmek

---

> **Sonraki Doküman:** [03 — OOP & POP](03%20-%20OOP%20ve%20POP.md)

---

*Phase 1 · Doküman 2/6 · Şubat 2026*
