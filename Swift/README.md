# 🔷 PHASE 1: Swift Dili Temelleri + OOP Güçlendirme

## Genel Bakış

> **Süre:** Hafta 1–3 · Günlük 1–2 saat  
> **Ön Koşul:** Temel programlama bilgisi (Flutter/Dart deneyimi avantaj)  
> **Hedef:** Swift dilinin temellerini sağlam bir şekilde öğrenmek, OOP bilgisini Swift üzerinden güçlendirmek ve Protocol-Oriented Programming paradigmasını kavramak.

---

## 📚 Doküman Haritası

| # | Doküman | İçerik | Tahmini Süre |
|---|---|---|---|
| 1 | [Swift Temelleri](01%20-%20Swift%20Temelleri.md) | Syntax, veri tipleri, koleksiyonlar, value vs reference types, struct vs class | 3–4 gün |
| 2 | [Optionals](02%20-%20Optionals.md) | Optional kavramı, `if let`, `guard let`, nil-coalescing, optional chaining, pattern matching | 2–3 gün |
| 3 | [OOP & POP](03%20-%20OOP%20ve%20POP.md) | Inheritance, polymorphism, encapsulation, abstraction, protocol-oriented programming, composition over inheritance | 4–5 gün |
| 4 | [Closures & Memory](04%20-%20Closures%20ve%20Memory.md) | Closure syntax, trailing closures, `@escaping`, capture lists, `[weak self]`, retain cycle'lar | 3–4 gün |
| 5 | [Advanced Enums](05%20-%20Advanced%20Enums.md) | Raw values, associated values, pattern matching, enum ile state management, recursive enums | 2–3 gün |
| 6 | [Error Handling, Generics & Access Control](06%20-%20Error%20Handling%20-%20Generics%20-%20Access%20Control.md) | `do-try-catch`, `Result` type, generic fonksiyonlar/tipler, protocol constraints, erişim seviyeleri | 3–4 gün |

---

## 🗓️ Önerilen Haftalık Plan

### Hafta 1: Dilin Temelleri
| Gün | Konu | Doküman |
|---|---|---|
| Gün 1–2 | Swift syntax, değişkenler, veri tipleri, koleksiyonlar | [01 — Swift Temelleri](01%20-%20Swift%20Temelleri.md) |
| Gün 3–4 | Value vs reference types, struct vs class, mutating | [01 — Swift Temelleri](01%20-%20Swift%20Temelleri.md) |
| Gün 5–7 | Optionals derinlemesine | [02 — Optionals](02%20-%20Optionals.md) |

### Hafta 2: OOP, POP ve Closures
| Gün | Konu | Doküman |
|---|---|---|
| Gün 8–10 | OOP prensipleri + POP temelleri | [03 — OOP & POP](03%20-%20OOP%20ve%20POP.md) |
| Gün 11–12 | POP ileri: composition, protocol extensions, mülakat hazırlığı | [03 — OOP & POP](03%20-%20OOP%20ve%20POP.md) |
| Gün 13–14 | Closures, @escaping, [weak self] | [04 — Closures & Memory](04%20-%20Closures%20ve%20Memory.md) |

### Hafta 3: Enum'lar, Error Handling, Generics
| Gün | Konu | Doküman |
|---|---|---|
| Gün 15–17 | Advanced Enums + pattern matching | [05 — Advanced Enums](05%20-%20Advanced%20Enums.md) |
| Gün 18–21 | Error handling, generics, access control + genel tekrar | [06 — Error Handling, Generics & Access Control](06%20-%20Error%20Handling%20-%20Generics%20-%20Access%20Control.md) |

---

## ✅ Phase 1 Tamamlama Checklist'i

Phase 2'ye geçmeden önce aşağıdaki her maddeyi işaretleyebildiğinden emin ol:

### Swift Temelleri
- [ ] `let` vs `var` farkını biliyorum, varsayılan olarak `let` kullanıyorum
- [ ] Type inference ve açık tür belirtiminin ne zaman gerektiğini biliyorum
- [ ] Array, Dictionary ve Set arasındaki farkları ve kullanım alanlarını biliyorum
- [ ] `struct` vs `class` farkını (value type vs reference type) açıklayabilirim
- [ ] Copy-on-Write mekanizmasını anlıyorum
- [ ] `mutating` keyword'ünü ne zaman kullanacağımı biliyorum

