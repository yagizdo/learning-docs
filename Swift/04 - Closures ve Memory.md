# 📘 04 — Closures & Memory Management

> **Phase 1 · Doküman 4/6**  
> **Konu:** Closure syntax, trailing closures, @escaping, capture lists, [weak self], retain cycles  
> **Tahmini Süre:** 3–4 gün  
> **Ön Koşul:** [01](01%20-%20Swift%20Temelleri.md), [02](02%20-%20Optionals.md), [03](03%20-%20OOP%20ve%20POP.md)

---

## 📑 İçindekiler

1. [Closure Nedir?](#1-closure-nedir)
2. [Closure Syntax Kısaltmaları](#2-closure-syntax-kısaltmaları)
3. [Trailing Closure Syntax](#3-trailing-closure-syntax)
4. [@escaping vs Non-escaping](#4-escaping-vs-non-escaping)
5. [Capture Lists ve Capture Semantics](#5-capture-lists-ve-capture-semantics)
6. [Retain Cycles — Memory Leak'in Kaynağı](#6-retain-cycles--memory-leakin-kaynağı)
7. [[weak self] ve [unowned self]](#7-weak-self-ve-unowned-self)
8. [Gerçek Hayat: Retain Cycle Senaryoları ve Çözümleri](#8-gerçek-hayat-retain-cycle-senaryoları-ve-çözümleri)
9. [Ne Zaman [weak self] Gerekli, Ne Zaman Gereksiz?](#9-ne-zaman-weak-self-gerekli-ne-zaman-gereksiz)

---

## 1. Closure Nedir?

Closure, **isimsiz bir fonksiyondur** (anonymous function). Bir değişkene atanabilir, fonksiyona parametre olarak geçilebilir ve çevresindeki değişkenleri "yakalayabilir" (capture).

```swift
// Normal fonksiyon
func multiply(a: Int, b: Int) -> Int {
    return a * b
}

// Aynı mantığın closure versiyonu
let multiplyClosure: (Int, Int) -> Int = { (a: Int, b: Int) -> Int in
    return a * b
}

// Kullanım
multiply(a: 3, b: 4)     // 12
multiplyClosure(3, 4)      // 12

// Closure bir fonksiyona parametre olarak geçilebilir
func applyOperation(_ a: Int, _ b: Int, operation: (Int, Int) -> Int) -> Int {
    return operation(a, b)
}

let result = applyOperation(10, 5, operation: multiplyClosure)  // 50
```

---

## 2. Closure Syntax Kısaltmaları

Swift, closure syntax'ını kademeli olarak kısaltmana izin verir. Bu kısaltmaları bilmek önemlidir çünkü başkalarının kodunu okurken bunlarla karşılaşacaksın.

```swift
let numbers = [5, 2, 8, 1, 9, 3]

// ── Adım 1: Tam syntax ──
let sorted1 = numbers.sorted(by: { (a: Int, b: Int) -> Bool in
    return a < b
})

// ── Adım 2: Tür çıkarımı — derleyici türleri bilir ──
let sorted2 = numbers.sorted(by: { a, b in
    return a < b
})

// ── Adım 3: Tek ifadeli closure'da implicit return ──
let sorted3 = numbers.sorted(by: { a, b in a < b })

// ── Adım 4: Shorthand argument names ($0, $1, ...) ──
let sorted4 = numbers.sorted(by: { $0 < $1 })

// ── Adım 5: Operator method (en kısa) ──
let sorted5 = numbers.sorted(by: <)

// Hepsi aynı sonucu verir: [1, 2, 3, 5, 8, 9]
```

### Pratik Kullanımlar

```swift
let products = ["MacBook", "iPhone", "iPad", "AirPods"]

// map ile dönüştürme
let uppercased = products.map { $0.uppercased() }
// ["MACBOOK", "IPHONE", "IPAD", "AIRPODS"]

// filter ile eleme
let shortNames = products.filter { $0.count <= 4 }
// ["iPad"]

// reduce ile biriktirme
let combined = products.reduce("Ürünler: ") { $0 + $1 + ", " }
// "Ürünler: MacBook, iPhone, iPad, AirPods, "

// compactMap — nil'leri atıp unwrap et
let prices = ["99.99", "abc", "149.50", "N/A"]
let validPrices = prices.compactMap { Double($0) }
// [99.99, 149.5]
```

---

## 3. Trailing Closure Syntax

Fonksiyonun **son parametresi** bir closure ise, parantez dışına yazabilirsin. iOS geliştirmede her yerde karşılaşırsın.

```swift
// ── Trailing closure olmadan ──
let doubled = [1, 2, 3].map({ $0 * 2 })

// ── Trailing closure ile — daha okunabilir ──
let tripled = [1, 2, 3].map { $0 * 3 }

// ── Tek parametreli fonksiyon + trailing closure → parantez bile gereksiz ──
UIView.animate(withDuration: 0.3) {
    // animasyon bloğu
}

// ── Çoklu trailing closure (Swift 5.3+) ──
UIView.animate(withDuration: 0.3) {
    // animations
} completion: { finished in
    // animasyon bitti
}
```

---

## 4. @escaping vs Non-escaping

Bu ayrım, closure'un **ne zaman çalışacağı** ile ilgilidir ve memory management'ı doğrudan etkiler.

### Non-escaping (Varsayılan)

Closure, fonksiyon bitmeden önce çalışır ve fonksiyonla birlikte yok olur. Bellek riski yoktur.

```swift
func processItems(_ items: [String], transform: (String) -> String) -> [String] {
    return items.map(transform)  // Closure HEMEN çalışır
    // Fonksiyon bittiğinde closure da yok olur
}

let result = processItems(["hello", "world"]) { $0.uppercased() }
// transform closure'ı fonksiyon içinde çalıştı ve bitti — @escaping gerekmez
```

### @escaping — Closure Fonksiyondan "Kaçar"

Closure, fonksiyon **bittikten sonra** da yaşamaya devam eder. Üç ana senaryo:

```swift
class TaskManager {
    // ── Senaryo 1: Async işlem — closure daha sonra çağrılacak ──
    func fetchData(completion: @escaping (String) -> Void) {
        DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
            completion("Veri geldi!")  // 2 saniye sonra çağrılacak
            // fetchData fonksiyonu çoktan return etti
        }
    }
    
    // ── Senaryo 2: Closure bir property'de saklanıyor ──
    private var savedCallbacks: [() -> Void] = []
    
    func onComplete(callback: @escaping () -> Void) {
        savedCallbacks.append(callback)  // Closure saklanıyor
    }
    
    // ── Senaryo 3: Başka bir @escaping closure'a geçiriliyor ──
    func fetchAndProcess(completion: @escaping (String) -> Void) {
        fetchData { data in              // Bu closure @escaping, o yüzden:
            completion(data.uppercased())  // completion da @escaping olmalı
        }
    }
}
```

### @escaping Gerekmediğinde Yazmak Hata Değil, Ama Gereksiz

```swift
// ❌ Gereksiz @escaping — closure hemen çalışıyor
func process(_ items: [Int], filter: @escaping (Int) -> Bool) -> [Int] {
    return items.filter(filter)  // Aslında @escaping gerekmez
}

// ✅ Doğru — @escaping yok
func process(_ items: [Int], filter: (Int) -> Bool) -> [Int] {
    return items.filter(filter)
}
```

---

## 5. Capture Lists ve Capture Semantics

Closure'lar, tanımlandıkları scope'taki değişkenlere **referans** tutarlar. Buna "capture" (yakalama) denir.

### Value Type Capture'ı — Referans, Kopya Değil!

```swift
var counter = 0

let incrementer = {
    counter += 1  // counter'a REFERANS tutuyor
    print("Counter: \(counter)")
}

counter = 100     // counter dışarıda değişti

incrementer()     // "Counter: 101" — 0+1 değil, 100+1!
incrementer()     // "Counter: 102"

// Closure, counter'ın KENDİSİNE değil, counter'ın REFERANSINA bağlı
```

### Capture List ile Kopyalama

Capture list `[...]` syntax'ı ile değerin closure oluşturulduğu andaki **kopyasını** yakalayabilirsin:

```swift
var score = 10

let captureByReference = {
    print("Referans: \(score)")   // Mevcut değeri okur
}

let captureByValue = { [score] in
    print("Kopya: \(score)")       // Closure oluşturulduğu andaki değer: 10
}

score = 99

captureByReference()  // "Referans: 99" — güncel değer
captureByValue()      // "Kopya: 10"    — yakalandığı andaki değer
```

---

## 6. Retain Cycles — Memory Leak'in Kaynağı

Bu bölüm, iOS mülakat sorularının **en sık sorulanlarından** biridir.

### ARC Temelleri (Kısa Özet)

Swift, class instance'larının belleğini **ARC (Automatic Reference Counting)** ile yönetir. Her class instance'ının kaç tane güçlü referansı (strong reference) olduğu sayılır. Sayı 0'a düştüğünde instance bellekten silinir (deallocate).

```swift
class Person {
    let name: String
    init(name: String) {
        self.name = name
        print("✅ \(name) oluşturuldu")
    }
    deinit {
        print("🗑️ \(name) bellekten silindi")
    }
}

var person1: Person? = Person(name: "Ahmet")  // Referans sayısı: 1
var person2 = person1                           // Referans sayısı: 2

person1 = nil   // Referans sayısı: 1 — hâlâ yaşıyor
person2 = nil   // Referans sayısı: 0 → "🗑️ Ahmet bellekten silindi"
```

### Retain Cycle Nedir?

İki veya daha fazla nesne birbirine **güçlü referans** tuttuğunda, hiçbirinin referans sayısı 0'a düşemez ve bellekten silinmezler. Bu bir **memory leak**'tir.

```swift
class Employee {
    let name: String
    var department: Department?
    
    init(name: String) { self.name = name }
    deinit { print("🗑️ Employee \(name) silindi") }
}

class Department {
    let name: String
    var manager: Employee?  // ⚠️ Güçlü referans
    
    init(name: String) { self.name = name }
    deinit { print("🗑️ Department \(name) silindi") }
}

var emp: Employee? = Employee(name: "Zeynep")
var dept: Department? = Department(name: "Engineering")

emp?.department = dept   // Employee → Department (güçlü)
dept?.manager = emp      // Department → Employee (güçlü)
// DÖNGÜ: Employee ↔ Department

emp = nil    // Employee'in dış referansı kalktı AMA Department hâlâ tutuyor
dept = nil   // Department'ın dış referansı kalktı AMA Employee hâlâ tutuyor

// ❌ Hiçbir deinit çağrılmadı — MEMORY LEAK!
```

### Closure ile Retain Cycle

```swift
class ProfileViewController {
    var name = "Ahmet"
    var onUpdate: (() -> Void)?
    
    func setupCallback() {
        // ❌ RETAIN CYCLE
        onUpdate = {
            print("Profil güncellendi: \(self.name)")
            // self → onUpdate → closure → self (döngü!)
        }
    }
    
    deinit { print("🗑️ ProfileVC silindi") }
}

var vc: ProfileViewController? = ProfileViewController()
vc?.setupCallback()
vc = nil  // ❌ deinit ÇAĞRILMIYOR — memory leak
```

---

## 7. [weak self] ve [unowned self]

Retain cycle'ı kırmak için capture list'te `weak` veya `unowned` kullanılır.

### [weak self] — Güvenli Tercih

`weak` referans, nesneyi bellekte **tutmaz**. Nesne silinirse referans otomatik olarak `nil` olur. Bu yüzden `weak` referans her zaman **Optional** olur.

```swift
class ProfileViewController {
    var name = "Ahmet"
    var onUpdate: (() -> Void)?
    
    func setupCallback() {
        // ✅ [weak self] ile retain cycle kırıldı
        onUpdate = { [weak self] in
            guard let self else { return }  // self nil ise çık
            print("Profil güncellendi: \(self.name)")
        }
    }
    
    deinit { print("🗑️ ProfileVC silindi") }
}

var vc: ProfileViewController? = ProfileViewController()
vc?.setupCallback()
vc = nil  // ✅ "🗑️ ProfileVC silindi" — memory leak yok
```

### [unowned self] — Daha Riskli

`unowned` referans da nesneyi bellekte tutmaz, ancak Optional olmaz. Nesne silinmişse ve `unowned` referans kullanılırsa **crash** olur.

```swift
class Timer {
    var onTick: (() -> Void)?
    
    func start() {
        // Timer'ın self'ten ÖNCE yok olmayacağını biliyorsan:
        onTick = { [unowned self] in
            // self silinmişse → 💥 CRASH
            print("Tick")
        }
    }
}
```

### Karar Tablosu

| | `weak` | `unowned` |
|---|---|---|
| Referans tipi | Optional (`self?`) | Non-optional (`self`) |
| Nesne silinirse | nil olur, güvenli | 💥 CRASH |
| Kullanım | **Varsayılan tercih** | self kesinlikle closure'dan uzun yaşayacaksa |
| guard let gerekli mi? | Evet | Hayır |
| Risk | Düşük | Yüksek |

**Pratik Kural:** Emin değilsen **her zaman `[weak self]`** kullan. `unowned` sadece lifecycle ilişkisini kesinlikle kontrol edebildiğin nadir durumlarda kullan.

---

## 8. Gerçek Hayat: Retain Cycle Senaryoları ve Çözümleri

### Senaryo 1: Network Call

```swift
class ProductListViewModel {
    private let networkService: NetworkServiceProtocol
    var products: [Product] = []
    var onDataLoaded: (() -> Void)?
    
    init(networkService: NetworkServiceProtocol) {
        self.networkService = networkService
    }
    
    func loadProducts() {
        networkService.fetch(endpoint: "products") { [weak self] (result: Result<[Product], Error>) in
            guard let self else { return }
            
            switch result {
            case .success(let products):
                self.products = products
                self.onDataLoaded?()
            case .failure(let error):
                print("Hata: \(error)")
            }
        }
    }
    
    deinit { print("🗑️ ProductListViewModel silindi") }
}
```

### Senaryo 2: Timer / Animation

```swift
class CountdownViewController {
    var timer: Timer?
    var secondsRemaining = 60
    
    func startCountdown() {
        // ⚠️ Timer, closure'ı tutar, closure self'i tutar
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
            guard let self else { return }
            self.secondsRemaining -= 1
            print("Kalan: \(self.secondsRemaining)s")
            
            if self.secondsRemaining <= 0 {
                self.timer?.invalidate()
            }
        }
    }
    
    deinit {
        timer?.invalidate()
        print("🗑️ CountdownVC silindi")
    }
}
```

### Senaryo 3: Delegate Pattern (class-only protocol)

```swift
// Protocol'ü class-only yap (AnyObject) ki weak referans olabilsin
protocol DataManagerDelegate: AnyObject {
    func didLoadData(_ data: [String])
    func didFailWithError(_ error: Error)
}

class DataManager {
    weak var delegate: DataManagerDelegate?  // ← weak!
    // weak olmazsa: ViewController → DataManager → delegate → ViewController (döngü)
    
    func fetchData() {
        // simülasyon
        delegate?.didLoadData(["item1", "item2"])
    }
}
```

---

## 9. Ne Zaman [weak self] Gerekli, Ne Zaman Gereksiz?

Bu, sık sorulan bir mülakat sorusudur. Kurallar:

### ✅ [weak self] GEREKLİ

```swift
// 1. self bir closure property'sinde saklanıyorsa
class A {
    var callback: (() -> Void)?
    func setup() {
        callback = { [weak self] in /* ... */ }
    }
}

// 2. Async işlemlerde self'e referans varsa
class B {
    func load() {
        URLSession.shared.dataTask(with: url) { [weak self] data, _, _ in
            self?.process(data)
        }.resume()
    }
}

// 3. Timer, NotificationCenter, veya KVO gibi uzun ömürlü gözlemcilerde
class C {
    func observe() {
        NotificationCenter.default.addObserver(
            forName: .someNotification, object: nil, queue: .main
        ) { [weak self] _ in
            self?.handleNotification()
        }
    }
}
```

### ❌ [weak self] GEREKSİZ (veya Zararsız)

```swift
// 1. Non-escaping closure'larda — closure fonksiyon bitince yok olur
let filtered = items.filter { $0.isActive }  // weak self gerekmez

// 2. UIView.animate — Apple'ın implementasyonu retain cycle oluşturmaz
UIView.animate(withDuration: 0.3) {
    self.view.alpha = 0  // Güvenli — Apple closure'ı saklamaz
}

// 3. DispatchQueue (dikkatli ol — duruma bağlı)
// Eğer self zaten dışarıdan tutuluyorsa ve closure kısa ömürlüyse:
DispatchQueue.main.async {
    self.updateUI()  // Genellikle güvenli — ama uzun delay varsa [weak self] düşün
}

// 4. guard let self pattern'inden sonra
// guard let self'ten sonra self non-optional — güvenle kullanılabilir
someAsyncCall { [weak self] in
    guard let self else { return }
    // Buradan sonra self güvenli
    self.doSomething()
}
```

### Hızlı Karar Akışı

```
Closure @escaping mi?
├── HAYIR → [weak self] gerekmez
└── EVET → Closure self'e referans tutuyor mu?
    ├── HAYIR → [weak self] gerekmez
    └── EVET → self closure'ı (doğrudan veya dolaylı) tutuyor mu?
        ├── HAYIR → Gerekli değil ama zararsız (isteğe bağlı)
        └── EVET → ⚠️ RETAIN CYCLE! [weak self] ZORUNLU
```

---

## 📝 Bölüm Sonu Notları

Bu dokümanı tamamladıktan sonra şunları yapabilmelisin:

1. Closure syntax kısaltmalarını tanımak ve kullanmak
2. Trailing closure syntax'ını doğru kullanmak
3. `@escaping` vs non-escaping farkını açıklamak
4. Capture semantics'i anlamak (referans vs kopya capture)
5. Retain cycle'ın ne olduğunu ve nasıl oluştuğunu bilmek
6. `[weak self]` ve `[unowned self]` farkını bilmek
7. Hangi senaryolarda `[weak self]` gerekli olduğunu belirlemek

---

> **Sonraki Doküman:** [05 — Advanced Enums](05%20Advanced%20Enums.md)

---

*Phase 1 · Doküman 4/6 · Şubat 2026*
