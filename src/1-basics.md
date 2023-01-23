# Merhaba Dünya
"Merhaba Dünya"'nın esas amacı, C'nin ilk versiyonu yazıldığından beri, derleyiciyi test etmek ve gerçek bir program çalıştırmaktır.

```rust
// hello.rs
fn main() {
    println!("Hello, World!");
}
```
```bash
$ rustc hello.rs
$ ./hello
Hello, World!
```
Rust'ta süslü ayraçlar ve noktalı virgül vardır, C++ tarzı yorum satırları bulunur ve bir de `main` fonksiyonu bulunur. Şimdiye kadar bu kısmı tanıyorsunuz. Ünlem işareti, bunun bir *makro* çağrısı olduğunu gösterir. C++ programcıları için bu biraz caydırıcı olabilir, çünkü onların tek bildiği makrolar o abuk subuk C makrolarıdır - ama bu makroların çok daha yetenekli ve kabul edilebilir olduğunu rahatlıkla söyleyebilirim.

"Güzel de bu ünlem işaretini nereye sıkıştıracağımı nereden bileyim" diye aklından geçirenler olmuştur. Ancak derleyici beklemediğiniz kadar yardımsever; eğer ünlem işaretini unutursanız şunu görürsünüz:
```rust
error[E0425]: unresolved name `println`
 --> hello2.rs:2:5
  |
2 |     println("Hello, World!");
  |     ^^^^^^^ did you mean the macro `println!`?
```

Bir dili öğrenmek o dilin hatalarıyla barışık olmak demektir. Derleyiciyi sizi *azarlayan* bir bilgisayar olarak görmek yerine katı ama dostane davranan bir yardımcı olarak görmeye çalışın, çünkü başlangıçta epeyce kırmızı yazılar göreceksiniz. Derleyicinin sizin hatalarınızı yüzünüze vurması, insanların sizin yüzünüze vurmasından kat kat daha iyidir.

Bir sonraki aşama *değişken* atamaktır.

```rust
// let1.rs
fn main() {
    let answer = 42;
    println!("Hello {}", answer);
}
```

Yazım hataları *derleme* zamanında anlaşılır, Python ya da JavaScript gibi çalışma zamanını beklemenize gerek yoktur. Bu, sizi daha sonra pek çok stresten kurtaracak! Eğer "answer" yerine "answr" yazarsam, derleyici bu konuda epey kibar davranır:

```rust
4 |     println!("Hello {}", answr);
  |                         ^^^^^ did you mean `answer`?
```

