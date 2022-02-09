# Modüller ve Kargo

# Modüller

Programlar büyüdükçe onları bir dosyanın dışına taşımak ve fonksiyonlarla tipleri farklı *isim alanlarına (namespace)* taşımak gereklidir. Rust'ın bu iki şeye çözümü *modüllerdir. (modules)*

C ile başladı ama C ile bitmedi, bir süre sonra kendinizi `primitive_display_set_width` gibi rezilce isimler koyarken bulabilirsiniz. Sadece dosya isimlerini keyfinizce isimlendirebiliyorsunuz. 

Rust'ta aynı şeyi `primitive::display::set_width` şeklinde isimlendirebiliyorsunuz. Üstelik `use primitive::display` kullandıktan sonra bunu kısaca `display::set_width` olarak çağırabilirsiniz. Hatta `use primitive::display::set_width` dedikten sonra onu doğrudan `set_width` diye çağırabilirsiniz fakat nasıl kullandığınıza dikkat etmelisiniz. `rustc` tarafında sorun olmaz ancak sizin kafanız karışabilir. Ancak, bu sistemin çalışabilmesi için dosya isimlerinin basit bir kaç kurala bakması gereklidir.

Yeni bir anahtar kelimemiz var, `mod`, bir bloğu içine yazılan tip ve fonksiyonlarla beraber topyekûn modül olarak ilan etmeye yarar. 

```rust
mod foo {
    #[derive(Debug)]
    struct Foo {
        s: &'static str
    }
}

fn main() {
    let f = foo::Foo{s: "hello"};
    println!("{:?}", f);
}
```
Ancak bu çalışmayacaktır - "Foo yapısı gizlidir (struct Foo is private)" diye bir hata alacağız. Bunu çözmek için `pub` anahtar kelimesi aracılığıyla `Foo`'yu görünür kılmalıyız. Sonra bu hata "foo::Foo yapısının s alanı gizlidir (field s of struct foo::Foo is private)" olacaktır, `pub` anahtar kelimesini alanın başına eklemeliyiz ki `Foo::s` de görünür olsun. Sonra güzelinden çalışacaktır.

```rust
    pub struct Foo {
        pub s: &'static str
    }
```

Bir alanı açıkça `pub` olarak belirmek bir modulün içerisinden neyin ulaşılabilir olduğunu *seçmek* demektir. Bir modülün içerisindeki erişebilen tiplere ve fonksiyonlara modulün *arayüzü (interface)* denir.

Bir yapının içindekileri gizlemek ve erişimi metotlarla sağlamak çoğunlukla doğru bir tercihtir.

```rust
mod foo {
    #[derive(Debug)]
    pub struct Foo {
        s: &'static str
    }

    impl Foo {
        pub fn new(s: &'static str) -> Foo {
            Foo{s: s}
        }
    }
}

fn main() {
    let f = foo::Foo::new("hello");
    println!("{:?}", f);
}
```

Neden bu yapıları gizlemek daha iyidir? Çünkü arayüzün canına okumadan ve modüle erişen diğer parçaların ayrıntılarıyla boğuşmadan onu değişebilirsiniz. Geniş ölçekli bir programın en büyük belası kodun birbirine girmeye olan meyilidir ki kodun gerekli parçasını izole etmeyi imkansız hâle getirir bu.

Cesur yeni dünyada modüller tek bir şeyi yapar ve kendi sırlarını kendilerine saklarlar.

Peki ne zaman gizlememeliyiz? Stroustrup'ın dediği gibi arayüzün kendisi kullanıldığı zaman, mesela `struct Point{x: f32, y: f32}`. 

Bir modülün içinde bütün nesneler birbirine görünürler. Burada herkesin birbirini tanıdığı ve sırlarını bildiği bir mahalle yaşantısı vardır. 

Herkesin programı çeşitli dosyalara ayırmaya başladığı bir sınırı vardır. Ben mesela 500 satıra geldiğimde bunu düşünmeye başlıyorum ancak hepimiz 2000 satırdan sonra sıkılırız.

Peki ya bir programı çeşitli dosyalara nasıl ayırırız?

`foo` kodunu `foo.rs` içine koyalım.

```rust
// foo.rs
#[derive(Debug)]
pub struct Foo {
    s: &'static str
}

impl Foo {
    pub fn new(s: &'static str) -> Foo {
        Foo{s: s}
    }
}
```

