# Yapılar, Numaralandırmalar ve Eşleştirme

# Rust Alekta Movik Movik
> Ç.N: Orijinal başlık - Rust likes to Move It, Move It. "I like to move it" isimli bir şarkıya gönderme. Bu şarkıyı Türk milleti olarak "Alekta Movik Movik" diye biliyoruz :(

Fazla ileri gitmiyor muyuz? Mesela kaçırdığımız bazı şeyler var:

```rust
// move1.rs
fn main() {
    let s1 = "hello dolly".to_string();
    let s2 = s1;
    println!("s1 {}", s1);
}
```

Kod çalışınca da şu hatayı alırız:

```
error[E0382]: use of moved value: `s1`
 --> move1.rs:5:22
  |
4 |     let s2 = s1;
  |         -- value moved here
5 |     println!("s1 {}", s1);
  |                      ^^ value used here after move
  |
  = note: move occurs because `s1` has type `std::string::String`,
  which does not implement the `Copy` trait
```

Rust diğer dillerden biraz daha farklı davranır. Bütün değişkenleri birer referans olduğu dillerde (Java ve Python gibi) `s2` `s1`'in karakter dizesi objesine bir başka referans olur. C++'da ise `s1` bir veridir ve `s2`'ye kopyalanır. Ancak Rust veriyi *taşır (move)*, karakter dizelerini ise kopyalanabilir bir tür olarak da görmez. ("does not implement the Copy trait" - "Kopyala özelliğini barındırmıyor")

Böyle bir şeyi sayılar gibi "ilkel (primitive)" tiplerde görmeyiz çünkü onlar sadece veridir; kopyalanabilmelerine izin vardır çünkü kopyalaması ucuzdur. Ama `String` "Hello Dolly" için bellekte yer tahsis eder ve kopyalama daha fazla belleğin tahsis edilmesini ve karakterlerin tek tek kopyalanmasını içerir. Rust'ın bunu sessiz sedasız yapmasını bekleyemezsiniz. 

`String`'in bütün "Moby Dick"i barındırdığını düşünün. Bu karmaşık bir *yapı (struct)* olmazdı; sadece yazının bulunduğu bellek bölgesini tutan adresi, büyüklüğünü ve ne kadar bellekte alan tahsis edildiğini barındırırdı. Kopyalamak epey bir yük olurdu çünkü bellek *heap* bölgesinde tahsis edilmişti ve kopyalamanın kendisi de bellekte alan tahsis etmeyi gerektirirdi. 

```
    String
    | addr | ---------> Call me Ishmael.....
    | size |                    |
    | cap  |                    |
                                |
    &str                        |
    | addr | -------------------|
    | size |

    f64
    | 8 bytes |
```

İkinci veri ise karakter dizisi dilimidir (`&str`) ve `String` ile aynı bellek alanına yönlendirir, büyüklüğü ile birlikte. Kopyalaması çok basit!

Üçüncü verimiz ise `f64` - sadece 8 bayt tutuyor. Herhangi bir bellek alanına yönlendirilmiyor, yani kopyalaması onu taşımak kadar basit.

`Copy` verileri bellekteki karşılıklarıyla tanımlanır ve Rust kopyaladığı zaman bu baytları sadece başka bir yere kopyalar. Buna benzer olarak `Copy` olmayan bir veri ise *sadece taşınır*. C++'ın aksine kopyalama ve taşımada herhangi bir karmaşa yoktur.

Aynı şeyi bir fonksiyon çağrısı olarak yazmak da aynı soruna sebep olur:

```rust
// move2.rs

fn dump(s: String) {
    println!("{}", s);
}

fn main() {
    let s1 = "hello dolly".to_string();
    dump(s1);
    println!("s1 {}", s1); // <---error: 'value used here after move'
}
```

Şimdi bir tercih yapmanız gerekiyor. Ya `String`'i bir referans olarak kullanacaksınız ya da açık açık `clone` metotu ile onu kopyalayacaksınız. Genelde ilk olan daha iyi bir seçenektir.

```rust
fn dump(s: &String) {
    println!("{}", s);
}

fn main() {
    let s1 = "hello dolly".to_string();
    dump(&s1);
    println!("s1 {}", s1);
}
```

Artık hatadan çok uzaktayız. Ancak `String` referansını çok nadir görürsünüz, çünkü bir karakter dizisi kalıbını bu şekilde kullanmak gerçekten çirkin ve bu yolla geçici bir `String` oluşturmak zorunda kalırsınız. 

```rust
    dump(&"hello world".to_string());
```

Onun yerine en iyi yol şudur:

```rust
fn dump(s: &str) {
    println!("{}", s);
}
```

Ve böylece `dump(&s1)` ve `dump("hello world")` kullanımlarının ikisi de geçerli olacaktır. (Burada Rust'ın `Deref` zorlaması işin içine girer ve `&String`'i `&str` yapar.)

Sonuç olarak, `Copy` olmayan bir değerin değişkene atanması bir konumdan öbürüne taşınmasıdır. Eğer bu olmasaydı Rust *gizlice* kopyalamak zorunda kalırdı ve bellek tahsislerini *aleni* yapma sözünü tutamazdı.

# Değişkenlerin Kapsamları
Birinci kural, verileri kopyalamak yerine orijinal veriye referans göstermektir - yani "ödünç almak."

Ancak bir referans sahibinden daha uzun *asla* yaşayamaz!

Öncelikle Rust blok kapsamlı bir dildir. Değişkenler kendi blokları kadar yaşar:

```rust
{
    let a = 10;
    let b = "hello";
    {
        let c = "hello".to_string();
        // a,b and c are visible
    }
    // the string c is dropped
    // a,b are visible
    for i in 0..a {
        let b = &b[1..];
        // original b is no longer visible - it is shadowed.
    }
    // the slice b is dropped
    // i is _not_ visible!
}
```

(`i` gibi) Döngü değişkenleri biraz farklıdır, onlar sadece döngülerinin blokları için geçerlidir. Aynı isimle yeni bir değişken oluşturmak ("gölgelemek/*shadowing*") bir hata değildir ama kafa karıştırıcı olabilir.

Bir değişken "kapsam dışına çıkınca" *düşürülür (dropped)*. Kullanılan her bir bellek tanesi geri dönüştürülür ve sistemden alınan kaynaklar iade edilir - örneğin, `File`'ı düşürmek onu kapatır. Bu iyi bir şey. Kullanılmayan kaynaklar ihtiyaç olmayınca hemen geri teslim edilir.

(Rust'a özgü bir başka sorun da verinin taşınmış olmasına rağmen kapsam dahilinde görünmüş olmasıdır.)

Bu örnekte `rs1` isminde bir referans hazırladık ve değerini sadece iç bloğun ömrü kadar uzun kalan `tmp`'ye ayarladık.

```rust
01 // ref1.rs
02 fn main() {
03    let s1 = "hello dolly".to_string();
04    let mut rs1 = &s1;
05    {
06        let tmp = "hello world".to_string();
07        rs1 = &tmp;
08    }
09    println!("ref {}", rs1);
10 }
```

`s1`'in verisini ödünç aldık ve sonra da `tmp`'i ödünç aldık. Ancak `tmp`, bloğun dışında yok!

```
error: `tmp` does not live long enough
  --> ref1.rs:8:5
   |
7  |         rs1 = &tmp;
   |                --- borrow occurs here
8  |     }
   |     ^ `tmp` dropped here while still borrowed
9  |     println!("ref {}", rs1);
10 | }
   | - borrowed value needs to live until here
```

`Tmp` nerede? Gitti, yok, öldü o artık: *düşürüldü*. Rust sizi burada C'nin "sarkan işaretçiler (dangling pointer)" belasından koruyor - çoktan yitip gitmiş bir veriye işaret eden referanslardan yani.

# Demetler (Tuple)
Bir fonksiyondan öoklu veriler dönmeyi gerektiren zamanlar gerekecektir. Demetler bunun için gayet uygun bir gözümdür.

```rust
// tuple1.rs

fn add_mul(x: f64, y: f64) -> (f64,f64) {
    (x + y, x * y)
}

fn main() {
    let t = add_mul(2.0,10.0);

    // can debug print
    println!("t {:?}", t);

    // can 'index' the values
    println!("add {} mul {}", t.0,t.1);

    // can _extract_ values
    let (add,mul) = t;
    println!("add {} mul {}", add,mul);
}
// t (12, 20)
// add 12 mul 20
// add 12 mul 20
```

Demetlerin dizilerden temel farkları, demetler *farklı* tipler barındırabilmesidir.

```rust
let tuple = ("hello", 5, 'c');

assert_eq!(tuple.0, "hello");
assert_eq!(tuple.1, 5);
assert_eq!(tuple.2, 'c');
```

Bazen `Iterator` metotlarından karşınıza fırlarlar. `enumerate` tıpkı Python'daki aynı isimli oluşturucu gibi çalışır:

```rust
    for t in ["zero","one","two"].iter().enumerate() {
        print!(" {} {};",t.0,t.1);
    }
    //  0 zero; 1 one; 2 two;
```

`zip` ise iki döngüleyiciyi birbiriyle eşleştirir ve bir demet içerisinde veri dönen tek bir döngüleyici olarak birleştirir.

```rust
    let names = ["ten","hundred","thousand"];
    let nums = [10,100,1000];
    for p in names.iter().zip(nums.iter()) {
        print!(" {} {};", p.0,p.1);
    }
    //  ten 10; hundred 100; thousand 1000;
```

# Yapılar (Struct)
Demetler fena şeyler değiller ancak `t.1` gibi anlaşılmaz şeylerle parçalarını incelemek biraz can sıkıcı olabilirler. 

Rust *yapıları* ise isimli *alanlar (field)* barındırır:

```rust
// struct1.rs

struct Person {
    first_name: String,
    last_name: String
}

fn main() {
    let p = Person {
        first_name: "John".to_string(),
        last_name: "Smith".to_string()
    };
    println!("person {} {}", p.first_name,p.last_name);
}
```

Sizin bunu fark etmemenize rağmen yapıların verileri bellekte yanyana dururlar çünkü derleyici belleği verimliliğe göre düzenler, büyüklüğüne göre değil ve arada bazı boşluklar olabilir.

Bu yapıyı ilklemek (initalize) biraz şekilsiz görünebilir, bundan dolayı `Person` yapısını oluşturmayı bir fonksiyon içerisine taşıyorum. Bu fonksiyon bir `impl` bloğunun içerisine taşınarak `Person`'a ait bir *ilişkili fonksiyona (associated function)* dönüştürülebilir.

```rust
// struct2.rs

struct Person {
    first_name: String,
    last_name: String
}

impl Person {

    fn new(first: &str, name: &str) -> Person {
        Person {
            first_name: first.to_string(),
            last_name: name.to_string()
        }
    }

}

fn main() {
    let p = Person::new("John","Smith");
    println!("person {} {}", p.first_name,p.last_name);
}
```

`new` ile ilişkili özel bir şey yok. C++ tarzı `::` notasyonu ile bu fonksiyona ulaşabiliyoruz.

Bir de argüman olarak *kendisini referans alan (reference self)* `Person` metotu hazırlayalım.

```rust
impl Person {
    ...

    fn full_name(&self) -> String {
        format!("{} {}", self.first_name, self.last_name)
    }

}
...
    println!("fullname {}", p.full_name());
// fullname John Smith
```

`self`, bir referans olarak açıkça belirtildi. (`&self`'i `self: &Person`'un kısaltması olarak düşünebilirsiniz.)

`Self` kelimesi `struct` tipine atıfta bulunur - kafanızda `Person` yerine `Self` koyabilirsiniz:

```rust
    fn copy(&self) -> Self {
        Self::new(&self.first_name,&self.last_name)
    }
```

Metotlar veri düzenlemek için kendilerini *`mutable self`* olarak argüman alırlar.

```rust
    fn set_first_name(&mut self, name: &str) {
        self.first_name = name.to_string();
    }
```

Ve sadece `self` kullanıldığında veri *taşınacaktır*:

```rust
    fn to_tuple(self) -> (String,String) {
        (self.first_name, self.last_name)
    }
```

(Bunu bir de `&self` ile deneyin ve yapıların (struct) kendi verileri konusunda ne kadar inatçı olduğunu bir de siz görün!)

`v.to_tuple()` çağrıldığı zaman `v`'nin taşındığını ve kullanılamaz hâle geldiğini göreceksiniz.

Özetlersek:
- `self` kullanılmazsa: fonksiyonları bu şekilde bağlayabilirsiniz, `new` "oluşturucusu" gibi .
- `&self` ile: Yapının verilerini kullanabilir ancak değiştiremezsiniz.
- `&mut self` ile: Yapının verilerini düzenleyebilirsiniz.
- `self` ile: Veriyi tüketirsiniz, yani taşırsınız.

 Eğer `Person`'u veri ayıklama şeklinde ekrana yazdırırsanız, bilgilendirici bir hata alırsınız:
```
error[E0277]: the trait bound `Person: std::fmt::Debug` is not satisfied
  --> struct2.rs:23:21
   |
23 |     println!("{:?}", p);
   |                     ^ the trait `std::fmt::Debug` is not implemented for `Person`
   |
   = note: `Person` cannot be formatted using `:?`; if it is defined in your crate,
    add `#[derive(Debug)]` or manually implement it
   = note: required by `std::fmt::Debug::fmt`
```

Derleyici bazı tavsiyesine uyuyoruz ve `Person`'un tanımı üstüne  `#[derive(Debug)]` ekliyoruz, böylece işe yarar bir çıktımız oluyor:

```
Person { first_name: "John", last_name: "Smith" }
```

Bu direktif, derleyicinin faydalı bir özellik olan `Debug`'u eklemesine yarıyor ki bu da sizin kendi yapılarınızla (struct) ekrana yazdırarak pratik yapmanıza yardımcı olur. (Ya da `format!` ile yazdırabilirsiniz). (Bunu sürekli olarak yapmak pek Rust geleneğine yakışmaz doğrusu.)

İşte minik programımızın son hâli:

```rust
// struct4.rs
use std::fmt;

#[derive(Debug)]
struct Person {
    first_name: String,
    last_name: String
}

impl Person {

    fn new(first: &str, name: &str) -> Person {
        Person {
            first_name: first.to_string(),
            last_name: name.to_string()
        }
    }

    fn full_name(&self) -> String {
        format!("{} {}",self.first_name, self.last_name)
    }

    fn set_first_name(&mut self, name: &str) {
        self.first_name = name.to_string();
    }

    fn to_tuple(self) -> (String,String) {
        (self.first_name, self.last_name)
    }
}

fn main() {
    let mut p = Person::new("John","Smith");

    println!("{:?}", p);

    p.set_first_name("Jane");

    println!("{:?}", p);

    println!("{:?}", p.to_tuple());
    // p has now moved.

}
// Person { first_name: "John", last_name: "Smith" }
// Person { first_name: "Jane", last_name: "Smith" }
// ("Jane", "Smith")
```

# Yaşam Sürelerinin Yüreğimizi Dağlamaya Başladığı O An

Yapıların çoğu zaman veri taşır ancak bazen referans taşıması da gerekebilir. Mesela düşünelim ki yapımıza karakter dizisi değeri yerine bir karakter dizisi dilimi ekleyeceğiz.

```rust
// life1.rs

#[derive(Debug)]
struct A {
    s: &str
}

fn main() {
    let a = A { s: "hello dammit" };

    println!("{:?}", a);
}
```
```
error[E0106]: missing lifetime specifier
 --> life1.rs:5:8
  |
5 |     s: &str
  |        ^ expected lifetime parameter
```

Buradaki sorunu anlayabilmek için problemi bir de Rust'ın gözünden görmeniz gerekmekte. Rust, bir referansın ömrünün ne kadar uzun süreceğini hesaplamadan o referansa izin vermeyecektir. Bütün referanslar bir veriyi önüç alır ve her verinin bir yaşam süresi vardır. Referansların yaşam süreleri o verinin yaşam süresinden uzun olamaz. Rust, referansın geçersiz olduğu bir koşulun oluşma ihtimaline izin vermeyecektir. 

Şimdi, karakter dizisi diliminin referansı bir `String` değerini ya da "selam" gibi bir *karakter dizisi kalıbını* ödünç alır. Karakter dizesi kalıpları programın yaşamı boyunca yaşar ki buna "statik (static)" yaşam süresi deriz.

İşte şimdi tıkır tıkır çalışıyor - Rust'ın bir karakter dizisi kalıbının sürekli olarak var olacağını garanti etmiş olduk. 

```rust
// life2.rs

#[derive(Debug)]
struct A {
    s: &'static str
}

fn main() {
    let a = A { s: "hello dammit" };

    println!("{:?}", a);
}
// A { s: "hello dammit" }
```

Tabii bu hâli de çok şık görünmüyor ama kesin olmak için bazı bedeller ödmemiz gerekiyor.

Bunu bir fonksiyondan karakter dizisi dilimi döndürmek için de kullanabiliriz.

```rust
fn how(i: u32) -> &'static str {
    match i {
    0 => "none",
    1 => "one",
    _ => "many"
    }
}
```

Kısıtlayıcı olmasına karşın statik karakter dizilerinin bu tarz durumları için işe yarar.

Buna karşın, biz bir referansın yaşam ömrünü *en az yapının ömrü kadar uzun* olarak da belirtebiliriz. 

```rust
// life3.rs

#[derive(Debug)]
struct A <'a> {
    s: &'a str
}

fn main() {
    let s = "I'm a little string".to_string();
    let a = A { s: &s };

    println!("{:?}", a);
}
```

Yaşam ömürleri geleneksel olarak "a", "b" gibi harflerle belirtilir ancak siz dilerseniz "ben" gibi kelimelerle de ifade edebilirsiniz.

Bu ekleme ile beraber, bizim `A` yapısı ile `s` karakter dizisi birbirine sıkı sıkıya bağlanmıştır: `a`, `s`ten ödünç alır ve o olmadan yaşayamaz.

Bu tanımla birlikte şu şekilde `A` dönen bir fonksiyon yazabiliriz.

```rust
fn makes_a() -> A {
    let string = "I'm a little string".to_string();
    A { s: &string }
}
```

Ancak bu sefer de `A`'nın açıkça yaşam süresinin belirtilmesine ihtiyaç vardır - "expected lifetime parameter" (beklenilen yaşam süresi parametresi)

```
  = help: this function's return type contains a borrowed value,
   but there is no value for it to be borrowed from
  = help: consider giving it a 'static lifetime
```

`rustc`'nin verdiği tavsiyeye uyalım:

```rust
fn makes_a() -> A<'static> {
    let string = "I'm a little string".to_string();
    A { s: &string }
}
```

Ve hatamız:

```
8 |      A { s: &string }
  |              ^^^^^^ does not live long enough
9 | }
  | - borrowed value only lives until here
```

Bunu güvenli bir şekilde yapmanın bir yolu yok, çünkü fonksiyon sona verdiği zaman `string` düşecek ve `string`e yapılan referanslar kendinden daha uzun süre yaşayamaz.

Bazen, bir yapının değer ve o değeri içeren bir referans taşıması iyi bir fikirmiş gibi görünebilir. Ama bu çok basit bir şekilde imkansızdır çünkü yapılar *taşınabilir* olmalıdır, ve her türlü taşınma referansı geçersiz kılacaktır. Üstelik bunu yapmanın bir gereği de yok - mesela yapınızın karakter dizisi alanı varsa ve bunun dilimlerini sunmaya ihtiyacınız varsa, indeks numaralarını tutabilir ve bir metot içerisinde gerçek dilimleri dönebilirsiniz.

# Özellikler (Trait)
Rust'ta `struct`'ı *sınıf (class)* olarak görmediğine dikkat edin. `class` kelimesinin anlamı diğer dillerde içi öylesine doldurulmuştur ki size nasıl düşüneceğinizi dikte eder hâle gelmiştir.

Şimdi şunlara dikkat edin: Rust'ta yapılar birbirini *miras (inherit)* alamaz; hepsi özgün tiplerdir. *Alt-tip* diye bir şey yok, onlar sadece bir saçmalıktan ibaretti.

Peki ya tipler arasındaki ilişkiler nasıl kurulur?

`rustc` bazen "implementing X trait (X özelliğini uygulamak)" diye gevezelik eder ve şimdi özellikler (tipler) hakkında konuşmanın tam zamanı.

Aşağıdaki bir özellik tanımlamanın ve belirli tiplere nasıl *uygulandığının* örneğini görüyorsunuz.  

```rust
// trait1.rs

trait Show {
    fn show(&self) -> String;
}

impl Show for i32 {
    fn show(&self) -> String {
        format!("four-byte signed {}", self)
    }
}

impl Show for f64 {
    fn show(&self) -> String {
        format!("eight-byte float {}", self)
    }
}

fn main() {
    let answer = 42;
    let maybe_pi = 3.14;
    let s1 = answer.show();
    let s2 = maybe_pi.show();
    println!("show {}", s1);
    println!("show {}", s2);
}
// show four-byte signed 42
// show eight-byte float 3.14
```

Şahane; `i32` ve `f64` içerisine *yeni bir metot* ekledik.

Rust ile haşır neşir oldum diyebilmek için standart kütüphanedeki basit özellikleri de bilmeniz gerekir. (Ki genelde bir arada bulunurlar.)

`Debug` epey yaygındır. `Person` üzerinde `#[derive(Debug)]` ile uyguladık, ancak isteseydik tam ismi görüntüleyecek şekilde de uygulayabilirdik.

```rust
use std::fmt;

impl fmt::Debug for Person {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}", self.full_name())
    }
}
...
    println!("{:?}", p);
    // John Smith
```

`write!` da epey kullanışlı bir makrodur - burada `f` `Write` özelliğini uygulamış her şeyi temsil ediyor. (Mesela bu bir `File` olabilir - ya da sadece bir `String`)

`Display` ise "{}" ile yazdırılabilen verileri kontrol ve tıpkı `Debug` gibi uygulanır. Ve faydalı bir yan etki olarak, `ToString` `Display`'e sahip olan her türlü tipe uygulanır. Mesela `Display`'ı `Person` için uygularsak `p.to_string()` de çalışır hâle gelir.

`Clone` ise `clone` metotunu tanımlar ve sadece `#[derive(Clone)]` ile tanımlanabilir - eğer bütün alanların (fields) tipleri `Clone`'a sahipse. (Ç.N: Clone - İngilizce Klonlamak)

# Örnek: Noktalı sayı aralıklarının döngüleyicisi
Daha önce aralıklarla (range, `0..n`) karşılaştık ancak noktalı sayı kabul etmiyorlar. (*Şansınızı zorlayabilirsiniz* ancak pek de numarası olmayan 1.0'da takılıp kalırsınız.) 

Bir döngüleyici (iterator) için yaptığımız gayriresmi tanımı hatırlayın; `Some` veya `None` dönebilen bir `next` metotuna sahip yapı. Bu süreçte, döngüleyicinin kendisi düzenlenir ve döngülemenin durumu hakkında bilgi tutar. (Sonraki indeks gibi) Döngülenen verinin içeriği genellikle değişmez. (Ancak `Vec::drain` gibi kendi verisini düzenleyen enteresan bir döngüleyiciyi de inceleyeceğiz.)

Ve şimdi de resmi tanımı görelim: ["Iterator" özelliği](https://doc.rust-lang.org/std/iter/trait.Iterator.html)

```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    ...
}
```

`Iterator` için [ilişkili tipi (associated type)](https://doc.rust-lang.org/stable/book/associated-types.html) de tanımış olduk. Bu özelliğin (trait) çalışması için bir tipe ihtiyaç vardır ve dönüş tipini de belirtmeniz gerekmektedir. `next` metotu belirli bir tip belirtilmeden çalışabilir, sadece `Self` üzerinden `Item`'e atışta bulunulması yeterlidir.

`f64` tipi için uygulanmış bir `Iterator`, `Iterator<Item=f64>` ile belirtilir ki bunu "f64 tipi ile ilişkilendirilmiş bir döngüleyici" olarak okuyabilirsiniz. 

`...` ile gösterilen kısım `Iterator`ün tedarik ettiği metotlardır. Sadece `Item` ve `next`'i belirttikten sonra pek çok metot da sizin için sunulacaktır.

```rust
// trait3.rs

struct FRange {
    val: f64,
    end: f64,
    incr: f64
}

fn range(x1: f64, x2: f64, skip: f64) -> FRange {
    FRange {val: x1, end: x2, incr: skip}
}

impl Iterator for FRange {
    type Item = f64;

    fn next(&mut self) -> Option<Self::Item> {
        let res = self.val;
        if res >= self.end {
            None
        } else {
            self.val += self.incr;
            Some(res)
        }
    }
}


fn main() {
    for x in range(0.0, 1.0, 0.1) {
        println!("{} ", x);
    }
}
```

Ve şöyle biçimsiz bir görüntüyü elde etmiş oluyoruz:

```
0
0.1
0.2
0.30000000000000004
0.4
0.5
0.6
0.7
0.7999999999999999
0.8999999999999999
0.9999999999999999
```

`0.1` tam olarak noktalı sayı olarak gösterilemediğinden böyle tuhaf şeyler yaşıyoruz, minik bir formatlama yardımı ile bundan kurtulabiliriz. `println!` kısımını şöyle düzeltelim:
```rust
println!("{:.1} ", x);
```
Ve daha temiz bir çıktımız olmuş oluyor. (Bu [formatlama](https://doc.rust-lang.org/std/fmt/index.html) "noktadan sonra bir nokta" anlamına geliyor.)

Şimdi bütün döngüleyici metotlarını kullanabiliriz, hadi bütün verileri bir vektörde toplayalım, `map` kullanalım ve daha da coşalım:

```rust
    let v: Vec<f64> = range(0.0, 1.0, 0.1).map(|x| x.sin()).collect();
```

# Jenerik Fonksiyonlar
Diyelim ki `Debug` özelliiğine sahip herhangi bir tipi argüman olarak alan bir fonksiyon yazacağız. Burada jenerik fonksiyon kullanmamızın bir örneğini görüyorsunuz, herhangi bir verinin referansını argüman olarak alabilir. `T`, tip parametresi oluyor ki fonksiyon ismi yazıldıktan hemen sonra tanımlandı:

```rust
fn dump<T> (value: &T) {
    println!("value is {:?}",value);
}

let n = 42;
dump(&n);
```

Ancak, Rust kelimenin tam anlamıyla `T` tipi hakkında hiçbir şey bilmiyor.

```
error[E0277]: the trait bound `T: std::fmt::Debug` is not satisfied
...
   = help: the trait `std::fmt::Debug` is not implemented for `T`
   = help: consider adding a `where T: std::fmt::Debug` bound
```

Bunun çalışması için, `T`'nin `Debug` içermesi gerektiğinden bahsetmeliyiz:

```rust
fn dump<T> (value: &T)
where T: std::fmt::Debug {
    println!("value is {:?}",value);
}

let n = 42;
dump(&n);
// value is 42
```

Rust'ın jenerik fonksiyonlarının tipe *özellikleri bağlaması (trait bounds)* gerekir - burada "T is any type that implements Debug" kısmını anlatıyoruz. (T, Debug'ı içeren herhangi bir tiptir) `rustc` epey yardımcı oluyor ve hangi tipin tam olarak belirtilmesi gerektiğini bize bildiriyor.

Şimdi Rust, `T` için tip bağlarını biliyor, artık derleyiciden mantıklı mesajlar alabiliriz.

```rust
struct Foo {
    name: String
}

let foo = Foo{name: "hello".to_string()};

dump(&foo)
```

Buradaki hata ise  "the trait `std::fmt::Debug` is not implemented for `Foo` (`std::fmt::Debug` özelliği `Foo` için uygulanmadı)"

Fonksiyonlar dinamik dillerde aslında jeneriktir çünkü değerler beraberinde türlerini taşırlar ve tür denetimi çalışma zamanı denetlenir - ya da başarısız olur. Karmaşık programlarda daha derleme zamanında tiplerin kontrol edilmesini ciddi anlamda isteriz! Bu dillerdeki bir programcı, derleme hatalarını sakince incelemek yerine programın çalışma anındadaki sürprizleri incelemek zorundadır. Murphy kanununa göre sorunlar en uygunsuz, ters zamanda ortaya çıkmaya meyillidir.

Bir sayının karesini almak jeneriktir; tam sayılar, noktalı sayılar ve çarpım operatörünü içeren her türlü şeyin karesini `x*x` ile alabilirsiniz. Peki ya tip bağları?

```rust
// gen1.rs

fn sqr<T> (x: T) -> T {
    x * x
}

fn main() {
    let res = sqr(10.0);
    println!("res {}",res);
}
```

Sorun, Rust'ın `T`'nin çarpılabilir olduğunu bilmemesidir.

```
error[E0369]: binary operation `*` cannot be applied to type `T`
 --> gen1.rs:4:5
  |
4 |     x * x
  |     ^
  |
note: an implementation of `std::ops::Mul` might be missing for `T`
 --> gen1.rs:4:5
  |
4 |     x * x
  |     ^
```

Derleyicinin tavsiyesine uyarak bu tipi `*` çarpım operatörünü barındıran [ilgili özelliğe](https://doc.rust-lang.org/std/ops/trait.Mul.html) zorlamayı deneyelim.

```rust
fn sqr<T> (x: T) -> T
where T: std::ops::Mul {
    x * x
}
```

Yine de hâlen daha çalışmıyor:

```
error[E0308]: mismatched types
 --> gen2.rs:6:5
  |
6 |     x * x
  |     ^^^ expected type parameter, found associated type
  |
  = note: expected type `T`
  = note:    found type `<T as std::ops::Mul>::Output`
```

Bu tipi daha da kısıtlamayı deneyelim:

```rust
fn sqr<T> (x: T) -> T::Output
where T: std::ops::Mul + Copy {
    x * x
}
```

(Ancak) şimdi oldu! Derleyiciyi sakince dinlemek sizi esas noktaya yaklaştırır, ta ki temizce program derlenene dek.

Tabii *bunu* `C++`'da yapmak daha kolay.

```cpp
template <typename T>
T sqr(x: T) {
    return x * x;
}
```

Ama (dürüst olmak gerekirse), C++ laz müteahhit mantığını benimsiyor. C++'ın şablon (template) hataları berbattır çünkü derleyicinin tek bildiği şey bazı metotların ya da operatörlerin tanımlanıp tanımlanmadığıdır. C++ komitesi bu sorunu biliyor ve [konseptler](https://en.wikipedia.org/wiki/Concepts_(C%2B%2B)) üzerinde çalışıyorlar ki bunlar daha çok özelliklerle kısıtlanmış tip parametrelerine çok benziyorlar. 

Jenerik fonksiyonlar başta biraz zorlayıcı gelebilir ancak net olmak, ne tür değerleri güvenle kullanabileceğinizi sadece tanıma bakarak kullanabileceğiniz anlamına geliyor.

Bu fonksiyonlar *çok biçimli*nin tersi olarak *tek biçimli* olarak bilinir. (ÇN: Tek biçimli - monomorfik, çok bilimli - polimorfik) Fonksiyonun gövdesi her bir tip için ayrı ayrı derleme yapar. Çok biçimli fonksiyonlarda ise makine eşlesen her tip için aynı kodu kullanır, dinamik olarak doğru metota *yönlendirir (dispatch)*.

Tek biçimlilik hızlı kod üretir, tipler için özelleştirilmiştir ve *satır içi* çalışabilirler. `sqr(x)` görüldüğü anda hemen `x*x` ile değiştirirlir. Ancak bunun dezavantajı, büyük jenerik fonksiyonların her için çok fazla kod üretmesidir ki buna *kod şişmesi (code bloat)* denir. Her zaman bir takas vardır ve deneyimli bir kişi hangi iş için doğru aracı seçeceğini bilmelidir.

# Basit Numaralandırmalar
Numaralandırmalar (Enums) birkaç verisi bulunan tiplerdir. Örneğin, bir yön dört farklı şekil alabilir:

```rust
enum Direction {
    Up,
    Down,
    Left,
    Right
}
...
    // `start` is type `Direction`
    let start = Direction::Left;
```

Çeşitli metotlar alabilirler, tıpkı yapılar gibi. `Match` ifadesi `enum` tiplerini kontrol etmenin en basit yoludur.

```rust
impl Direction {
    fn as_str(&self) -> &'static str {
        match *self { // *self has type Direction
            Direction::Up => "Up",
            Direction::Down => "Down",
            Direction::Left => "Left",
            Direction::Right => "Right"
        }
    }
}
```

Noktalama da önemlidir. `self`'ten önce `*` operatörünü kullandığımıza dikkat edin. Unutması kolaydır çünkü çoğu zaman Rust böyle düşünür. (`self.first_name` deriz, `(*self).first_name` değil.) Fakat eşleştirmenin biraz daha net olması gerekir. Olduğu gibi bırakmak buna kadar varan bir sürü çıktıya sebep olur:

```
   = note: expected type `&Direction`
   = note:    found type `Direction`
```


Çünkü `self` `&Direction` tipidir, bundan dolayı `*` ile deferans ederiz. 

Yapılar gibi numaralandırmalar da özellikleri içerebilir, `#[derive(Debug)]` arkadaş da `Direction`'a eklenebilir.

```rust
        println!("start {:?}",start);
        // start Left
```

Yani `as_str` metotu aslında o kadar da gerekli değil, `Debug` ile isimleri her zaman alabiliriz. (Ancak `as_str` alan tahsis etmez, ki bu önemli olabilir.)

Ancak burada net bir sıralama aramamalısınız - numaralandırmalar tam sayı değeri barındırmaz. 

(Ç.N: Numaralandırma olarak çevrilen `enum` sözcüğü gerçekten de C ve C++'da sayılandırma işlemi için kullanılır ancak Rust'ta böyle bir özellik yoktur. C++'daki karşılığı `enum` değil, `enum class`'tır.)

Şimdi her `Direction` değerinin ardılını gösteren bir metot yazdık. *Yıldız jokerini* kullanmak metotun içeriğine bütün numaralandırma değerlerini sıraladığı için epey kullanışlıdır.

```rust
    fn next(&self) -> Direction {
        use Direction::*;
        match *self {
            Up => Right,
            Right => Down,
            Down => Left,
            Left => Up
        }
    }
    ...

    let mut d = start;
    for _ in 0..8 {
        println!("d {:?}", d);
        d = d.next();
    }
    // d Left
    // d Up
    // d Right
    // d Down
    // d Left
    // d Up
    // d Right
    // d Down
```

Bu şekilde istenen ve belirlenmiş düzende bütün yönleri sonsuza dek sıralamaya izin verir. (Aslında) bu oldukça basit bir *durum makinesidir*.

Numaralandırma veriler kıyaslanamaz:

```
assert_eq!(start, Direction::Left);

error[E0369]: binary operation `==` cannot be applied to type `Direction`
  --> enum1.rs:42:5
   |
42 |     assert_eq!(start, Direction::Left);
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
note: an implementation of `std::cmp::PartialEq` might be missing for `Direction`
  --> enum1.rs:42:5
```
Çözüm, `enum Direction` tanınımının üstüne `#[derive(Debug,PartialEq)]` eklemektir.

Önemli bir nokta, Rust'ın kullanıcı tiplerinin bir eklenti ile birlikte gelmemesidir. Genel özellikleri (trait) tanımlayarak onlara olağan davranışları tanımlarsınız. Bu yapılar için de geçerlidir - eğer bir yapıya `PartialEq` verirseniz akla yatkın bir şey belirleyecektir, tüm alanların `PartialEq`'e sahip olduğunu düşünerek bir kıyas yapacaktır. Eğer alanlar buna sahip değilse, eşitliği tanımlananız gerekmektedir ki bunu açıkça tanımlamanıza izin vardır.

Rust'ta "C tarzı numaralandırmalar" da kullanılabilir.

```rust
// enum2.rs

enum Speed {
    Slow = 10,
    Medium = 20,
    Fast = 50
}

fn main() {
    let s = Speed::Slow;
    let speed = s as u32;
    println!("speed {}", speed);
}
```

İlklendiği zaman tam sayı değeri alırlar ve tip dönüşümleriyle tam sayıya dönüşebilirler.

Bunun için sadece ilk isme değer vermeniz yeterlidir, diğerleri de bir arttırarak onu takip edecektir:

```rust
enum Difficulty {
    Easy = 1,
    Medium,  // is 2
    Hard   // is 3
}
```

Tabii bunun isim olarak çağırınca havada kaldı, tıpkı her şeye "şey" demek gibi. Esas kullanılması gereken terim *varyanttır* - `Speed`in varyantları `Slow`, `Medium` ve `Fast`tır.

Numaralandırmalar doğal bir sıralama da alabilir, ancak bunu kibarca istemelisiniz. `enum Speed`'in başına `#[derive(PartialEq,PartialOrd)]` ekledikten sonra `Speed::Fast > Speed::Slow` ve `Speed::Medium != Speed::Slow` gibi ifadeler kullanılabilir olur.

# Numaralandırmalar Tam Teçhizatlıyken
Rust'ın numaralandırmaları tam anlamıyla kullanıldığı zaman C'deki birliklerin (union) steroidli hâline benzer, tıpkı Ferrari ile Fiat Uno arasındaki ilişki gibi. Çeşitli tiplerden verileri bir araya güvenlice toplamanın zorluğunu düşünün.

```rust
// enum3.rs

#[derive(Debug)]
enum Value {
    Number(f64),
    Str(String),
    Bool(bool)
}

fn main() {
    use Value::*;
    let n = Number(2.3);
    let s = Str("hello".to_string());
    let b = Bool(true);

    println!("n {:?} s {:?} b {:?}", n,s,b);
}
// n Number(2.3) s Str("hello") b Bool(true)
```

Numaralandırma bu verilerden sadece birisini taşıyabilir, büyüklüğü bir varyantın en büyük değeri kadardır.

Şimdiye kadar bir süper araba etmese de numaralandırmaların kendilerini yazdırabilmeleri de güzel şey. Bunun yanında verilerinin ne tarz veriler olduğunu da biliyorlar ki bu `match`'ın süpergücüdür. 

```rust
fn eat_and_dump(v: Value) {
    use Value::*;
    match v {
        Number(n) => println!("number is {}", n),
        Str(s) => println!("string is '{}'", s),
        Bool(b) => println!("boolean is {}", b)
    }
}
....
eat_and_dump(n);
eat_and_dump(s);
eat_and_dump(b);
//number is 2.3
//string is 'hello'
//boolean is true
```

(`Result` ve `Option` kardeşleri hatırladınız mı? Onlar da bir numaralandırma.)

`eat_and_dump` fonksiyonu hiç fena değil ancak veriyi bir referans olarak iletsek iyi olur çünkü şu an verinin yerini taşıyor ve onu "yiyor":

```rust
fn dump(v: &Value) {
    use Value::*;
    match *v {  // type of *v is Value
        Number(n) => println!("number is {}", n),
        Str(s) => println!("string is '{}'", s),
        Bool(b) => println!("boolean is {}", b)
    }
}

error[E0507]: cannot move out of borrowed content
  --> enum3.rs:12:11
   |
12 |     match *v {
   |           ^^ cannot move out of borrowed content
13 |     Number(n) => println!("number is {}",n),
14 |     Str(s) => println!("string is '{}'",s),
   |         - hint: to prevent move, use `ref s` or `ref mut s`
```

Ödünç alınmış referanslarla yapamayacağınız bazı şeyler var. Rust, orijinal değerin içerisindeki karakter dizisini *dışarı çıkartmanıza* izin vermeyecektir. `Number` üzerinde sorun yok çünkü `f64`'ün kopyalanmasında bir sakınca yok ama `String` `Copy`'i içermez.

`match`'ın kesin tipler hakkında seçici olduğunu söyledim, şimdi ipucunu takip edelim ve sıkıntı çıkartmayacaktır, şimdi içerideki karakter dizisine bir referans ödünç alıyoruz.

```rust
fn dump(v: &Value) {
    use Value::*;
    match *v {
        Number(n) => println!("number is {}", n),
        Str(ref s) => println!("string is '{}'", s),
        Bool(b) => println!("boolean is {}", b)
    }
}
    ....

    dump(&s);
    // string is 'hello'
```

Devam etmeden önce, başarılı bir Rust derlemesinin mutluluğu ile dolup taşmışken, bir saniye bekleyelim. `Rustc` o kadar iyi ki sorunu tam olarak *anlamadan* onu çözmemizi sağlıyor. 

Sorun, eşleştirmenin kesinliğinden ve ödünç kontrolünün kuralların çiğnenmemesinden kaynaklanıyor. Bu kurallardan birisi, sahipliği olan bir tipe dahil olan veriyi zart diye çekemiyor olmamızdan geliyor. Biraz C++ bilmek burada kafa karıştırabilir çünkü akla yatkın olsa bile C++ problemin yolunu kopyalayacaktır. Bir vektörden karakter dizesi alırken de aynı hatayı alabilirsiniz, mesela `*v.get(0).unwrap` ile deneyin. (`*` kullanmanızın sebebi indekslemenin referans dönmesi) Buna yapmanıza izin vermecektir. (Bu tarz durumlarda `Clone` çok da kötü bir tercih olmayabilir.)

(Bu arada, `v[0]` karakter dizeleri gibi kopyalanamaz verilerde tam olarak bundan dolayı çalışmayacaktır. `&v[0]` ile ödünç almanız ya da `v[0].clone()` kullanmanız gerekmektedir.)


`match` kullanırken `Str(s: String) =>` yerine `Str(s)` yazıldığını görebilirsiniz. Yeni bir yerel değişken yaratılır. (bazen *bağlama (binding)* olarak anılır) Çoğu zaman tatmin edilen tip tutar, veriyi alıp onun içinden çıkartırken. Ancak burada `s: &String` yazmaya ihtiyacımız oldu ve `ref` ile sadece `String`'i ödünç almak istediğimizi bildirmiş olduk.

Burada da bir karakter dizisini dışarı çıkartıyoruz ve değerin daha sonra ne olacağını umursamıyoruz. `_` her şeyle eşleşecektir.

```rust
impl Value {
    fn to_str(self) -> Option<String> {
        match self {
        Value::Str(s) => Some(s),
        _ => None
        }
    }
}
    ...
    println!("s? {:?}", s.to_str());
    // s? Some("hello")
    // println!("{:?}", s) // error! s has moved...
```

İsimlendirme önemlidir -, `as_str` olarak değil de `to_str` olarak tanımlamamıza dikkat edin. (Ç.N: To Str - Str'ye çevir, As Str - Str olarak) Bir karakter dizisini `Option<&String>` olarak dönen  bir metot yazabilirsiniz. (Referansın da numaralandırma değeri ile aynı yaşam süresinde olmasına gerek vardır) Ancak onu `to_str` olarak isimlendirmemelisiniz. 

`to_str` örneğimizi şöyle yazabilirsiniz - tamamen aynı işi yapar:

```rust
    fn to_str(self) -> Option<String> {
        if let Value::Str(s) = self {
            Some(s)
        } else {
            None
        }
    }
```

# Eşleştirme Hakkında Daha Fazlası
"()" kullanarak bir demeti dışarı çıkartabileceğinizi hatırladınız mı?

```rust
    let t = (10,"hello".to_string());
    ...
    let (n,s) = t;
    // t has been moved. It is No More
    // n is i32, s is String
```

Bu *parçalama* işleminin özel bir durumudur; elimizdeki bazı veriler var ve (buradaki gibi) parçalara ayırmayı ya da verilerini ödünç almayı düşünebiliriz. Her iki durum da da bir bütünün parçalarına ulaşmaya çalışıyoruz.

Sözdizimi `match`'taki gibi kullanılabilir. Burada açıkça ödünç alınmış verileri ödünç alıyoruz.

```rust
    let (ref n,ref s) = t;
    // n and s are borrowed from t. It still lives!
    // n is &i32, s is &String
```

Yapıları parçalamak da pekâlâ mümkün:

```rust
    struct Point {
        x: f32,
        y: f32
    }

    let p = Point{x:1.0,y:2.0};
    ...
    let Point{x,y} = p;
    // p still lives, since x and y can and will be copied
    // both x and y are f32
```

`match`'ı yeni örüntülerle tekrar inceleyelim. İlk iki örüntü `let` parçalaması gibi çalışır - ilki ilk elemanı sıfır olan, ikinci indeksi karakter dizesi olan her türlü demetle eşleşir, ikincisi ise sadece `(1, "hello")` ile eşleşir. Son koşulda ise olarak, bir değişken *herhangi bir şeyle* eşleşir. Eğer `match` bir ifadeyi eşleştiriyorsa ancak bunu değişkene bağlamak istemiyorsanız bu epey kullanışlıdır. `_` da bir değişken gibi çalışır ancak görmezden gelinir, bir `match`'ı bitirmenin yaygın bir yoludur.

```rust
fn match_tuple(t: (i32,String)) {
    let text = match t {
        (0, s) => format!("zero {}", s),
        (1, ref s) if s == "hello" => format!("hello one!"),
        tt => format!("no match {:?}", tt),
        // or say _ => format!("no match") if you're not interested in the value
     };
    println!("{}", text);
}
```

Peki neden sadece `(1, "hello")` kullanmıyoruz? Eşleştirme kesin olarak çalışır ve derleyici de bundan bahsedecektir:

```
  = note: expected type `std::string::String`
  = note:    found type `&'static str`
```

Neden `ref s`'e ihtiyacımız var? Bu biraz belirsiz bir durum (E00008 numaralı hataya bakın.) ve eğer bir koşula bağlayacaksanız bunu ödünç almanız gerekir, koşula bağlamanız farklı bir bağlamda gerçekleştiğinden bellekteki alanın taşınması gerekebilir. Bu, işin en civcivli olduğu yerlerden birisi.

Eğer tipimiz `&str` olsaydı bunu doğrudan eşleştirebilirdik:

```rust
    match (42,"answer") {
        (42,"answer") => println!("yes"),
        _ => println!("no")
    };
```

`match` için geçerli olan `if let` için de geçerlidir. Bu mesela güzel bir örnek, bir `Some` verimiz olduğu için içindeki veriyi çekebiliriz ve içinden sadece bir karakter dizisini çıkartabiliriz. İç içe geçmiş `if let` ifadelerine ihtiyacımız da yok üstelik. Burada `_` kullanıyoruz çünkü demetin ilk parçası ilgimizi çekmiyor. 

```rust
    let ot = Some((2,"hello".to_string());

    if let Some((_,ref s)) = ot {
        assert_eq!(s, "hello");
    }
    // we just borrowed the string, no 'destructive destructuring'
```

`parse` ile ilgili bir enteresan bir sorunumuz da var. (Ya da dönüş tipini bilmesi gereken fonksiyonlar için de bunu düşünebiliriz)

```rust
    if let Ok(n) = "42".parse() {
        ...
    }
```

`n`'in tipi nedir? Bir ipucu vermeniz gerekir, ne tür bir tam sayılı değer bu? Hatta bu tam sayı mıdır?

```rust
    if let Ok(n) = "42".parse::<i32>() {
        ...
    }
```

Bu rezil söz diziminin adı "[turbofish operatörüdür](https://turbo.fish/)". 

Eğer `Result` dönen bir fonksiyonun içerisindeyseniz, soru işareti ile çok daha şık bir çözüm kullanabilirsiniz:

```rust
    let n: i32 = "42".parse()?;
```

Her neyse, herhangi bir `parse` hatası `Result`'ın hata tipine dönüştürülebilir bir tipe ihtiyaç duyar ki bunu sonra [hata kontrolü](#bu_yayınlanınca_düzenlenecek) kısmında ele alacağız.

# Kapamalar (Closure)

Rust'ın gücünün büyük bir kısmı bu kapamalardan gelir. En basit hâliyle bir fonksiyonun kısa yoluna benzerler:

```rust
    let f = |x| x * x;

    let res = f(10);

    println!("res {}", res);
    // res 100
```

Burada açıkça belirtilmiş bir tip yoktur - bir "10" tam sayı kalıbının kullanılmasına kadar her şey birer tahmin edilmiştir. 

Ancak `f`'i farklı farklı tipler için kullanırsak hata alırız - Rust `f`'in tam sayılarla çalışması gerektiğine karar vermişti.

```
    let res = f(10);

    let resf = f(1.2);
  |
8 |     let resf = f(1.2);
  |                  ^^^ expected integral variable, found floating-point variable
  |
  = note: expected type `{integer}`
  = note:    found type `{float}`
```

İlk kullanım `x` için argümanı belirlemişti. Aslında yaptığımız şey şudur:

```rust
    fn f (x: i32) -> i32 {
        x * x
    }
```

Ancak açıkça tiplerin yazılması gerektiği sorunun dışında fonksiyonlar ve kapamaların bir farkı daha vardır. Doğru fonksiyonunu inceleyelim:

```rust
    let m = 2.0;
    let c = 1.0;

    let lin = |x| m*x + c;

    println!("res {} {}", lin(1.0), lin(2.0));
    // res 3 5
```

Bunu `fn` ile böyle yapamayız, kapsamının dışında kalan hiçbir şeyle `fn` ilgilenmez. Buradaki kapama, `m` ve `c`'yi kendi kapsamı içerisine ödünç aldı.

Peki ya `lin`'in tipi nedir? Ancak `rustc` bilebilir. Aslında görünenin altında kapama, çağrılabilir bir (çağırma operatörünü içeren bir) *yapıdır (struct).* Şu şekilde yazılmış gibi davranır:

```rust
struct MyAnonymousClosure1<'a> {
    m: &'a f64,
    c: &'a f64
}

impl <'a>MyAnonymousClosure1<'a> {
    fn call(&self, x: f64) -> f64 {
        self.m * x  + self.c
    }
}
```

Derleyici bu konuda epey yardımcı oluyor ve basit bir kapamayı buna dönüştürüyor! Tek bilmeniz gereken kapama bir *yapıdır* ve verileri içinde bulunduğu çevreden *ödünç alır.* Bu referansların da bir *yaşam ömrü* vardır. 

Bütün kapamaların benzersiz tipleri vardır ancak benzer özellikleri (trait) içerirler. Türü tam bilmesek de en azından jeneriklerde nasıl ifade edeceğimi biliyoruz:

```rust
fn apply<F>(x: f64, f: F) -> f64
where F: Fn(f64)->f64  {
    f(x)
}
...
    let res1 = apply(3.0,lin);
    let res2 = apply(3.14, |x| x.sin());
```

El-meal: `apply` `Fn(f64)->f64`'e sahip herhangi bir T tipi ile çalışabilir - yani `f64` alıp `f64` dönen bir fonksiyon olabilir bu.

`apply(3.0, lin)` şeklinde çağırdıktan sonra `lin`'e erişmek şu tuhaf hatayı ortaya çıkartıyor:

```
    let l = lin;
error[E0382]: use of moved value: `lin`
  --> closure2.rs:22:9
   |
16 |     let res = apply(3.0,lin);
   |                         --- value moved here
...
22 |     let l = lin;
   |         ^ value used here after move
   |
   = note: move occurs because `lin` has type
    `[closure@closure2.rs:12:15: 12:26 m:&f64, c:&f64]`,
     which does not implement the `Copy` trait
```

Ve bu kadar, `apply` bizim kapamamızı yedi. Ve ayrıca, `rustc`'nin kullanmaya çalıştığı yapının (struct) gerçek tipi. Kapamaları bir yapı olarak düşünmek işi epey kolaylaştırıyor.

Bir kapama çağırmak aslında *metot çağrısıdır*: Üç tip fonksiyon özelliği (trait) üç tip metoda sahiptir:
- `Fn`, `&self` olarak geçer.
- `FnMut`, `&mut self` olarak geçer.
- `FnOnce` ise sadece `self` olarak geçer.

 Bir kapama içerisinde *yakalanmış referansları* düzenlemek de mümkündür.
 
 ```rust
    fn mutate<F>(mut f: F)
    where F: FnMut() {
        f()
    }
    let mut s = "world";
    mutate(|| s = "hello");
    assert_eq!(s, "hello");
```

`mut`'a dikkat edin - `f`'in değişebilir olması gerekiyor.

Yine de, ödünç alma ile ilgili kurallardan kaçınamazsınız. Şuna bakın:

```rust
let mut s = "world";

// closure does a mutable borrow of s
let mut changer = || s = "world";

changer();
// does an immutable borrow of s
assert_eq!(s, "world");
```

Çalışamaz! Çünkü `s`'i `assert` deyiminde ödünç alamıyoruz, çünkü daha önce `changer` kapamasında değişken olarak ödünç almıştır. Kapama düşürülmediği sürece `s`'e hiç kimse erişemez, bundan dolayı iç bir kapsam alanı içerisinde kullanarak yaşam süresini kontrol etmek en iyi çözümdür.

```rust
let mut s = "world";
{
    let mut changer = || s = "world";
    changer();
}
assert_eq!(s, "world");
```

Eğer Lua ve JavaScript gibi dillere aşinaysanız, bu dillerde gayet sade iken Rust'ta kapamaların bu denli karmaşık olduğunu merak ediyor olabilirsiniz. Bu, Rust'ın gizlice bellek tahsis etmemesi için gerekli bir bedeldir. JavaScript'te, `mutate(function() {s = "hello";})` gibi bir ifadenin karşılığı her zaman dinamik bellek tahsis edilmiş kapamadır. 

Bazen kapamaların verileri ödünç almasını değil direkt taşımasını isteyebilirsiniz.

```rust
    let name = "dolly".to_string();
    let age = 42;

    let c = move || {
        println!("name {} age {}", name,age);
    };

    c();

    println!("name {}",name);
```

Burada alacağımız hata son `println`'dadır: "taşınmış verinin kullanımı: `name` (use of moved value: `name`)". Burada tek bir çözüm var, kapamanın içine verinin klonunu taşımak: 

```rust
    let cname = name.to_string();
    let c = move || {
        println!("name {} age {}",cname,age);
    };
```

Neden taşıyan kapamalara ihtiyacımız var? Çünkü orijinal verinin erişilemeyeceği bir durumda onları çağırmamız gerekebilir. En basit örneği *iş parçacıklarıdır.* Taşıyan kapamalar ödünç almaz, bundan dolayı yaşam süresi açısından hiçbir sorunları olmaz.

Kapamaların esas kullanımı döngüleyici metotlarıdır. Noktalı sayılar için hazırladığımız `range` döngüleyicisini hatırlayın. Kapama kullanarak bu döngüleyici (veya başka döngüleyiciler) üzerinde işlem yapmak gayet kolay:

```rust
    let sine: Vec<f64> = range(0.0,1.0,0.1).map(|x| x.sin()).collect();
```

`map` vektörler üzerinde tanımlanmadı (Bunu kullanan bir özellik (trait) yaratmak oldukça kolay olmasına rağmen) çünkü `map`'ın yeni bir vektör yaratması gerekirdi. Bu şekilde elimizde seçeneklerimiz oluyor. Üstelik, geçici hiçbir öğe yaratılmış olmuyor:

```rust
 let sum: f64 = range(0.0,1.0,0.1).map(|x| x.sin()).sum();
```

(İşin özü) Tıpkı bir döngü yazmak kadar kadar hızlı. Eğer Rust kapamaları JavaScript kapamaları kadar "acısız" olsaydı bu performansı garanti edemezdik.

`filter` da ayrıca bir iterator metotudur - geriye sadece koşullara uyanlar kalır:

```rust
    let tuples = [(10,"ten"),(20,"twenty"),(30,"thirty"),(40,"forty")];
    let iter = tuples.iter().filter(|t| t.0 > 20).map(|t| t.1);

    for name in iter {
        println!("{} ", name);
    }
    // thirty
    // forty
```

# Üç çeşit döngüleyici
Üç farklı çeşit (yine) üç basit argüman tipine denk düşüyor. Bir `String` vektörümüz olduğunu düşünelim. Bunlar bizim döngüleyici tiplerimiz, ilk üçü aleni bir şekilde sonraki üçü de gizil bir şekilde belirtilmiştir. 

```rust
for s in vec.iter() {...} // &String
for s in vec.iter_mut() {...} // &mut String
for s in vec.into_iter() {...} // String

// implicit!
for s in &vec {...} // &String
for s in &mut vec {...} // &mut String
for s in vec {...} // String
```

Şahsen ben aleni bir şekilde ifade etmeyi tercih ediyorum, ancak iki formu da anlamak ve nasıl kullanıldığını bilmek önemlidir. 

`into_iter` vektörü tüketir ve içeriğindeki karakter dizilerini çıkartır, ve ardından artık vektör kullanılamaz - artık taşınmış olur. Pythonistalar alışkanlıktan `for s in vec` dediği zaman başlarına bu gelir.

`for s in &vec` şeklindeki gizil form muhtemelen kullanmak isteyeceğiniz şekildir, tıpkı fonksiyon argümanlarında `&T` kullanmak gibi.

Üç çeşidi de anlamak önemlidir çünkü Rust tip tahminlerini epeyce kullanır - kapama argümanlarında tip bildirimlerini pek görmezsiniz. Bu iyi bir şey çünkü bu tiplerin hepsi yazılsaydı kafa şişirici olurdu. Ancak, bu ufak kodun bedeli gizil tiplerin ne olduğunu net olarak bilmenizin gerekmesidir!

`map` döngüleyicinin değerini ne olursa olsun alır ve onu başka bir şeye dönüştürür, ancak `filter` veriye bir *referans* alır. Aşağıda, `iter` kullanıyoruz ve bundan dolayı döngüleyici tipi `&String`tir. `filter`'ın her veriyi referansını aldığını gözden kaçırmayın:
```rust
for n in vec.iter().map(|x: &String| x.len()) {...} // n is usize
....
}

for s in vec.iter().filter(|x: &&String| x.len() > 2) { // s is &String
...
}
```

Metotları çağırdığınız zaman Rust kendiliğinden dereferans eder, ondan dolayı sorunu pek anlamazsınız. Ancak `|x: &&String| x == "one"` çalışmayacaktır çünkü operatörler tip eşleştirmesinden daha katıdır. `rustc`, `&str` ve `&&String`'i kıyaslayacak bir operatör olmadığını bildirecektir. Bundan dolayı eşleşme yapabilmek için `&&String`'i `&String`e çevirmek için dereferans etmeniz gerekecektir.

```rust
for s in vec.iter().filter(|x: &&String| *x == "one") {...}
// same as implicit form:
for s in vec.iter().filter(|x| *x == "one") {...}
```

Eğer tipleri bildirmeyi bırakırsanız, argümanı şu şekilde düzeltebilirsiniz ki bu sefer s'in tipi `&String` olur.

```rust
for s in vec.iter().filter(|&x| x == "one")
```

Ve çoğu zaman bu şekilde yazıldığını görürsünüz.

# Dinamik Verili Yapılar
*Kendisine yönlenen yapı* tekniği en güçlü tekniktir.

Aşağıda C ile yazılmış bir *ikili ağacın* temel tuğlasını görüyorsunuz. (C... Âdeta Beyoğlu'nun arka sokakları gibi... "Acaba başıma ne gelecek?" demeden dolaştığınız tarihî sokaklarda nefes kesici bir gezi...)

```rust
    struct Node {
        const char *payload;
        struct Node *left;
        struct Node *right;
    };
```


Bunu doğrudan `Node` alanlarını içererek yapamazsınız çünkü `Node`'un büyüklüğü yine `Node`'a dayanır. Ki bu hesaplanamaz. Bundan dolayı `Node` yapılarının göstericilerini (pointer) kullanıyoruz, ki göstericinin boyutu her zaman kestirilebilir.

Eğer `left`, `NULL` değilse `Node`'un `left` tarafı bir başka `Node` gösteriyordur ve bu böyle sonsuza kadar gidebilir.

Rust'ta `NULL` yoktur (en azından bu güvensiz hâliyle yok), bu Option'un işidir. Ancak `Node`'u doğrudan `Option` içerisine ekleyemezsiniz çünkü `Node`'un boyutunu bilemezsiniz. (gibi gibi) Bu da `Box`'un işidir, kendisinin sabit bir boyutu vardır ancak bellekte alanı tahsis edilmiş veriyi içerir. 

İşte Rust'taki karşılığına bakalım, `type` ile tipimize bir takma ad verdik:

```rust
type NodeBox = Option<Box<Node>>;

#[derive(Debug)]
struct Node {
    payload: String,
    left: NodeBox,
    right: NodeBox
}
```

(Rust işte böyle kalender meşreptir, ileriye dönük bildirimlere ihtiyacınız yoktur.)

Şimdi bunu test edelim:

```rust
impl Node {
    fn new(s: &str) -> Node {
        Node{payload: s.to_string(), left: None, right: None}
    }

    fn boxer(node: Node) -> NodeBox {
        Some(Box::new(node))
    }

    fn set_left(&mut self, node: Node) {
        self.left = Self::boxer(node);
    }

    fn set_right(&mut self, node: Node) {
        self.right = Self::boxer(node);
    }

}


fn main() {
    let mut root = Node::new("root");
    root.set_left(Node::new("left"));
    root.set_right(Node::new("right"));

    println!("arr {:#?}", root);
}
```

Çıktı beklediğimizden çok daha iyi, "{:#?}" sağolsun. ("#" genişletilmiş demektir.)

```
root Node {
    payload: "root",
    left: Some(
        Node {
            payload: "left",
            left: None,
            right: None
        }
    ),
    right: Some(
        Node {
            payload: "right",
            left: None,
            right: None
        }
    )
}
```

Peki ya `root` düşerse? Bütün alanlar da düşer, ağacın `dalları` düşerse kendi alanlarını da kaybolur ve böyle devam eder. `Box::new`, `new` anahtar kelimesine en çok ulaşacağınız alandır ancak `delete` veyahut `free` gibi bir kelimeye ihtiyacınız yoktur.

Bu ağacı kullanmak için bir yol bulmalıyız. Karakter dizilerinin sıralanabildiğine dikkat edin: "hede" < "hödö", "ayı" > "abi"; sözde alfabetik sıralama olarak anılır. (Aslını söylemek gerekirse, insan dillerinin çeşitliliğinden ve tuhaf kurallarına istinaden buna sözlüksel sıralama denir.)

Aşağıda `Node`ları sözlüksel sıralamaya göre yerleştiren bir metot görüyorsunuz. Veriyi mevcut `Node` ile kıyaslıyoruz - eğer küçükse soluna yerleştiriyoruz, değilse de sağına yerleştirmeye çalışıyoruz. Solda bir `Node` olmayabilir, bundan dolayı `set_left` kullanıyoruz.

```rust
    fn insert(&mut self, data: &str) {
        if data < &self.payload {
            match self.left {
                Some(ref mut n) => n.insert(data),
                None => self.set_left(Self::new(data)),
            }
        } else {
            match self.right {
                Some(ref mut n) => n.insert(data),
                None => self.set_right(Self::new(data)),
            }
        }
    }

    ...
    fn main() {
        let mut root = Node::new("root");
        root.insert("one");
        root.insert("two");
        root.insert("four");

        println!("root {:#?}", root);
    }
```

`match`'a dikkat edin - `Box` içerisinden değişken bir referans çıkartıyoruz, eğer `Option`'un içeriği `Some` ise `insert` kullanıyoruz. Değilse, sol tarafa yeni bir `Node` ekliyoruz ve böyle devam ediyoruz. `Box`, akıllı bir göstericidir; `Node` metotlarını çağırmak için "kutudan çıkarmamıza" gerek yok!   

İşte ağacımızın görüntüsü:

```
root Node {
    payload: "root",
    left: Some(
        Node {
            payload: "one",
            left: Some(
                Node {
                    payload: "four",
                    left: None,
                    right: None
                }
            ),
            right: None
        }
    ),
    right: Some(
        Node {
            payload: "two",
            left: None,
            right: None
        }
    )
}
```

Diğerlerinden daha "küçük" olan karakter dizileri sol eklenir, aksi durumda ise sağa eklenirler.

Şimdi gezinti zamanı. Bu *iç-sıralı gezinmedir. (inorder traversal)* - önce solu ziyaret ediyoruz, bir şeyler yapıyoruz ve sonra da sağa geçiyoruz.

```rust
    fn visit(&self) {
        if let Some(ref left) = self.left {
            left.visit();
        }
        println!("'{}'", self.payload);
        if let Some(ref right) = self.right {
            right.visit();
        }
    }
    ...
    ...
    root.visit();
    // 'four'
    // 'one'
    // 'root'
    // 'two'
```

Karakter dizilerini bir sıralamaya göre geziyoruz! `ref`'in `if let` için kullanıldığına dikkat edin, `match` ile aynı kurallara sahiptir.

# Jenerik Yapılar
Önceki örneğimizde kullandığımız ikili ağaç yapısını düşünün. Bütün `payload` tipleri için yeniden yazmak epey *çıldırtıcı* olurdu doğrusu. `T` tip parametresiyle `Node`'u yeniden jenerik şekilde yazıyoruz.

```rust
type NodeBox<T> = Option<Box<Node<T>>>;

#[derive(Debug)]
struct Node<T> {
    payload: T,
    left: NodeBox<T>,
    right: NodeBox<T>
}
```

Bu kullanım diller arasındaki farkları da belli ediyor. `Payload` üzerindeki temel işlem karşılaştırmadır, bundan dolayı T ile `<` kullanılabilmelidir ki `PartialOrd` bunu sağlar. Tip parametresi `impl` bloğu içerisinde özellik kısıtlamasıyla birlikte yazılmalıdır.

```rust
impl <T: PartialOrd> Node<T> {
    fn new(s: T) -> Node<T> {
        Node{payload: s, left: None, right: None}
    }

    fn boxer(node: Node<T>) -> NodeBox<T> {
        Some(Box::new(node))
    }

    fn set_left(&mut self, node: Node<T>) {
        self.left = Self::boxer(node);
    }

    fn set_right(&mut self, node: Node<T>) {
        self.right = Self::boxer(node);
    }

    fn insert(&mut self, data: T) {
        if data < self.payload {
            match self.left {
                Some(ref mut n) => n.insert(data),
                None => self.set_left(Self::new(data)),
            }
        } else {
            match self.right {
                Some(ref mut n) => n.insert(data),
                None => self.set_right(Self::new(data)),
            }
        }
    }
}


fn main() {
    let mut root = Node::new("root".to_string());
    root.insert("one".to_string());
    root.insert("two".to_string());
    root.insert("four".to_string());

    println!("root {:#?}", root);
}
```

Tıpkı C++ gibi jenerik yapımız tip parametrelerinin köşeli ayraçlarla gösterilmesine ihtiyaç duyar. Rust genellikle bu tür tip parametresini bağlamdan tahmin edebilecek kadar zekidir - Bunun `Node<T>` olduğunu biliyor ve `T` üzerinde `insert` kullanıyor. İlk `insert` tasarısı sadece `String` ile takılıp kalmıştı. Ancak yeni kullanım uymuyorsa muhtemelen bir şekilde bunu bildirecektir.

Ancak, tipi uygun biçimde kısıtlamanız gerektiğine dikkat edin.