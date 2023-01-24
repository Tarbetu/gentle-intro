# Dosya Sistemi ve Süreçler 

# Dosya Okumaya Farklı Bir Bakış
Birinci bölümün sonunda bütün dosyayı bir karakter dizisi içerisine aktarmayı göstermiştim. Doğal olarak bu her zaman iyi bir fikir olmayabilir, bu yüzden bir dosyayı satır satır okumayı göstereceğim.

`fs::File`, `io::Read`'ı tanımlar ki bu okunabilen her şeyin özelliğidir. (trait) Bu özellik, `u8` ile bayt içeren bir dilimi dolduran `read` metotunu tanımlar - bu özelliğin *gerekli* tek metotudur ve *beraberinde gelen* pek çok metotu bedavaya kapmış olursunuz, `Iterator` gibi. Bir bayt vektörünü doldurmak için `read_to_end`'u veya bir karakter dizisini doldurmak için `read_to_string`'i kullanabilirsiniz - UTF-8 ile kodlanmış olması bir zorunluluktur.

Bu arabellek kullanılmayan (buffering) "saf" bir okumadır. Arabellekli okuma için `read_line` ve `lines` döngüleyicisini sunan `io::BufRead` özelliğini kullanabilirsiniz. `io::BufReader`, `io::BufRead`'ın kullanımlarını okunabilir *herhangi* bir şey için sunacaktır.

`fs::File` ise aynı zamanda `io::Write`'ı da barındırır.

Bütün bu özelliklerin kullanılabilir olduğunu görmenin en kolay yolu `use std::io::prelude::*` kullanmaktır.

```rust
use std::fs::File;
use std::io;
use std::io::prelude::*;

fn read_all_lines(filename: &str) -> io::Result<()> {
    let file = File::open(&filename)?;

    let reader = io::BufReader::new(file);

    for line in reader.lines() {
        let line = line?;
        println!("{}", line);
    }
    Ok(())
}
```

`let line = line?` gözünüze biraz tuhaf görünebilir. Döngüleyiciden dönen `line`  aslında `io::Result<String>` tipidir ve `?` ile onu paketinden dışarı çıkartıyoruz. Bunu yapmamızın sebebi döngü esnasında bir şeylerin yanlış gide*bile*ceğidir. - Girdi/çıktı hataları, UTF-8 olmayan bir bayt serisini almak gibi şeyler.

Bir döngüleyici olarak `lines`'i `collect` ile bir vektöre kolayca çevirebiliriz ya da `enumerate` döngüleyicisi ile her satırın sırasını öğrenebiliriz.

Yine de bütün satırları okumak için en iyi tercih bu değildir çünkü her satır için yeni bir `String` tahsis edilir. En iyi yöntem `read_line` kullanmaktır, biraz daha acayip görünse de. Satırın bir satır sonu karakteri içerdiğine dikkat edin ki bunu `trim_right` ile siliyoruz.

```rust
    let mut reader = io::BufReader::new(file);
    let mut buf = String::new();
    while reader.read_line(&mut buf)? > 0 {
        {
            let line = buf.trim_right();
            println!("{}", line);
        }
        buf.clear();
    }
```

Sonuçta çok daha az tahsis etme işlemimiz oluyor çünkü bir karakter dizisini *temizlemek* onun tahsis edilmiş alanını boşaltmıyor, bu alanı işgal eden yeni tahsis işlemleriyle uğraşmıyoruz.