Ve sonra da ana dosyada `mod foo` deyimini bir blok olmadan kullanalım.

```rust
// mod3.rs
mod foo;

fn main() {
    let f = foo::Foo::new("hello");
    println!("{:?}", f);
}
```

Şimdi `rustc mod3.rs` komutu herhangi bir hata olmadan derlenecektir. "Makefile"lar ile boğuşmaya hiç ihtiyacımız yok!

Ç.N: Makefile çoğunlukla C ve C++ ile kullanılan ancak Crystal, Go gibi yüksek seviye dillerde bile tercih edilen bir dosya. İşlevi birden çok kod dosyasını bir araya getirmek, onu yönetmektir. 

Derleyici aynı zamanda `MODULADI/mod.rs` içine de bakacaktır, mesela ben `boo` isminde bir dizin açıp içerisine `mod.rs` diye bir dosya yerleştirebilirim:

```rust
// boo/mod.rs
pub fn answer()->u32 {
    42
}
```

Ana dosya bunu farklı bir dosyadaki farklı bir modül olarak tanımlayacaktır:

```rust
// mod3.rs
mod foo;
mod boo;

fn main() {
    let f = foo::Foo::new("hello");
    let res = boo::answer();
    println!("{:?} {}", f,res);
}
```

Şu ana kadar içinde `main`  fonksiyonu olan bir `mod3.rs` dosyamızla beraber `boo/mod.rs` dosyamız da vardır ki diğer modüller bunu `boo` olarak görür. Genel alışkanlık, `main` fonksiyonunu barındıran dosyanın adını `main.rs` yapmaktır.

Neden bir şeyi yapmanın iki farklı yolu var? Çünkü `boo/mod.rs` aracılığıyla `boo` içerisinde yeni modüller tanımlayabilirsiniz. `boo/mod.rs`'yi değiştirelim ve yeni bir modül ekleyelim - bunu dışarıdan erişilebilir olmasına dikkat edin. (`pub` olmazsa `bar`a sadece `boo` modülü içerisinden erişebilirsiniz.)

```rust
// boo/mod.rs
pub fn answer()->u32 {
    42
}

pub mod bar {
    pub fn question() -> &'static str {
        "the meaning of everything"
    }
}
```

Ç.N: Question: Soru, Answer: Cevap