`println!` makrosu bir [format karakter dizesi](https://doc.rust-lang.org/std/fmt/index.html) alır; Python3'te kullanılan formatlama stiline epey benzerdir.

Bir başka kullanışlı makro ise `assert_eq!`. Bu Rust testlerinin direğidir, iki şeyin birbirine eşit olduğu *varsayarsınız. (assert = varsaymak)* Eğer eşit değillerse, *panik*.

```rust
// let2.rs
fn main() {
    let answer = 42;
    assert_eq!(answer,42);
}
```

Herhangi bir çıktı olmayacaktır. Ancak 42'yi 40 ile değiştirirseniz:

```shell
thread 'main' panicked at
'assertion failed: `(left == right)` (left: `42`, right: `40`)',
let2.rs:4
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

Ve bu bizim Rust'taki karşımıza çıkan ilk *çalışma zamanı hatası*.

# Döngüler ve Koşullamalar
Enteresan olan her şey tekrar tekrar yapılabilir:

```rust
// for1.rs
fn main() {
    for i in 0..5 {
        println!("Hello {}", i);
    }
}
```

*Aralık (Range)* kapsayıcı değildir, bundan dolayı `i`'nin değeri 0 ila 4 arasında değişir.  Dizilerin indekslerinin sıfırdan başladığı bir dilde pek de olağandışı değildir.

Enteresan şeyler de *koşula bağlı olarak* da gerçekleştirilebilir.

```rust
// for2.rs
fn main() {
    for i in 0..5 {
        if i % 2 == 0 {
            println!("even {}", i);
        } else {
            println!("odd {}", i);
        }
    }
}
```
```
even 0
odd 1
even 2
odd 3
even 4
```

`i % 2` eğer i, 2'ye tam olarak bölünebiliyorsa sıfır olur; Rust C-tarzı operatörler kullanır. Koşulların etrafında parantez yoktur, tıpkı Go'daki gibi, ama blokların etrafında süslü parantezlerin kullanımı *zorunludur.*

Aynı şey, daha da ilginç bir yoldan yapılabilir:
```rust
// for3.rs
fn main() {
    for i in 0..5 {
        let even_odd = if i % 2 == 0 {"even"} else {"odd"};
        println!("{} {}", even_odd, i);
    }
}
```

Klasik olarak, programlama dillerinde *deyimler (statement)* (`If` gibi) ve *ifadeler (expression)* (`1+i` gibi) bulunur. Rust'ta, her şeyin değeri olabilir ve bunlar bir ifade olabilir. C'nin o garabet "ternary/üçlü operatörüne" burada ihtiyacımız yok.

Aynı zamanda bloklarda noktalı virgül olmadığına da dikkat edin!

# Şeyleri Şeylere Eklemek
Bilgisayarlar aritmatik konusunda epey iyidir. 0'dan 4'e kadar bütün sayıları toplamayı deneyelim.

```rust
// add1.rs
fn main() {
    let sum = 0;
    for i in 0..5 {
        sum += i;
    }
    println!("sum is {}", sum);
}

```

Ama derlenirken hata verecektir:

```shell
error[E0384]: re-assignment of immutable variable `sum`
 --> add1.rs:5:9
3 |     let sum = 0;
  |         --- first assignment to `sum`
4 |     for i in 0..5 {
5 |         sum += i;
  |         ^^^^^^^^ re-assignment of immutable variable

```

"Immutable"? Değişemeyen değişen mi? `let` değişkenlerinin değeri sadece atanırken belirtilebilir. `mut` ismindeki sihirli sözlük (*nolur* bu değişkeni değişebilir yap) işi halledecektir:

```rust
// add2.rs
fn main() {
    let mut sum = 0;
    for i in 0..5 {
        sum += i;
    }
    println!("sum is {}", sum);
}
```

Değişkenlerin varsayılan olarak yeniden yazılabilir olduğu dillerden geçerken bu biraz kafa karıştırıcı olabilir. Bir şeyi *değişken* yapan şey onun değerinin çalışma zamanında atanmasıdır, sabitlerin (constant) aksine. Bu kavramlar matematikte de kullanılır, mesela "let n be the largest number in set S (N'i S kümesi içerisindeki en büyük değer yap)" derken.

Değişkenlerin varsayılan olarak *salt okunur* olmasının ardında bir neden vardır. Büyük bir programda, değerlerin nerede atandığını bulmak oldukça güçleşebilir. Bundan dolayı Rust, değişimlerin bildirilmesini ister. Dilde zekice epey şey var ancak dil hiçbir şeyin örtük kalmamasına ayrıca özen gösteriyor.

Rust hem statik hem de güçlü tiplenen bir dildir - bu kavramlar genelde karıştırılır, ama C (statik ama zayıf tiplenen) ve Python'u (dinamik ama güçlü tiplenen) göz önüne getirin. Statik tiplemede tip derleme zamanında bilinir, dinamik tiplemede ise çalışma zamanında.

Tam da bu anda, Rust'ın sizden tipleri gizlediğini sezebilirsiniz. Mesela `i`'nin tam değeri nedir? Derleyici için bu sorun değildir, 0'dan başlarken, tip çıkarımı ile (*type referance*) bu sayılar `i32` (Dört bitlik işaretli tam sayı) oluverir.

Hadi net bir değişim yapalım, 0'ı 0.0 ile değiştirip hataları görelim:
```shell
error[E0277]: the trait bound `{float}: std::ops::AddAssign<{integer}>` is not satisfied
 --> add3.rs:5:9
  |
5 |         sum += i;
  |         ^^^^^^^^ the trait `std::ops::AddAssign<{integer}>` is not implemented for `{float}`
  |

```

Pekâlâ, şimdi güldük eğlendik ama bu da nesi? Bütün operatörler (Mesela `+=`) bir özelliğe (trait) denk gelir ki özellik (trait) somut tiplere yeni özellikler ekleyen soyut arabirimlerdir. Özelliklerle daha sonra ilgileneceğiz, ama burada bilmeniz gereken bütün şey `AddAssign`, `+=` operatörünü sağlayan özelliğin adı olduğudur ve hata mesajının demek istediği şey bu özelliğin noktalı sayılara bu operatör tam sayılarla işlem yapmak için uygulanmadığıdır. (Operatör özelliklerinin tam listesi [burada](https://doc.rust-lang.org/std/ops/index.html).)

Rust'ta her şey bellidir - sırf sizin gönlünüz olsun diye tam sayıyı noktalı sayıya gizlice çevirmeyecektir.

```rust
// add3.rs
fn main() {
    let mut sum = 0.0;
    for i in 0..5 {
        sum += i as f64;
    }
    println!("sum is {}", sum);
}
```

# Fonksiyonların Tipleri de Apaçık Ortadadır 
Fonksiyonlar da derleyicinin sizin için tipleri tahmin etmekle uğraşmayacağı yerlerden birisidir. Aslında bu üzerinde düşünülerek alınmış bir karardır çünkü Haskell gibi güçlü tip çıkarımlarına sahip dillerde tip isimleri nadiren yazılır. Aslında Haskell için tipleri açıkça yazmak iyi yaklaşımdır. Rust ise her zaman bunu mecbur tutar.

İşte tanımladığımız basit bir fonksiyon:

```rust
// fun1.rs

fn sqr(x: f64) -> f64 {
    return x * x;
}

fn main() {
    let res = sqr(2.0);
    println!("square is {}", res);
}
```

Rust biraz eski bir argüman bildirimi tarzı kullanmakta, tip isimden sonra gelir. Bu, Pascal gibi Algol'dan türemiş dillerde kullanılan tarzdır.

Hatırlatalım, tam sayı noktalı sayıya dönüşmez - eğer `2.0`'ı `2` ile değiştirirseniz nurtopu gibi bir hatanız olmuş olur:
```rust
8 |     let res = sqr(2);
  |                   ^ expected f64, found integral variable
  |
```

Rust'da fonksiyonlarda çok az `return` deyiminin kullanıldığının görürsünüz. Daha çok, şuna benzer ifadeler vardır:

```rust
fn sqr(x: f64) -> f64 {
    x * x
}
```

Fonksiyonun gövdesi (`{ }` içi) tıpkı "ifade olarak kullanılan if"teki gibi son ifadenin değerini alır.

Noktalı virgülleri refleks olarak kazara ekleyebilirsiniz ve o zaman şöyle bir hata alırsınız:

```rust
  |
3 | fn sqr(x: f64) -> f64 {
  |                       ^ expected f64, found ()
  |
  = note: expected type `f64`
  = note:    found type `()`
help: consider removing this semicolon:
 --> fun2.rs:4:8
  |
4 |     x * x;
  |       ^

```

`()` tipi boş tiptir, yokluktur, `void`dir, "nothing"dir, tasavvuftaki fakrdır. Rust'ta her şeyin değeri vardır, ama bazen sadece yoktur. Derleyici bunun sıkça karşılaşılan bir durum olduğunu bilir, ve size aslında *yardım* eder. (C++ derleyicileriyle vakit harcamış zavallı ruhlar bunun ne kadar faydalı olduğunun farkındadır.)

`Return` kullanılmayan ifadelere biraz daha örnek verelim: 
```rust
// absolute value of a floating-point number
fn abs(x: f64) -> f64 {
    if x > 0.0 {
        x
    } else {
        -x
    }
}

// ensure the number always falls in the given range
fn clamp(x: f64, x1: f64, x2: f64) -> f64 {
    if x < x1 {
        x1
    } else if x > x2 {
        x2
    } else {
        x
    }
```

`Return` kullanmak yanlış değil, ama kod onsuz daha temiz. Yine de, bir fonksiyondan *erken dönmek* için `return` kullanabilirsiniz. 

Bazı işlemler zarif bir yoldan özyinelemeli olarak yazılabilir:
```rust
fn factorial(n: u64) -> u64 {
    if n == 0 {
        1
    } else {
        n * factorial(n-1)
    }
}
```

Başta tuhaf görünebilir ve en iyisi kağıt kalemle örnekler üzerinde düşünmektir. Ancak, bir işlemi yapmanın en etkili yolu değildir.

Değerler aynı zamanda *referans* olarak da iletilebilir. `&` ile yaratılmış bir referans `*` *dereferans* edilebilir. (Ç.N: De- olumsuzlaştırma öneki)

```rust
fn by_ref(x: &i32) -> i32{
    *x + 1
}

fn main() {
    let i = 10;
    let res1 = by_ref(&i);
    let res2 = by_ref(&41);
    println!("{} {}", res1,res2);
}
// 11 42
```

Bir fonksiyonun argümanlarını değiştirebilmesini mi istiyorsunuz? *Değişebilir referans (Mutable referance)* kullanın:

```rust
// fun4.rs

fn modifies(x: &mut f64) {
    *x = 1.0;
}

fn main() {
    let mut res = 0.0;
    modifies(&mut res);
    println!("res is {}", res);
}
```

Bu C++'dan çok C'ye benzedi. Açıkça referansı (`&` ile) belirtmelisiniz ve aynı şekilde `*` ile deferans etmelisiniz. Sonra da `mut`'u ekleyin çünkü varsayılan değişebilir değiller. (Bana hep C++ referansları C'ye göre gözden kaçırılmaya daha müsaitmiş gibi gelir.)

Temel olarak, Rust burada biraz bizi *yoruyor* ve fonksiyonlardan değer döndürmeye zorluyor.  Neyse ki, Rust'ın "işlem başarılı, bu da sonucu" gibi güçlü ifadeleri olduğundan `&mut`'u sıklıkla kullanmayız. Referans kullanmak, büyük bir nesnemiz olduğunda ve onu kopyalamak istemediğimizde dikkate değerdir. 

"Değişkenden sonra tip gelir" tarzı `let` için de gayet uyuyor, bir değişkenin türünü belirtmek istersek eğer:

```rust
let bigint: i64 = 0;
```


# Yolumuzu Yordamımızı Bilmek
Şimdi belgelendirmeye bakmanın tam zamanı. Belgeler makinenize yüklenmiş olmalı ve onu `rustup doc --std` komutu ile tarayıcınızda açabilir olmalısınız.

Arama kutucuğunun en üstte olduğuna dikkat edin, zira bu sizin en yakın dostunuz olacak; çalışmak için İnternet'e gerek duymaz.

Diyelim ki matematiksel fonksiyonların nerede olduğunu merak ediyorsunuz, "cos" diye aratmanız yeterli. Klavyede dokunduğunuz ilk iki tuş her iki noktalı sayı tipi için de var olduğunu gösterir. Aradığımız işlem, değerin kendisinde metot olarak tanımlıdır, mesela şöyle:

```rust
let pi: f64 = 3.1416;
let x = pi/2.0;
let cosine = x.cos();
```

Sonuç sıfıra epey yakın çıkacaktır, belli ki tahmini değere değil gerçek PI sayısına ihtiyacımız var.

(Sahi, neden `f64` diye belirtmemize gerek var ki? Aslına bakarsanız o olmadan değerimiz `f32` veya `f64` olabilir ki bunlar epey farklı şeyler.)
(Ç.N: Noktalı sayı tutan)

`Cos` için verilen örneğe bakalım, bunu çalışabilir bir programa çevirdik. (`assert!` de `assert_eq!`'in amcaoğlu oluyor, verilen ifade kesinlikle doğru olmalıdır.)

```rust
fn main() {
    let x = 2.0 * std::f64::consts::PI;

    let abs_difference = (x.cos() - 1.0).abs();

    assert!(abs_difference < 1e-10);
}
```

`std::f64::consts::PI` şu güzel ortamı iyice bozdu! `::` C++'daki anlamıyla aynı şeye denk geliyor, (Bazı dillerde yerine `.` kullanılır) - bu *tam yolu belirtilmiş bir isim*. Bu tam adı, `PI` için yaptığımız aramayı yaparken ikinci klavye tuşlamasında alıyoruz. 

Şimdiye dek, bizim ufak Rust programımıza "Merhaba Dünya" tartışmalarında gündemi meşgul eden `import` ya da `include` gibi şeyleri eklemedik. Hadi, programımızı bir `use` deyimi ile şenlendirelim:

```rust
use std::f64::consts;

fn main() {
    let x = 2.0 * consts::PI;

    let abs_difference = (x.cos() - 1.0).abs();

    assert!(abs_difference < 1e-10);
}
```

Tamam da buna neden şimdiye dek ihtiyaç duymadık? Çünkü Rust *prelude* aracılığıyla, `use` deyimini kullanmaya gerek bırakmadan pek çok temel işlevi görünür kılar (ama siz kullanana kadar yüklemez).


# Diziler ve Dilimler
Bütün statik tiplenen dillerde *diziler (array)* bulunur, bu birden çok veriyi bellek içerisinde baştan sona kontrol eder. Diziler, sıfırdan itibaren *indekslenir*.

```rust
// array1.rs
fn main() {
    let arr = [10, 20, 30, 40];
    let first = arr[0];
    println!("first {}", first);

    for i in 0..4 {
        println!("[{}] = {}", i,arr[i]);
    }
    println!("length {}", arr.len());
}
```

Ve çıktı:

```
first 10
[0] = 10
[1] = 20
[2] = 30
[3] = 40
length 4
```

Burada Rust dizinin büyüklüğünü net olarak bilir ve eğer `arr[4]`'e erişmeye çalışırsanız derleme hatası alırsınız. 

Yeni bir dil öğrenmek aynı zamanda diğer dillerden edindiğiniz alışkanlıkları da *terk etmek* demektir; eğer bir Pythonista iseniz bu köşeli parantezleri `List` diye isimlendirebilirsiniz. Rust'taki `List`'in muadiline daha sonra bakacağız, ancak diziler düşündüğünüz işi yapmıyor; *sabit* bir büyüklükleri vardır. (Eğer yalvarırsak) *değişebilir*ler ancak yeni değerler ekleyemeyiz.

Diziler Rust'ta o kadar çok kullanılmaz, çünkü her dizi tipi uzunluğunun bilgisini de taşır. Mesela `[i32; 4]` dizi tipine bakabilirsiniz; aynı zamanda `[10, 20]` olan bir dizinin tipi de `[i32; 2]` olacaktır vs, hepsinin farklı tipi vardır. Yani bunlar aslında fonksiyon argümanı olmaktan başka şeye yaramayan başıboş serserilerdir.

Esas sık kullanılan*lar* *dilimlerdir*. Bunları bir dizinin *parçalanmış hâli[^slice]* olarak düşünebilirsiniz. Tıpkı dizilerin davrandığı gibi davranırlar ve *uzunluklarını bilirler*, C'deki *gösterici (pointer)* denen korkunç yaratıkların tam tersi olarak.

İki önemli noktaya dikkat edin - bir dilimin tipi nasıl yazıldığına ve fonksiyona ne zaman `&` eklemeniz gerektiğine.

[^slice]: Ç.N: Esas çeviride "parçalanmış hâl" yerine "görünüm (view)" kelimesi kullanılıyor. İngilizce için cümle gayet geçerli, ancak Türkçe'de tuhaf duruyor.

```rust
// array2.rs
// read as: slice of i32
fn sum(values: &[i32]) -> i32 {
    let mut res = 0;
    for i in 0..values.len() {
        res += values[i]
    }
    res
}

fn main() {
    let arr = [10,20,30,40];
    // look at that &
    let res = sum(&arr);
    println!("sum {}", res);
}
```

`Sum`'da kodu bir saniyeliğine görmezden gelin ve `&[i32]`'ye bakın. Rust dizileri ve dilimleri arasındaki ilişki C'deki diziler ve göstericiler arasındaki ilişkiye benzer, iki detay hariç - Rust dilimleri kendi uzunluğunun takibini yaparlar (ve eğer bu uzunluğun dışına çıkarlarsa paniklerler) sonra da `&` 'ı operatörünü kullanarak dilim olarak kullanmak istediğinizi açıkça belirtmeniz gereklidir.

Bir C programcısı `&`'ı gördüğü zaman "falancanın adresi" diye okur, Rust programcısı ise "ödünç (borrow)" olarak. Bu, Rust öğrenirken dikkat etmeniz gereken kilit sözcüktür. Ödünç alma, esasında programlamada kullanılan genel bir terimdir ve (Dinamik dillerde her zaman olduğu gibi) referans olarak bir veriyi ya da C'de bir gösterici (pointer) yollamanıza denir. Ödünç alınan her şey esas sahibinde kalır.

# Dilimleme ve Biçme
Bir diziyi `{}` yolu ile ekrana yazamazsınız fakat *hata ayıklama (debug)* yani `{:?}` ile ekrana yazdırabilirsiniz.

Bu: 

```rust
// array3.rs
fn main() {
    let ints = [1, 2, 3];
    let floats = [1.1, 2.1, 3.1];
    let strings = ["hello", "world"];
    let ints_ints = [[1, 2], [10, 20]];
    println!("ints {:?}", ints);
    println!("floats {:?}", floats);
    println!("strings {:?}", strings);
    println!("ints_ints {:?}", ints_ints);
}
```

Bunu yazdırır:

```rust
ints [1, 2, 3]
floats [1.1, 2.1, 3.1]
strings ["hello", "world"]
ints_ints [[1, 2], [10, 20]]
```

Bu arada, dizilerin dizileri de olabilir ancak dizide sadece bir tipten değerler bulunmalıdır. Dizideki değerler verimlilikten dolayı bellekte yanyana bulunurlar ki bu erişim için oldukça faydalıdır.

Eğer bir değişkenin gerçek tipini merak ediyorsanız, size bir hile gösterebilirim. Bir değişkeni, geçersiz olduğunu bildiğiniz bir tiple bildirin:

```rust
let var: () = [1.1, 1.2];
```

İşte aradığınız şeyi gösteren hata:

```
3 |     let var: () = [1.1, 1.2];
  |                   ^^^^^^^^^^ expected (), found array of 2 elements
  |
  = note: expected type `()`
  = note:    found type `[{float}; 2]`
```

(`{float}`, "bir nevi noktalı sayı ama tam tipi henüz belirtilmedi." demektir.)

Dilimler size aynı dizinin farklı *parçalarını* sunar:

```rust
// slice1.rs
fn main() {
    let ints = [1, 2, 3, 4, 5];
    let slice1 = &ints[0..2];
    let slice2 = &ints[1..];  // open range!

    println!("ints {:?}", ints);
    println!("slice1 {:?}", slice1);
    println!("slice2 {:?}", slice2);
}
```

Bu, Python'daki dilim anlayışına epey yakındır ancak arada büyük bir fark vardır: Veri asla kopyalanmadı. Bu dilimler, bütün verilerini dizilerden *ödünç* alırlar. Dilimlerin dizilerle epey sıkı bir bağ vardır ve Rust bu bağın kopmaması için elinden gelen her şeyi kuvvetle yapar.

# Opsiyonel Değerler (Optional Values)
Dilimler, diziler gibi, indekslenebilir. Rust, derleme zamanında dizinin değerini bilir, ama bir dilimin değeri ancak çalışma zamanında bilinebilir. Bundan dolayı, `s[i]` kullanımı belleğin yanlış bir yerine erişmeye sebep olabilir ve bu durumda program *panikleyecektir.* Bu gerçekten de olmasını istediğiniz şey değildir - aradaki fark Florida'dan atılacak çok pahalı bir uydunun gökyüzünde parçalanması ile atışın güvenlice iptal edilmesine kadar varabilir. Ve burada *hata yakalama mekanizmaları (exceptions)* yok.

Şimdi hazır olun zira gelecek şey sizi şok edecek. Burada panikleyebilecek kodları try-bloğu ve hatayı yakala (try - catch) yapısı yok - en azından her gün kullandığınız şekliyle yok. Peki, Rust nasıl güvenli kalabiliyor?

İşte size bir paniklemeyen `get` metotu. İyi de, bu ne dönüyor?

```rust
// slice2.rs
fn main() {
    let ints = [1, 2, 3, 4, 5];
    let slice = &ints;
    let first = slice.get(0);
    let last = slice.get(5);

    println!("first {:?}", first);
    println!("last {:?}", last);
}
// first Some(1)
// last None
```

`last` çuvalladı. (Sıfır temelli indekslemeyi unuttuk.) Ama `None` diye bir şey döndü. `first` için sorun yok, ama `Some` diye bir şey dönüverdi. `Option` tipini selamlayın! Bu tip `Some` *olabilir*, `None` *olabilir*.

`Option` tipinin gayet faydalı metotları vardır:

```rust
    println!("first {} {}", first.is_some(), first.is_none());
    println!("last {} {}", last.is_some(), last.is_none());
    println!("first value {}", first.unwrap());

// first true false
// last false true
// first value 1
```

Eğer `last` üzerinde *unwrap* kullanırsanız nurtopu gibi bir paniğiniz olur. Bunun yerine en azından `is_some` kullanabilirsiniz - varsayılan bir değeriniz varsa gayet faydalıdır:

```rust
    let maybe_last = slice.get(5);
    let last = if maybe_last.is_some() {
        *maybe_last.unwrap()
    } else {
        -1
    };
```
`*` operatörünün kullanıldığına dikkat edin - `Some` içindeki esas tip bir referans olan `&i32`'dir. `i32` verisini almak için veriyi deferans ediyoruz.

Bu biraz işi uzatıyor, onun yerine bir kısayol kullanabiliriz - `unwrap_or` metodu `Option` `None` ise yerine bir değer atayabilir. Tipler muhakkak uyuşmalı -`get` referans dönecek. Bundan ötürü bir `&i32` olan `&-1`'ı kullanmamız gereklidir. Şimdi tekrar  `*` operatörünü `i32` değeri almak için kullanalım.

```rust
    let last = *slice.get(5).unwrap_or(&-1);
```

`&`'ı unutmak gayet olası ama derleyici arkanızı toplayacaktır. `-1` yazsaydık, `rustc` şuna benzer bir hata verecekti: "&{integer} bekleniyordu ancak tam sayı alındı" ve eklerdi ki: "yardım: `&-1`'ı deneyin" \*

\* "expected &{integer}, found integral variable", "help: try with `&-1`"

`Option` tipini veri taşıyan bir paket olarak zihninizde canlandırabilirsiniz, ya da hiçbir şey ifade etmeyen bir değer (`None`). (Haskell'deki karşılığı `Maybe`dir.) Bu tip tipi belirtilebilen herhangi bir veriyi barındırabilir. Bizim örneğimizde üzerinde çalıştığımız tip `Option<&i32>`'dir, C++'ın "Genellemeler (generics)" yazılımı ile gösterirsek. Paketi açmak bir patlamaya sebep olabilir ancak bu mevzu Schrödinger'in kedisi kadar karmaşık değil ve önceden paketin içinde ne var bilebiliriz.  

Rust fonksiyonlarının ve metotların bu tür paketleri döndürmesi gayet olağandır ve üzerinde uzmanlaşana kadar nasıl kullanıldığını öğrenin.

# Vektörler
Ç.N: Doğrusunu isterseniz ilk gördüğümde bu vektörleri geometrideki vektörlerle ilişkili zannetmiştim. Yüzde yüz programlama deyimi, aklınıza farklı şeyler gelmesin. :)

Tekrar dilim metotlarına döneceğiz ancak vektörleri gözden geçirelim. Bunlar *yeniden biçimlendirilebilen* dizilerdir ve Python'un `List`ine ve C++'ın `std::vector`'üne epey benzerler. Rust'ın `Vec` tipi ("vektör" olarak okunur.) dilimlere çok benzerler; esas farklılıkları ise vektöre yeni bir veri ekleyebiliyor olmanız - *değişebilir (mutable)* olarak değişkenin bildirilmesi kaydı ile.

```rust
// vec1.rs
fn main() {
    let mut v = Vec::new();
    v.push(10);
    v.push(20);
    v.push(30);

    let first = v[0];  // will panic if out-of-range
    let maybe_first = v.get(0);

    println!("v is {:?}", v);
    println!("first is {}", first);
    println!("maybe_first is {:?}", maybe_first);
}
// v is [10, 20, 30]
// first is 10
// maybe_first is Some(10)
```

Yeni başlayanların başına sıklıkla gelen şey `mut` eklemeyi unutmalarıdır; bunu yaparsanız dostça uyarılırsınız.

```
3 |     let v = Vec::new();
  |         - use `mut v` here to make mutable
4 |     v.push(10);
  |     ^ cannot borrow mutably
```

Vektörler ve dilimler arasında çok yakın bir bağ vardır.

```rust
// vec2.rs
fn dump(arr: &[i32]) {
    println!("arr is {:?}", arr);
}

fn main() {
    let mut v = Vec::new();
    v.push(10);
    v.push(20);
    v.push(30);

    dump(&v);

    let slice = &v[1..];
    println!("slice is {:?}", slice);
}
```

Ufak ama önemli ödünç alma operatörümüz `&`, vektörü dilime çevirmeye *zorluyor. (coercing)*. Ve bu pek mantıksız değil çünkü vektörler bellekte *dinamik* bir yer tutarlar ve dizi gibi çalışırlar.

Eğer dinamik tipli bir dilden geliyorsanız, sizinle bazı şeyleri konuşmanın vakti geldi. Sistem programlama dillerinde iki farklı bellek yönetim tarzı vardır:  Yığıt (Stack) ve Öbek (Heap).[^heapstack] Stack bellek üzerinde oldukça hızlı bir şekilde alan tahsis ederler ancak yapıları ancak bir kaç megabaytla çıkabilecek kadar sınırlıdır. Heap ise gigabaytlara kadar çıkabilir ancak alan tahsis etme süreci biraz meşakkatlidir ve bu bellek alanının sonradan temizlemesi gereklidir. Bazı sözüm ona "yönetilen (managed)" dillerde (Bunlar Java olur, Go olur, bazı sözde betik dilleri olur) bu tarz detaylar sizden gizlenir ve belediyemizin *çöp toplayıcıları (garbage collector)* tarafından bu pis işler halledilir. Sistem, bir verinin başka bir veriye referans gösterilmediğine emin olunca kullanılabilir bellek alanına geri döner.

[^heapstack]: Yığıt ve Öbek, benim çeviri standartlarıma göre bile aşırı yapay kalıyor. Bundan ötürü kafa karışıklığını ortadan kaldırmak için Heap ve Stack kelimelerinden devam ettim. 

İşin özü bu durumun faydaları olsa da bazı sorunları da vardır. Stack ile oynamanın bazı tehlikeleri var ve içinde bulunduğunuz fonksiyonun dönüş adresini bozabilirsiniz, sonra da iğrenç bir şekilde can verirsiniz. Ya da daha da kötüsü, Hacker Okan'ın elini öpmek zorunda kalabilirsiniz.

İlk C programım (DOS'ta yazmıştım) tüm bilgisayarı çökertmişti. Unix sistemleri bu tarz şeylere karşı daha iyi tavır alırdı ve *segfault* mekanizması ile kontrolden çıkan süreçler "öldürülür". Peki, bu neden Rust'ın (ya da Go'nun) paniklemesinden daha kötüdür? Çünkü panik sorunun olduğu yerde meydana gelir, bütün program birbirine girdiğinde ve ev ödevlerine dadandığında değil. Panikler *bellek için emniyetlidir (memory safe)* çünkü belleğin canına okunmadan hemen önce gerçekleşirler. Bu, C'deki güvenlik sorunlarının yaygın bir nedenidir çünkü bütün bellek erişimleri emniyetsizdir ve işi bilen bir saldırgan bu güvensizlikten faydalanabilir.

Panikler kulağınıza korkunç ve plansız gelebilir ama Rust'ın panikleri bile yapılandırılmıştır - stack tek tek serbest bırakılır. Bellekte tahsis edilmiş alanı olan bütün veriler boşatılır ve geriye dönük bir rapor oluşturulur.

Peki çöp toplayıcıların (garbage collector) dezavantajları nedir? Birincisi belleği çok hoyratça kullanıyorlar, sizin için önemli olmayabilir ama gömülü mikroçiplerde bu çok fena bir sorun oluşturur. İkincisi, en olur olmaz zamanlarda belleği temizlemeye başlamasıdır. (Odanızda uzanmış telefonda sevgilinizle hassas bir konuşma yaparken birden odanızı temizlemeye kalkışan annenizi düşünün.) Gömülü sistemlerin olaylara *gerçekleştiği anda* yanıt vermesi gerekir ve planlanmamış bir temizliğe hiç tahammütleri yoktur. Roberto Lerusalimsch, Lua gibi çok zarif bir dinamik dilin baş tasarımcısı, çöp toplayıcılı bir yazılımın kullanıldığı uçakta asla uçmak isteyemeyeceğini söylemiştir.

Vektörlere geri dönelim, bir vektör yaratıldığı ya da düzenlendiğinde heap içerisinden alan tahsis eder ve bu tahsis edilen alanın sahibi olur. Vektör öldüğünde ya da bellekten temizlendiğinde, bellek de serbest bırakılır.

# Döngüleyiciler (Iterators)
Rust bilinmezinin en temel noktasından henüz bahsetmedik - döngüleyiciler. Bir aralık (range) üzerinde kullanılan for döngüsü bir döngüleyici (iterator) kullanır. (`0..n` Python3'teki `range` fonksiyonuna benzer.)

Bir döngüleyiciyi fark etmek oldukça kolaydır. `Option` değerini bize dönen `next` metotuna sahip bir "objeye" döngüleyici deriz. `None` dönene kadar, `next` kullanabiliriz.

```rust
// iter1.rs
fn main() {
    let mut iter = 0..3;
    assert_eq!(iter.next(), Some(0));
    assert_eq!(iter.next(), Some(1));
    assert_eq!(iter.next(), Some(2));
    assert_eq!(iter.next(), None);
}
```

`for var in iter {}` 'in yaptığı da tam olarak budur.

Bu size for döngüsü tanımlamanın faydasız bir yolu gibi görünebilir ancak `rustc`'nin yapacağı akıl almaz optimizasyonların sonucunda While döngüsü kadar hızlı çalışacaktır.

Bir dizi üzerinde döngü kurmayı deneyelim:

```rust
// iter2.rs
fn main() {
    let arr = [10, 20, 30];
    for i in arr {
        println!("{}", i);
    }
}
```

Elbette ki hata dönecek. Ama çıktıya bakınca:
```
4 |     for i in arr {
  |     ^ the trait `std::iter::Iterator` is not implemented for `[{integer}; 3]`
  |
  = note: `[{integer}; 3]` is not an iterator; maybe try calling
   `.iter()` or a similar method
  = note: required by `std::iter::IntoIterator::into_iter`
```

`Rustc`'nin tavsiyesiyle programımız bir kez daha çalıştı:

```rust
`// iter3.rs
fn main() {
    let arr = [10, 20, 30];
    for i in arr.iter() {
        println!("{}", i);
    }

    // slices will be converted implicitly to iterators...
    let slice = &arr;
    for i in slice {
        println!("{}", i);
    }
}
```

Aslında, dizi üzerinde `for i in 0..slice.len() {}` gibi bir kullanımdansa bu yöntem çok daha verimlidir çünkü Rust'ı obsesifçe her indeks operasyonunda bir ton şeyi kontrol etmeye yönlendirmemiş oluyoruz.

Bir de bir aralığın hepsini hızlıca toplamanın bir başka örneğine bakalım. Daha önce bir döngü ve `mut` değişkenini kullanıyordum. Burada ise toplamanın "*idiomatic*" ve profesyonelce bir yolu var:

```rust
// sum1.rs
fn main() {
    let sum: i32  = (0..5).sum();
    println!("sum was {}", sum);

    let sum: i64 = [10, 20, 30].iter().sum();
    println!("sum was {}", sum);
}
```

Rust'ta tiplerin önceden bildirilmiş olması gereken durumlardan birisi olduğuna dikkat edin, aksi taktirde Rust tam olarak ne yapması gerektiğini bilemeyecektir. Burada farklı tam sayı tipleriyle çalışmaktayız ki bu sorun teşkil etmez. (Aynı zamanda, isim sıkıntısı çektiğiniz zamanlarda aynı ismi tekrar kullanmanız da sorun teşkil etmez.)

Bu bilgiyle birlikte, [dilim metotları](https://doc.rust-lang.org/std/primitive.slice.html) gözünüze biraz daha anlamlı gelecektir. (Belgelendirme hakkında bir bilgi; her belgenin sağ tarafında '[-]' işareti bulunur ve bu butona tıklayınca metot listesini kapatabilirsiniz. İlginizi çeken şeylerin detaylarını da genişletebilirsiniz. Şimdilik biraz tuhaf görünüyor, görmezden geliverin.)

`windows` metotu size dilimlerin döngüleyicisini verir - birbiriyle örtüşen pencereler olarak değerler (Ç.N: Burada ne demek istediği benim için bile bir bilinmez.)
```rust

// slice4.rs
fn main() {
    let ints = [1, 2, 3, 4, 5];
    let slice = &ints;

    for s in slice.windows(2) {
        println!("window {:?}", s);
    }
}
// window [1, 2]
// window [2, 3]
// window [3, 4]
// window [4, 5]
```
`Chunks` da benzer işlevi yapabilir.
```rust
 for s in slice.chunks(2) {
        println!("chunks {:?}", s);
    }
// chunks [1, 2]
// chunks [3, 4]
// chunks [5]
```

# Vektörler Üzerine Biraz Daha Konuşalım
Bir vektör kurmak için `vec!` isminde oldukça kullanışlı bir makromuz var. Bununla birlikte vektörün sonundaki verileri `pop` ile *silebiliriz* ve vektörü bir başka vektörle *genişletebiliriz (extend).*

```rust
// vec3.rs
fn main() {
    let mut v1 = vec![10, 20, 30, 40];
    v1.pop();

    let mut v2 = Vec::new();
    v2.push(10);
    v2.push(20);
    v2.push(30);

    assert_eq!(v1, v2);

    v2.extend(0..2);
    assert_eq!(v2, &[10, 20, 30, 0, 1]);
}
```

Vektörler birbirleriyle kıyaslanacağı zaman dilim olarak karşılaştırılır.

Vektörün belirli noktalarına `insert` kullanarak verileri yerleştirebilirsiniz, `remove` ile de silebilirsiniz. Bu vektörün sonuna veri eklemekten (*push*) ya da veri çıkartmaktan (*pop*) daha verimsizdir çünkü veriler yeni yer yaratmak için taşınır, büyük boyutlu vektörlerle çalışırken bu duruma dikkat etmeniz gerekir.

Vektörlerin bir büyüklüğü ve *kapasitesi* vardır. Eğer bir vektörü `clear` ile temizlerseniz, büyüklüğü sıfır olur ama eski kapasitesini korur. Bu durumda `push` vs ile yeni veriler eklerken yeniden bellek alanı tahsis edilmesi yalnızca eski kapasitenin aşımıyla gerekli olur.

Vektörler sıralanabilir ve içinde tekrarlayan veriler temizlenebilir - bu işlemler vektörü değiştirir. (Eğer önce vektörü kopyalamak isterseniz, `clone` kullanın.)

```rust
// vec4.rs
fn main() {
    let mut v1 = vec![1, 10, 5, 1, 2, 11, 2, 40];
    v1.sort();
    v1.dedup();
    assert_eq!(v1, &[1, 2, 5, 10, 11, 40]);
}
```

# Karakter Dizileri (String)
Rust'taki karakter dizileri diğer dillerden biraz daha gelişkindir. `String` tipi, `Vec` gibi, belleği dinamik olarak tahsis eder ve yeniden boyutlandırılabilir. (C++'ın `std::string` tipine çok benzer ancak Java'nın ve Python'nun değişemez karakter dizileri gibi değildir.) Ancak bir program, pek çok `string` kalıbı *(string literal)* de barındırabilir ("merhaba" gibi) ve bir sistem programlama dili bunları çıktı dosyasının içinde barındırabilmelidir. Gömülü mikroçiplerde bunun anlamı, bunları pahalı RAM yerine ucuz ROM'a yerleştirmektir. (Düşük seviyeli cihazlar için, RAM'ın pahalılığı aynı zamanda enerji üretimi pahalılığıdır.) Bir sistem programlama dilinde iki tür karakter dizisi bulunmalıdır, statik ya da bellekte yeri tahsis edilmiş.

Yani "merhaba" bir `String` değildir. Onun tipi `&str`'dir. ("Karakter dizisi dilimi *String Slice* olarak okunur.") Bu ayrım, C++'daki `const char*` ve `std::string` arasındaki fark gibidir ancak `&str` biraz daha kullanışlıdır. Doğrusu, `&str` ve `String` ilişki `&[T]` ile `Vec<T>` arasındaki ilişkiye çok benzer.

```rust
// string1.rs
fn dump(s: &str) {
    println!("str '{}'", s);
}

fn main() {
    let text = "hello dolly";  // the string slice
    let s = text.to_string();  // it's now an allocated string

    dump(text);
    dump(&s);
}
```

Tekrar edelim, ödünç alma operatörü tıpkı `Vec<T>`'yi `&[T]`'ye çevirmesi gibi `String`'i de `&str`'ye çevirir.

Aslında içten içe, `String` aslında bir `Vec<u8>`'dir ve `&str` de bir `&[u8]`'dir, ancak bu baytlar UTF-8'e *kesinlikle* uygun olmalıdır.

Vektör gibi, bir karakteri `push`layabilirsiniz veyahut sonundaki karakteri `pop`layabilirsiniz.

```rust
// string5.rs
fn main() {
    let mut s = String::new();
    // initially empty!
    s.push('H');
    s.push_str("ello");
    s.push(' ');
    s += "World!"; // short for `push_str`
    // remove the last char
    s.pop();

    assert_eq!(s, "Hello World");
}
```
Pek çok tipi String'e `to_string` diyerek çevirebilirsiniz. (Eğer onları "{}" ile ekranda gösterebiliyorsanız, çevrilebilirler.) `format!` makrosu da tıpkı `println!` gibi karmaşık karakter dizileri üretmek için kullanılabilir.

```rust
// string6.rs
fn array_to_str(arr: &[i32]) -> String {
    let mut res = '['.to_string();
    for v in arr {
        res += &v.to_string();
        res.push(',');
    }
    res.pop();
    res.push(']');
    res
}

fn main() {
    let arr = array_to_str(&[10, 20, 30]);
    let res = format!("hello {}", arr);

    assert_eq!(res, "hello [10,20,30]");
}
```

`v.to_string()`'in önündeki `&` operatörüne dikkat edin - operatör bir karakter dizesi dilimi üzerinde tanımlanmış, `String`'in kendisine değil, uyuşması için bazı detaylar eklememiz gerekiyor.

Dilimlerde kullanılan ifade şekli karakter dizilerinde de gösterilebilir:

```rust
// string2.rs
fn main() {
    let text = "static";
    let string = "dynamic".to_string();

    let text_s = &text[1..];
    let string_s = &string[2..4];

    println!("slices {:?} {:?}", text_s, string_s);
}
// slices "tatic" "na"
```

Ancak karakter dizilerini indeksleyemezsiniz. Çünkü onlar tek ve gerçek kodlama olan UTF-8'i kullanırlar ki bu kodlamada bazı "karakterler" sadece baytların sayısı olabilir.

```rust
// string3.rs
fn main() {
    let multilingual = "Hi! ¡Hola! привет!";
    for ch in multilingual.chars() {
        print!("'{}' ", ch);
    }
    println!("");
    println!("len {}", multilingual.len());
    println!("count {}", multilingual.chars().count());

    let maybe = multilingual.find('п');
    if maybe.is_some() {
        let hi = &multilingual[maybe.unwrap()..];
        println!("Russian hi {}", hi);
    }
}
// 'H' 'i' '!' ' ' '¡' 'H' 'o' 'l' 'a' '!' ' ' 'п' 'р' 'и' 'в' 'е' 'т' '!'
// len 25
// count 18
// Russian hi привет!
```

Şimdi şuna bakalım - 25 baytımız var ama sadece 18 kataktere sahibiz! Fakat, eğer `find` gibi bir metot kullanırsak (bulunması hâlinde) geçerli bir indeks elde alırsınız ve herhangi bir dilimleme doğru çalışacaktır. 

(Rust'ın `char` tipi 4-baytlık Unicode karakteridir. Karakter dizileri ise `char`ların dizisi değildir!)

Karakter dizelerini dilimlemek vektör dilimlemek gibi riskli bir iştir, çünkü bayt "aralıkları" kullanılır. Alttaki koşulda karakter dizesi iki bayttan oluşur, bunun ilk baytını almaya çalışmak bir Unicode hatasıdır. Bundan dolayı karakter dizisi metotlarından gelen uygun aralıkları kullanmaya dikkat edin.

```rust
    let s = "¡";
    println!("{}", &s[0..1]); <-- bad, first byte of a multibyte character
```

Karakter dizilerini parçalamak popüler ve faydalı bir meşgaledir.  `split_whitespace` metotu bir döngüleyici döner ve bunun ne yapacağımızı biz belirleriz. Genelde bir karakter dizisini daha ufak karakter dizilerinin vektörünü kurmak için buna ihtiyaç duyarız.

`collect` ise çok geneldir ve *neyi* topladığımız (collect) hakkında bir ipucu ister - bundan dolayı açıkça tip belirtilir.

```rust
    let text = "the red fox and the lazy dog";
    let words: Vec<&str> = text.split_whitespace().collect();
    // ["the", "red", "fox", "and", "the", "lazy", "dog"]
```

Döngüleyicilerin `extend` metotuyla da aynı işi yapabilirdiniz.

```rust
    let mut words = Vec::new();
    words.extend(text.split_whitespace());
```

Pek çok dilde bunları yapmak için *bellekte ayrıca alanı tahsis edilmiş karakter dizilerine* ihtiyacımız olurdu, oysa burada sadece bir ödünç alma olayı var. Tek tahsis edilen alan, dilimleri bellekte tutacak alandır.

Şu şirin çift-satıra bir bakınız; karakterler üzerine bir döngüleyici kuruyoruz ve boşluk olmayan karakterleri alıyoruz. Hatırlatalım, `collect` ipucu ister. (Ve biz de karakterler vektörü istemiş olabiliriz)

```rust
	let stripped: String = text.chars()
        .filter(|ch| ! ch.is_whitespace()).collect();
    // theredfoxandthelazydog
```

`filter` metotu ise argüman olarak *kapama (closure)* alır, kapama dediğimiz de Rust'ın dilinde lambdalara veya anonim fonksiyonlara verdiğimiz isim. Argüman, işlevsel olarak çalışmayı bozmadığı için apaçık tip belirtme kuralını genişletebiliyoruz.

Tabii bunca şeyi apaçık bir döngü ile değişebilir bir vektöre karakter dizilerini iterek de yapabilirsiniz, ama şimdi yaptığımız daha kısa, daha okunaklı (*alışınca* tabii) ve denk bir hızda. Bir döngü kullanmak elbet ayıp değildir ama bu şekilde yazmanızı şiddetle tavsiye ederim.

# Reklam Arası: Komut Satırından Argümanları Almak 
Şimdiye kadar programlarımız neyin ne olduğundan habersiz bir şekilde kendi kendilerine yaşayıp gittiler. Artık onları gerçek dünya ile tanıştırmalıyız.

`std::env::args` ile komut satırındaki argümanlara ulaşabilirsiniz; size bütün argümanları, programın ismi de dahil olmak üzere, birer karakter dizisi olarak döner.

```rust
// args0.rs
fn main() {
    for arg in std::env::args() {
        println!("'{}'", arg);
    }
} 
```
```
src$ rustc args0.rs
src$ ./args0 42 'hello dolly' frodo
'./args0'
'42'
'hello dolly'
'frodo'
```

Bir `Vec` dönse daha iyi olmaz mıydı? Bunu `collect` ile bir vektöre çevirmek de pek zor değildir, hatta döngüleyicilerin `skip` metotu ile programın adını atlayabilirsiniz.

```rust
    let args: Vec<String> = std::env::args().skip(1).collect();
    if args.len() > 0 { // we have args!
        ...
    }
```

İşin en iyi tarafı, bütün dillerde bunu bu şekilde kullanıyor olmanız.

Biraz daha Rustça yaklaşım ise tek bir argümanı okumak. (Ve aynı zamanda bir tam sayı verisini okumak):

```rust
// args1.rs
use std::env;

fn main() {
    let first = env::args().nth(1).expect("please supply an argument");
    let n: i32 = first.parse().expect("not an integer!");
    // do your magic
}
```

`nth(1)` size bir döngüleyicinin ikinci verisini döner, `expect` ise `unwrap` gibi çalışır ancak bir de mesaj belirtmenize izin verir.

Bir karakter dizisini bir sayıya çevirmenin yolu gayet bariz, ancak dönüştürülecek tipi açıkça belirtmeniz gerekmekte - yoksa `parse` bunu nereden bilecek?

# Örüntü Eşleştirme (Matching)

`string3.rs` dosyasındaki Rusça selamlamayı kullandığımız kodda aslında bu tarz durumları o şekilde çözmeyiz. `Match` ile deneyelim:

```rust
    match multilingual.find('п') {
        Some(idx) => {
            let hi = &multilingual[idx..];
            println!("Russian hi {}", hi);
        },
        None => println!("couldn't find the greeting, Товарищ")
    };
```

`Match` şu kızılderili okunu barındıran, virgüllerle ayrılmış pek çok örüntü tanımından oluşur. Bu ifade, çok rahat bir şekilde `Option` içerisinden ifadeyi ayıklayabilir ve `idx`'e bağlayabilir. Bütün koşulların *muhakkak* karşılanması gerektiği için `None`'u da ele alıyoruz. 

Buna bir kere alışınca (yani, bir kaç kez yazınca) size ayrıca `Option` tutacak ek bir değişken gerektiren `is_some` yazmaktan çok daha rahat gelecek.

Ancak hatalarla ilgilenmiyorsanız, mahalleden `if let`'i çağırabiliriz:

```rust
    if let Some(idx) = multilingual.find('п') {
        println!("Russian hi {}", &multilingual[idx..]);
    }
```

`Match` C'deki `switch` gibi de çalışabilir ve diğer Rust kurucuları gibi veri de dönebilir:

```rust
    let text = match n {
        0 => "zero",
        1 => "one",
        2 => "two",
        _ => "many",
    };
```

`_`'ı C'deki `default` olarak düşünebilirsiniz - varsayılan değer ifadesidir kendileri. Eğer bunu belirtmezseniz `rustc` bunun bir hata olduğunu düşünür. (C++'da bekleyebileceğiniz ilgili koşullar hakkında pek çok şeyi belirtilen bir uyarıyı almak olur.)

Rust'ın `match` deyimleri aralıkları da eşleştirebilir. Bu aralıkların üç noktalı olduğuna dikkat edin, bunlar kapsayan aralıklardır, mesela ilk koşul "3" sayısıyla eşleşecektir.

```rust
    let text = match n {
        0...3 => "small",
        4...6 => "medium",
        _ => "large",
     };
```

# Dosyaları Okumak
Bizim programlarımızı dünyaya açacak olan bir sonraki adımımız ise *dosyaları okumaktır.*

`expect`'in `unwrap` gibi çalıştığını ancak fazladan bir hata mesajı girmemize izin verdiğini aklınızda tutun. Şimdi birden çok hata hortlatacağız:

```rust
// file1.rs
use std::env;
use std::fs::File;
use std::io::Read;

fn main() {
    let first = env::args().nth(1).expect("please supply a filename");

    let mut file = File::open(&first).expect("can't open the file");

    let mut text = String::new();
    file.read_to_string(&mut text).expect("can't read the file");

    println!("file had {} bytes", text.len());

}
```
```
src$ file1 file1.rs
file had 366 bytes
src$ ./file1 frodo.txt
thread 'main' panicked at 'can't open the file: Error { repr: Os { code: 2, message: "No such file or directory" } }', ../src/libcore/result.rs:837
note: Run with `RUST_BACKTRACE=1` for a backtrace.
src$ file1 file1
thread 'main' panicked at 'can't read the file: Error { repr: Custom(Custom { kind: InvalidData, error: StringError("stream did not contain valid UTF-8") }) }', ../src/libcore/result.rs:837
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

Dosyanın var olmadığı veyahut okunmasına izin olmadığı durumlarda `open` hata dönebilir, `read_to_string` ise dosya içeriğinin UTF-8 olmaması durumunda hata döner. (Tabii, bu koşulda yerine `read_to_end` kullanıp içeriği bayt vektörlerine koymak da bir seçenek.) Çok da büyük olmayan dosyaları tek hamlede okumak daha faydalı ve basittir.

Eğer diğer dillerde dosya işleme nasıl olur bir fikriniz varsa dosyanın ne zaman kapatılması gerektiğini düşünüyor olabilirsiniz. Eğer dosyaya bir şeyler yazdırsaydık kapatmamak veri kaybına sebebiyet verebilirdi ancak burada dosya kendiliğinden kapatılıyor ve fonksiyon sona erdiği zaman da `file` değişkeni *düşürülüyor.*

Bu "hata hortlatma işi"ne biraz fazla alıştık galiba. Bütün programı böyle çökertebilen bir kodu kendi fonksiyonlarınıza yerleştirmek istemezsiniz. O zaman `File::open`'ın ne döndüğüne bakalım. Eğer `Option` bir şeyin varlığını ya da yokluğunu işaret ediyorsa `Result` da bir şeyin olup olmadığını gösterir. İkisi de `unwrap`'i bilir (ve amcaoğlu `expect`i de) ancak biraz farklıdırlar. `Result`, `Ok` ve `Err` için iki farklı tür parametre içerir. `Result` "paketi" iki farklı kompartmana sahiptir, birisi `Ok` ve diğeri de `Err`.

```rust
fn good_or_bad(good: bool) -> Result<i32,String> {
    if good {
        Ok(42)
    } else {
        Err("bad".to_string())
    }
}

fn main() {
    println!("{:?}",good_or_bad(true));
    //Ok(42)
    println!("{:?}",good_or_bad(false));
    //Err("bad")

    match good_or_bad(true) {
        Ok(n) => println!("Cool, I got {}",n),
        Err(e) => println!("Huh, I just got {}",e)
    }
    // Cool, I got 42

}
```

(Aslında "hata (error)" için seçtiğimiz tip biraz gereksiz - pek çok insan `Rust`'ın hata tiplerine alışana kadar karakter dizelerini tercih eder.) Bu, bir veriyi ya da başka bir veriyi döndürmenin gayet uygun bir yoludur.

Dosya okumamızın bu şekli çökmez. `Result` döner ve onu *çağırana* gelen verinin nasıl işlenmesi gerektiğini seçtirir.

```rust
// file2.rs
use std::env;
use std::fs::File;
use std::io::Read;
use std::io;

fn read_to_string(filename: &str) -> Result<String,io::Error> {
    let mut file = match File::open(&filename) {
        Ok(f) => f,
        Err(e) => return Err(e),
    };
    let mut text = String::new();
    match file.read_to_string(&mut text) {
        Ok(_) => Ok(text),
        Err(e) => Err(e),
    }
}

fn main() {
    let file = env::args().nth(1).expect("please supply a filename");

    let text = read_to_string(&file).expect("bad file man!");

    println!("file had {} bytes", text.len());
}
```

Birinci eşleşme `Ok` içindeki veriyi güvenli bir şekilde dışarı çıkartır ve eşleşmenin değeri yapar. Eğer bir `Err` verisi ise, hatayı döner ve onu tekrar `Err` içerisine paketler.

İkinci eşleşme ise `Ok` içerisine paketlenmiş bir karakter dizesi döner ya da hatayı tekrar eder. `Ok` içindeki esas veriye ihtiyacımız yok ondan dolayı `_` ile yok sayıyoruz.

Bu biraz sıkıcı, yazdığımız kodu büyük kısmı hatayı işlemekten ibaret olunca "işin ruhunu" kaybediyoruz. Mesela Go'da bunu hissedersiniz, düzinesiyle erken dönen hataları kontrol etmeniz gerekir ya da sadece *görmezden gelirsiniz*. (Rust evreninde bu tuvalette ekmek çiğnemek kadar kötü bir şeydir.)

Neyse ki, bir kısayolumuz var.

`std::io` modülü `io::Result<T>` diye bir tipe sahiptir ki bu `Result<T, io::Error>` ile aynı şeydir ve daha kolay yazılabilir.

```rust
fn read_to_string(filename: &str) -> io::Result<String> {
    let mut file = File::open(&filename)?;
    let mut text = String::new();
    file.read_to_string(&mut text)?;
    Ok(text)
}
```

`?` operatörü `File::open` üzerinde denediğimiz eşleştirmelerle birebir aynı şeyi yapıyor; eğer sonuç hataysa hemen fonksiyonu döndürüyor. Değilse, `Ok` sonucunu dönüyor. Sonuç olarak hâlâ daha karakter dizisini paketlememiz gerekiyor.

2017 senesi Rust için iyi bir yıldı ve `?` gibi karizmatik şeyler kararlı hâle geldi. Eski kodlarda `try!` diye bir makroyu görebilirsiniz:

```rust
fn read_to_string(filename: &str) -> io::Result<String> {
    let mut file = try!(File::open(&filename));
    let mut text = String::new();
    try!(file.read_to_string(&mut text));
    Ok(text)
}
```

Sonuç olarak, tek tek hataları bildirmeden güvenli Rust kodu yazmak düşündüğünüz kadar çirkin değil.