### Optionals
- [ ] Optional'ın ne olduğunu ve neden var olduğunu açıklayabilirim
- [ ] `if let`, `guard let` ve `??` operatörünü doğru senaryolarda kullanabiliyorum
- [ ] Optional chaining ile güvenli erişim yapabiliyorum
- [ ] Force unwrapping'in (`!`) neden tehlikeli olduğunu biliyorum
- [ ] `if let` vs `guard let` arasındaki farkı ve tercih kriterlerini biliyorum

### OOP & POP
- [ ] Inheritance, polymorphism, encapsulation ve abstraction'ı Swift örnekleriyle açıklayabilirim
- [ ] Protocol tanımlayabilir, protocol extension yazabilir, default implementation verebilirim
- [ ] "Composition over Inheritance" prensibini uygulayabilirim
- [ ] POP vs OOP farkını mülakat sorusu olarak cevaplayabilirim
- [ ] Protocol-based dependency injection yapabiliyorum

### Closures & Memory
- [ ] Closure syntax'ını biliyorum: trailing closure, shorthand arguments (`$0`, `$1`)
- [ ] `@escaping` vs non-escaping farkını biliyorum
- [ ] `[weak self]` ve `[unowned self]` arasındaki farkı biliyorum
- [ ] Retain cycle'ın ne olduğunu ve nasıl önleneceğini biliyorum

### Advanced Enums
- [ ] Raw values ve associated values arasındaki farkı biliyorum
- [ ] `switch` ile pattern matching yapabiliyorum
- [ ] Enum ile state management uygulayabiliyorum

### Error Handling, Generics & Access Control
- [ ] `do-try-catch` mekanizmasını kullanabiliyorum
- [ ] `Result` type ile hata yönetimi yapabiliyorum
- [ ] Generic fonksiyon ve tip yazabiliyorum
- [ ] Protocol constraint'leri (`where` clause) kullanabiliyorum
- [ ] 5 erişim seviyesini (`open`, `public`, `internal`, `fileprivate`, `private`) biliyorum

---

## 📚 Önerilen Kaynaklar

| Kaynak | Tür | Link | Not |
|---|---|---|---|
| The Swift Programming Language | Resmi Kitap | [swift.org](https://docs.swift.org/swift-book/) | Bedava, en güvenilir kaynak |
| 100 Days of Swift | Kurs | [hackingwithswift.com](https://www.hackingwithswift.com/100) | Proje bazlı, günlük ilerleme |
| Sean Allen | YouTube | [youtube.com/@seanallen](https://www.youtube.com/@seanallen) | Kısa, öz Swift/iOS videoları |
| Swift Playground | Uygulama | App Store'dan indir | İnteraktif pratik ortamı |

---

## 🎯 Phase 1 Final Projesi

Tüm dokümanları tamamladıktan sonra, aşağıdaki mini projeyi Swift Playground'da veya Xcode'da yap ve GitHub'a koy:

**Kütüphane Yönetim Sistemi (Console App)**

Gereksinimler:
- `Book`, `Member`, `Library` modelleri (`struct` kullan)
- `Searchable`, `Persistable`, `Displayable` protocol'leri (POP)
- Kitap ekleme/silme/arama (generics + protocol constraint)
- Ödünç alma/iade sistemi (enum ile state management)
- Hata yönetimi (`throws` + `Result` type)
- Proper access control (`private`, `internal`)
- Closure kullanımı (sorting, filtering, completion handlers)

Bu proje, Phase 1'deki tüm konuları tek bir çatı altında pekiştirir.

---

> **Sonraki Phase:** [Phase 2 — UIKit Temelleri: Programmatic UI](../phase2/00_Phase2_Index.md)

---

*Bu doküman serisi, Flutter/Dart deneyimli geliştiricilerin iOS (Swift) ekosistemine geçişi için hazırlanmıştır. Şubat 2026.*
