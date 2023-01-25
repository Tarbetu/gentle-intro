# Hataları Yönetmek

# Hataları Yönetmenin Temelleri

Eğer soru işareti operatörünü kullanmazsanız Rust'ta hata yönetimi epey sıkıcı olabilir. Ancak bunu yapabilmek için bazen herhangi bir hatayı barındırabilecek bir `Result` tipi oluşturabilmemiz gerekir. Bütün hatalar `std::error::Error` dönebildiğine göre herhangi bir hatayı `Box<Error>` ile gösterebiliriz. 

Düşünün ki hem girdi/çıktı işlemlerinden gelen hatayı hem de karakter dizisini sayıya çevirmekten gelen hatayı aynı fonksiyon içinde dönmek istiyoruz:

```rust
// box-error.rs
use std::fs::File;
use std::io::prelude::*;
use std::error::Error;

fn run(file: &str) -> Result<i32,Box<Error>> {
    let mut file = File::open(file)?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents.trim().parse()?)
}
```

Burada girdi/çıktı işlemleri için iki farklı soru işareti operatörü kullanılıyor (Ya dosya açılamazsa? Ya `String`e çevrilemezse?) ve bir de çeviri için ayrıca bir soru işareti operatörü kullanıyoruz. En sonunda da sonucu `Ok` ile dönüyoruz. Rust, `parse` üzerinden dönen `i32` tipi ile çalışabilir. 

`Result` tipi için bir kısayolu oluşturmak da oldukça olaydır:

```rust
type BoxResult<T> = Result<T,Box<Error>>;
```


Ancak bizim programımızın kendisine özgü hata tipleri olacağı için kendi hata tiplerimizi hazırlamamız gerekecek. Bunun için gereken malzemeler:

-   Tercihen `Debug`
-   Bir tutam `Display`
-   Ve son olarak, olmazsa olmazımız `Error`

Eğer bunlar olmazsa hata tipiniz kafası nasıl eserse öyle çalışabilir.

```rust
// error1.rs
use std::error::Error;
use std::fmt;

#[derive(Debug)]
struct MyError {
    details: String
}

impl MyError {
    fn new(msg: &str) -> MyError {
        MyError{details: msg.to_string()}
    }
}

impl fmt::Display for MyError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f,"{}",self.details)
    }
}

impl Error for MyError {
    fn description(&self) -> &str {
        &self.details
    }
}

// a test function that returns our error result
fn raises_my_error(yes: bool) -> Result<(),MyError> {
    if yes {
        Err(MyError::new("borked"))
    } else {
        Ok(())
    }
```

Sürekli sürekli `Result<T, MyError>` yazmak biraz yorucu olduğu için çeşitli Rust modüllerinin kendi `Result` tipleri vardır. Mesela `Result<T,io::Error>` yazmak yerine `io::Result<T>` yazabilirsiniz.

Sonraki örneğimizde çevrilemeyecek bir karakter dizisini ondalıklı sayıya çevirirken karşımıza çıkan hatayı kontrol etmemiz gerekecek.

Şimdiye kadar `?` ile elimizdeki ifadenin bir hata dönüp dönemeyeceğine bakarak çalıştı. Bunu belirleyen özellik (trait), `From` özelliğidir. `Box<Error>` ise `From`'a sahip bütün `Error` tiplerini kabul eder. 

Devam etmeden önce kendi yarattığınız `BoxResult` isimlendirmesini kullanabilir ve her şeyi tek elde toplayabilirsiniz, bu kendi yarattığımız hata tipini `Box<Error>`'a döndürecektir. Ufak uygulamalar için pekâlâ iyi bir tercih olabilir. Ancak ben size diğer hata türlerini kendi hata türümüze dâhil edebileceğiniz daha iyi bir örnek göstereceğim.

`ParseFloatError`, `Error`'u içerdiğinden dolayı kendi içinde `description()`'un da tanımlanmış olması gerek.

```rust
use std::num::ParseFloatError;

impl From<ParseFloatError> for MyError {
    fn from(err: ParseFloatError) -> Self {
        MyError::new(err.description())
    }
}

// and test!
fn parse_f64(s: &str, yes: bool) -> Result<f64,MyError> {
    raises_my_error(yes)?;
    let x: f64 = s.parse()?;
    Ok(x)
}
```

İlk `?` için pek bir olay yok. (`From` ile her tip kendisine dönüştürülür.) İkinci `?` ise `ParseFloatError` hatasını `MyError`'a çevirir.

Ve sonuç:

