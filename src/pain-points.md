## Rust ve Çektirdiği Çile

Rust'ın öğrenmesi pek çok "ana akım" dillere kıyasla öğrenmesi zor dillerden olduğunu söylemek doğru olur. Bazı istisnası insanlar o kadar da zor bulmuyor, fakat "istisna" ne anlama gelir buna dikkat edin - onlar *istinaidir*. Çoğu insan başta zorlanır ama sonra başarır. Başlangıçta çektiğiniz çilelerle neticede elde edeceğiniz şeyler arasında bir bağlantı yoktur.

Hepimiz bir yerlerden geliyoruz ve bu Python gibi "dinamik" veya "C++" gibi "statik" ana akım dilleriyle muhatap olmuş olduğumuz anlamına gelir. Her koşulda, Rust programlama anlayışımızı yeniden inşa etmeye zorlayacak kadar farklıdır. Zeki ve tecrübeli insanlar bu işin içine girip kendi zekiliklerinin fayda etmediğini görünce bir hayal kırıklığına uğruyorlar; kendisine yeterince değer vermeyenler de düşündükleri kadar "zeki" olmadıklarını düşünüyorlar.

Dinamik programlama tecrübesi olanlar (buna Java'yı da dahil edebilirim) her şeyin bir referans ve referansların da varsayılan olarak değişebilir olduğunu düşünür. Çöp toplayıcıları da bellek açısından emniyetli programlama yapmamıza yardımcı *olur*. Daha fazla bellek kullanımı ve öngörülememezlik pahasına da olsa JVM'i daha hızlı hâle getirmek için çok şey yapıldı. Bazen buna değer diye düşünülür, hiç modası geçmeyen bir fikir programcı üretkenliğinin bilgisayar performansından daha önemli olduğudur.

Fakat dünyadaki pek çok bilgisayarın - mesela arabalardaki gaz kelebeğini kontrol etmek gibi gibi hayati önemde şeyler yapan minik bir bilgisayarın - en ucuz laptop kadar kaynağa sahip değildir ve olaylara *eşzamanlı (realtime)* olarak cevap vermesi gerekir. Aynı şekilde yazılım altyapılarının doğru, sağlam ve hızlı olması gerekir. (Eski bir mühendislik kuralı). Şimdiye dek bu tarz işler kendi yapısı gereği hiçbir emniyet bulundurmayan C ve C++ ile yapıldı - bu raddedeki *emniyetsizliğin* toplam maaliyeti dikkat edilmesi gereken genel şey oldu. Bir programı çalışır hâle getirmek kolaydır, ama esas olay buradan sonra başlar.

Sistem programlama dillerinde bir çöp toplayıcısı bulunamaz çünkü onlar her şeyin üzerine inşa edildiği bina temelidir. Kaynakların sizin ihtiyaç duyduğunuz ve uygun gördüğünüz şekilde kullanılmasını sağlarlar. 

Eğer çöp toplayıcısı olmazsa bellek başka türde şekillerde yönetilir. Manuel bellek yönetiminde belleği alırsınız, kullanırsınız ve bunu geri verirsiniz - kulağa kolay gibi gelse de aslında oldukça zordur. Üretken ve facia yaratmaya elverişli bir C programcısı olmak sizin için bir hafta sürer - ancak ne yaptığını bilen, bütün muhtemel hataları kontrol eden iyi bir C programcısı olmak yıllar sürer. 

Rust belleği modern C++ gibi yönetir, nesneler yok edildiği zaman arka kalan bellek tekrar kullanılabilir. `Box` ile heap üzerinde bellek tahsis edebilirsiniz, ancak bu fonksiyon bittiğinde `Box` "kapsam sonuna" ulaşmış olur. `new` diye bir şey var ancak `delete` diye bir şey yok. `File` oluşturabilirsiniz, (önemli bir kaynak olan) dosya iş bittiğinde kendiliğinden kapatılır. Buna Rust'ta *düşürme (dropping)* denir.

Kaynakları paylaşmanız gerekir, her şeyin kopyasını üretmek verimsizdir, işi ilginç kılar da budur. C++'da da referanslar vardır ve Rust referansları daha çok C işaretçilerine benzer - veriye ulaşmak için `*r` kullanmanız gerekir ve veri referansını iletmek için de `&`  kullanırsınız.

Rust'ın *ödünç alma denetçisi (borrow checker)* esas veri yok edildikten sonra referansının var olma ihtimalini ortadan kaldırır. 

## Tip Çıkarımı
"Statik" ve "dinamik" arasındaki fark her şey değildir. Her şeyde olduğu gibi bu da çok katmanlı bir konudur. C, statik tipli (her değişkenin derleme zamanında tipi vardır) ancak zayıf tipli (weakly typed) (mesela `void*` her şeye işaret edebilir) bir dildir; Python ise dinamik tipli (Tip değişkende tanımlanmaz değil veriyle beraber gelir) fakat güçlü tipli (strongly typed) bir dildir. Java ise statik/güçlümtraktır (akla uygun görünen fakat tehlikeli olabilen veri dönüştürme (?) mekanizmasıyla) ve Rust ise statik/güçlüdür; çalışma zamanında gizlice veri dönüştürmez.

Java her bir tipin ne olduğunun detaylıca yazdırdığı yaygın olarak bilinirken Rust tipleri *tahmin (infer)* etmeyi sever. Bu genelde zekicedir ancak fakat bazen hangi tiple çalıştığınızı merak edersiniz. Mesela `let n = 100` gibi bir ifade görürsünüz ve merak edersiniz - bu sayının değeri ne? Varsayılan olarak bu `i32`dir, dort bitlik işaretli sayı. Herkes C'nin belirsiz sayı tiplerinin (`int` ve `long` gibi) kötü bir fikir olduğundan emindir, açık olmak en iyisidir. Bazen tipi belirtebilirsiniz, mesela `let n: u32 = 100` gibi ya da sayıyla beraber tipi de belirtebilirsiniz; `let n = 100u32` gibi. Ama tip çıkarımı dediğimi şey bundan ibaret değil! Eğer `let n = 100` yazarsanız `rustc` bunun *bir çeşit* sayı olduğunu bilir. `n` değişkenini bir yere iletirseniz ve fonksiyon `u64` bekliyorsa `n`'in tipi `u64` olacaktır!

Bu noktadan sonra eğer `u32` bekleyen bir yerde `n` değişkenini kullanırsanız `rustc` buna izin vermeyecektir çünkü `n`'in tipi `u64` olarak işaretlenmiştir ve bunu dönüştürmenin gizli ve kolay bir yolu olma*ya*caktır. İşte bu güçlü tiplemedir - "Interger overflow"dan muzdarip olana dek hayatınızı kolaylaştıracak ufak tip dönüşümleri Rust'ta yoktur. Açıkça `n` değerini `n as u32` olarak iletmeniz gerekir - Rust'ta tipler böyle dönüştürülür. Neyse ki, `rustc` sorunları müdahale edilebilirken vermekte iyidir - derleyicinin tavsiyesiyle sorunu düzeltebilirsiniz. 

Bu sayede Rust açık tip ifadeleri olmaksızın yazılabilir:

```rust
let mut v = Vec::new();
// v is deduced to have type Vec<i32>
v.push(10);
v.push(20);
v.push("hello") <--- Bunu yapma işte, yapma
```
Bir sayı vektörüne karakter dizisi iletememek bir sorun değil, özelliktir. Dinamik tipleme esnek olduğu kadar beladır da.

(Eğer bir vektör tipine hem sayı hem de karakter dizisi iletmeniz lazımsa, Rust'ın `enum` tipleri bu işi emniyetlice görür.)

Bazen ufak bir *ipucu* vermeniz gerekir. `collect` çok güzel bir döngüleyici metotudur, ama bazen ipucuna ihtiyaç duyar. Mesela `char` üzerinde çalışan bir döngüleyicim var. `collect` iki farklı biçimde çalışır.

```rust
// a vector of char ['h','e','l','l','o']
let v: Vec<_> = "hello".chars().collect();
// a string "doy"
let m: String = "dolly".chars().filter(|&c| c != 'l').collect();
```
Bazen değişken tipinden emin olamazsanız bunun da ufak bir yolu var, o da `rustc`'yi işleme alınan tipi bir hata mesajında yazdırmaktır:

```rust
let x: () = var;
```
`rustc` aşırı spesifik bir tip seçebilir. Burada farklı referansları `&Debug` vektörüne koyuyoruz ancak tipleri açıkça belirtmemiz gereklidir.

```rust
use std::fmt::Debug;

let answer = 42;
let message = "hello";
let float = 2.7212;

let display: Vec<&Debug> = vec![&message, &answer, &float];

for d in display {
    println!("got {:?}", d);
}
```

## Değişebilir Referanslar
Kural basit: bir anda sadece değişken referans olabilir. Bunun sebebi değişimin *her yerde* gerçekleşmesinin önüne geçmek. Küçük programlar yazarken fark etmiyor olabilirsiniz ancak büyük kod yapılarında başınıza ciddi bela olabilir.

Bir diğer kural ise ortada değişemez referanslar varken değişebilir referans elde edemiyor oluşunuzdur. Aksi takdirde kimse bu değişemez referansların değişemeyeceğini garantezi edemezdi. C++ da değişemez referanslara sahiptir (mesela `const string&`) gibi ancak başka birisinin `string&` referansı alıp veriyi değiştiremeyeceğini garanti *edemez*.

Her referansın değişebilir olduğu dillerle çalışmaya alışıksanız bu size ters gelecektir. Emniyetsiz, "rahat" diller tamamen programcının kötü bir şey yapmayacağına ve kendi programını kesinlikle iyi bir şekilde anladığını düşünerek çalışır. Fakat bir kişiden daha fazla kişi tarafından geliştirilmiş büyük programlar bir kişinin programın ne olduğunu anlayıp anlamamasından çok daha ötededir.

İşin *gıcık eden* eden tarafı ödünç alma denetçisinin düşündüğümüz kadar zeki olmamasıdır.

```rust
let mut m = HashMap::new();
m.insert("one", 1);
m.insert("two", 2);

if let Some(r) = m.get_mut("one") { // <-- mutable borrow of m
    *r = 10;
} else {
    m.insert("one", 1); // can't borrow mutably again!
}
```

Eğer `None` elde etmişsek HashMap'ten hiçbir şey ödünç alınmıyor, yani burada *gerçekten* hiçbir şey ödünç alma kurallarını çiğnemiyor.

Hâliyle iş çirkinleşiyor:

```rust
let mut found = false;
if let Some(r) = m.get_mut("one") {
    *r = 10;
    found = true;
}
if !found {
    m.insert("one", 1);
}
```
Rezalet olsa da işe yarıyor çünkü sadece "if" ifadesinde bir adet referasımız tutulmuş oluyor.

Daha iyisi `HashMap` içindeki [entry](https://doc.rust-lang.org/std/collections/hash_map/enum.Entry.html) metotunu kullanmaktır.

```rust
use std::collections::hash_map::Entry;

match m.entry("one") {
    Entry::Occupied(e) => {
        *e.into_mut() = 10;
    },
    Entry::Vacant(e) => {
        e.insert(1);
    }
};
```

Ödünç alma mekanizmasının bu sıralar *sözcüksel olmayan yaşam süreleri (non-lexical lifetimes)* hakkında daha yumuşak olacağı konuşuluyor. 

Yine de ödünç alma mekanizması bazı önemli vakaları *anlıyor*. Mesela bir yapınız varsa alanları bağımsız olarak ödünç alınabilir. Birleşke mantığı işinize yarayacaktır, büyük yapılar (struct) kendi metotları olan daha ufak yapıları barındırmalıdır. Değişken metotları büyük yapılar içinde tanımlamak, tek bir alan değiştirilmiş olsa bile bütün yapının değiştirilemeyeceği durumlara yol açacaktır.

Değişebilir verilerin söz konusu olduğu durumlarda verilerin parçalarının bağımsız olarak işlendiği özel durumlar vardır. Örneğin değişebilir bir diliminiz varsa, `split_at_mut` size değişebilir iki referans verecektir. Bu gayet emniyetlidir çünkü Rust bu iki dilimin kazara aynı verileri sahiplenmeyeceğinden emin olacaktır.

## Referanslar ve Yaşam Süreleri
Rust, referasın veriden daha uzun süre yaşadığı durumlara izin vermez. Aksi taktirde ölü veriye işaret eden "ölü referanslarımız" bulurdu - `segfault` kaçınılmaz olurdu.

`rustc` genelde fonksiyonlardaki yaşam süreleri hakkında mantıklı çıkarımlar yapabilir:

```rust
fn pair(s: &str, ch: char) -> (&str, &str) {
    if let Some(idx) = s.find(ch) {
        (&s[0..idx], &s[idx+1..])
    } else {
        (s, "")
    }
}
fn main() {
    let p = pair("hello:dolly", ':');
    println!("{:?}", p);
}
// ("hello", "dolly")
```

Gayet emniyetlidir çünkü burada herhangi bir sınırlayıcı yoktur. `rustc` iki karakter dizisinin de fonksiyona aktarılan karakter dizisinin referansları olduğuna karar verir.

Aslında fonksiyon tanımı şu şekildedir:

```rust
fn pair<'a>(s: &'a str, ch: char) -> (&'a str, &'a str) {...}
```

Bu noktasyon çıktıdaki karakter dizilerinin *en fazla* girdideki karakter dizileri kadar yaşayacağına karar verir. Yaşam sürelerinin aynı olduğu anlamına gelmez bu, onları istediğimiz an düşürebiliriz, sadece "`s`"ten fazla yaşamayacağını belirtiriz.

Yani, `rustc` *yaşam süresi saptaması (lifetime ellision/?)* işimizi biraz daha rahatlatır. 

Şimdi eğer fonksiyon *iki* karakter dizisi alırsa, yaşam sürelerini açıkça belirtmemiz gerekir ki hangi çıktının hangi girdiyi referans aldığını bilmiş olalım.

Bir referans tutan bir yapı tanımladığımız sırada neyin ne kadar yaşam ömrü olduğunu belirtmeliyiz:

```rust
struct Container<'a> {
    s: &'a str
}
```
Burada da yapının referanstan daha uzun süre yaşamayamayacağını vurgulamış oluyoruz. Hem yapılar hem de fonksiyonlar için yaşam süresinin `<>` içerisinde tıpkı tip belirtilir gibi belirtilmesi gereklidir.

Kapamalar gayet akılcı ve güçlü bir özelliktir - Rust kapamalarının gücü buradan gelir. Eğer onları bir yerde saklamak isterseniz onlara bir yaşam süresi belirtmeniz gerekir. Çünkü kapamalar kendi özünde çevresinden referanslar alan ve çağrılabilen bir yapıdır. Mesela `m` ve `c` şeklinde değişemez referanslar alabilen `linear` kapamasına bakalım.

```rust
let m = 2.0;
let c = 0.5;

let linear = |x| m*x + c;
let sc = |x| m*x.cos()
...
```

`linear` ve `sc` kapamalarının ikisi de `Fn(x: f64) -> f64` özelliğine sahiptir ancak ikisi de aynı yaratık *değildir* - ikisinin de farklı tipleri ve boyutları vardır! Eğer saklamak isterseniz tiplerini `Box<Fn(x: f64)->f64 + 'a>` olarak belirtilmelilerdir.

JavaScript veya Lua'da kapamaların nasıl da su gibi aktığına görmüşseniz size biraz sinir bozucu gelecektir. Ancak C++ da Rust ile aynı şeyi yapar ve sanal çağrılar için ufak bir bedel karşılığında farklı türden kapamaları saklamak için `std::function`'a ihtiyaç duyar.

## Karakter Dizileri

Rust'ta karakter dizilerinin başlangıçta sinir bozucu gelmesi olağandır. Onları üretmenin birden çok yolu vardır ve hepsi gereksiz detaylı gelir.
```rust
let s1 = "hello".to_string();
let s2 = String::from("dolly");
```
"hello" hâli hazırda bir karakter dizisi değil midir? Yani, bir şekilde. `String` *sahipli* bir karakter dizisidir ve heap üzerinde yer tutar. Bir karakter dizisi kalıbı olan `&str` (karakter dizisi dilimi) tipi ise (sabit olarak) çalıştırılabilir dosyanın içinde bekler ya da `String`'ten ödünç alınarak oluşturulur. Sistem programlama dillerinin bu ayrıma ihtiyacı vardır - ufak bir mikrokontrollerı düşünür, azıcık RAM ve biraz daha fazla RM'u bulunur. Karakter dizisi kalıpları daha az enerji harcayan ve ucuz olan ROM'da ("read-only/salt okunur") depolanır. 

Fakat bu C++'da çok daha kolay (diyebilirsiniz):

```C
std::string s = "hello";
```
Kısaca evet, fakat bir karakter dizisinin örtükçe nasıl oluşturulduğunu da gizlemektedir. Rust bellek tahsis etme konusunda açık olmayı tercih eder, hâliyle `to_string` gibi şeyler var. Öbür taraftan, C++ karakter dizilerini ödünç almak için `c_str` kullanmalısınız ve C'nin karakter dizileri çok kullanışsızdır.

Neyse ki, Rust'ta işler çok daha iyi işliyor - `String` ve `&str` tiplerinin ikisinin de gerekli olduğunu *bir kere* kabul ederseniz. `String` metotları çoğunlukla karakter dizisini değiştirmek içindir, mesela `push` bir karakter ekler (alttan alta `Vec<u8>` gibi çalışır). Fakat `&str` metotlarının tamamına da sahiptir. `Deref` mekanizması aracılığıyla bir `String` aynı zamanda `&str` olarak iletilebilir - bu yüzden nadiren fonksiyon tanımlarında `&String` görürsünüz.

Çeşitli özellikler (trait) aracılığıyla `&str`'den `String` elde etmenin pek çok yolu var. Rust özelliklerin tiplerinin genellenerek çalışmasına izin verir. Pratik bir kural olarak, `Display` özelliğine sahip her şey `to_string`'e sahiptir; `42.to_string()` gibi.

Bazı operatörler beklediğiniz gibi davranmaz:

```rust
    let s1 = "hello".to_string();
    let s2 = s1.clone();
    assert!(s1 == s2);  // cool
    assert!(s1 == "hello"); // fine
    assert!(s1 == &s2); // WTF?
```

Hatırlayın ki `String` ve `&String` birbirinden farklı tiplerdir ve `==` bu tarz komibasyonlarda tanımlı değildir. C++ programlamaya alışık bir kişi değerlerin yerine referansların konulduğunu görmeyi beklediğinden şaşırabilir. Ek olarak `&s2` kendiğinden `&str` olmayacaktır çünkü *deref zorlaması* bir  `&str` değişkeni veya argümanı atadığınız zaman çalışacaktır. (`s2.as_str()` diye açıkça ifade etmek işinize yarayacaktır.)

Ancak bu harbiden bir "has\*\*\*tir artık" denmeyi hakeder:

```rust
let s3 = s1 + s2;  // <--- no can do
```
İki `String` değerini birleştiremezsiniz, ancak bir `String` ile `&str`'i birleştirebilirsiniz. Fakat `&str` ile `String`'i birleştiremezsiniz. Bu yüzden pek çok insan `+` yerine `format!` makrosunu tercih eder, biraz daha tutarlıdır ancak o kadar da verimli değildir.

Bazı karakter dizisi işlemleri de mevcuttur ancak farklı çalışır. Mesela pek çok dilin `split` metotu vardır ki bu da bir karakter dizisini, karakter dizisi listesine dönüştürür. Rust'ta karakter dizisi metodu bir döngüleyici döner ve bunu "sonra" bir vektör içerisine toplayabilirsiniz.

```rust
let parts: Vec<_> = s.split(',').collect();
```

Eğer hızlıca bir vektör almak istiyorsanız biraz göze batabilir fakat yeni bir vektörü belelkte tahsis etmeden önce birkaç işlem yapabilirsiniz. Mesela parçalanmış bir metin içerisinden en uzun kelimeyi mi almak istiyorsunuz?

```rust
let max = s.split(',').map(|s| s.len()).max().unwrap();
```
(Eğer doş bir döngüleyici varsa maksimum değer de olmayacaktır ve bu durumu kontrol etmek için `unwrap` kullanıyoruz.)

`collect` metotu bize parçaların orijinal karakter dizisinden ödünç alındığı bir `Vec<&str>` döner - sadece referanslar için bellekte alan tahsis etmemiz gerekir. C++'da çalıştığı gibi bir metot yok fakat son zamana dek her alt diziye ayrıca alan tahsis edilmesi gerekiyordu. (C++ 17 ile beraber Rust'taki `&str` gibi çalışan `std::string_view` geldi.)

## Noktalı virgüller hakkında bir not
Noktalı virgüller bu dilde de zorunludur, fakat C'de noktalı virgül konulmaması gereken yerlerde Rust'ta da konulmaz, mesela `{}` bloklarından sonra. Aynı zamanda  `enum` ve `struct` içinde de onlara ihtiyaç yoktur. (Bu C'den gelen bir acayiplik) Fakat, eğer blok bir değer dönmeliyse noktalı virgüllere ihtiyaç kalmaz.

```rust
    let msg = if ok {"ok"} else {"error"};
```
Bütün bir `let` deyiminin ardından yine bir noktalı virgül koymak zorunda olduğumuza dikkat edin!

Eğer noktalı karakter dizisi kalıplarından sonra noktalı virgül koysaydık bize `()` dönerdi. (`Nothing` veya `void` gibi) Bu fonksiyondan değer dönerken karşılaşılan olağan bir hatadır.

```rust
fn sqr(x: f64) -> f64 {
    x * x;
}
```

`rustc` size bu durumda açıklayıcı bir hata mesajıyla geri dönüş yapacaktır.

> Rust, Haskell ve Ruby gibi expression-based bir dildir ve bu kavramı "ifade odaklı"  olarak düşünebilirsiniz. Bu tarz dillerde her ifadenin bir değeri vardır 

## C++ ile Alakalı Konular

### Rust'ta Değer Semantikleri Farklıdır

C++'da ilkel tipler gibi davranacak ve kendisini kopyalayacak tipler tanımlamak mümkündür. Ek olarak, bir değerin geçici bir bağımdan başka bir bağlama nasıl taşınacağını belirlemek adına taşıma oluşturucusu kullanılır.

Rust'ta ilkel tipler beklendiği gibi davranır fakat `Copy` özelliği sadece kopyalanabilir türler içeriyorsa (yapılar, demetler veya numaralandırma) kullanıcının tanımladığı tiplere eklenebilir. Diğer tiplere `Clone` eklenebilir ancak bu sefer de verilerin `clone` metotunu çağırmanız gerekir. Rust, herhangi bir bellekte alan tahsis etme işleminin açıktan olmasını ister ve atama operatörlerini ya da kopyalama oluşturucularını gizlemez.

Yani, kopyalama ile taşıma her zaman birkaç biti hareket etmesi olarak tanımlanır ve geçersiz kılınamaz.

Eğer `s1` `Copy` özelliğini içermeyen bir türse `s2 = s1` bir taşımaya sebep olur ve bu `s1`'i *tüketir*!  Eğer gerçek bir kopya üretmek istiyorsanız `clone` kullanın.

Ödünç alma çoğu zaman kopyalamadan daha iyidir ancak bu sefer de ödünç alma kurallarını takip etmelisiniz. Neyse ki, ödünç alma işlemi yeniden düzenlenebilir bir davranıştır. Mesela `String`, `&str` olarak ödünç alınabilir ve `&str`'nin değişmeyen metotlarını kullanabilir. *Karakter dizisi dilimleri* de C++'ın `const char*`dan farksız `c_str` anlayışındaki ödünç alma yöntemine kıyasla çok daha güçlüdür. `&str` sahiplenilmiş birkaç baytın işaretçisinden (veya bir karakter dizisi kalıbından) ve *boyut bilgisinden* oluşur. Bu, bellek açısından oldukça verimli örüntüler kurmamıza yardımcı olur. Mesela bütün karakter dizilerinin bir karakter dizisinden ödünç alındığı bir `Vec<&str>` oluşturulabilir - tek ihtiyacınız olan vektör için ek alan olacaktır:

Mesela, boşluklardan bölerken:

```rust
fn split_whitespace(s: &str) -> Vec<&str> {
    s.split_whitespace().collect()
}
```

Aynı şekilde, C++'daki `s.substr(0,2)` her zaman karakter dizisinin kopyasını oluşturur ancak dilim sadece ödünç alır: `&[0..2]`

Buna benzer ilişki `Vec<T>` ve `&[T]` arasında da bulunur.

### Paylaşılan Referanslar
C++'da bulunduğu gibi Rust için de *akıllı işaretçiler (smart pointers)* bulunur - mesela `std::unique_ptr` muadili `Box`tur. Bellek ve tahsis edilmiş diğer kaynaklar `Box` kapsam dışına çıktığı zaman geri iade edildiğinden `delete` kullanmaya ihtiyaç yoktur. (Rust RAII'yi epeyce benimsemiştir.)

```rust
let mut answer = Box::new("hello".to_string());
*answer = "world".to_string();
answer.push('!');
println!("{} {}", answer, answer.len());
```

İnsanlar başlangıçta `to_string`'i başlangıçta pek sevmez ancak işleri *açıktan* yapmayı sağlar.

Açık dereferans operatörü olan `*` önemli ancak metotlar üzerinde herhangi bir özel notasyon kullanmadığımıza dikkat edin. (Mesela burada `(*answer).push('!') yok`)

Ödünç almanın sadece orijinal içeriğin sahibi belli olduğu zaman işe yaradığı açıktır. Çoğu tasarımda bu mümkün değildir.

Bu C++'da `std::shared_ptr`'in kullanıldığı yerdir; kopyalama sadece veri üzerindeki referans sayısını attırır. Bunun da bir bedeli var, üstelik:

- veri sadece salt okunabilir olsa bile, sürekli referans sayımının arttırılması önbelleğin doğrulanamamasına sebep olabilir. 
- `std::shared_ptr` süreçler arası emniyetlice paylaşılabilecek şekilde tasarlanmıştır ve kendi kilidini de beraberinde taşıması kaba maliyeti arttırmaktadır.

Rust'ta `std::rc::Rc` da aynı zamanda referans sayımı yapan paylaşılan akıllı işaretçi gibi davranır. Fakat, bu sadece değişemez referanslar içindir! Eğer süreç anlamında emniyetli bir türünü istiyorsanız, `std::sync::Arc` ("Atomik Rc") kullanabilirsiniz. Rust iki farklı tür sunduğu için biraz tuhaf görünebilir fakat süreçlerin işin içine girmediği işlemler için kaba maliyeti arttırmamış olursunuz.

Değişemez referanslar olmalarının sebebi bunun Rust'ın bellek modelinin esaslarıyla olmasıyla alakalıdır. Fakat yine de sıyrılmanın bir yolu vardır: `std::cell:RefCell`. Eğer paylaşılan referansınızı `Rc<RefCell<T>>` olarak tanımlarsanız `borrow_mut` ile değişebilir referans edinebilirsiniz. Bu sefer Rust kurallarını *dinamik* olarak uygularsınız - mesela zaten bir referans varken ek olarak `borrow_mut` kullanırsanız bir paniğe sebep olacaktır.

Yine de bu hâlen daha *emniyetlidir*. Panikler bellekte yanlış bir yere dokunulduğu andan *önce* gerçekleşir! Tıpkı fırlatılan hatalar gibi, çağrı sırası teker teker boşaltılır. Bu derece yapılandırılmış bir süreç için talihsiz bir kelime seçimi diyebiliriz - bu panikleyerek kapanmak yerine sıralı bir temizliktir.

`Rc<RefCell<T>>` tipi göze biraz biraz batıyor olabilir, ancak kullanılış şekli kesinlikle kötü değil. Burada, Rust (tekrardan) işlerin açıktan yürümesini tercih etmiş oluyor.

Ortak durumu eğer bellek açısından güvenli bir şekilde paylaşmak istiyorsanız `Arc<T>` tek *emniyetli* yoldur. Eğer değişebilir erişimlere ihtiyaçlarınız varsa `Rc<RefCell<T>>` yerine `Arc<Mutex<T>>` kullanırsınız. `Mutex` tanımlandığından biraz daha farklı çalışır, bu veri için bir kutu görevi görür. Veriyi `lock` ile alır ve düzenlersiniz.
```rust
let answer = Arc::new(Mutex::new(10));

// in another thread
..
{
  let mut answer_ref = answer.lock().unwrap();
  *answer_ref = 42;
}
```
Neden `unwrap`? Eğer kilidi elinde tutan süreç paniklerse `lock` hata verir. (Resmi dokümentasyon bu tarz durumlarda `unwrap` kullanmanın mantıklı olduğunu düşünür çünkü belli bir şeyler çok yanlış gitmiş. Panikler süreçlerin içerisinde yakalanabilir.)

Özel kilidi mümkün olduğu sürece kısa sürece tamamlayıp işi teslim etmek (Mutexlerde her zaman olduğu gibi) önemlidir. Onları sınırlı bir kapsam içerisinde tutmak yaygın bir tercihtir - değişebilir referans kapsam dışına çıktığı zaman kilit de sona ermiş olur.

C++'ın sadeliği ile bunu kıyaslayınca ("dostum sadece `shared_ptr` kullan") göze biraz acayip görünüyor. Fakat bu şekilde paylaşılan veriler arasındaki herhangi bir *düzenleme* daha kolay fark ediliyor ve `Mutex` kilidi örüntüsü süreç emniyetine yönlendiriyor.

Her şeyde olduğu gibi, paylaşılan referansları kullanırken [dikkatli olun.](https://news.ycombinator.com/item?id=11698784).

### Döngüleyiciler
C++'da döngüleyiciler olağan bir yoldan yapılamaz; akıllı işaretçilere sahiptirler ve genellikle `c.begin()` ile başlar ve `c.end()` ile biterler. Döngüleyiciler üzerindeki operasyonlar yalnız başına şablon fonksiyonları olarak kullanılır, `std::find_if` gibi.

Rust döngüleyicileri ise `Iterator` özelliği ile tanımlanırlar; `next` bize bir `Option` döner ve `Option` artık bir `None` olduğu zaman işimiz biter.

Bilinen işlemler artık birer metot. Aşağıda `find_if`'in muadilini görebilirsiniz. Bize bir `Option` döner (çünkü hiç yoksa `None` cevabı alırız) ve `if let` deyimi aracılığıyla `None` olmayan durumda değere erişebiliriz: 

```rust
let arr = [10, 2, 30, 5];
if let Some(res) = arr.find(|x| x == 2) {
    // res is 2
}
```

### Emniyetsizlik ve Bağlı Listeler
Rust'ın standart kütüphanesinde `unsafe` (emniyetsiz kod bloğu) kullanıldığı bir sır değil. Bu ödünç alma kontrolünün muhafazakar anlayışını ihlal etmez. "emniyetsiz (unsafe)" kelimenin özel bir anlamı olduğuna dikkat edin - Rust'ın derleme zamanında anlayamadığı işlemler. Rust'ın perspektifinden C++ her an ve her zaman emniyetsiz modda çalışır! Büyük uygulamalarda birkaç düzine satırlık emniyetsiz kod gerekiyorsa, ki bu da olabilir, bir hata olduğu zaman bu satırların bir insan tarafından dikkatlice incelenmesi yeterli olacaktır. Bilirsiniz, insanoğlu 100 bin satır üstü kodları okumakta pek başarılı değildir.

Bundan bahsediyorum çünkü ortada bir örüntü var, tecrübeli bir C++ programcısı bir ağaç yapısını ya da bağlı liste oluşturduğu zaman kendisini biraz yılgın hisseder. Pekâlâ, çift bağlı liste üretmek emniyetli Rust için mümkündür; `Rc` akışı yönlendirir ve `Weak` referanslar aradan çekilir. Ancak standart kütüphane daha fazla performans elde etmek için... işaretçileri kullanır.