Şimdi, cevabımızı anlamlandıracak bir sorumuz var. (`bar` modülü, `boo`'nun içindedir.)

```rust
let q = boo::bar::question();
```

Dilersek modül bloğunu `boo/bar.rs` altına taşıyabiliriz.

```rust
// boo/bar.rs
pub fn question() -> &'static str {
    "the meaning of everything"
}
```

Ve `boo/mod.rs` şuna dönüşür:

```rust
// boo/mod.rs
pub fn answer()->u32 {
    42
}

pub mod bar;
```

Sonuç olarak modüller organizasyon ve erişilebilirlikle alakalı ve tercihen başka dosyalara erişebilir.  

Lütfen `use`'ın herhangi bir içe aktarma işleminde kullanılmadığını ve kısayol oluşturduğumuza not edin. Örneğin:

```rust
{
    use boo::bar;
    let q = bar::question();
    ...
}
{
    use boo::bar::question();
    let q = question();
    ...
}
```

Bir başka önemli nokta ise Rust'ta *parçalı derleme* işlemlerinin bulunmamasıdır. Ana program ve onun modül dosyaları sil baştan yeniden derlenir. Büyük programların derlenme süresini kayda değer bir sürede uzatır, `rustc` zaman içerisinde artımlı derlemede iyileşmesine rağmen.

# Sandıklar
> *Ç.N: Crate kelimesinin yaygınlığından ötürü "sandık" ya da "crate" arasında aklımda uzunca bir süre seçim yaptım. Çünkü bu genel programlamaya ait bir kelime değil, Rust terminolojisinin bir parçası ki bu da onu çevrilmemesi gereken bir özel isim yapar. Ancak "Sandık" kelimesi gerçekten mantığa uygun ve "Sandık" olarak düşünmenin hiçbir zararı yok. "Crate" diyerek geçseydim, İngilizce bilmeyen kişiler için bunu salt ezberlenmesi gereken, anlamsız bir kelimeye dönüştürürdüm.*

"Her bir derleme parçasına" *sandık (crate)* denir ki bu bir kütüphane veyahut çalıştırılabilir bir dosya olabilir. 

Geçen bölümdeki dosyaları hep birlikte değil de ayrıca derlemek için, önce `foo.rs`'ı bir *statik kütüphane sandığına* çevirelim.

```
src$ rustc foo.rs --crate-type=lib
src$ ls -l libfoo.rlib
-rw-rw-r-- 1 steve steve 7888 Jan  5 13:35 libfoo.rlib
```

Şimdi bunu bizim ana programımıza *ilişkilendirebiliriz. (linking)*

```
src$ rustc mod4.rs --extern foo=libfoo.rlib
```

Ana programımızın bu yeni yapıya uyum sağlaması gerekmektedir, `extern` (Dışsal) ile kullandığımız isim ilişkilendirdiğimiz zaman kullandığımız isimle aynı olmalıdır. Yeni kütüphanemiz artık `foo` modülü aracılığıyla görünür olacaktır:

```
// mod4.rs
extern crate foo;

fn main() {
    let f = foo::Foo::new("hello");
    println!("{:?}", f);
}
```

İnsanlar "Cargo! Cargo!" diye zikre başlamadan önce Rust'ın inşa araçlarını neden bu kadar düşük seviyeden gösterdiğini anlatmam için bana izin verin. Ben "Aletlerin Fıkhı"na dahilim ve bunları bilmek sizin Cargo ile yeni projeler yönetirken daha az "sinir"le karşılaşmanızı sağlar. Modüller basit dil işlevleridir ve Cargo olmadan da kullabilirler.

Şimdi, Rust'ın çalıştırılabilir dosyaları neden bu kadr büyük onu anlayalım:

```
src$ ls -lh mod4
-rwxrwxr-x 1 steve steve 3,4M Jan  5 13:39 mod4

```
Yarım dünya olmuş! Aslında bu çalıştırılabilir dosyada *pek çok* hata ayıklama bilgisi bulunur. Eğer niyetiniz bir hata ayıklayıcı kullanmaksa ve program paniklediği zaman anlamlı geri dönüşler almak istiyorsanız bu kötü bir şey değildir.

Hata ayıklama bilgisini silelim ve bir de böyle bakalım:

```
src$ strip mod4
src$ ls -lh mod4
-rwxrwxr-x 1 steve steve 300K Jan  5 13:49 mod4
```

Yine de basit bir şey için büyük bir dosya olduğunu düşünebilirsiniz ancak bu program Rust'ın standart kütüphanesine *statik* linklenmiştir. Bu iyi bir şey, bu programı doğru işletim sistemini kullanan herkesle paylaşabilirsiniz - Rust ile ilişkili hiçbir araca ihtiyaçları yoktur. (`rustup` sayesinde farklı işletim sistemlerine ve platformlara derleyebilirsiniz.)

Rust'ın kütüphanelerine dinamik linkleyebiliriz ki bu koşulda gerçekten küçük çalıştırılabilir dosyalar elde ederiz.

```
src$ rustc -C prefer-dynamic mod4.rs --extern foo=libfoo.rlib
src$ ls -lh mod4
-rwxrwxr-x 1 steve steve 14K Jan  5 13:53 mod4
src$ ldd mod4
    linux-vdso.so.1 =>  (0x00007fffa8746000)
    libstd-b4054fae3db32020.so => not found
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f3cd47aa000)
    /lib64/ld-linux-x86-64.so.2 (0x00007f3cd4d72000)
```

"not found (bulunamadı)" çıktısının sebebi `rustup`'ın dinamik kütüphanelerinin sistem çapında kurulamamış olması. En azından Unix'te şöyle bir şey yapabiliriz. (Sembolik bağların en iyi çözüm olduğunu ben de biliyorum.)

```
src$ export LD_LIBRARY_PATH=~/.rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib
src$ ./mod4
Foo { s: "hello" }
```

*Teorik olarak* Rust'ın dinamik linklemeyle ilgili herhangi bir sorunu yok, tıpkı Go gibi. Sadece her altı haftada yeni bir stabil sürüm yayınlandığı için her şeyi tekrar tekrar derlemek biraz tuhaf kaçacaktır. Eğer her şeyin sizin için uygun olacağı bir stabil sürüm bulursanız, bunda sorun olmayacaktır. İşletim sistemlerinin paket yöneticileri Rust'ın standart kütüphanelerini sunmaya başlayacağı zaman dinamik linkleme daha popüler olacaktır.

# Cargo
Java veya Python ile kıyaslarsanız Rust'ın standart kütüphanesi o kadar da büyük değildir, tabii yine de çoğu şeyini işletim sistemi kütüphanelerinden alan C ve C++'dan daha fazla şey bulursunuz.

Bu durumu telafi etmek için **Cargo** aracılığı ile [crates.io](https://crates.io)'da yayınlanan topluluk kütüphanelerine ulaşabilirsiniz. Cargo sizin için doğru sürümü arayacak ve kaynağı indirecek ve diğer bağımlılıkların kurulduğunu da kontrol edecektir. 

JSON okuyan basit bir program yapalım. Bu veri formatı yaygın olarak kullanılır ancak standart kütüphaneye eklenemeyecek kadar da karmaşıktır. Bundan ötürü yeni bir Cargo projesi açıyoruz, "--bin" de ekliyoruz ki çalıştırılabilir bir proje yapalım yoksa kütüphane projesi hazırlar.

Ç.N: Hayır hazırlamaz. Çevrilen belgenin eskiliğinden dolayı böyle bahsetmiş. Varsayılan davranış çalıştırılabilir proje hazırlamaktır, kütüphanesi projesi başlatmak için `--lib` kullanmanız gerekir. Yine de `--bin` kullanabilirsiniz ancak buna gerek yoktur. Bu arada bahsi geçen JSON sandığı bu yazının yazıldığı tarihe (8 şubat 2022) iki yıldır güncellenmemiş görünmektedir. Rust'ta kullanılan esas JSON çözümü [serde_json sandığıdır.](https://crates.io/crates/serde_json) Yazının devamını okuyabilirsiniz çünkü `serde_json` ve `json` sandıkları arasında pratikte pek fark yoktur. Kaldı ki bu bölümün ardından yazar `serde_json`'u inceliyor.

```
test$ cargo init --bin test-json
     Created binary (application) project
test$ cd test-json
test$ cat Cargo.toml
[package]
name = "test-json"
version = "0.1.0"
authors = ["Your Name <you@example.org>"]

[dependencies]
```

[JSON sandığını](https://github.com/maciejhirsz/json-rust) kullanan bir proje yapmak için "Cargo.toml" dosyasını düzenleyin:

```
[dependencies]
json="0.11.4"
```
Sonra Cargo ile ilk derlememizi yapalım:

```
test-json$ cargo build
    Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading json v0.11.4
   Compiling json v0.11.4
   Compiling test-json v0.1.0 (file:///home/steve/c/rust/test/test-json)
    Finished debug [unoptimized + debuginfo] target(s) in 1.75 secs
```

Programımızın `main` dosyası hâlihazırda oluşturdu - "src" dizinindeki "main.rs" dosyasıdır. Şimdilik henüz "hello world" çıktısı vermekten başka bir şeye yaramıyor, hadi onu doğru düzgün bir test programına çevirelim.

"raw" karakter dizesinin nasıl kullanıldığına da dikkat edin - eğer kullanmasaydık kaçış dizelerini kullanmamız gerekirdi ki bu bayağı bir çirkinliğe sebep olurdu:

```rust
// test-json/src/main.rs
extern crate json;

fn main() {
    let doc = json::parse(r#"
    {
        "code": 200,
        "success": true,
        "payload": {
            "features": [
                "awesome",
                "easyAPI",
                "lowLearningCurve"
            ]
        }
    }
    "#).expect("parse failed");

    println!("debug {:?}", doc);
    println!("display {}", doc);
}
```

Ç.N: Cargo aracılığıyla kurduğunuz sandıkların "extern crate *sandık_adı*" şeklinde çağrılmasına gerek yoktur. 

Şimdi projeyi inşa edip çalıştırabilir - sadece `main.rs` değişti.

```
test-json$ cargo run
   Compiling test-json v0.1.0 (file:///home/steve/c/rust/test/test-json)
    Finished debug [unoptimized + debuginfo] target(s) in 0.21 secs
     Running `target/debug/test-json`
debug Object(Object { store: [("code", Number(Number { category: 1, exponent: 0, mantissa: 200 }),
 0, 1), ("success", Boolean(true), 0, 2), ("payload", Object(Object { store: [("features",
 Array([Short("awesome"), Short("easyAPI"), Short("lowLearningCurve")]), 0, 0)] }), 0, 0)] })
display {"code":200,"success":true,"payload":{"features":["awesome","easyAPI","lowLearningCurve"]}}
```

Hata ayıklama çıktısı JSON belgesi hakkında bazı iç detayları sundu ancak `Display` özelliğini kullanan sade `{}` bizim için taranmış JSON'u döner.

Şimdi JSON API'sini keşfe çıkalım. Eğer verileri dışarı çıkartamasaydık bunun pek anlamı olmazdı. `as_TİP` metotu bize `Option<TİP>` döner, bunun nedeni belirtilen alanın varlığı kesin değildir ve doğru tipe çevirmeyebiliriz.

```rust
    let code = doc["code"].as_u32().unwrap_or(0);
    let success = doc["success"].as_bool().unwrap_or(false);

    assert_eq!(code, 200);
    assert_eq!(success, true);

    let features = &doc["payload"]["features"];
    for v in features.members() {
        println!("{}", v.as_str().unwrap()); // MIGHT explode
    }
    // awesome
    // easyAPI
    // lowLearningCurve
```

`features`, `JsonValue` tipine bir referanstır - referans olması gerekir çünkü *veriyi* JSON dökümanı dışına taşımış oluruz. Ayrıca bir tür dizi olduğunu bildiğimiz için `members()` bize `&JsonValue` üzerinde çalışan dolu bir döngüleyici dönecektir. 

Ya eğer "payload"'ın "feratures" diye bir anahtarı olmasaydı? O zaman `features` bir `Null` olurdu, elimizde patlamazdı. Bu yaklaşım biraz serbestlik tanıyor ki bu JSON'un gevşek doğasına da pekâlâ uyuyor. Belgenin yapısını incelemek ve yapı uyuşmazsa hataları idare etmek size kalmış.

Bu yapıları düzenleyebilirsiniz. Eğer `let mut doc` diye tanımlasaydık pekâlâ bunu yapabilirdik:

```rust
    let features = &mut doc["payload"]["features"];
    features.push("cargo!").expect("couldn't push");
```

`push` başarısız olabilir çünkü `features` bir dizi olmayabilir, bundan ötürü `Result<()>` döner.

*JSON kalıplarını* kullanmak için gerçekten şık bir makromuz da var:

```rust
    let data = object!{
        "name"    => "John Doe",
        "age"     => 30,
        "numbers" => array![10,53,553]
    };
    assert_eq!(
        data.dump(),
        r#"{"name":"John Doe","age":30,"numbers":[10,53,553]}"#
    );
```

Bunun çalışabilmesi için makroları JSON sandığından açıkça içe aktarmanız gerekir:

```rust
#[macro_use]
extern crate json;
```

Bu sandığın kötü bir tarafı da var, o da JSON'un dengesiz ve dinamik tipli doğasını Rust'ın statik ve yapılandırmış doğası arasında uyumsuzluktan doğuyor. ("Readme" dosyasında bu sürtüşmeden ("friction") söz eder.) Eğer JSON'u Rust'ın veri yapılarına çevirmek isterseniz en sonunda pek çok düzenleme yapmanız gerekir çünkü elde ettiğiniz veriyi yapılarınıza uyacağınızdan emin olamazsınız! Bunun üstesinden gelebilmek için [serde_json](https://github.com/serde-rs/json) kullanabilirsiniz, bununla Rust veri yapılarınızı JSON'a *serileştirebilir/çevirebilir (serialize)* ya da JSON'dan Rust'a *serisizleştirebilirsiniz/geri çevirebilirsiniz. (deserialize)*

Ç.N: "Muvaffakiyetsizleştiricileştiriveremeyebileceklerimizdenmişsinizcesine" gibi bir karmaşa yarattığımın farkındayım. Bundan dolayı bazı yerlerde *serialize* ve *deserialize* için *çevirmek* karşılığını kullanacağım. 

Bunun için `cargo new --bin test-serde-json` ile çalıştırılabilir bir Cargo projesi başlatalım ve `test-serde-json` dizinine girip `Cargo.toml'u` değiştirelim. Buna benzer bir şey yapabiliriz:

```
[dependencies]
serde="0.9"
serde_derive="0.9"
serde_json="0.9"
```

`src/main.rs`'yi şu şekilde dolduralım:

```rust
#[macro_use]
extern crate serde_derive;
extern crate serde_json;

#[derive(Serialize, Deserialize, Debug)]
struct Person {
    name: String,
    age: u8,
    address: Address,
    phones: Vec<String>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Address {
    street: String,
    city: String,
}

fn main() {
    let data = r#" {
     "name": "John Doe", "age": 43,
     "address": {"street": "main", "city":"Downtown"},
     "phones":["27726550023"]
    } "#;
    let p: Person = serde_json::from_str(data).expect("deserialize error");
    println!("Please call {} at the number {}", p.name, p.phones[0]);

    println!("{:#?}",p);
}
```

`derive` özelliğini daha çok görünüz ancak `serde_derive` sandığı içerisinde `Serialize` ve `Deserialize` gibi önemli özellikleri içeren `derive`lar bulunmaktadır. Sonuç, oluşturulan Rust yapısını gösterecektir:

```
Please call John Doe at the number 27726550023
Person {
    name: "John Doe",
    age: 43,
    address: Address {
        street: "main",
        city: "Downtown"
    },
    phones: [
        "27726550023"
    ]
}
```

Eğer bunu `json` sandığı ile yapsaydınız pek çok satırda çoğu hata yönetimiyle ilişkili birkaç yüz tanecik çeviri kodu yazmanız gerekecekti. Bunaltıcı, batırması bir hataya bakar ve bunun için oturup uğraşmak anlamsız. 

Eğer dışarıdan gelen iyi yapılandırılmış JSON'u işleyecekseniz (gerekirse alanları yeniden isimlendirebilirsiniz) ve başka programlarla ağ üzerinden veri paylaşacaksanız (Şu günlerde JSON'u herkes anlıyor) bu bariz en iyi çözümdür. `serde` ("SERialization DEserialization") hakkındaki ilgi çeken bir nokta diğer dosya türlerini de desteklemesidir, mesela Cargo'da kullanılan yapılandırması kolay ve popüler `toml` da bunlardan birisidir. Eğer programınızın .toml dosyalarını yapılara çevirmesi gerekiyorsa bu tıpkı .json dosyalarında yaptığınız gibi bu yapıları hazırlayabilirsiniz.

Serileştirme, Java ve Go'da da benzerleri bulunan önemli bir tekniktir - ancak büyük bir fark vardır. Bu dillerde verinin yapısı çalışma zamanında *yansıma (reflection)* kullanılarak bulunur. Ancak bu koşulda serileştirme işlemi *çalışma zamanında* oluşturulur - bu çok daha verimlidir.

Cargo, Rust ekosisteminin ağır toplarından birisidir çünkü bizim için çok fazla işi halleder. Eğer o olmasaydı Github'dan tek tek kütüphaneleri indirmek, statik kütüphaneler olarak inşa etmek ve programa ilişkilendirmemiz gerekirdi. Bunu C++ projelerinde yapmak çiledir ve aynı çile Cargo olmasaydı Rust projelerinde de olacaktır. C++'ın çektirdiği çilenin eşi benzeri de yoktur bu arada, o yüzden diğer dillerin paket yöneticileriyle kıyaslamamız daha doğru olacaktır. (JavaScript için) npm, (Python için) pip sizin için bağımlılıkları kontrol eder ve indirir ancak programı dağıtmak zordur çünkü kullanıcının da sizin yerine NodeJS ve Python kurmuş olması gerekir. Ancak Rust programın bağımlılıkları statik linklenmiştir, ek bir bağımlılık gerekmeksizin istediğiniz kişiye yollayabilirsiniz.

# Vali Kebabı
Basit bir yazının dışında herhangi bir veriyi işleme alacaksanız düzenli ifadeler (**reg**ular **ex**pressions) hayatınızı kolaylaştıracaktır. Bunlar pek çok dilde vardır ve sizin temel regex kalıplarına aşina olduğunuzu varsayıyorum. [regex](https://github.com/rust-lang/regex) sandığını kullanmak için Cargo.toml dosyasına "[dependencies]" altına 'regex = "0.2.1"' ifadesini koymanız yererlidir.

Terk eğik çizgilerin özel anlamlar yaratmaması için "çiğ/raw karakter dizilerini" kullanacağım. İnsanın anlayacağı şekilde anlatırsak aşağıdaki düzenli ifade " ':' karakterinden önce iki rakam, sonrasında da herhangi uzunluktaki bir rakamı alın" anlamına gelmektedir:

```rust
extern crate regex;
use regex::Regex;

let re = Regex::new(r"(\d{2}):(\d+)").unwrap();
println!("{:?}", re.captures("  10:230"));
println!("{:?}", re.captures("[22:2]"));
println!("{:?}", re.captures("10:x23"));
// Some(Captures({0: Some("10:230"), 1: Some("10"), 2: Some("230")}))
// Some(Captures({0: Some("22:2"), 1: Some("22"), 2: Some("2")}))
// None
```

Başarılı bir çıktı üç parçadan oluşur - bütün eşleşme ile iki parça olarak sayılar. Düzenli ifadeler genel varsayılan olarak *mıhlanmamıştır (anchored)*, yani **regex** ifademiz ilk eşleşmeyi arayacak ve gerisine bakmayacaktır. (Eğer sadece "()" olarak bir düzenli ifade yazarsanız her şeyle eşleşecektir.)

Bu eşleşmeleri *isimlendirebiliriz* ve düzenli ifadeleri birden fazla satır hâlinde yazabiliriz, satırları da içine alacak şekilde. Mesela burada sonucu ilişkisel bir dizi olarak kullanabiliriz ve eşleşmeleri isme göre arayabiliriz. 

```rust
let re = Regex::new(r"(?x)
(?P<year>\d{4})  # the year
-
(?P<month>\d{2}) # the month
-
(?P<day>\d{2})   # the day
").expect("bad regex");
let caps = re.captures("2010-03-14").expect("match failed");

assert_eq!("2010", &caps["year"]);
assert_eq!("03", &caps["month"]);
assert_eq!("14", &caps["day"]);
```

Düzenli ifadeler karakter dizilerini örüntü eşleştirmelere göre bölebilir ancak ne anlama geldiğini anlayamaz. Mesela ISO-tarzı bir sözdizimini belirtip eşleşeyebilirsiniz, ancak *anlamsal* olarak saçma sapan şeylere işaret edebilirler. Mesela ki "2014-24-52".

Bu koşulda ayrıca tarih-zaman işlemeye ihtiyacınız olabilir, bu bize [chrono](https://github.com/lifthrasiir/rust-chrono) tarafından sunulur. Tarihleri üretirken zaman dilimini belirtmeniz de gerekebilir:

```rust
extern crate chrono;
use chrono::*;

fn main() {
    let date = Local.ymd(2010,3,14);
    println!("date was {}", date);
}
// date was 2010-03-14+02:00
```

Ancak bu tarz kullanımda kötü tarihler şuna sebep olabilir: panik! (Mesela sahte bir tarih kullanmayı deneyebilirsiniz.) Esas ihtiyacınız olan metot `ymd_opt`'dir ki size `LocalResult<Date>` döner.

```rust
    let date = Local.ymd_opt(2010,3,14);
    println!("date was {:?}", date);
    // date was Single(2010-03-14+02:00)

    let date = Local.ymd_opt(2014,24,52);
    println!("date was {:?}", date);
    // date was None
```
Doğrudan tarihi ve zamanı tarayabilirsiniz, standart UTC biçiminde ya da farklı  [biçimler](https://lifthrasiir.github.io/rust-chrono/chrono/format/strftime/index.html#specifiers)'da olabilir. Bu hemen hemen aynı formatlar istediğiniz tarzda biçimlendirmenize yardımcı olur.

Bu iki sandıktan özellikle bahsettiğm çünkü normalde bunlar diğer dillerde standart kütüphanenin birer parçasıdır. Ve üstelik bu kütüphanelerin çok çok ilkel bir tipi aslında Rust'ın standart kütüphanesinin parçasıydı, ancak ayrıştırıldı. Bu bilinçli bir karardı, Rust takımı standart kütüphanenin kararlılığını oldukça ciddiye alıyor ve ekleyecekleri yeni özellikleri önce kararsız "nightly" yayınında sonra yarı kararlı "beta" yayınında test ederler ve en sonunda "stable" yayınına alırlar. Pek çok deneyim ve iyileştirme gereken kütüphaneleri bağımsız olarak geliştirmek ve Cargo ile erişilir kılmak çok daha iyidir. Sonuca bakarsak, bu iki sandık standart*tır* - tekrardan standart kütüphaneye alınmayacaktır ve ortadan kaybolmayacaklardır.