```rust
fn main() {
    println!(" {:?}", parse_f64("42",false));
    println!(" {:?}", parse_f64("42",true));
    println!(" {:?}", parse_f64("?42",false));
}
//  Ok(42)
//  Err(MyError { details: "borked" })
//  Err(MyError { details: "invalid float literal" })
```

Birazcık işi yokuşa sürse de hiç de karmaşık değil. İşin tadını kaçıran kısım yazdığımız `From` dönüşümleri yazdığımız hataların bizim `MyError` ile iyi anlaşması gerektiği - ya da bunları hiç düşünmeyin `Box<Error>` kullanın olsun bitsin. Yeni başlayanlar tek bir şeyi birden fazla yapabilmenin yolunu görünce kafaları karışır. Bir avakadoyu soymanın (ya da yeterince manyaksanız bir kediyi yüzmenin) her zaman başka bir yolu vardır. Bu esnekliğin bedeli birden çok seçeneğe sahip olmaktır. Hata kontrolü 200 satırlık bir program için büyük bir programdan daha basit olabilir. Ve eğer bu kıymetli kod atıklarınızı bir Cargo paketine dönüştürmek isterseniz hata işleme çok daha kıymetli bir hâle gelir.

Şu an için soru işareti operatörü yalnızca `Result` için çalışmakta, `Option` için değil, ve bu bir özelliktir, bir kısıtlama değil. `Option` tipinin `ok_or_else` isminde kendisini `Result`'a dönüştüren bir metotu vardır. Mesela, düşünün ki bir `HashMap`'ta aradığımız anahtar bulunmuyor:

```rust
let val = map.get("my_key").ok_or_else(|| MyError::new("my_key not defined"))?;
```

Şimdi hatamız gayet anlaşılır bir şekilde dönmüş oldu! (Bu form içerisinde bir kapama kullanılıyor, böylece buradaki hata ancak gerekirse yaratılırsa olacak.) 

**Çeviri Notu:** Yazarın dediğine karşın, daha sonra soru işareti operatörü `Option` tipine dönüşebilir bir şekilde güncellendi. Yani artık şu kullanım geçerlidir:

```Rust
let val = map.get("my_key")?
```