Bu arada ödünç alma işlemini bozmamak için blok kullandığımız durumlardan birisiyle karşı karşıyayız. `line`, `buf` tarafından ödünç alınıyor ve bu ödünç `buf`'ı tekrardan düzenlemeden önce belleği terk etmelidir. Yine Rust bizi aptalca bir şey yapmaktan alıkoymaya çalışıyor, ara belleği boşalttıktan *sonra* `line`'a ulaşmak gibi. (Ödünç alma mekanizması bazen sizi darlayabilir. Rust bu kodu inceleyecek ve `line`'ın, `buf.clear()`'dan sonra kullanılmadığını görecektir, bu "sözcüksel olmayan yaşam sürelerini (non-lexical lifetimes)" ayıklamasından kaynaklanır.

Bu pek şık görünmüyor. Belki size arabelleğe referanslar dönen bir döngüleyici veremem ama en azından döngüleyiciye *benzeyen* bir şeyler verebilirim.

Şimdi jenerik bir yapı tanımlayalım; `R` ismindeki tip parametremiz `Read` içeren her tipi kabul edebilir. Bu yapının içeriğinde okuyucu (reader) ve referansını alabileceğimiz bir arabelleğimiz (buffer, ya da kısaca buf) olacak.

```rust
// file5.rs
use std::fs::File;
use std::io;
use std::io::prelude::*;

struct Lines<R> {
    reader: io::BufReader<R>,
    buf: String
}

impl <R: Read> Lines<R> {
    fn new(r: R) -> Lines<R> {
        Lines{reader: io::BufReader::new(r), buf: String::new()}
    }
    ...
}
```

Şimdi `next` metotunu kullanalım. Tıpkı bir döngüleyici gibi `Option` dönecek, `None` döndüğü zaman döngüleyici başa dönmüş olacak. İçerisinden dönen tip `Result` olacak çünkü `read_line` başarısız olabilir ve *asla hataları görmezden gelmiyoruz*. Eğer başarısız olursa hatayı `Some<Result>` olarak dönebiliriz. Eğer dosyanın doğal sınırı olan "sıfır baytları" okunursa bu bir hata değildir, sadece `None`'dur.  

```rust
    fn next<'a>(&'a mut self) -> Option<io::Result<&'a str>>{
        self.buf.clear();
        match self.reader.read_line(&mut self.buf) {
            Ok(nbytes) => if nbytes == 0 {
                None // no more lines!
            } else {
                let line = self.buf.trim_right();
                Some(Ok(line))
            },
            Err(e) => Some(Err(e))
        }
    }
```

Şimdi, yaşam sürelerinin nasıl çalıştığına dikkat edin. Açık bir yaşam süresi belirtmemiz gerekti çünkü Rust hiçbir zaman karakter dizelerini yaşam sürelerini belirtmeden ödünç almaya izin vermeyecektir ve burada ödünç alınan karakter dizisinin yaşam süresini `self` ile birlikte belirtiyoruz.

Bu yaşam süresiyle birlikte bu tanım `Iterator` özelliği ile uyumsuz. Ancak uyumlu olsaydı sorunları kolayca görebilirdik, `collect`'in karakter dizileririnden vektör yapmaya çalıştığını düşünün. Bu mümkün değil zira hepsi aynı değişebilir (mutable) karakter dizisini ödünç almış olurdu! (Eğer *bütün* dosyayı bir karakter dizesine çevirmek isteseydiniz karakter dizelerinin `lines` metotu karakter dizeleri dönebilirdi çünkü hepsi esas karakter dizilerinden ödünç alınmıştır.)

Neticedeki döngü çok daha temizdir ve ara belleğe alma işlemi pek çok kullanıcı tarafından görünmez.

```rust
fn read_all_lines(filename: &str) -> io::Result<()> {
    let file = File::open(&filename)?;

    let mut lines = Lines::new(file);
    while let Some(line) = lines.next() {
        let line = line?;
        println!("{}", line);
    }

    Ok(())
}
```

Hatta eşleştirme karakter dizisi dilimini dışarı çıkartacağından, döngüyü bu şekilde de yazabilirsiniz:

```rust
    while let Some(Ok(line)) = lines.next() {
        println!("{}", line)?;
    }
```

Bunu yapmak isteyebilirsiniz ancak muhtemel hataları halının altına süpürmüş olursunuz; döngü bir hata söz konusu olduğu zaman sessizce duracaktır. Daha da ötesi, Rust'ın `UTF-8`'e çeviremediği ilk yerde duracaktır. Gündelik kodlar için kabul edilebilir ancak kodu yayına aldığınız zaman kötüdür.

# Dosyalara Yazmak

`Debug`'u kullanırken `write!` ile tanışmıştık - `Write`'ın kullanıldığı her yerde aynı zamanda `write!` kullanabiliriz. `print!` demenin farklı bir yoluna bakalım:

```rust
    let mut stdout = io::stdout();
    ...
    write!(stdout,"answer is {}\n", 42).expect("write failed");
```
Eğer bir hata *söz konusu olabilirse* bunu idare edebilirsiniz. Bu *genellikle* gerçekleşmez ancak yine de söz konusu olabilir. Eğer bir girdi/çıktı işlemi yapıyor Bu genellikle kabul edilebilir çünkü bir dosyayla oynaşıyorsanız `?` eklemeniz gereken bazı yerler olabilir. 

Ancak bir fark var; `print!`, `stdout`'u her yazım için kitler. Çoğu zaman istediğiniz şey budur çünkü çoklu süreçlerin yazıldığı programlarda `stdout`'u kitlemezseniz çıktınız tuhaf bir şekilde karmaşıklaşabilir. Ancak çok fazla metin yolluyorsanız `write!` çok daha hızlı davranacaktır.

Çeşitli dosyalar için `write!` kullanmamız gerekiyor. `out`,  `write_out`'un sonunda düşürüldüğü zaman dosya kapanır ki bu hem istenen şeydir hem de önemlidir.

```rust
// file6.rs
use std::fs::File;
use std::io;
use std::io::prelude::*;

fn write_out(f: &str) -> io::Result<()> {
    let mut out = File::create(f)?;
    write!(out,"answer is {}\n", 42)?;
    Ok(())
}

fn main() {
  write_out("test.txt").expect("write failed");
}
```

Eğer performansı önemsiyorsanız Rust'ın varsayılan olarak önbelleğe alınmadığını bilmeniz gerekir. Her bir yazma talebi doğrudan işletim sistemine gönderilir ki bu işleri oldukça yavaşlatır. Bundan bahsediyorum çünkü diğer programlama dillerinde varsayılan davranış farklıdır ve Rust'ın betik dilleri tarafından geride bırakıldığını görmek sizi epeyce şaşırtabilir! Nasıl ki `Read`'ın `io::BufReader`'ı varsa `io::BufWriter`ın da `Write`'ı var.

# Dosyalar, Konumlar ve Dizinler
Şimdi makinedeki Cargo dizinini bulan bir program yazalım. En basit yöntem `~/.cargo` altına bakmaktır. Ancak bu Unix kabuğu için geçerlidir, çoklu ortam desteği için `env::home_dir` fonksiyonunu kullanacağız. (Başarısız olabilir ancak ev dizini olmayan bir bilgisayar da Rust araçlarını barındırmaz.)

Ç.N: [`env::home_dir`](https://doc.rust-lang.org/std/env/fn.home_dir.html) fonksiyonu beklenildiği gibi çalışmadığı için `1.29.0`'dan itibaren tedavülden kaldırılmıştır. Bu fonksiyonu kullanmayın, Windows ve Unix ortamları için ev dizinlerini kendi yöntemlerinizle bulmanızı veya bir [crates.io](https://crates.io)'yu karıştırmanızı tavsiye ederim.

Sonra bir `PathBuf` yaratalım ve `push` metotunu *parçalardan* tam bir dosya konumu inşa etmek için kullanalım. (Bu `/` gibi bir şeyle ile debelenmekten çok daha kolaydır.)

```rust
// file7.rs
use std::env;
use std::path::PathBuf;

fn main() {
    let home = env::home_dir().expect("no home!");
    let mut path = PathBuf::new();
    path.push(home);
    path.push(".cargo");

    if path.is_dir() {
        println!("{}", path.display());
    }
}
```

`PathBuf`, `String` gibi çalışır - karakterlerin büyüyebilen bir paketidir ancak konum inşa etmek için kendi araçlarını kullanır. Ancak özelliklerinin çoğunluğu `Path`'ın referans versiyonundan gelir, tıpkı `&str` gibi. Yani, mesela, `is_dir` bir `Path` metotudur.

Bu kulağınıza şüphe uyandıran bir miras alma (inheritance) tarzı gibi gelebilir, ancak bu [Deref](https://doc.rust-lang.org/book/deref-coercions.html) özelliğinin maharetidir. Gözünüze tıpkı `String/&str` gibi görünebilir - bir `PathBuf` referansı bir  `Path` referansına *dönüşür/zorlanır. (coerced)* ("Zorlamak (Coerce)" kelimesi biraz ağır kaçmış olabilir ancak bu Rust sizin için dönüşüm uyguladığı nadir yerlerden birisidir.)

```rust
fn foo(p: &Path) {...}
...
let path = PathBuf::from(home);
foo(&path);
```

`PathBuf`'un en yakın arkadaşı `OsString`'tir, kendisi sistemden doğrudan aldığımız karakter dizilerini gösterir. (Aynı şekilde buna karşılık `OsString/&OsStr` ilişkisi de vardır.)

Bu tarz karakter dizilerinin UTF-8 olacağını *garanti edilmemiştir!* Gerçek hayatta her şey [karmaşıktır](https://news.ycombinator.com/item?id=10519932), özellikle "Her şey neden bu kadar zor" diye düşünürken. Sadede gelelim. Birincisi antik ASCII kodlamanın ve diğer diller için özel kodlamanın kullanıldığı yıllar oldu. İkincisi kendi aramızda konuştuğumuz dillerin kendisi de epey karmaşıktır. Mesela "noël" kelimesi *beş* Unikod kodu kadar yer tutar.

Modern işletim sistemlerinin dosya adlarının çoğu zaman Unikod olabileceği doğrudur. (Unix tarafı için UTF-8, Windows için UTF-16) Ama olmadığı zamanlar da vardır! Ve Rust bu olasılığı dikkatlice ele almalıdır. Örneğin `Path`, `as_os_str` diye `&OsStr` dönen bir metota sahiptir. Ancak `to_str`, bazen `Option<&str>` döner. Yani her zaman mümkün değildir!

İnsanlar genelde bu konuda biraz takılır çünkü "karakter" ve "karakter dizesi"ne fazlasıyla alıştılar. Einstein'ın dediği gibi programlama dilleri sade olmalı, basit değil. Bir sistem programlama dilinin `String/&str` ayrımına ihtiyacı vardır (ödünç alınmışa karşılık sahiplenmiş: kafaya epeyce yatıyor) ve Unikod karakter dizilerini standartlaştırmak için Unikod olmayan stilleri de kapsamalıdırlar - işte `OsString/&OsStr` kardeşlerin doğuşu. Bunların içeriğinde `String` benzeri enteresan metotlar bulunmadığına dikkat edin, çünkü tiplemelerinden tam olarak emin değiliz. 

Ancak, insanlar dosya isimlerini olağan karakter dizileriymiş gibi işlemeye alışkınlardır ki kolayca dosya konumlarını kullanmak ve değiştirmek için Rust'ta `PathBuf` vardır.

Bir konumun adresinin parçalarını temizleyebilmek için `pop` kullanabilirsiniz. Mesela bulunduğumuz dizinden başlayalım.

```rust
// file8.rs
use std::env;

fn main() {
    let mut path = env::current_dir().expect("can't access current dir");
    loop {
        println!("{}", path.display());
        if ! path.pop() {
            break;
        }
    }
}
// /home/steve/rust/gentle-intro/code
// /home/steve/rust/gentle-intro
// /home/steve/rust
// /home/steve
// /home
// /
```

Bu da daha kullanışlı bir hâli. Bir konfigrasyon dosyasını aramak için bir program yazdık ve bütün altdizinleri bu dosya için arıyoruz. Bunun için `/home/steve/rust/config.txt` diye bir dosya yazdık ve program `/home/steve/rust/gentle-intro/code` içerisinden başlıyor:

```rust
// file9.rs
use std::env;

fn main() {
    let mut path = env::current_dir().expect("can't access current dir");
    loop {
        path.push("config.txt");
        if path.is_file() {
            println!("gotcha {}", path.display());
            break;
        } else {
            path.pop();
        }
        if !path.pop() {
            break;
        }
    }
}
// gotcha /home/steve/rust/config.txt
```

Bu **git**'in nasıl çalıştığı depoyu nasıl bulduğunun yöntemidir de aynı zamanda. 

Bir dosya hakkındaki bilgiler *metaveri (metadata)* olarak geçer. Her zaman olduğu gibi bir hata olabilir - "Bulunamadı" dışında mesela dosyayı okumamız için gerekli izinlere sahip olamayabiliriz.

```rust
// file10.rs
use std::env;
use std::path::Path;

fn main() {
    let file = env::args().skip(1).next().unwrap_or("file10.rs".to_string());
    let path = Path::new(&file);
    match path.metadata() {
        Ok(data) => {
            println!("type {:?}", data.file_type());
            println!("len {}", data.len());
            println!("perm {:?}", data.permissions());
            println!("modified {:?}", data.modified());
        },
        Err(e) => println!("error {:?}", e)
    }
}
// type FileType(FileType { mode: 33204 })
// len 488
// perm Permissions(FilePermissions { mode: 436 })
// modified Ok(SystemTime { tv_sec: 1483866529, tv_nsec: 600495644 })
```

Bir dosyanın (bayt cinsinden) uzunluğu ve son değiştirilme tarihini bulmak oldukça kolaydır. (Ancak bunu yapamayacağımız zamanlar da vardır.) `File` tipinin ilgili metotları `is_dir` (Ç.N: Dizin midir?), `is_file` (Ç.N: Dosya mıdır?) ve `is_symlink`'dir. (Ç.N: "Sembolik bağ mıdır?")

`permissions` da ilginç bir metottur. Rust platformlar arası geçerli olmaya özen gösterir ve bu da bir "en düşük ortak paydaya erişme" durumudur. Genel olarak, dosya salt okunur ise onu sorgulayabilirsiniz - "yetkiler (permissions) konsepti Unix'te oldukça geniştir ve kullanıcı/grup/diğerleri için okuma/yazma/çalıştırma yetkilerini barındırır. 

Ancak Windows ile çalışmıyorsanız en azından yetki bitlerini öğrenmek için platforma yönelik özellikleri (trait) kapsama çağırabilirsiniz. (Genelde olduğu gibi özellikler kapsama çağrıldığı zaman etkinleşirler.) Sonrasında da programın dosyasını şöyle değiştirebiliriz:

```rust
use std::os::unix::fs::PermissionsExt;
...
println!("perm {:o}",data.permissions().mode());
// perm 755
```

("{:o}" sekizli (octal) sayı sistemine göre biçimlendirir.)

(Windows'ta bir dosyanın çalıştırılıp çalıştırılamayacağı dosyanın uzantısından tanımlanır. Çalıştırılabilir dosya uzantıları `PATHEXT` çevre değişkeninden öğrenilebilir - ".exe", ".bat" vs.)

Dosyalarla çalışmak için `std::fs` bize pek çok faydalı fonksiyonlar sunar, bir dosyayı kopyalayıp taşımak, sembolik bağ kurmak ve dizin oluşturmak gibi.

Bir dizinin içeriğini öğrenmek için bir döngüleyici sunan `std::fs::read_dir`'i kullanabilirsiniz. İşte karşınızda boyutları 1024 bayttan büyük olan ve uzantısı `.rs` ile biten bütün dosyalar!

```rust
fn dump_dir(dir: &str) -> io::Result<()> {
    for entry in fs::read_dir(dir)? {
        let entry = entry?;
        let data = entry.metadata()?;
        let path = entry.path();
        if data.is_file() {
            if let Some(ex) = path.extension() {
                if ex == "rs" && data.len() > 1024 {
                    println!("{} length {}", path.display(),data.len());
                }
            }
        }
    }
    Ok(())
}
// ./enum4.rs length 2401
// ./struct7.rs length 1151
// ./sexpr.rs length 7483
// ./struct6.rs length 1359
// ./new-sexpr.rs length 7719
```

`read_dir`'in başarısız olabileceği aşikar ("bulunamadı" ya da "yetki yok" gibi bir hata çıkarabilir) ancak aynı zamanda her yeni bir girdiye erişmek de başarısız olabilir. (Okunmuş veriyi arabelleğe alan `lines` döngüleyicini düşünün.) Ek olarak, ilgili girdi için uygun metaveriye ulaşamayabiliriz de. Bir dosyanın uzantısı da pekala olmayabilir, bunu ayrıca kontrol etmemiz gerekir.

Neden konumların üzerinde bir döngüleyici kullanmıyoruz? Unix'te `opendir` sistem çağrısı bu şekilde çalışır ancak Windows'ta dosyaların metaverisini almadan üzerinde bir döngü kuramazsınız. Dolayısıyla platformlar arasındaki kodun en verimli çalışmasının en zarif yoludur. 

Bu noktada "hatalarla boğuştuğunuz" için kendinizi bitkin hissedebilirsiniz. Ancak *hatalar her zaman* vardı - Rust sizin için yeni hatalar icat etmedi. Sadece hataları görmezden gelmemeniz için elinden geleni yapıyor. Herhangi bir işletim sistemi çağrısı başarısız olabilir. 

Java ve Python gibi dillerde hata atarsınız (throw exceptions); Go ve Lua gibi diller ise size iki veri döner ve birincisi sonuç ikincisi de hata olur, Rust'ta kitaplık fonksiyonlarının hata oluşturması kötü bir davranış olarak kabul edilir. Bu nedenle pek çok hata denetimi vardır ve fonksiyonlar erkenden dönebilir.

Ya hata alırsınız ya da almazsınız, Rust bu yüzden `Result` kullanır: hem hata hem de sonuç elde edemezsiniz. Ve soru işareti operatörü kontrol oldukça kolaylaştırır.

# Süreçler

Esas ihtiyaç duyduğumuz şeylerden bir şey programların başka bir programı çalıştırması ya da *süreç başlatmaktır*. Programınız pek çok alt süreç çalıştırabilir ve alt süreçler üst süreçlerle pek çok ilişkide bulunabilir.

Bir programı, argümanları da beraberinde kullanmanızı sağlayan `Command` yapısı ile çalıştırabilirsiniz. 

```rust
use std::process::Command;

fn main() {
    let status = Command::new("rustc")
        .arg("-V")
        .status()
        .expect("no rustc?");

    println!("cool {} code {}", status.success(), status.code().unwrap());
}
// rustc 1.15.0-nightly (8f02c429a 2016-12-15)
// cool true code 0
```

`new` programın adını alır (Eğer kesin bir dosya konumu değilse `PATH` içinde arar), `arg` yeni argümanlar ekler ve `status` programın çalışmasını tetikler. Program çalışınca bir `Result` alırsınız, `Ok` programın çalıştığını belirtir ki içeriğinde `ExitStatus` bulunur. Bizim örneğimizde programımız başarıyla çalıştı ve çıkış değeri olarak 0 döndü. (`unwrap` kullanmamızın sebebi eğer program bir sinyal ile "öldürülürse" her zaman çıkış değerini öğrenemeyecek olmamız.)

Eğer `-V`'yi `-v` ile değiştirirsek `rustc` başarısız olur.

```
error: no input filename given

cool false code 101
```

Üç farklı koşul gerçekleşebilir:
- Program var olmayabilir, çalıştırma iznimiz olmayabilir ya da kullanılamaz hâlde olabilir
- Program çalışmış ancak başarısız olmamış olabilir - sıfır olmayan çıkış değeri
- Program sıfır olan çıkış değeri ile sonuçlanmış olabilir. Yani başarılıdır!

Varsayılan olarak programın standart çıktısı (stdout/Standart output) ve standart hata çıktısı (stderr/Standart Error) terminale yönlendirilir.

Bazen çıktıyı yakalamakla da ilgilenebiliriz ki `output` metotu bu işe yarar.

```rust
// process2.rs
use std::process::Command;

fn main() {
    let output = Command::new("rustc")
        .arg("-V")
        .output()
        .expect("no rustc?");

    if output.status.success() {
        println!("ok!");
    }
    println!("len stdout {} stderr {}", output.stdout.len(), output.stderr.len());
}
// ok!
// len stdout 44 stderr 0
```

`status` ile alt süreçler sonuçlanana dek programımız durduruluyordu ve üç şey alıyorduk - sonuç (Önceden olduğu gibi), `stdout`'un ve `stderr`'un içeriği.

Şimdi ise yakaladığımız çıktı `Vec<u8>` içerisinde tutuluyor - sadece bayt olarak. İşletim sisteminden aldığımız şeylerin her zaman geçerli bir UTF-8 karakter dizisi olamayacağını hatırlayın. Aslında, bunun bir karakter dizisi *bile* olamayacağını bilmemiz gerekiyor - programlar tuhaf ikili veriler dönebilirler.  

Eğer çıktının UTF-8 olacağından eminsek bu vektörü ya da baytları `String::from_utf8` ile dönüştürebiliriz. Sonuç `Result` dönecektir çünkü dönüşümün gerçekleşeceğinden emin değiliz. İşi biraz daha gevşekçe yapan başka bir fonksiyonumuz var, `String::from_utf8_lossy`, bununla çeviriyi deneyebilir ve dönüştürülemeyen karakterlerin yerlerine � koyabilirsiniz. 

Aşağıda kabukta programda çalıştıran kullanışlı bir fonksiyon görmektesiniz. Programımız `stderr` ile `stdout`'u birleştirmek için sıradan bir kabuk tekniği kullanıyor. Kabuğun ismi Windows'ta biraz farklı ancak diğerleriyle de sorunsuz çalışacaktır.

```rust
fn shell(cmd: &str) -> (String,bool) {
    let cmd = format!("{} 2>&1",cmd);
    let shell = if cfg!(windows) {"cmd.exe"} else {"/bin/sh"};
    let flag = if cfg!(windows) {"/c"} else {"-c"};
    let output = Command::new(shell)
        .arg(flag)
        .arg(&cmd)
        .output()
        .expect("no shell?");
    (
        String::from_utf8_lossy(&output.stdout).trim_right().to_string(),
        output.status.success()
    )
}


fn shell_success(cmd: &str) -> Option<String> {
    let (output,success) = shell(cmd);
    if success {Some(output)} else {None}
}
```

Sağ taraftaki boşlukları biraz törpülüyorum ve böylece `shell("which rustc")` dediğimiz zaman doğrudan konumu alabiliyoruz. 

`Process` ile çalıştırılmış bir programın `current_dir` ile çalışma dizinini, `env` ile çevre değişkenlerini belirleyerek çalışma şeklini kontrol edebilirsiniz.

Şimdiye kadar programımız alt süreçlerin tamamlanmasını bekledi. Eğer `spawn` metotunu kullanırsanız program size hemen döner ve programın basitçe bitişini bekler ki bu esnada gidip başka şeyler yapabiliriz. Aşağıdaki örnek `stdout` ve `stderr`in de aynı zamanda susturulmasına da örnektir. 

```rust
// process5.rs
use std::process::{Command,Stdio};

fn main() {
    let mut child = Command::new("rustc")
        .stdout(Stdio::null())
        .stderr(Stdio::null())
        .spawn()
        .expect("no rustc?");

    let res = child.wait();
    println!("res {:?}", res);
}
```

Varsayılan olarak alt süreç üst sürecin standart girdisini ve çıktısını "miras alır". Ancak bu örnekte alt sürecin çıktısını "hiçliğe" yönlendirmiş olduk. Unix kabuğunda `> /dev/null 2> /dev/null` demekle aynı şey yani.

Rustta yaptığımız bu şeyleri sistem kabuğu (`sh` veya `cmd`) ile de yapabilirdik. Ancak bu yolla tamamen programatik bir şekilde süreç oluşturmayı kontrol etmiş oldunuz.

Bu örnekte eğer sadece `.stdout(Stdio::piped())` kullanmış olsaydık alt sürecin çıktısını bir izole etmiş olurduk. Sonra da `child.stdout` üzerinden izole ettiğimiz çıktıyı doğrudan okuyabilirdik. (`Read` özelliğini kullanıyor). Aynı şekilde doğrudan `child.stdin`'e yazabilmek için de `.stdout(Stdio::piped())` kullanabilirsiniz.

Ancak `wait` yerine `wait_with_output`'u kullansaydık bize `Result<Output>` dönerdi ve alt sürecin çıktısı `Output`'un `stdout` alanında daha önce olduğu gibi `Vec<u8>` olarak sunulurdu.

`Child` yapısında aynı zamanda aleni bir şekilde `kill` (öldür) metotu da bulunur.

Ç.N: Alt süreç İngilizce'de "çocuk süreç" anlamına gelen "Child process" olarak bahsedilir.