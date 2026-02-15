# 📘 01 — Swift Temelleri

> **Phase 1 · Doküman 1/6**  
> **Konu:** Swift syntax, veri tipleri, koleksiyonlar, value vs reference types, struct vs class  
> **Tahmini Süre:** 3–4 gün  
> **Ön Koşul:** Herhangi bir programlama dilinde temel deneyim

---

## 📑 İçindekiler

1. [Değişkenler ve Sabitler (let / var)](#1-değişkenler-ve-sabitler-let--var)
2. [Type Inference (Tür Çıkarımı)](#2-type-inference-tür-çıkarımı)
3. [Temel Veri Tipleri](#3-temel-veri-tipleri)
4. [String İşlemleri ve Interpolation](#4-string-i̇şlemleri-ve-interpolation)
5. [Koleksiyonlar: Array, Dictionary, Set](#5-koleksiyonlar-array-dictionary-set)
   - [5.1 Array](#51-array)
   - [5.2 Dictionary](#52-dictionary)
   - [5.3 Set](#53-set)
   - [5.4 Koleksiyonlarda Higher-Order Functions](#54-koleksiyonlarda-higher-order-functions)
6. [Kontrol Akışı (Control Flow)](#6-kontrol-akışı-control-flow)
   - [6.1 if / else](#61-if--else)
   - [6.2 switch](#62-switch)
   - [6.3 Döngüler (for-in, while)](#63-döngüler-for-in-while)
7. [Fonksiyonlar](#7-fonksiyonlar)
8. [Value Types vs Reference Types](#8-value-types-vs-reference-types)
   - [8.1 Temel Fark: Kopyalama vs Referans Paylaşımı](#81-temel-fark-kopyalama-vs-referans-paylaşımı)
   - [8.2 struct Derinlemesine](#82-struct-derinlemesine)
   - [8.3 class Derinlemesine](#83-class-derinlemesine)
   - [8.4 Copy-on-Write (COW)](#84-copy-on-write-cow)
   - [8.5 mutating Keyword](#85-mutating-keyword)
   - [8.6 Ne Zaman struct, Ne Zaman class?](#86-ne-zaman-struct-ne-zaman-class)
9. [Gerçek Hayat Projesi: Ürün Kataloğu](#9-gerçek-hayat-projesi-ürün-kataloğu)

---

## 1. Değişkenler ve Sabitler (let / var)

Swift'te değerler iki şekilde tanımlanır: **değişmeyen** (`let`) ve **değişebilen** (`var`). Bu iki anahtar kelime, dilin güvenlik felsefesinin temel taşıdır.

```swift
// ── Sabit (Immutable) — bir kez ata, bir daha değiştirme ──
let appName = "Bookstore"
let maxLoginAttempts = 5
let pi = 3.14159

// ── Değişken (Mutable) — gerektiğinde değiştir ──
var currentTemperature = 22.5
var loginCount = 0
var isUserLoggedIn = false

// Değişken güncelleme
currentTemperature = 18.0   // ✅ Sorun yok
loginCount += 1              // ✅ Sorun yok

// Sabit güncelleme denemesi
// appName = "NewApp"        // ❌ Derleme hatası: Cannot assign to value: 'appName' is a 'let' constant
```

### Swift Topluluğunun Altın Kuralı

> **"Varsayılan olarak `let` kullan. Sadece değerin gerçekten değişeceğinden eminsen `var` yap."**

Bu kural neden önemli?

**Okunabilirlik:** Kodu okuyan bir geliştirici, `let` gördüğünde "Bu değer hiçbir yerde değişmiyor, güvenle okuyabilirim" der. `var` gördüğünde ise "Bu değer bir yerlerde değişecek, dikkat etmeliyim" der.

**Güvenlik:** Derleyici, yanlışlıkla değiştirmemen gereken bir değeri değiştirmeni engeller.

**Performans:** Derleyici, `let` ile tanımlanan değerleri optimize edebilir çünkü değişmeyeceğini bilir.

**Geliştirici Bakış Açısı:** Xcode, bir `var` tanımlayıp hiç değiştirmezsen şu uyarıyı verir: *"Variable was never mutated; consider changing to 'let'"*. Profesyonel projelerde bu uyarılar ciddiye alınır ve code review'da not düşülür.

---

## 2. Type Inference (Tür Çıkarımı)

Swift, **statically typed** bir dildir — her değişkenin derleme zamanında bilinen bir türü vardır. Ancak bunu her seferinde elle yazmak zorunda değilsin. Swift'in güçlü **tür çıkarım** (type inference) motoru, atanan değere bakarak türü otomatik belirler.

```swift
// ── Tür çıkarımı ile (önerilen, daha temiz) ──
let userName = "Zeynep"       // String olarak çıkarıldı
let userAge = 28              // Int olarak çıkarıldı
let accountBalance = 1520.75  // Double olarak çıkarıldı
let isPremium = true          // Bool olarak çıkarıldı

// ── Açık tür belirtimi ile ──
let userId: String = "usr_a1b2c3"
let retryLimit: Int = 3
let taxRate: Double = 0.18
let isActive: Bool = false
```

### Ne Zaman Açık Tür Belirtmelisin?

Çoğu zaman tür çıkarımı yeterlidir, ancak birkaç durumda açık belirtim gerekir:

```swift
// 1. Derleyici yanlış tür çıkarabilir
let temperature: Float = 36.6   // Açık belirtme olmazsa Double olarak çıkarılır

// 2. Boş koleksiyon oluştururken
let names: [String] = []        // Tür çıkarımı yapacak bir değer yok
var scores: [String: Int] = [:] // Boş dictionary

// 3. Bağlam belirsizken okunabilirlik için
let statusCode: Int = 200
let timeout: TimeInterval = 30  // TimeInterval aslında Double'dır, ama bağlamı netleştirir
```

---

## 3. Temel Veri Tipleri

Swift'in temel tipleri, Dart'takilerle işlevsel olarak benzerdir ancak önemli davranışsal farklar içerir.

### Sayısal Tipler

```swift
// Tam sayılar
let count: Int = 42          // Platformun mimarisine göre 64-bit (modern cihazlarda)
let positive: UInt = 100     // Unsigned — sadece 0 ve pozitif değerler

// Kayan noktalı sayılar
let price: Double = 99.99    // 64-bit — varsayılan ve önerilen
let ratio: Float = 0.75      // 32-bit — nadir kullanılır (grafik işlemleri, vb.)
```

### Metin ve Karakter

```swift
let greeting: String = "Merhaba Dünya"
let initial: Character = "A"

// String birleştirme
let firstName = "Ahmet"
let lastName = "Yılmaz"
let fullName = firstName + " " + lastName  // "Ahmet Yılmaz"
```

### Mantıksal

```swift
let isValid: Bool = true
let hasPermission: Bool = false
```

### Kritik Fark: Implicit Type Conversion Yoktur

Bu, Dart'tan gelen bir geliştirici için en önemli farktır.

```swift
let intValue = 10
let doubleValue = 3.5

// ❌ DERLEME HATASI — Swift implicit (otomatik) dönüşüm YAPMAZ
// let result = intValue + doubleValue

// ✅ Açık (explicit) dönüşüm gerekli
let result = Double(intValue) + doubleValue  // 13.5

// Dart'ta bu çalışır:
// int a = 10;
// double b = 3.5;
// var result = a + b;  // 13.5 — Dart otomatik dönüştürür
```

Bu neden böyle? Swift'in tasarım felsefesi **"açıklık güvenliği getirir"** (clarity brings safety) prensibine dayanır. Tür dönüşümünde veri kaybı olabilir (örn. `Double` → `Int` kesirli kısmı atar) ve Swift bunu bilinçli bir karar olarak senden bekler.

```swift
let pi = 3.14159
let rounded = Int(pi)  // 3 — kesirli kısım KAYBOLUR, ama sen bunu bilerek yaptın
```

### Type Aliases

Karmaşık tiplere okunabilir isimler verebilirsin:

```swift
typealias UserID = String
typealias Price = Double
typealias CompletionHandler = (Bool, Error?) -> Void

let currentUserId: UserID = "usr_12345"
let productPrice: Price = 299.99
```

---

## 4. String İşlemleri ve Interpolation

### String Interpolation

Değişkenleri ve ifadeleri string içine gömme yöntemidir. Dart'taki `$variable` ve `${expression}` yerine Swift `\(expression)` kullanır.

```swift
let userName = "Elif"
let orderCount = 5
let totalPrice = 149.99

// Temel interpolation
let message = "Merhaba, \(userName)!"

// İfade (expression) ile
let summary = "\(userName) toplam \(orderCount) sipariş verdi. Tutar: \(totalPrice) TL"

// İçinde hesaplama
let discountMessage = "İndirimli fiyat: \(totalPrice * 0.8) TL"
```

### Çok Satırlı String (Multi-line String Literals)

```swift
let htmlTemplate = """
<html>
<head>
    <title>\(userName) Profili</title>
</head>
<body>
    <h1>Hoş geldin, \(userName)!</h1>
    <p>Toplam sipariş: \(orderCount)</p>
</body>
</html>
"""
```

### Sık Kullanılan String İşlemleri

```swift
let email = "  Ahmet@Example.COM  "

// Temizleme ve dönüştürme
let cleaned = email
    .trimmingCharacters(in: .whitespaces)  // Baş/son boşlukları kaldır
    .lowercased()                           // Küçük harfe çevir
// "ahmet@example.com"

// Kontroller
let hasAtSign = cleaned.contains("@")      // true
let isEmpty = cleaned.isEmpty               // false
let charCount = cleaned.count               // 19

// Alt string işlemleri
let domain = cleaned.split(separator: "@").last  // Optional("example.com")
let hasPrefix = cleaned.hasPrefix("ahmet")       // true
let hasSuffix = cleaned.hasSuffix(".com")         // true

// Karakter erişimi (Swift'te string indexing farklıdır!)
let firstChar = cleaned[cleaned.startIndex]  // "a"
// ⚠️ cleaned[0] ÇALIŞMAZ — Swift'te integer subscript yok
// Bunun nedeni: Swift stringleri Unicode-safe'dir ve her karakter farklı boyutta olabilir
```

**Geliştirici Bakış Açısı:** Swift'te `string[0]` yazamazsın. Bu kasıtlıdır — Swift, Unicode'u birinci sınıf vatandaş olarak destekler ve bir "karakter" tek bir byte olmayabilir (emoji'ler, aksanlı harfler vb.). Bu yüzden string indexing `String.Index` tipiyle yapılır. Başta garip gelir ama büyük projelerde Unicode bug'larını önler.

---

## 5. Koleksiyonlar: Array, Dictionary, Set

### 5.1 Array

Sıralı, indeksli eleman koleksiyonu. Dart'taki `List<T>` karşılığı.

```swift
// Oluşturma
var cities: [String] = ["İstanbul", "Ankara", "İzmir"]
var numbers = [10, 20, 30, 40]  // Tür çıkarımı: [Int]
let emptyList: [String] = []     // Boş array (tür belirtilmeli)

// Eleman ekleme
cities.append("Bursa")                    // Sona ekle
cities.insert("Antalya", at: 2)           // Belirli indekse ekle
cities += ["Trabzon", "Adana"]            // Birleştirerek ekle

// Eleman erişimi
let first = cities[0]                     // "İstanbul"
let last = cities.last                    // Optional("Adana")
let safeAccess = cities.indices.contains(99) ? cities[99] : nil  // Güvenli erişim

// Eleman silme
cities.remove(at: 0)                      // İndeks ile sil
cities.removeLast()                        // Sonuncuyu sil
cities.removeAll { $0.hasPrefix("A") }     // Koşullu silme

// Bilgi
let count = cities.count                   // Eleman sayısı
let hasIstanbul = cities.contains("İstanbul")  // Varlık kontrolü
```

### 5.2 Dictionary

Anahtar-değer çiftleri. Dart'taki `Map<K, V>` karşılığı.

```swift
// Oluşturma
var userAges: [String: Int] = [
    "Ahmet": 28,
    "Zeynep": 25,
    "Mehmet": 32
]

// Eleman ekleme / güncelleme
userAges["Elif"] = 30                 // Yeni eleman
userAges["Ahmet"] = 29               // Güncelleme

// ⚠️ KRİTİK: Dictionary'den değer okumak HER ZAMAN Optional döner
let ahmetAge = userAges["Ahmet"]      // Optional(29)
let unknownAge = userAges["Bilinmeyen"]  // nil

// Güvenli erişim — varsayılan değerle
let age = userAges["Bilinmeyen", default: 0]  // 0 (Optional DEĞİL)

// Eleman silme
userAges.removeValue(forKey: "Mehmet")
userAges["Zeynep"] = nil              // Bu da siler

// Döngü
for (name, age) in userAges {
    print("\(name): \(age) yaşında")
}

// Sadece anahtarlar veya değerler
let allNames = Array(userAges.keys)    // [String]
let allAges = Array(userAges.values)   // [Int]
```

**Geliştirici Bakış Açısı:** Dictionary'den değer okumanın Optional döndürmesi, Dart'taki `Map`'ten önemli bir farktır. Dart'ta `map['key']` nullable döner ama bunu handle etmek zorunda değilsin. Swift'te **derleyici seni zorlar** — bu da runtime crash'leri önler.

### 5.3 Set

Benzersiz elemanlar koleksiyonu, sırasız. Dart'taki `Set<T>` ile aynı mantık.

```swift
// Oluşturma
var skills: Set<String> = ["Swift", "UIKit", "SwiftUI"]

// Eleman ekleme — zaten varsa hiçbir şey olmaz
skills.insert("Swift")      // Zaten var — set değişmez
skills.insert("Combine")    // Eklendi

// Küme operasyonları (matematiksel)
let frontendSkills: Set = ["Swift", "SwiftUI", "UIKit"]
let backendSkills: Set = ["Swift", "Vapor", "PostgreSQL"]

let commonSkills = frontendSkills.intersection(backendSkills)  // {"Swift"}
let allSkills = frontendSkills.union(backendSkills)            // Tüm benzersiz elemanlar
let onlyFrontend = frontendSkills.subtracting(backendSkills)   // {"SwiftUI", "UIKit"}
```

**Ne zaman Set kullanırsın?**
- Benzersizlik garantisi gerektiğinde (tag'ler, izinler, ID'ler)
- Sıranın önemli olmadığı durumlarda
- Hızlı `contains()` kontrolü gerektiğinde — Set'te `O(1)`, Array'de `O(n)`

### 5.4 Koleksiyonlarda Higher-Order Functions

Bu fonksiyonlar iOS geliştirmede **her gün** kullanılır. Bunlara hakim olmak kodunun kalitesini büyük ölçüde artırır.

```swift
struct Product {
    let id: String
    let name: String
    let price: Double
    let category: String
    let isInStock: Bool
}

let products = [
    Product(id: "1", name: "MacBook Pro", price: 45000, category: "Bilgisayar", isInStock: true),
    Product(id: "2", name: "iPhone 16", price: 65000, category: "Telefon", isInStock: true),
    Product(id: "3", name: "AirPods", price: 8000, category: "Aksesuar", isInStock: false),
    Product(id: "4", name: "iPad Air", price: 28000, category: "Tablet", isInStock: true),
    Product(id: "5", name: "Apple Watch", price: 15000, category: "Aksesuar", isInStock: true),
]

// ── map: Her elemanı dönüştür ──
let productNames = products.map { $0.name }
// ["MacBook Pro", "iPhone 16", "AirPods", "iPad Air", "Apple Watch"]

// ── filter: Koşula uyanları seç ──
let inStockProducts = products.filter { $0.isInStock }
// AirPods hariç 4 ürün

// ── reduce: Tüm elemanları tek bir değere indirge ──
let totalValue = products.reduce(0) { $0 + $1.price }
// 161000.0

// ── compactMap: nil değerleri atar, Optional'ları unwrap eder ──
let optionalPrices: [Double?] = [100, nil, 200, nil, 300]
let validPrices = optionalPrices.compactMap { $0 }
// [100, 200, 300]

// ── flatMap: İç içe koleksiyonları düzleştir ──
let nestedNumbers = [[1, 2, 3], [4, 5], [6]]
let flat = nestedNumbers.flatMap { $0 }
// [1, 2, 3, 4, 5, 6]

// ── sorted: Sıralama ──
let cheapToExpensive = products.sorted { $0.price < $1.price }

// ── Zincirleme (Chaining) — gerçek hayat senaryosu ──
let affordableAccessoryNames = products
    .filter { $0.category == "Aksesuar" && $0.isInStock }  // Stokta olan aksesuarlar
    .sorted { $0.price < $1.price }                         // Ucuzdan pahalıya
    .map { $0.name }                                        // Sadece isimler
// ["Apple Watch"]

// ── forEach: Side-effect için (print, log, vs.) ──
products
    .filter { !$0.isInStock }
    .forEach { print("⚠️ Stok dışı: \($0.name)") }
```

---

## 6. Kontrol Akışı (Control Flow)

### 6.1 if / else

```swift
let temperature = 35

if temperature > 30 {
    print("Çok sıcak")
} else if temperature > 20 {
    print("Güzel hava")
} else {
    print("Serin")
}

// Ternary operator
let status = temperature > 30 ? "Sıcak" : "Normal"
```

### 6.2 switch

Swift'te `switch` son derece güçlüdür ve Dart'taki `switch`'ten çok daha fazla şey yapabilir.

```swift
let statusCode = 404

// Temel kullanım — break yazmana gerek yok (fall-through yok)
switch statusCode {
case 200:
    print("Başarılı")
case 201:
    print("Oluşturuldu")
case 400...499:              // Aralık (range) kontrolü
    print("Client hatası: \(statusCode)")
case 500...599:
    print("Server hatası")
default:
    print("Bilinmeyen durum kodu")
}

// Tuple ile pattern matching
let coordinate = (x: 0, y: 5)

switch coordinate {
case (0, 0):
    print("Orijin noktası")
case (let x, 0):
    print("X ekseninde, x = \(x)")
case (0, let y):
    print("Y ekseninde, y = \(y)")
case let (x, y):
    print("Başka bir nokta: (\(x), \(y))")
}

// where ile koşul ekleme
let age = 25

switch age {
case let a where a < 0:
    print("Geçersiz yaş")
case 0...12:
    print("Çocuk")
case 13...17:
    print("Ergen")
case 18...64:
    print("Yetişkin")
case let a where a >= 65:
    print("Yaşlı")
default:
    break
}
```

**Önemli:** Swift'te `switch` **exhaustive** olmak zorundadır — yani tüm olası durumlar karşılanmalıdır. Karşılanmayan durum varsa `default` case zorunludur. Bu derleyici seviyesinde garanti edilir.

### 6.3 Döngüler (for-in, while)

```swift
// for-in — en yaygın
for number in 1...5 {
    print(number)  // 1, 2, 3, 4, 5
}

// Range ile (üst sınır dahil değil)
for i in 0..<3 {
    print(i)  // 0, 1, 2
}

// Array üzerinde
let fruits = ["Elma", "Armut", "Portakal"]
for fruit in fruits {
    print(fruit)
}

// Index ile birlikte
for (index, fruit) in fruits.enumerated() {
    print("\(index): \(fruit)")
}

// Dictionary üzerinde
let scores = ["Ahmet": 95, "Zeynep": 88]
for (name, score) in scores {
    print("\(name) → \(score)")
}

// where filtresi ile
for number in 1...20 where number.isMultiple(of: 3) {
    print(number)  // 3, 6, 9, 12, 15, 18
}

// while
var countdown = 5
while countdown > 0 {
    print("\(countdown)...")
    countdown -= 1
}

// repeat-while (Dart'taki do-while)
var attempts = 0
repeat {
    attempts += 1
    print("Deneme \(attempts)")
} while attempts < 3
```

---

## 7. Fonksiyonlar

```swift
// ── Temel fonksiyon ──
func greet(name: String) -> String {
    return "Merhaba, \(name)!"
}

let message = greet(name: "Ahmet")

// ── Birden fazla dönüş değeri (Tuple) ──
func calculateStats(numbers: [Int]) -> (min: Int, max: Int, average: Double) {
    let min = numbers.min() ?? 0
    let max = numbers.max() ?? 0
    let average = Double(numbers.reduce(0, +)) / Double(numbers.count)
    return (min, max, average)
}

let stats = calculateStats(numbers: [5, 2, 8, 1, 9])
print("Min: \(stats.min), Max: \(stats.max), Avg: \(stats.average)")

// ── Argument Labels ve Parameter Names ──
func sendMessage(from sender: String, to receiver: String, body message: String) {
    print("\(sender) → \(receiver): \(message)")
}
sendMessage(from: "Ahmet", to: "Zeynep", body: "Merhaba!")
// "from", "to", "body" → argument labels (çağrı tarafında)
// "sender", "receiver", "message" → parameter names (fonksiyon içinde)

// Label'ı gizleme (underscore)
func square(_ number: Int) -> Int {
    return number * number
}
let result = square(5)  // square(number: 5) yerine daha temiz

// ── Varsayılan değerler ──
func createUser(name: String, role: String = "member", isActive: Bool = true) -> [String: Any] {
    return ["name": name, "role": role, "isActive": isActive]
}

let admin = createUser(name: "Ahmet", role: "admin")
let defaultUser = createUser(name: "Zeynep")  // role: "member", isActive: true

// ── Variadic parameters ──
func calculateTotal(_ prices: Double...) -> Double {
    return prices.reduce(0, +)
}
let total = calculateTotal(29.99, 15.50, 42.00)  // 87.49

// ── inout parameters — değeri fonksiyon içinden değiştir ──
func doubleValue(_ value: inout Int) {
    value *= 2
}

var myNumber = 10
doubleValue(&myNumber)  // & işareti zorunlu — "bunu değiştireceksin" demek
print(myNumber)          // 20
```

### Fonksiyon Tipleri

Swift'te fonksiyonlar **birinci sınıf vatandaştır** — bir değişkene atanabilir, parametre olarak geçilebilir ve dönüş değeri olabilir.

```swift
// Fonksiyon tipi: (Int, Int) -> Int
func add(_ a: Int, _ b: Int) -> Int { a + b }
func multiply(_ a: Int, _ b: Int) -> Int { a * b }

// Fonksiyonu değişkene ata
var operation: (Int, Int) -> Int = add
print(operation(3, 4))  // 7

operation = multiply
print(operation(3, 4))  // 12

// Fonksiyonu parametre olarak geç
func applyOperation(_ a: Int, _ b: Int, using op: (Int, Int) -> Int) -> Int {
    return op(a, b)
}

let sum = applyOperation(10, 5, using: add)       // 15
let product = applyOperation(10, 5, using: multiply)  // 50
```

---

## 8. Value Types vs Reference Types

Bu bölüm, Swift'e geçişte yaşanacak **en önemli zihinsel değişimi** temsil eder.

### 8.1 Temel Fark: Kopyalama vs Referans Paylaşımı

Swift'te iki temel bellek modeli vardır:

**Value Type (Değer Tipi):** Bir değişkeni başka bir değişkene atadığında **bağımsız bir kopya** oluşur. Birindeki değişiklik diğerini etkilemez.

**Reference Type (Referans Tipi):** Bir değişkeni başka bir değişkene atadığında **aynı nesneye ikinci bir referans** oluşur. Birindeki değişiklik diğerini de etkiler.

```swift
// ══════════════════════════════════════
//  VALUE TYPE — struct
// ══════════════════════════════════════
struct Coordinate {
    var latitude: Double
    var longitude: Double
}

var locationA = Coordinate(latitude: 41.0082, longitude: 28.9784) // İstanbul
var locationB = locationA   // 📋 BAĞIMSIZ KOPYA oluştu

locationB.latitude = 39.9334  // Ankara'ya değiştirdik

print(locationA.latitude)  // 41.0082 — İstanbul, DEĞİŞMEDİ ✅
print(locationB.latitude)  // 39.9334 — Ankara


// ══════════════════════════════════════
//  REFERENCE TYPE — class
// ══════════════════════════════════════
class UserAccount {
    var name: String
    var balance: Double
    
    init(name: String, balance: Double) {
        self.name = name
        self.balance = balance
    }
}

var accountA = UserAccount(name: "Ahmet", balance: 1000)
var accountB = accountA   // 🔗 AYNI NESNEYE referans — kopya DEĞİL

accountB.balance = 500

print(accountA.balance)  // 500 — ⚠️ accountA da DEĞİŞTİ!
print(accountB.balance)  // 500
```

Bunu somut bir benzetmeyle anlayalım:

- **struct (Value Type)** → Bir belgeyi fotokopi makinesinde kopyalamak. Kopya üzerinde karalama yapsan da orijinal belge sapasağlam durur.
- **class (Reference Type)** → Bir Google Docs linkini paylaşmak. Herkes aynı belgeye erişir, biri düzenleme yapınca herkes görür.

### 8.2 struct Derinlemesine

```swift
struct ShoppingCartItem {
    let productId: String
    let productName: String
    var quantity: Int
    let unitPrice: Double
    
    // Computed property — hesaplanmış değer
    var totalPrice: Double {
        return Double(quantity) * unitPrice
    }
    
    // Memberwise initializer — Swift struct'lara otomatik verir
    // ShoppingCartItem(productId: "1", productName: "Kitap", quantity: 2, unitPrice: 45.0)
}

// Swift'te struct'lar otomatik olarak bir "memberwise initializer" alır
// Bu init'i elle yazmana gerek yok — derleyici üretir
var item = ShoppingCartItem(
    productId: "P001",
    productName: "Swift Programlama Kitabı",
    quantity: 2,
    unitPrice: 89.90
)

print(item.totalPrice)  // 179.80

// Kopyalama davranışı
var giftItem = item
giftItem.quantity = 1      // Sadece kopya değişti
print(item.quantity)        // 2 — orijinal etkilenmedi
print(giftItem.quantity)    // 1
```

**Swift'te Neler Value Type'tır?**

`Int`, `Double`, `Float`, `Bool`, `String`, `Array`, `Dictionary`, `Set` ve tüm `struct`'lar ve `enum`'lar value type'tır. Yani Swift standard library'nin büyük çoğunluğu value type'tır.

### 8.3 class Derinlemesine

```swift
class ShoppingCart {
    var items: [ShoppingCartItem] = []
    let cartId: String
    
    // Class'larda init YAZMAN GEREKİR — otomatik memberwise init yok
    init(cartId: String) {
        self.cartId = cartId
    }
    
    var totalAmount: Double {
        return items.reduce(0) { $0 + $1.totalPrice }
    }
    
    func addItem(_ item: ShoppingCartItem) {
        items.append(item)
    }
    
    func removeItem(productId: String) {
        items.removeAll { $0.productId == productId }
    }
    
    // deinit — class bellekten silindiğinde çağrılır (struct'ta YOK)
    deinit {
        print("🗑️ Sepet \(cartId) bellekten silindi")
    }
}

// Referans paylaşımı
var myCart = ShoppingCart(cartId: "C001")
myCart.addItem(ShoppingCartItem(productId: "P1", productName: "Kitap", quantity: 1, unitPrice: 50))

var sharedCart = myCart  // 🔗 Aynı sepete referans
sharedCart.addItem(ShoppingCartItem(productId: "P2", productName: "Kalem", quantity: 3, unitPrice: 10))

print(myCart.items.count)     // 2 — ⚠️ sharedCart üzerinden eklenen ürün burada da görünüyor
print(sharedCart.items.count) // 2

// Kimlik kontrolü — aynı nesne mi?
print(myCart === sharedCart)   // true — aynı nesne (=== sadece class için geçerli)
```

### struct vs class: Tam Karşılaştırma

| Özellik | struct (Value Type) | class (Reference Type) |
|---|---|---|
| Atama davranışı | Bağımsız kopya | Referans paylaşımı |
| Memberwise init | Otomatik verilir | Elle yazılmalı |
| Inheritance (Kalıtım) | ❌ Desteklemez | ✅ Destekler |
| deinit | ❌ Yok | ✅ Var |
| Kimlik kontrolü (===) | ❌ | ✅ |
| mutating keyword | Gerekli (property değiştirmek için) | Gerekli değil |
| ARC (Memory Management) | ❌ Gerekli değil | ✅ Referans sayımı ile yönetilir |
| Thread Safety | Varsayılan olarak güvenli (kopya) | Dikkat gerekli (paylaşımlı state) |

### 8.4 Copy-on-Write (COW)

"Her atamada kopya oluşuyorsa, büyük array'ler için bu çok yavaş olmaz mı?" — Çok haklı bir soru.

Swift standard library tipleri (`Array`, `Dictionary`, `String`, `Set`) **Copy-on-Write** optimizasyonu kullanır. Kopya, yalnızca **değişiklik yapıldığında** gerçekleşir.

```swift
var originalArray = Array(1...1_000_000)  // 1 milyon eleman
var copiedArray = originalArray           // ⚡ Henüz kopya YOK — aynı belleği paylaşıyor

// Bu noktada iki değişken aynı bellek adresine bakıyor
// Hiçbir performans maliyeti yok

copiedArray.append(0)  // 📋 ŞİMDİ gerçek kopya oluşturuldu
// Çünkü copiedArray'i değiştirmeye çalışıyoruz ve orijinali korumak lazım
```

**Önemli:** COW, Swift standard library tipleri için otomatiktir. Kendi yazdığın `struct`'larda COW otomatik **uygulanmaz** — ancak Apple'ın önerdiği pattern'lerle kendin implement edebilirsin (advanced konu).

### 8.5 mutating Keyword

struct'lar value type olduğu için, bir method kendi property'lerini değiştirmek istiyorsa `mutating` keyword'ü gereklidir. Bu keyword, aslında struct'ın **yeni bir kopyasını oluşturup eski yerine koyma** işlemini temsil eder.

```swift
struct Counter {
    private(set) var count: Int = 0
    
    // ✅ mutating — kendi property'sini değiştiriyor
    mutating func increment() {
        count += 1
    }
    
    mutating func decrement() {
        count = max(0, count - 1)
    }
    
    mutating func reset() {
        count = 0
    }
    
    // mutating gerektirmeyen method — sadece okuyor
    func displayCount() -> String {
        return "Sayaç: \(count)"
    }
}

// var ile tanımlanan struct — mutating method'lar çağrılabilir
var counter = Counter()
counter.increment()
counter.increment()
counter.increment()
print(counter.displayCount())  // "Sayaç: 3"

// let ile tanımlanan struct — mutating method'lar ÇAĞRILAMAZ
let fixedCounter = Counter()
// fixedCounter.increment()  // ❌ DERLEME HATASI
// Neden? let = immutable. struct zaten value type, değiştirmek = yeni kopya oluşturmak.
// Ama let bunu engelliyor.
```

### 8.6 Ne Zaman struct, Ne Zaman class?

Apple'ın resmi rehberi ve Swift topluluğunun konsensüsü:

**Varsayılan olarak `struct` kullan.** Sadece aşağıdaki durumlarda `class`'a geç:

```
┌─────────────────────────────────────────────────┐
│           KARAR AKIŞI: struct vs class           │
├─────────────────────────────────────────────────┤
│                                                  │
│  1. Kalıtım (inheritance) gerekli mi?            │
│     EVET → class                                 │
│     HAYIR → devam et ↓                           │
│                                                  │
│  2. Referans semantiği gerekli mi?               │
│     (Birden fazla yer aynı instance'ı            │
│      paylaşmalı mı?)                             │
│     EVET → class                                 │
│     HAYIR → devam et ↓                           │
│                                                  │
│  3. Kimlik (identity) önemli mi?                 │
│     (İki nesnenin "aynı nesne" olup olmadığını   │
│      kontrol etmen gerekecek mi?)                │
│     EVET → class                                 │
│     HAYIR → devam et ↓                           │
│                                                  │
│  4. Objective-C API'si ile etkileşim var mı?     │
│     EVET → class                                 │
│     HAYIR → struct ✅                            │
│                                                  │
└─────────────────────────────────────────────────┘
```

**Yaygın kullanım örnekleri:**

| Tip | struct | class |
|---|---|---|
| Veri modelleri | `User`, `Product`, `Order` | — |
| Konfigürasyon | `APIConfig`, `Theme` | — |
| Geometri | `Point`, `Size`, `Rect` | — |
| ViewController | — | `LoginViewController` |
| Singleton servisler | — | `NetworkManager.shared` |
| UIKit view'ları | — | `UIView`, `UILabel` (Apple'ın kararı) |
| Delegate nesneler | — | Delegation pattern genelde class gerektirir |

---

## 9. Gerçek Hayat Projesi: Ürün Kataloğu

Bu bölümdeki tüm kavramları birleştiren, profesyonel yapıda bir örnek:

```swift
// ══════════════════════════════════════
//  MODEL KATMANI — struct kullanıyoruz (veri modeli)
// ══════════════════════════════════════

struct Product {
    let id: String
    let name: String
    let price: Double
    let category: Category
    let stockCount: Int
    
    var isInStock: Bool {
        return stockCount > 0
    }
    
    var formattedPrice: String {
        return String(format: "%.2f TL", price)
    }
}

enum Category: String {
    case electronics = "Elektronik"
    case clothing = "Giyim"
    case books = "Kitap"
    case home = "Ev & Yaşam"
}


// ══════════════════════════════════════
//  KATALOG YÖNETİMİ — struct (value type, bağımsız kopyalar)
// ══════════════════════════════════════

struct ProductCatalog {
    private var products: [Product] = []
    
    var count: Int { products.count }
    var isEmpty: Bool { products.isEmpty }
    
    // ── Ekleme ──
    mutating func add(_ product: Product) {
        guard !products.contains(where: { $0.id == product.id }) else {
            print("⚠️ Ürün zaten mevcut: \(product.name)")
            return
        }
        products.append(product)
    }
    
    // ── Silme ──
    mutating func remove(id: String) -> Product? {
        guard let index = products.firstIndex(where: { $0.id == id }) else {
            return nil
        }
        return products.remove(at: index)
    }
    
    // ── Arama ──
    func search(query: String) -> [Product] {
        let lowercasedQuery = query.lowercased()
        return products.filter {
            $0.name.lowercased().contains(lowercasedQuery) ||
            $0.category.rawValue.lowercased().contains(lowercasedQuery)
        }
    }
    
    // ── Filtreleme ──
    func inStock() -> [Product] {
        return products.filter { $0.isInStock }
    }
    
    func byCategory(_ category: Category) -> [Product] {
        return products.filter { $0.category == category }
    }
    
    func affordable(maxPrice: Double) -> [Product] {
        return products
            .filter { $0.price <= maxPrice && $0.isInStock }
            .sorted { $0.price < $1.price }
    }
    
    // ── İstatistikler ──
    func statistics() -> CatalogStatistics {
        let inStockCount = products.filter { $0.isInStock }.count
        let totalValue = products.reduce(0) { $0 + ($1.price * Double($1.stockCount)) }
        let averagePrice = products.isEmpty ? 0 : products.reduce(0) { $0 + $1.price } / Double(products.count)
        
        let categoryBreakdown = Dictionary(grouping: products, by: { $0.category })
            .mapValues { $0.count }
        
        return CatalogStatistics(
            totalProducts: products.count,
            inStockProducts: inStockCount,
            outOfStockProducts: products.count - inStockCount,
            totalInventoryValue: totalValue,
            averagePrice: averagePrice,
            categoryBreakdown: categoryBreakdown
        )
    }
}

struct CatalogStatistics {
    let totalProducts: Int
    let inStockProducts: Int
    let outOfStockProducts: Int
    let totalInventoryValue: Double
    let averagePrice: Double
    let categoryBreakdown: [Category: Int]
    
    func display() {
        print("═══════════════════════════════")
        print("📊 Katalog İstatistikleri")
        print("═══════════════════════════════")
        print("Toplam ürün:        \(totalProducts)")
        print("Stokta:             \(inStockProducts)")
        print("Stok dışı:          \(outOfStockProducts)")
        print("Envanter değeri:    \(String(format: "%.2f TL", totalInventoryValue))")
        print("Ortalama fiyat:     \(String(format: "%.2f TL", averagePrice))")
        print("───────────────────────────────")
        for (category, count) in categoryBreakdown.sorted(by: { $0.value > $1.value }) {
            print("  \(category.rawValue): \(count) ürün")
        }
        print("═══════════════════════════════")
    }
}


// ══════════════════════════════════════
//  KULLANIM
// ══════════════════════════════════════

var catalog = ProductCatalog()

catalog.add(Product(id: "E001", name: "MacBook Pro 16\"", price: 89999, category: .electronics, stockCount: 15))
catalog.add(Product(id: "E002", name: "iPhone 16 Pro", price: 74999, category: .electronics, stockCount: 50))
catalog.add(Product(id: "C001", name: "Kışlık Mont", price: 2499, category: .clothing, stockCount: 0))
catalog.add(Product(id: "B001", name: "Swift Programlama", price: 189, category: .books, stockCount: 100))
catalog.add(Product(id: "H001", name: "Robot Süpürge", price: 15999, category: .home, stockCount: 8))

// Arama
let searchResults = catalog.search(query: "pro")
print("'pro' araması: \(searchResults.map { $0.name })")
// ["MacBook Pro 16\"", "iPhone 16 Pro"]

// Uygun fiyatlı ürünler
let budget = catalog.affordable(maxPrice: 5000)
print("5000 TL altı: \(budget.map { "\($0.name) — \($0.formattedPrice)" })")
// ["Swift Programlama — 189.00 TL"]
// (Mont stok dışı olduğu için listede yok)

// İstatistikler
catalog.statistics().display()

// Value type kanıtı — katalog kopyalandığında bağımsız
var backupCatalog = catalog
backupCatalog.remove(id: "E001")

print("Orijinal: \(catalog.count) ürün")    // 5 — etkilenmedi
print("Yedek:    \(backupCatalog.count) ürün")  // 4
```

---

## 📝 Bölüm Sonu Notları

Bu dokümanı tamamladıktan sonra şunları yapabilmelisin:

1. `let` ve `var`'ı bilinçli şekilde kullanmak
2. Swift'in temel veri tiplerini ve tür güvenliği felsefesini anlamak
3. String interpolation ve çok satırlı string'leri kullanmak
4. Array, Dictionary ve Set'i doğru senaryolarda kullanmak
5. `map`, `filter`, `reduce`, `compactMap` ile fonksiyonel koleksiyon işlemleri yapmak
6. `struct` vs `class` farkını derinlemesine anlamak ve karar verebilmek
7. Value type ve reference type davranışlarını tahmin edebilmek
8. `mutating` keyword'ünü doğru kullanmak
9. Copy-on-Write mekanizmasını bilmek

---

> **Sonraki Doküman:** [02 — Optionals](02%20-%20Optionals.md)

---

*Phase 1 · Doküman 1/6 · Şubat 2026*
