## Rust ve Nesne Yönelimli Programlama
Her disiplinden birileri geliyor ve Nesne Yönelimli bir dilden gelmeniz olasılığınız oldukça yüksektir:

- "Sınıf"lar nesne (kimi zaman *örnek* (instance) deriz) üreten fabrikalar gibi çalışırlar.
- Sınılar diğer sınıfların (*üst sınıf/ebeveyn*) alanlarını (*field*) ve davranışlarını (*metotlar*) *miras (inheritance)* alabilir.
- Eğer B, A'dan miras alıyorsa, o zaman B aynı zamanda A olarak kabul edilebilir. (*alttip/subtyping*)
- Nesne kendi verilerini gizlemelidir (*kapsülleme/encapsulation*), sadece metotlarıyla etkileşime geçmelidir.
Nesne yönelimli *tasarım* sınıfların (yani isimleri) ve yöntemlerin (yani sıfatları) tanımlandığı anlayıştır. Aynı zamanda nesne yönelimli tasarımda bunlar arasındaki ilişkiler de tanımlanır ve bu ilişkiler *sahip olmak (has-a*) ve *onun türünden olmak (is-a)* olarak tanımlanır.

Eski Star Trek serilerinde doktorun kaptana "Bu bir yaşam Jim, sadece bizim bildiğimiz yaşam değil" (It's Life, Jim, just not Life as we know it) dediği bir an vardır. Bu deyim aynen olduğu gibi Rust'taki nesne yönelimi anlayışını da açıklıyor: Önce "Bu ne ya?" diyorsunuz, çünkü Rust'taki veri yapıları (yapılar, numaralandırmalar ve demetler) pek de uçan kaçan tarzda değiller. Onlara metot tanımlayabilirsiniz, verinin kendisini gizleyebilirsiniz ve bütün kapsülleme yöntemlerini kullanabilirsiniz ancak her birisi *birbiriyle alakasız tiplerdir.* Alt tipler yoktur, miras alma yöntemi yoktur. (Sadece `Deref` zorlamaları miras olarak kabul edilebilir [^yorum])

[^yorum]: Çevirmenin yorumu: Nesne yönelimli programlamada miras alan tip, doğal olarak miras aldığı tip olduğu kabul edilir. Rust'ta bir literatür farklılığı vardır, o da tipin aslında o olmadığı ancak zorlanarak dönüştürüldüğü vurgusudur. Yani bu bir miras alma mıdır? Evet, belki de. Ancak Rust, "coercion" kelimesi ile Rust'ın bir tipi başka bir tipe zorla dönüştürdüğünü vurgulamaktadır.

Çeşitli tipler arasındaki ilişkiler *özellikler (trait)* ile inşa edilir. Rust öğrenme sürecinin büyük bir kısmı standart kütüphanede bulunan özelliklerin nasıl çalıştığını anlamaktan geçer çünkü bu bütün verileri birbiriyle çalışmasına izin veren ve onlara anlamlar yükleyen bir sistemdir.

Özellik sistemi ilginçtir çünkü ana akım programlama dillerinde birebir benzeri yoktur.  Tabii, bu dinamiklik ya da statiklik arasındaki farklı düşünürsek. Dinamik olarak Java ve Go arayüzlerine (interface) benzerler.

### Özellik Nesneleri
Özellikleri anlatırken kullandığımız ilk örneği düşünün:

 ```rust
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
```
Bu da büyük `impl` bloklarıyla beraber ufak bir program:

```rust
# trait Show {
#     fn show(&self) -> String;
# }
#
# impl Show for i32 {
#     fn show(&self) -> String {
#         format!("four-byte signed {}", self)
#     }
# }
#
# impl Show for f64 {
#     fn show(&self) -> String {
#         format!("eight-byte float {}", self)
#     }
# }

fn main() {
let answer = 42;
let maybe_pi = 3.14;
let v: Vec<&Show> = vec![&answer,&maybe_pi];
for d in v.iter() {
println!("show {}",d.show());
}
}
// show four-byte signed 42
// show eight-byte float 3.14
```
Burası Rust'ın tip bildirimi gerektirdiği bazı nadir noktalardan birisi - `Show` özelliğini içeren herhangi bir vektörü açıkça istemek durumundayım. `i32` ve `f64` arasında hiçbir şey bağlantı olmadığına dikkat edin, fakat ikisi de `show` metotuna sahip çünkü ikisi de aynı özelliğe sahip. Bu *sanal (virtual)* bir metottur, çünkü bu metot her tip için farklı bir kod çalıştırır ve çalışma zamanındaki duruma göre doğru yöntem çağrılır. Bu referanslara [özellik nesneleri](https://doc.rust-lang.org/stable/book/trait-objects.html) denir.

Ve *bu* farklı tipleri nasıl aynı vektöre koyabileceğinizin yoludur. Eğer Java veyahut Go temeliniz varsa, `Show`'u bir arayüz (interface) olarak düşünebilirsiniz.

Biraz daha kurcalayalım ve değerleri bir `Box` işaretçisi içine koyalım. `Box`, heapta tahsis edilmiş alana yerleştirilen verinin referansını referansını temsil eder ve bir ödünç alma gibi çalışır - bu bir *akıllı işaretçidir (smart pointer)*. `Box` ortadan kalktığı zaman `Drop` devreye girer ve bellek serbest bırakır.

```rust
# trait Show {
#     fn show(&self) -> String;
# }
#
# impl Show for i32 {
#     fn show(&self) -> String {
#         format!("four-byte signed {}", self)
#     }
# }
#
# impl Show for f64 {
#     fn show(&self) -> String {
#         format!("eight-byte float {}", self)
#     }
# }

let answer = Box::new(42);
let maybe_pi = Box::new(3.14);

let show_list: Vec<Box<Show>> = vec![answer,maybe_pi];
for d in &show_list {
println!("show {}",d.show());
}
// show four-byte signed 42
// show eight-byte float 3.14
```

Buradaki fark, bu şekilde bu vektörü alıp bir yere referans takibi yapmaksızın başka yerlere ödünç verebilirsiniz. Vektör düşürüldüğü zaman, `Box` nesneleri de düşürülür ve bellek yeniden tahsis edilebilir.

## Hayvanat Bahçesi
Nesne yönelimli programlamadan ve miras alma işleminden bahsettiğimiz ilk andan itibaren hayvanlardan konuşmaya başlarız. Kulağa da fena gelmez, "Bak işte, kedi bir etoburdur ve etoburlar bir hayvandır". Ruby evreninden klasik bir sloganı analım: "Eğer vaklıyorsa, o bir ördektir". `quack` (vak!) metotuna sahip bütün nesneler ördek olarak tanımlanabilir, biraz kulağa tuhaf gelse de.

```rust

trait Quack {
fn quack(&self);
}

struct Duck ();

impl Quack for Duck {
fn quack(&self) {
println!("quack!");
}
}

struct RandomBird {
is_a_parrot: bool
}

impl Quack for RandomBird {
fn quack(&self) {
if !self.is_a_parrot {
println!("quack!");
} else {
println!("squawk!");
}
}
}

let duck1 = Duck();
let duck2 = RandomBird{is_a_parrot: false};
let parrot = RandomBird{is_a_parrot: true};

let ducks: Vec<&Quack> = vec![&duck1,&duck2,&parrot];

for d in &ducks {
d.quack();
}
// quack!
// quack!
// squawk!
```

Elimizde iki farklı tip var (o kadar işlevsiz ki hiçbir veri tutmuyorlar), ve evet, hepsinin `quack()` metotu var. Bir tanesinin davranışı bir ördek için azıcık tuhaf, ancak yine de hepsi ortak bir metot ismini paylaşıyor ve Rust hepsini tip emniyetli bir şekilde bir araya getiriyor.

Tip emniyeti müthiş bir şey. Eğer tip emniyeti olmasaydı, ortalığı çalışma zamanında darmaduman edecek bir *kediyi* vakvak kardeşlerin arasına sokabilirdik.

Şimdi saçma bir şey yapalım:

```rust
// and why the hell not!
impl Quack for i32 {
fn quack(&self) {
for i in 0..*self {
print!("quack {} ",i);
}
println!("");
}
}

let int = 4;

let ducks: Vec<&Quack> = vec![&duck1,&duck2,&parrot,&int];
...
// quack!
// quack!
// squawk!
// quack 0 quack 1 quack 2 quack 3
```

Ne diyebilirim ki? Vaklıyorsa ördektir. İlginç olan şey, özellikleri herhangi bir şeye ekleyebilirsiniz, sadece "nesnelere" değil. (`quack` bir referans olarak iletildiği için, defererans operatörü ile sayıyı alabilirsiniz.)

Yine de bunu sadece kendi sandığınızda tanımladığınız tipler üzerinde veya o sandıkta tanımlanmış özelliklerle yapabilirsiniz. Yani standart kütüphaneyi yamalı bohçaya (monkey patch) çeviremezsini, ki bu da Ruby vatandaşlarının başka bir alışkanlığı. (Çok da bayılan bir şey değildir doğrusu.)

Şimdiye kadar `Quack` bir Java arayüzü gibi davrandı ve isterseniz modern Java tipleri arayüzleri gibi çeşitli uygulamaları eğer gereken metotları sağladıysanız tipinize ekleyebilirsiniz. (`Iterator` özelliğini hatırlayın.)

Şimdiye kadar, özellikler bir tip *tanımının* parçası değildi ve isterseniz özellikleri istediğiniz tipe ekleyebilirsiniz,  ancak aynı sandık kısıtlamasını gözden kaçırmayın.

`Quack` özelliğine sahip bütün nesneleri referans olarak da görebilirsiniz:

```rust
fn quack_ref (q: &Quack) {
q.quack();
}

quack_ref(&d);
```

İşte bu Rust'ın anladığı şekilde alttiplemedir.

Burada "Programlama Dilleri Kapıştırmaca Dersleri" verdiğimiz için, Go'nun ilginç vaklama yaklaşımını da not etmek istiyorum. Eğer Go'da tanımlanmış bir `Quack` arayüzü varsa ve bir tipin `quack` metotu varsa, o tip `Quack` arayüzünü açık bir tanıma gerek duymadan kendisine implement eder. Bu Java'nın "Her şey tek tek tanımlanacak!" yaklaşımına ters düşer ve tip emnyetini biraz tehlikeye soksa da "ördek tiplemeye" izin verir. (Duck-typing)

Fakat ördek tipleme ile alakalı bir sorun var. Kötü bir nesne yönelimli programlamanın ilk işaretçisi çok fazla metotun `run` gibi her kalıba girebilecek isimlere sahip olmasıdır. "Eğer `run()` (Çalıştır) metotu varsa, o zaman `Runnable` (Çalıştırılabilir) olmalıdır" vaklıyorsa o ördektir kadar zarif gelmiyor. Yani *istemeden* bir `Go` arayüzü o tipte tanımlı olabilir. Rust için mesela, `Debug` ve `Display` aynı anda `fmt` metotunu barındırır, fakar iki özelliğin çok farklı anlamları vardır.

Yani, Rust özellikleri bildiğimiz anlamda çok biçimli nesne yönelimli programlamaya izin verir. Peki ya miras? İnsanlar genellikle "Miras alma"yı kastederken Rust genellikle "Arayüz mirasından" bahseder. `extend` yerine her zaman `implements` kullanmayı tercih eden bir Java programcısı gibi. Ve bu aslında Alan Jolub tarafından [tavsiye edilen bir alışkanlıktır.](http://www.javaworld.com/article/2073649/core-java/why-extends-is-evil.html) Der ki:

> James Gosling'in (Java'nın yaratıcısı) sunduğu bir Java kullanıcıları gurubu konferansında bir keresinde bulunmuştum. O akılda kalıcı soru - cevap kısmında birisi şöyle bir soru sordu: "Eğer Java'yı yeniden tasarlasaydınız, neyi değiştirirdiniz?". O da şöyle bir cevap verdi: "Sınıf anlayışını terk ederdim". İnsanların kahkahası sona erdikten sonra sorunun sınıflar olmadığını ancak "miras alma" işleminden ziyade (*extends* ilişkisi) olduğundan bahsetti. "Arayüz mirası" (*implements* ilişkisi) çok daha tercih edilesi. Miras almayı mümkün olduğunca az yapmanız faydanıza olur.

Java gibi bir dilde bile, çok fazla sınıf üretiyor olabilirsiniz!

Miras alma anlayışının çok ciddi sorunları var, fakat oldukça *akla yatkın* görünüyor. Aşırı geniş bir sınıf olan `Animal`ı tasarlıyoruz ve içerisine çok faydalı şeyler ekliyoruz (tehlikeli olsa bile), ve bizim `Cat` sınıfımız da bu faydalı şeyleri kullanabiliyor. Hepsi bu, kodları tekrar kullanmanın bir yolu. Ancak kodun yeniden kullanımı aslında başka bir konudur.

Miras alma ve arayüz mirası arasındaki farkı anlamak Rust'ı anlarken oldukça önemlidir.

Özelliklerin size başka metotlar *sağladığını* unutmayın. `Iterator` özelliğini düşünün, `next` metotunu tanımladıktan sonra başka metotları zahmetsizce tipinize eklemiş olursunuz. Modern Java arayüzlerindeki "varsayılan" metotlar gibi. Alttaki örnekte `name` kısmını tanımlıyoruz ve `upper_case` bizim yerimize tanımlanmış oluyor. *İstersek* `upper_case` metotunu yeniden yazabiliriz, ama bu *gerekli* değildir.  

```rust
trait Named {
fn name(&self) -> String;

fn upper_case(&self) -> String {
self.name().to_uppercase()
}
}

struct Boo();

impl Named for Boo {
fn name(&self) -> String {
"boo".to_string()
}
}

let f = Boo();

assert_eq!(f.name(),"boo".to_string());
assert_eq!(f.upper_case(),"BOO".to_string());
```
Bu da bir *çeşit* kodu tekrar kullanma yöntemi, evet, ama bunun verinin kendisinde geçerli olmadığını sadece arayüzde tanımlı olduğuna dikkat edin.

## Ördekler ve Genellemeler

Burada Rustla yazılmış, genelleme kullanılarak yazılan "ördek" fonksiyonu gözünüze anlamsız gelmiş olabilir:

```rust
fn quack<Q> (q: &Q)
where Q: Quack {
q.quack();
}

let d = Duck();
quack(&d);
```

Tip parametresi, `Quack` özelliğini barındıran herhangi bir şeyi işaret eder. Buradaki `quack` ve önceki bölümde tanımladığımız `quack_ref` arasında önemli bir fark var. Fonksiyonun içeriği, çağıran her bir tip için ayrıca oluşturulur ve sanal metotlara ihtiyacımız yok, bu tarz fonksiyonlar tamamen "satır içi" (inline) olabilir. Burada `Quack` tipi genellenen tip üzerinde bir kısıtlama olarak kullanılır.

Bu da genellenen `quack` metotumuzun C++ muadili (`const`a dikkat edin):

```cpp
template <class Q>
void quack(const Q& q) {
q.quack();
}
```

Tip parametresinin herhangi bir şeyle kısıtlanmadığına dikkat edin.

Bu daha çok derleme zamanında çalışan bir ördek tiplemeye benziyor - vaklamayan bir tipi iletirsek derleyici `quack` diye bir metot olmadığından bahsedecektir. En azından sonra derleme zamanında keşfediliyor. Go'da direkt bütün tipin `Quackable` arayüzüne sahip olması daha da kötü şeylere sebep olabilir. Daha karmaşık `template` fonksiyonları ve sınıfları berbat hata mesajlarına yol açacaktır çünkü bu sefer genellenen tipler üzerinde `hiçbir` kısıtlama olmayacaktır.

Vaklayan nesne üzerindeki işaretçilerle bir döngü tanımlamayı düşünebilirsiniz:

```cpp
template <class It>
void quack_everyone (It start, It finish) {
    for (It i = start; i != finish; i++) {
        (*i)->quack();
    }
}
```

Bu, her `It` döngüleyici türü için çalışacaktır. Rust muadili az biraz daha ilginçtir:

```rust
fn quack_everyone <I> (iter: I)
where I: Iterator<Item=Box<Quack>> {
for d in iter {
d.quack();
}
}

let ducks: Vec<Box<Quack>> = vec![Box::new(duck1),Box::new(duck2),Box::new(parrot),Box::new(int)];

quack_everyone(ducks.into_iter());
```
Rust'taki döngüleyiciler ördek tipli değildir ancak ilişkili tipler `Iterator` özelliğini barındırmalıdır ve ilgili örneğimizde `Box<Quack>` için döngüleyici tanımlanıyor. Hangi tiple çalışacağı hakkında bir belirsizlik yoktur ve ilişkili veriler muhakkak `Quack`'ı tanımlamalıdır. Bazen bir Rust fonksiyonu yazarken fonksiyon imzası yazmaktan Rust'tan bıkabilirsiniz, bu yüzden standart kütüphanenin kaynak kodlarını dikkatlice okumanızı şiddetle öneririm - ancak fonksiyon yazmak fonksiyon yazmaktan daha kolaydır! Örneğimizdeki tek tip parametresi döngüleyici tipidir, yani bu sadece vektör döngüleyicisiyle değil, `Box<Duck>` dizisi olan her şeyle çalışacaktır. 

## Miras Alma
Nesne yönelimli tasarımlı ilişkili en yaygın sorun nesnelerin bir "onun türünden olma" *(is-a)* ilişkisiyle tanımlanmasıdır, sahip olma ilişkileri *(has-a)* genellikle ihmal edilir. [GoF](https://en.wikipedia.org/wiki/Design_Patterns), "Design Patterns" (Dizayn örüntüleri) kitabında yirmi iki yıl önce "Birleşkeleri mirasa tercih edin" demiş.

İşte size iyi bir örnek: Bir şirketin çalışanlarını modellemek istiyorsunuz ve `Calisan` sizin için iyi bir sınıf ismi gibi görünüyor. Sonra, "Yönetici" bir çalışan türüdür (yani doğru) ve kendi hiyerarşimizi `Yonetici`'yi `Calisan`ın alt sınıfı olarak inşa ediyoruz kurguluyoruz. Zekice görünse de değil. İsimleri belirlemeye kendimizi kaptırmışken çalışanların ve yöneticilerin aslında birbirinden farklı canlılar olduğunu düşündük aslında. Belki de `Calisan` sınıfı sadece bir `Roller` dizisine sahip olmalıdır ve yöneticiyi daha fazla yetkiye sahip bir çalışan olarak tanımlamalıyız?

Taşıtları düşünelim, bisikletten damperli kamyonlara kadar geniş bir skalası var. Araçları kategorize etmenin çok farklı yolları var, üzerinde gittiği yoldan (şehir içinde, tarlada, rayda), kullandığı enerji türüne (dizel, hibrit, elektrikli vs), insan mı yoksa yük taşımacılığında mı kullanıldığı gibi. Sınıfların sabit bir hiyerarşisi, tek bir bakış açısı dışında diğer bakış açılarının görmezden gelindiği anlamına gelir. Gördüğünüz gibi, araçları çok farklı şekillerde sınıflandırabilirsiniz!

Rust için birleşkeler çok daha önemlidir çünkü başka bir sınıftan işlevleri olduğu gibi devrealmak tembelce bir yöntemdir.

Ödünç alma denetimi açısından da birleşkeler önemlidir çünkü çeşitli yapıların (struct) hangi alanlarının kullandıldığının takibi yapılabilir. Bir alanın değişken referansı alınırken diğer alanın değişmez referası ödünç alınabilir; bu yüzden yapılar kullanım rahatlığı için kendi metotlarına sahip olmalıdır. (Yapının *dışsal* arayüzü ise özellikler üzerinden sağlanabilir.)

Ayrık referansların net bir örneği bunu daha anlaşılır kılacaktır. Kendi `String` alanları olan bir yapı tanımladık ve tek bir `String`'in değişken referansını aldık.

```rust
struct Foo {
    one: String,
    two: String
}

impl Foo {
    fn borrow_one_mut(&mut self) -> &mut String {
        &mut self.one
    }
    ....
}
```
(Rust'ın isimlendirme anlayışının da aynı zamanda bir öneğidir - bu tarz metot isimleri `_mut` ile sona ermelidir.)

Şimdi ilk metotu tekrar kullanarak iki `String` alanını da ödünç alan bir yapı tanımlayalım:

```rust
    fn borrow_both(&self) -> (&str,&str) {
        (self.borrow_one_mut(), &self.two)
    }
```
Çalışamaz! `self`'in değişmez referansını aldık ve aynı zamanda `self`'in değişebilir referansını almaya çalışıyoruz. Eğer Rust bu tarz durumlara izin verseydi değişmez referansın değişemeyeceğini garanti edemeyebilirdi.

Çözüm basit:

```rust
    fn borrow_both(&self) -> (&str,&str) {
        (&self.one, &self.two)
    }
```
Bunda bir sorun yok çünkü bunların ikisini ödünç alma denetçisi bunların ikisini de bağımsız ödünç almalar olarak tanımlar. Alanların gelişigüzel yapılar olduğunu düşünün, rastgele çağırdığınız metotlar bu alanlar üzerinde çalışırken bir hataya sebep olmayacaktır.

Miras almanın sınırlandırılmış ancak önemli bir türü [Deref](https://rust-lang.github.io/book/second-edition/ch15-02-deref.html) özelliğidir, ismi "Dereferans" operatörü olan `*` ile gelir. `String` tipi `Deref<Target=str>` özelliğine sahiptir ve `&str` tipi için çalışacak bütün metotlar aynı zamanda `String` ile de çalışabilir! Benzer şekilde, `Foo` ile çalışan bütün metotlar aynı `Box<Foo>` üzerinden de doğruca çalışabilir. Başta biraz... kontrolsüz bir şekilde pratik görünse de aslında epey kullanışlıdır. Rust'ın içerisinde basit bir mantık var, ancak kullanımı o kadar da akılcı görünmüyor. Sadece değişken bir türün ödünç alındığı zaman daha basit davranması gerketiği zaman kullanılmalıdır.

Rust'ta *özellikler birbirini miras alabilir*:

```rust
trait Show {
    fn show(&self) -> String;
}

trait Location {
    fn location(&self) -> String;
}

trait ShowTell: Show + Location {}
```
Son özellik iki ayrıksı özelliği de barındırıyor, istenirse kendi içine metotlar tanımlanabilir. 

Şimdi alıştığımız şeye geri döndük:

```rust
#[derive(Debug)]
struct Foo {
    name: String,
    location: String
}

impl Foo {
    fn new(name: &str, location: &str) -> Foo {
        Foo{
            name: name.to_string(),
            location: location.to_string()
        }
    }
}

impl Show for Foo {
    fn show(&self) -> String {
        self.name.clone()
    }
}

impl Location for Foo {
    fn location(&self) -> String {
        self.location.clone()
    }
}

impl ShowTell for Foo {}

```

Eğer `Foo` türünden `foo` diye bir değerim varsa, bu türün değişkeni `&Show` ve `&Location`'u karşıladığı gibi (ikisini de barındıran) `&ShowTell`'i de karşılayacak.

İşte işimize yaracak ufak bir makro:

```rust
macro_rules! dbg {
    ($x:expr) => {
        println!("{} = {:?}",stringify!($x),$x);
    }
}
```
(`$x` ile gösterilen) tek bir argümanı alıyor ve bu argüman bir "ifade" olmalıdır. Bu değeri ekrana yazdırıyoruz ve veriyi ve metinin metinleştirilmiş hâlini ekrana yazdırıyoruz. C programcıları burada bıyık altından gülebilir, ancak eğer `1 + 2` (bir ifade) değerini verirsem `stringify!(1 + 2)` bize "1 + 2" şeklinde karakter dizisi verecektir. Bu, bize biraz kodları incelemek için yardımcı olacaktır:

```rust
let foo = Foo::new("Pete","bathroom");
dbg!(foo.show());
dbg!(foo.location());

let st: &ShowTell = &foo;

dbg!(st.show());
dbg!(st.location());

fn show_it_all(r: &ShowTell) {
    dbg!(r.show());
    dbg!(r.location());
}

let boo = Foo::new("Alice","cupboard");
show_it_all(&boo);

fn show(s: &Show) {
    dbg!(s.show());
}

show(&boo);

// foo.show() = "Pete"
// foo.location() = "bathroom"
// st.show() = "Pete"
// st.location() = "bathroom"
// r.show() = "Alice"
// r.location() = "cupboard"
// s.show() = "Alice"
```

İşte *bu* nesne yönelimli programlamadır, sadece alıştığınız türden değil.

`Show` referansına iletilen `show` değerinin *dinamik olarak* `ShowTell` olmayacağına dikkat edin! Daha dinamik sınıf sistemlerine sahip dillerin size objenin bir sınıfın nesnesi olup olmadığını denetleme imkanı verdiğine ve bu dinamik türe göre çağrı yapma izni verdiğine dikkat edin! Aslında bu pek de iyi bir fikir değildir ve Rust'ta bunu yapmanın bir yol yoktur çünkü Rust `Show` referansının aslında `ShowTell` referansı olduğunu unutmuştur bile.

Her zaman seçimleriniz vardır; özellik nesneleri aracılığıyla çok biçimlilik ya da özellik kısıtlamaları ile biçimlendirilmiş genelleme tanımlarıyla tek biçimlilik. Modern C++ ve Rust standart kütüphanesi genelleme yolunu tercih eder, ancak çok biçimlilik yolu da hâlen daha tercih edilebilir. Her zaman neyin karşılığında neyi aldığınızı bilmeniz gerekir - genellemeler daha hızlı kod üretir ve satır içi (inline) kullanılabilirler. Bunun neticesinde kodunuz şişebilir. (code bloat) Fakat her şeyin *mümkün olduğunca hızlı olmasına* da gerek yoktur - olağan bir program akışında birkaç defa bu gerçekleşebilir.

Sonuç olarak:

- Sınıfların rolü veri ve özellikler (trait) arasında paylaştırılmıştır.
- Yapılar ve numaralandırmalar veri gizlemesine ve metot tanımlanabilmesine rağmen fazla işleve sahip değildir.
- Alttiplemenin *sınırlandırılmış* bir türü `Deref` özelliği ile sağlanabilir.
- Özellikler bir veri tutmaz ancak her tipe uygunlanabilirler. (Yalnızca yapılara değil.)
- Özellikler, başka özellikleri miras alabilir.
- Özellikler metotlar sunabilir, kodların tekrar kullanımını mümkün kılarlar
- Özellikler size sanal metotlar (çok biçimlilik) ve genelleme sınırlamaları (tek biçimlilik) sunabilirler.

## Ördek: Windows API
Geleneksel nesne yönetimli programlamanın en çok kullanıldığı yerlerden birisi GUI (Ç.N: düğmeli menüyü arayüzler işte) kütüphaneleridir. `EditControl` ve `ListWindow`, `Window` türünden sınıflardır falan filan. Bu, GUI kütüphanelerine Rust bağlantıları yazmayı biraz daha zorlaştırır.

Rust'ta Win32 programlaması [direkt](https://www.codeproject.com/Tips/1053658/Win-GUI-Programming-In-Rust-Language) yapılabilir, ve orijinal C'den daha az gariptir. C'den C++'a geçer geçmez daha sade bir şey yapmak istedim ve kendi nesne tabanlı kodlarımı yazdım.

Tipik bir Win32 API fonksiyonu [ShowWindow'dur](https://docs.rs/user32-sys/0.0.9/i686-pc-windows-gnu/user32_sys/fn.ShowWindow.html) ve bir pencerenin görünüp görünmediğini denetlemek için kullanılır. Şimdi, `EditControl`'ün kendisine özgü nitelikleri bulunur fakat hepsi Win32'nin `HWND` ("pencere yönetimi") opak değeriyle yapılır. `EditControl`'ün aynı zamanda `show` metotu olmasını isteyebilirsiniz, genelde miras alma yöntemiyle halledilir. Fakat bu tipin her işlevi miras almasını istemeyebilirsiniz! Rust size güzel bir çözüm sunar, `Window` özelliğini düşünelim:

```rust
trait Window {
    // you need to define this!
    fn get_hwnd(&self) -> HWND;

    // and all these will be provided
    fn show(&self, visible: bool) {
        unsafe {
         user32_sys::ShowWindow(self.get_hwnd(), if visible {1} else {0})
        }
    }

    // ..... oodles of methods operating on Windows

}
```

Şimdi `EditControl` yapısı sadece bir `HWHD`'ye sahip olabilir ve `Window`'u eklemek tek bir metotu tanımlayarak mümkün olabilir. `EditControl` ise `Window` özelliğini miras alan bir yapıdır ve daha geniş bir arayüz tanımlar. `ComboBox` gibi düşünün - `EditControl` gibi davranır ve bir `ListWindow` da özellik mirası aracılığıyla eklenebilir.

Win32 API'sı ("32" artık "32-bit" anlamına gelmiyor) özünde nesne yönelimlidir, ancak daha eski bir şekilde, Alan Key'in tanımından etkilenmiş bir şekilde: nesneler gizli veriler tutar ve *mesajlar* üzerinden işlenir. Yani Windows uygulamalarının kalbi bir mesaj döngüsüdür ve çeşitli pencereler (pencere sınıfları da denir) kendi metotlarını kendi anahtar ifadeleriyle uygularlar. `WM_SETTEXT` diye bir mesaj vardır fakat bunun koda dökülüş hâli biraz daha farklıdır: bir etiketin yazısı değişebilir ya pencere başlığı değişir.

[Burada](https://gabdube.github.io/native-windows-gui/book_20.html) daha anlaşılır ve minimal bir Windows GUI frameworkü görebilirsiniz. Bana göre çok fazla `unwrap` öğeleri var ve bunların çoğu hata bile değil.  Bu, "NWG"nin mesajlaşmanın dinamik doğasına ters düşmesiyle alakalı. Daha tip güvenli bir arayüz sunmak için, hatalar derleme zamanında sunuluyor.

[Rust programlama kitabında "Nesne yönelimli nedir?"](https://doc.rust-lang.org/stable/book/ch17-00-oop.html) üzerine güzel bir tartışma mevcuttur.

> Çeviri notu: Bu bölümün sonlarına doğru bir dolu hata yapmış olabilirim, çünkü bu son kısımları yazarken saat gecenin üçüne yaklaşıyor ve kafa olarak biraz yorgundum. :)
> 
> Türk yazılımcısı çoğunlukla nesne yönelimli kavramını C# ve Java aracılığıyla öğrendiği için nesne yönelimi kavramı dilde `class` kelimesinin sunulmasıyla bağdaştırılıyor ve bu epeyce bir kafa karışıklığı yaratıyor. Nesne tabanlı programlama, kendi özünde verilerin soyutlanıp yazılımcıya daha çok anlam ifade eden "bütünlere" dönüştürülmesiyle alakalı bir tekniktir ve bu tekniğin uygulanması için verinin gizlenmesi, yalnızca metotlarla erişim gibi ilkeler vardır. `class` ile tanımlanan yaplar, bu ilkeleri otomatik olarak inşa etmeye yarar. Ortada `class` adı hiç geçmese bile nesne tabanlı programlama yapılabilir ve Lua, Nim, Nix gibi diller buna iyi bir örnek oluşturur. İşin özü, JavaScript da bu sınıfların listesindeydi ancak sonradan `class` kelimesi bazı prototip işlemlerini otomatikleştirmek için eklendi. Eklenene dek, JavaScript her zaman nesne tabanlıydı ancak "fonksiyonel" ve "nesne tabanlı" olmak arasında kalmış bir dil gibi düşünüldü; aslında JavaScript her zaman "prototip" üzerinden işleyen nesne yönelimli bir dildi.
> 
> Bir literatür hatası olarak, saf fonksiyonel diller aynı zamanda nesne yönelimli olamayacağı için fonksiyonel programlama ve nesne yönelimli programlama arasında bir zıtlık varmış gibi anlaşıldı. Sonradan Go, Rust gibi klasik nesne yönelimli diller sınıfına uymayan diller "fonksiyonel" diller olarak anlaşıldı. Rust'ta fonksiyonel esintiler bulunmasına rağmen Rust saf bir fonksiyonel dil değildir, nesne yönelimi ile Rust'ın alakası neyse fonksiyonel programlama ile Rust'ın alakası o seviyede diyebilirim. Ha, fonksiyonel programlama "işlevsel, işe yarayan programlama" demek değildir diye not edelim.
> 
> Peki, neden Rust'ta sınıflar yok? Bu bölümde bahsedildiği gibi sınıflar çok fazla işi bir anda halleder ve bu kesinlikle Rust'ın doğasına uymaz. Rust'ta her şey açık, belirgin ve bilindiktir. Rust, sınıfların yaptığı şeyleri yapmanıza izin verecek araçları size sunar ancak kolaylık olsun diye gizlice bir şeyler hazırlamaz. Bu yüzden, Rust nesne yönelimli bir dil gibi görünmese de aslında nesne yönelimini size sunar.
> 
> Eğer nesne yönelimli programlamaya karşı bakış açınızı gözden geçirmek ve nesne yönelimli programlamanın yaratıcısından da neyin ne olduğunu öğrenmek isterseniz, [bu bağlantıya tıklayabilirsiniz.](https://userpage.fu-berlin.de/~ram/pub/pub_jf47ht81Ht/doc_kay_oop_en)