`get` metotundan dönen `None` değerini olduğu gibi ya da bir hata dönmek size kalmış. İki durumunda kendince artıları ve eksileri var. Yazarın bahsettiği gibi, Rust'ta bir şeyi yapmanın birden fazla yolu var. Daha fazla bilgi için Rust'ın [referans kitabını](https://doc.rust-lang.org/reference/expressions/operator-expr.html#the-question-mark-operator) ya da [kutsal kitabı](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html) inceleyebilirsiniz.

# error-chain ve hatalarla baş etme sanatı
**Çeviri notu**: Acı bir şekilde bahsetmeliyim ki, error-chain isimli sandık [2019 gibi](https://users.rust-lang.org/t/error-chain-is-no-longer-maintained/27561/2) tedavülden kaldırıldı. Bunun yerine `failure` isminde alternatif bir sandığa kişiler yönlendirilmiş ancak o da [2020 yılı gibi](https://github.com/rust-lang-deprecated/failure/pull/347) tedavülden kaldırılmış. Benzer konseptleri sağlayan iki sandık var:
- [Anyhow](https://github.com/dtolnay/anyhow): Kabaca hataların tipi ne olursa olsun yönlendirebileceğiniz bir tip sunuyor.
- [thiserror](https://github.com/dtolnay/thiserror): Bir yapıyı `derive(Error)` gibi basit bir şekilde hata tipine dönüştürmeye yarar.

Yazının gerisini hem orijinal metni korumak hem belli başlı konseptleri tanıtmak hem de hata yönetiminin geçmişini göstermek için çeviriyorum.

Önemsiz olmayan uygulamalar için [`error_chain`](http://brson.github.io/2016/11/30/starting-with-error-chain) sandığına göz atmalısınız. Minik bir makro bu kadar faydalı olabilir.

`cargo new test-error-chain` komutuyla çalıştırılabilir bir sandık oluşturun ve oluşturulan dizinin içine girin. `Cargo.toml`'un sonuna `error-chain="0.8.1"`'i ekleyin.

`error-chain`'in olayı elle tek tek yazmanız gereken hata tiplerinin tanımlarını sizin yerinize yazmaktır; yapılar oluşturmak ve `Display`, `Debug`, `Error` gibi bir hata tipi yaratmak için kullanılan özellikleri eklemek. Aynı zamanda öntanımlı olarak `From` özelliği de dâhil edilir ki normal karakter dizileri de hatalara dönüştürülebilirler.

İlk `src/main.rs` dosyamız alttakine benzeyecektir. `Main` içerisinden `run` fonksiyonu çağrılıyor, hataları satır satır yazıyor ve programı sıfır olmayan bir çıkış kodu ile program sonlandırılıyor. Hepsi bu. `error_chain` makrosu ile bütün gerekli tanımlar üretilmiş olacaktır, `errors` modülünü gerekirse daha büyük programlarda kendi dosyasına yerleştirebilirsiniz. `errors` modülünü global kapsama dağıtmamız gerekti çünkü kodumuzun üretilmiş özellikleri (trait) görebilmesi gerekliydi. Varsayılan olarak, bir adet `Error` yapısı ve bu hatayla birlikte `Result` tanımlanacaktır.

Burada `foreign_links` kullanarak `std::io::Error`'ün `From` kullanarak bizim istediğimiz hata tipine dönüşmesini sağlıyoruz

```rust
#[macro_use]
extern crate error_chain;

mod errors {
    error_chain!{
        foreign_links {
            Io(::std::io::Error);
        }
    }
}
use errors::*;

fn run() -> Result<()> {
    use std::fs::File;

    File::open("file")?;

    Ok(())
}


fn main() {
    if let Err(e) = run() {
        println!("error: {}", e);

        std::process::exit(1);
    }
}
// error: No such file or directory (os error 2)
```

`foreign_links` hayatımızı oldukça kolaylaştırdı zira soru işareti operatörü artık `std::io::Error`'u nasıl `error::Error`'a dönüştüreceğini biliyor. (Kaputun altında tam da gerektiği gibi makromuz `Form<std::io::Error>` dönüşümü tanımlıyor.)

Bütün olay `run` içerisinde dönüyor; şimdi programa ilk argüman olarak verilen dosyanın ilk on satırını yazdırmayı deneyelim. Ortada verilmiş herhangi bir argüman olmayabilir, bunu bilemeyiz. Tek gereken şey `Option<String>`'i bir `Result<String>`'e dönüştürebilmek. Bunu yapabilmek için iki `Option` metotumuz var ve ben en basit olanını seçtim. `Error` tipimiz `&str` için `From`'u içerdiğinden basitçe bir karakter dizisiyle yeni bir hata oluşturabiliriz.

```rust
fn run() -> Result<()> {
    use std::env::args;
    use std::fs::File;
    use std::io::BufReader;
    use std::io::prelude::*;

    let file = args().skip(1).next()
        .ok_or(Error::from("provide a file"))?;

    let f = File::open(&file)?;
    let mut l = 0;
    for line in BufReader::new(f).lines() {
        let line = line?;
        println!("{}", line);
        l += 1;
        if l == 10 {
            break;
        }
    }

    Ok(())
}
```

`bail!` isminde hata "fırlatmak" için kullanılan küçük ama etkili makromuzu da görelim. Bunun yerine `ok_or` kullanabilirdiniz:

 ```rust
    let file = match args().skip(1).next() {
        Some(s) => s,
        None => bail!("provide a file")
    };
```

Tıpkı `?` gibi _fonksiyondan erken döner. (early return)_

Dönen hata içeriğinde `ErrorKind` isimli bir enum barındırır, bu bizi farklı türlü hataları seçebilmemizi sağlar. (`Error::from(str)` şeklinde oluşturduğunuz) her hata `Msg` isimli varyantla eşleşir. `Foreign_links` ile tanımladığımız `Io` ise Girdi/Çıktı hatalarıyla eşleşir:

```rust
fn main() {
    if let Err(e) = run() {
        match e.kind() {
            &ErrorKind::Msg(ref s) => println!("msg {}",s),
            &ErrorKind::Io(ref s) => println!("io {}",s),
        }
        std::process::exit(1);
    }
}
// $ cargo run
// msg provide a file
// $ cargo run foo
// io No such file or directory (os error 2)
```

Yeni tür hatalar eklemek de oldukça basittir. `error_chain` içerisine `errors` isimli bir kısım ekleyin:

```rust
    error_chain!{
        foreign_links {
            Io(::std::io::Error);
        }

        errors {
            NoArgument(t: String) {
                display("no argument provided: '{}'", t)
            }
        }

    }
```

Bu oluşturduğumuz yeni tür hata için `Display`'ın nasıl çalışacağını tanımlar. Ve şimdi "argüman yok" tarzı hataları daha spesifik bir şekilde tanımlamış olduk, `ErrorKind::NoArgument`'e bir `String` değeri verebiliriz:

```rust
    let file = args().skip(1).next()
        .ok_or(ErrorKind::NoArgument("filename needed".to_string()))?;
```

Şimdi eşleştirmeniz gereken yeni bir `ErrorKind` daha var:

```rust
fn main() {
    if let Err(e) = run() {
        println!("error {}",e);
        match e.kind() {
            &ErrorKind::Msg(ref s) => println!("msg {}", s),
            &ErrorKind::Io(ref s) => println!("io {}", s),
            &ErrorKind::NoArgument(ref s) => println!("no argument {:?}", s),
        }
        std::process::exit(1);
    }
}
// cargo run
// error no argument provided: 'filename needed'
// no argument "filename needed"
```

Genellikle mümkün olduğunca hataları spesifikleştirmek daha kullanışlıdır, _bilhassa_ bu bir kütüphane fonksiyonuysa! Bu türe göre eşleştirme tekniği geleneksel hata yönetimine oldukça benzer, sadece burada `except` ya da `catch` blokları yerine eşleştirme yöntemlerini kullanıyoruz.

Sonuç olarak, **error-chain** sizin yerinize bir `Error` tipi oluşturur ve `Result<T>`'i `std::result::Result<T, Error>` olarak tanımlar. `Error` ise içeriğinde `ErrorKind` isimli bir enum barındırır ve varsayılan olarak karakter dizilerinden oluşan hatalarla eşleşen `Msg`'i barındırır. Harici hataları da iki farklı iş yapan `foregin_links` ile tanımlayabilirsiniz. Birincisi, yeni bir `ErrorKind` varyantı oluşturabilirsiniz. İkincisi dış hataları kendi hatamıza çeviren `From` tanımlarını hazırlar. Böylece kolaylıkla çeşitli hata türleri eklenebilir hâle gelir. Böylece artık kalıplaşmış olan pek çok koddan kurtulmuş oluyoruz.

# Hataları Zincirlemek
Bu sandığın esas güzelliği *hataları zincirlemek*.

Bir *kütüphane kullanıcısı* olarak sadece gelişigüzel bir girdi/çıktı almak biraz can sıkar. Tamam, bir dosyayı açamadık. Ama hangi dosya? En basitinden, benim için önemli olan nokta nedir?

`error_chain` (Tr: Hata zinciri) bu tarz aşırı genelleme sorununa karşı *hata zincirleme* çözümünü sunar. Dosyayı açmak istediğimiz zaman tembelce `?` kullanma alışkanlığımıza devam edebilir ve `io::Error`'a dönüştürebiliriz, ya da hatayı *zincirleyebiliriz.*

```rust
// non-specific error
let f = File::open(&file)?;

// a specific chained error
let f = File::open(&file).chain_err(|| "unable to read the damn file")?;
```

Şimdi programımızın `foreign_links` kullanmadan yazılan yeni bir versiyonuna bakalım.

```rust
#[macro_use]
extern crate error_chain;

mod errors {
    error_chain!{
    }

}
use errors::*;

fn run() -> Result<()> {
    use std::env::args;
    use std::fs::File;
    use std::io::BufReader;
    use std::io::prelude::*;

    let file = args().skip(1).next()
        .ok_or(Error::from("filename needed"))?;

    ///////// chain explicitly! ///////////
    let f = File::open(&file).chain_err(|| "unable to read the damn file")?;

    let mut l = 0;
    for line in BufReader::new(f).lines() {
        let line = line.chain_err(|| "cannot read a line")?;
        println!("{}", line);
        l += 1;
        if l == 10 {
            break;
        }
    }

    Ok(())
}


fn main() {
    if let Err(e) = run() {
        println!("error {}", e);

        /////// look at the chain of errors... ///////
        for e in e.iter().skip(1) {
            println!("caused by: {}", e);
        }

        std::process::exit(1);
    }
}
// $ cargo run foo
// error unable to read the damn file
// caused by: No such file or directory (os error 2)
```

Görmüş olduğunuz üzere `chain_err` metotu orijinal hatayı alıyor ve orijinal hatayı barındıran yeni bir hata yaratıyor - bu böyle sonsuza kadar gider. İlgili kapamalar hataya *dönüştürülebilen* herhangi bir veri dönebilir.

Rust makroları sizi pek çok şey yazmaktan kurtarabilir. `error-chain`'in `main` yerine geçebilecek ayrı bir makrosu bile vardır:

```rust
quick_main!(run);
```

(Zaten `run`  bütün olayın gerçekleştiği yerdir.)