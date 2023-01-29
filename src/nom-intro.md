## Yazıları Nom ile Ayrıştırmak

[Nom](https://github.com/Geal/nom), [(burada anlatıldığı şekilde)](https://docs.rs/nom) öğrenmeye değer bir metin ayrıştırma için kullanılan bir Rust kütüphanesidir.

Eğer CSV veya JSON gibi türü bilinen bir veri türünü ayrıştırmak istiyorsanız bu işin özelleşmiş kütüphanelerden birisi olan [Rust CSV](https://github.com/BurntSushi/rust-csv) veya [Bölüm 4'te] bahsedilen JavaScript kütüphanelerinden birisine bakmak isteyebilirsiniz. 

Aynı şekilde, [ini](https://docs.rs/rust-ini/0.10.0/ini/) veya [toml](http://alexcrichton.com/toml-rs/toml/index.html). gibi yapılandırma dosyaları için onlara özel kendisine özgü kütüphanelere göz atabilirsiniz. ([serde_json](https://docs.rs/serde_json)'den bildiğimiz Serde Frameworkü ile uyumlu çalıştığı için Toml için hazırlanan kütüphane ayrıca hoştur.)

Fakat belli bir standarda ait olmayan, keyfe keder bir şekilde hazırlanmış verilerı taramak karakter dizileriyle geçireceğiniz sıkıcı saatlere işaret ediyor da olabilir. İlk fikir [regex](https://github.com/rust-lang/regex) olur, ancak regex bir yerden sonra alabildiğine mantıksız bir şeye dönüşebilir. Nom, metin ayrılmanın güçlü ve sadece basit araçları birleştirmekten ibaret olduğu güzel bir yol sunar. Aynı zamanda regexlerin bir sınırı vardır, mesela [HTML taramak için regex kullanamazsınız.](http://stackoverflow.com/questions/1732348/regex-match-open-tags-except-xhtml-self-contained-tags), fakat Nom ile HTML ayrıştırabilirsiniz. Hatta kendi programlama dilinizi yazmayı düşündüyseniz, Nom öğrenmek bu zorlu yolculuğun ilk adımı olabilir.

Nom öğrenmek için müthiş rehberler var, ancak ben biraz sindirerek gitmek istediğim için en basit yerden başlamak istiyorum. Bilmeniz gereken ilk şey, Nom baştan aşağıya makrolardan oluşur, ikincisi Nom karakter dizileri yerine bayt dilimleriyle çalışmayı tercih eder. Birinci şey, Nom'u kullanırken dikkatli olmanız gerektiğine işaret eder çünkü hata mesajlarından hiçbir şey anlamayabilirsiniz. İkincisi Nom'u *herhangi* bir veri türüyle kullanabileceğinizi işaret eder, sadece "metin" ayıklamak için değil. Nom kullanmış kişiler ikili biçimleri deşifre etmek veya dosya başlıklarını anlamak için kullandı. "UTF-8" ile kodlanmamış metinlerle de çalışabilirsiniz.

Nom'un son versiyonları karakter dizileriyle de çalışabilmeyi başladı, fakat karakter dizileriyle çalışabilen makroların sonunda `_s` bulunur.

```rust
#[macro_use]
extern crate nom;

named!(get_greeting<&str,&str>,
    tag_s!("hi")
);

fn main() {
    let res = get_greeting("hi there");
    println!("{:?}",res);
}
// Done(" there", "hi")

```
`named!` isimli makro (varsayılan olarak `&[u8]` tipinden) girdi alıp ve sivri parantezlerin ikinci argümanının tipinden geri dönen fonksiyonlar oluşturur. `tag_s!` ise kendisine iletilen karakter dizisi ile eşleşir, ve değer genellikle verilenin türünden olur. (Eğer `&[u8]` ile çalışmak isterseniz, bunun yerine `tag!` kullanabilirsiniz.) 

Tanımladığımız `get_greeting` ayrıştırıcısını bir `&str` ile çağırabiliriz ve bize [IResult](http://rust.unhandledexpression.com/nom/enum.IResult.html) dönecektir, bir de elbette ki eşleşen veriyi.

Boşlukları görmezden gelmek isteyebiliriz, `tag!` makrosunu `ws!` ile sarmalarsak aradığımız "hi" kelimesini eşleştirirken bütün boşluklar görmezden gelinecektir:
```rust
named!(get_greeting<&str,&str>,
    ws!(tag_s!("hi"))
);

fn main() {
    let res = get_greeting("hi there");
    println!("{:?}",res);
}
// Done("there", "hi")
```
Sonuç daha önce olduğu gibi "hi" olacaktır, ardında kalan karakter dizisi boşlukları kaldırılmış bir şekilde "there" olacaktır!

Tamam, "hi" eşleşmesi tıkırında çalışıyor ama bir şey yaramıyor. Hadi sadece "hi" yerine *hem* "hi"  *hem de* "bye" kısmını eşleştirelim.  `alt!` makrosu ("alternatif") `|` ile ayrılmış ayrıştırıcılardan *birisiyle* eşleşir. Aynı şekilde burada boşlukları okunaklı olması için kullanabilirsiniz: 
```rust
named!(get_greeting<&str>,
    ws!(alt!(tag_s!("hi") | tag_s!("bye")))
);
println!("{:?}", get_greeting(" hi "));
println!("{:?}", get_greeting(" bye "));
println!("{:?}", get_greeting("  hola "));
// Done("", "hi")
// Done("", "bye")
// Error(Alt)
```

Sonuncu hatalı, çünkü "hola" ile eşleşen bir metnimiz yok.

Doğrusu `IResult` tipini iyice anlamamız gerekiyor ki daha ileriye gidebilelim; fakat neden bunu bir "regex" ifadesiyle kıyaslamıyoruz?

```rust
    let greetings = Regex::new(r"\s*(hi|bye)\s*").expect("bad regex");
    let caps = greetings.captures(" hi ").expect("match failed");
    println!("{:?}",caps);
// Captures({0: Some(" hi "), 1: Some("hi")})
```

Doğrusu Regex göze daha *sade* görünüyor! Sadece parantez içine `|` koyduk ve bir tarafına "hi" diğer tarafına "bye" yerleştirdik. İlk sonuç girdi olarak aldığımız karakter dizisi, ikincisi de eşleşen ifade. (`|` regex için sözde "çeşitlilik (alternation)" operatörüdür, `alt!` makrosuna ilham vermiştir.)  

Fakat bu basit bir regex olsa bile bir anda herkes karmaşıklaşabilir. İşin ilginci metinlerde sıkça kullanılan `*` ve `(` gibi karakterlerden kaçınmanız gerekir ve `(hi)` veya `(bye)` ile eşleşen bir regex ifadesi yazmak isterseniz sevimli regeximix `\s*((hi | bye))\s*` gibi ucube bir hâl alacaktır. Bunun Nom muadili, gayet anlaşılır bir biçimde `alt!(tag_s!("(hi)") | tag_s!("(bye)"))` şeklindedir.

İşin kötüsü `regex` kütüphanesi ağır bir bağımlılıktır. Ananıza babanıza ancak verebileceğiniz bu i5 işlemcili laptota "Merhaba Dünya" seviyesi Nom örneklerinin derlenmesi sadece 0.55 saniye sürüyor. Fakat aynı şey regex için 0.90 saniye sürüyor. Aynı şekilde `strip` komutu uygulanmış ikili programın boyutu 0.3Mb tutarken (Statik linklenmiş bir Rust programının tutabileceği en küçük boyut) Regex örneği için 0.8Mb tutmaktadır. (Ç.N: Gözünüze bunlar anlamsız salt istatiksel veriler gibi görünebilir, ancak program büyüdükçe bu kütüphaneler kullanıldıkça bu farkın nasıl da katlanarak artacağını gözünüzde canlandırın.)

## Nom Ayrıştırıcısı Bize Ne Döner?

[IResult](http://rust.unhandledexpression.com/nom/enum.IResult.html) tipi standart `Result` tipinden daha çok şey döner. Üç ihtimal var:

  - `Done` - başarılı - sonucu ve geri kalan baytları alırsınız.
  - `Error` - ayrıştırma başarısız - bir hata alırsınız.
  - `Imcomplete` - (tamamlanmadı) daha fazla veriye ihtiyaç vardır.

Hata ayrışma çıktısını bize dönebilen herhangi bir veriyi argüman olarak alan genellenen bir `dump` fonksiyonu yazabiliriz. Bu örnek aynı zamanda bize bildiğimiz `Result`'u dönen `to_result` metodununu nasıl kullanılabileceğini de gösterir - bu metodu veriyi ya da hatayı istediğiniz durumların çoğunda sıkça kullanacaksınızdır.
```rust
#[macro_use]
extern crate nom;
use nom::IResult;
use std::str::from_utf8;
use std::fmt::Debug;

fn dump<T: Debug>(res: IResult<&str,T>) {
    match res {
      IResult::Done(rest, value) => {println!("Done {:?} {:?}",rest,value)},
      IResult::Error(err) => {println!("Err {:?}",err)},
      IResult::Incomplete(needed) => {println!("Needed {:?}",needed)}
    }
}


fn main() {
    named!(get_greeting<&str,&str>,
        ws!(
            alt!( tag_s!("hi") | tag_s!("bye"))
        )
    );

    dump(get_greeting(" hi "));
    dump(get_greeting(" bye hi"));
    dump(get_greeting("  hola "));

    println!("result {:?}", get_greeting(" bye  ").to_result());
}
// Done "" "hi"
// Done "hi" "bye"
// Err Alt
// result Ok("bye")
```

Ayrıştırıcılar bize ayrıştırılmamış verileri de dönüyor ve yeterince girdi almadıklarını da ortaya çıakrırlar, fakat genellikle `to_result`'u tercih edeceksiniz.

## Ayrıştırıcıları Birleştirmek
Selamlama örneğimizle devam edelim ve "hi" veya "bye", artı bir isimden oluşan bir selamlama tasarlayalım. `nom::alpha` alfabetik karakter serileriyle eşleşecek `pair!` ise iki ayrıştırıcıyı tek bir demekte birleştirecektir.

```rust
    named!(full_greeting<&str,(&str,&str)>,
        pair!(
            get_greeting,
            nom::alpha
        )
    );

    println!("result {:?}", full_greeting(" hi Bob  ").to_result());
// result Ok(("hi", "Bob"))
```
Şimdi, selamlayıcımızın pek sosyal olduğunu veya kimsenin adını bilmediğini de hesaba katalım, ismi opsiyonel yapalım. Doğal olarak demetteki ikinci veri bir `Option` olacaktır.  
```rust
    named!(full_greeting<&str, (&str,Option<&str>)>,
        pair!(
            get_greeting,
            opt!(nom::alpha)
        )
    );

    println!("result {:?}", full_greeting(" hi Bob  ").to_result());
    println!("result {:?}", full_greeting(" bye ?").to_result());
// result Ok(("hi", Some("Bob")))
// result Ok(("bye", None))
```
Selamlama için kullandığımız ayrıştırıcı ile isimleri yakalayan ayrıştırıcıyı birleştirmenin ve isim yakalamayı opsiyonel yapmanın ne seviye kolay olduğuna dikkat edin. Bu Nom'un geldiği gücün kaynağıdır ve bu yüzden ona "ayrıştırıcıları birleştiren kütüphane" (parse combinator library) denir. Basit ayrıştırıcılardan birleşerek inşa olan karmaşık ayrıştırıcılar inşa edebilir ve bunları teker teker test edebilirsiniz. (Buna eşdeğer bir regex bir Perl programı gibi görünmeye başlardı: çünkü regexlerin birleşmesi pek hayra alamet değildir.)

Fakat, henüz istediğimiz noktaya varamadık! `full_greeting(" bye ")` bize bir `Incomplete` hatası olarak dönecektir. Nom için "bye"dan sonra isim gelmelidir ve bu yüzden bizden isim namına bir şeyler isteyecektir. Bu bir *akış ayrıştırıcısının (streaming parser)* çalışmasının nasıl çalışması gerektiğidir, bu sayede dosyaları parça parça iletebilirsiniz; ancak burada Nom'a girdinin yetersiz olacağını bildirmemiz gerekir. 
```rust
    named!(full_greeting<&str,(&str,Option<&str>)>,
        pair!(
            get_greeting,
            opt!(complete!(nom::alpha))
        )
    );

    println!("result {:?}", full_greeting(" bye ").to_result());
// result Ok(("bye", None))
```

## Numaraları Ayrıştırmak
Nom bir dizi rakam serisini taramaya yarayan  `digit` fonksiyonuna sahiptir. `map!` kullanarak bir yazıyı bir sayıya dönüştürebilir ve bir `Result` tipi içinde geri dönebiliriz.
```rust
use nom::digit;
use std::str::FromStr;
use std::num::ParseIntError;

named!(int8 <&str, Result<i8,ParseIntError>>,
    map!(digit, FromStr::from_str)
);

named!(int32 <&str, Result<i32,ParseIntError>>,
    map!(digit, FromStr::from_str)
);

println!("{:?}", int8("120"));
println!("{:?}", int8("1200"));
println!("{:?}", int8("x120"));
println!("{:?}", int32("1202"));

// Done("", Ok(120))
// Done("", Err(ParseIntError { kind: Overflow }))
// Error(Digit)
// Done("", Ok(1202))
```

Burada `Result`'a dönüşebilen bir `IResult` ayrıştırıcısı elde ederiz - ve  elbette ki, burada mümkün olan birden çok hata vardır. Fonksiyonların içeriklerinin aynı olduğuna dikkat edin, esas dönüşüm fonksiyonun döndüğü tipe bağlıdır.

Sayıların işareti olabilir. Sayıları bir çift parça hâlinde yakabilirsiniz; önce bir işaret gelir ardından rakam gelir.

Mesela: 

```rust
named!(signed_digits<&str, (Option<&str>,&str)>,
    pair!(
        opt!(alt!(tag_s!("+") | tag_s!("-"))),  // maybe sign?
        digit
    )
);

println!("signed {:?}", signed_digits("4"));
println!("signed {:?}", signed_digits("+12"));
// signed Done("", (None, "4"))
// signed Done("", (Some("+"), "12"))
```

Eğer hedefe odaklıysanız ve ara sonuçları atlamak istiyorsanız, `recognize!` istediğiniz şeyi verebilir.

```rust
named!(maybe_signed_digits<&str,&str>,
    recognize!(signed_digits)
);

println!("signed {:?}", maybe_signed_digits("+12"));
// signed Done("", "+12")
```

Bu teknikle noktalı sayıları da yakalayabiliriz. Bu eşleşmeler üzerinden bayt dilimlerinden karakter dizilerine ulaşıyoruz.  `tuple!`, `pair!`'in oluşturulan demetle ilgilenmediğimiz türünden bir muadili. `complete!` ise "yarım kalan selamlama"da yaşadığımız sorunu çözmek için kullandığımız bir araç - "12", noktalı olmasa da aslında geçerli bir sayıdır.
```rust
named!(floating_point<&str,&str>,
    recognize!(
        tuple!(
            maybe_signed_digits,
            opt!(complete!(pair!(
                tag_s!("."),
                digit
            ))),
            opt!(complete!(pair!(
                alt!(tag_s!("e") | tag_s!("E")),
                maybe_signed_digits
            )))
        )
    )
);
```

Yardımcı olacak minik bir makro tanımlayarak bazı geçerli testler üretebilriz. Bu testler, `floating _point` verilen metinden sayı yakalayabildiyse geçerli sonuç verecektir.

```rust
macro_rules! nom_eq {
    ($p:expr,$e:expr) => (
        assert_eq!($p($e).to_result().unwrap(), $e)
    )
}

nom_eq!(floating_point, "+2343");
nom_eq!(floating_point, "-2343");
nom_eq!(floating_point, "2343");
nom_eq!(floating_point, "2343.23");
nom_eq!(floating_point, "2e20");
nom_eq!(floating_point, "2.0e-6");
```
(Makrolar kodu *biraz* kirletilmiş gösterse de, testlerinizi hazırlamak faydalı bir şeydir.)

Ve metinleri ayrıştırıp noktalı sayılara çevirebilirsiniz. Burada akışa odaklanacağım ve hatayı uzaklaştıracağım: 
```rust
    named!(float64<f64>,
        map_res!(floating_point, FromStr::from_str)
    );
```

Lütfen birbirinden karmaşık testler ayrıştırıcılar oluşturmanın adım adım nasıl mümkün olduğuna dikkat edin, her bir parçayı ayrıca test edebilirsiniz. Bu, birleştirilmiş ayrıştırıcıların regexler üzerinde güçlü bir avantajıdır. Bu gayet klasik bir programlama taktiği olan "böl ve yönettir". 
## Çeşitli eşlemeler üzerinde işlemler
Sabit bir sayıda örüntüyü yakalayan ve bir Rust demeti dönen `pairs!` ve `tuple!` ile tanıştık.

Bir de `many0` ve `many1` var - ikisi de değişken sayıda örüntüyü bir vektör içerisinde tanımlar. İkisi artasındaki fark birisinin "sıfır veya daha fazla", diğerinin ise "bir veya daha fazla" şeyi yakalalıyor olmasıdır. (regexteki `*` ve `+` karakterini düşünün) Yani, `many1!(ws(float64))`, "1 2 3" şeklinde bir karakter dizisi bize `vec![1.0, 2.0, 3.0]` olarak dönmeyi tercih edecek ancak boş bir karakter dizisinde hata verecektir.

`fold_many0` ise bir *azaltma (reduce)* işlemidir. Ayrıştırılan değerler tek bir değerde bir ikili operatör kullanılarak tek bir değerde toplanır. Mesela, Rust programcıları döngüleyicilerin içeriğini toplamak kullanmak için `sum` gelmeden önce ne yapıyorsa bu da ona benzer; aşağıdaki `fold` *işleyici (accumulator)* için bir başlangıç değerine (burada sıfır) sahiptir ve işleyicinin ne yapacağını bildirmesi için `+` operatörünü kullanır.

```rust
    let res = [1,2,3].iter().fold(0,|acc,v| acc + v);
    println!("{}",res);
    // 6
```

Nom muadili şöyledir:

```rust
    named!(fold_sum<&str,f64>,
        fold_many1!(
            ws!(float64),
            0.0,
            |acc, v| acc + v
        )
    );

    println!("fold {}", fold_sum("1 2 3").to_result().unwrap());
    //fold 6
```
Şimdiye dek bütün ifadeleri yakalamaya çalıştık veya eşleşen baytları `recognize!` ile aldık:

```rust
    named!(pointf<(f64,&[u8],f64)>,
        tuple!(
            float64,
            tag_s!(","),
            float64
        )
    );

    println!("got {:?}", nom_res!(pointf,"20,52.2").unwrap());
 //got (20, ",", 52.2)
```

Karmaşık ifadeler için, ayrıştırıcıların bütün sonuçlarını almış olmak bizi dağınık bir çalışma prensibine sokar! Daha iyisini yapabiliriz.

`do_parse!` sadece ihtiyacınız olan değerlere erişmesinize izin verir. Yakalanan veriler `>>` ile ayrılır - ilginizi çeken verileri `isim: ayrıştırıcı` formatında işaretleyebilirsiniz. Son olarak, parantezler arasında kodunuzu belirtirsiniz.
```rust
    #[derive(Debug)]
    struct Point {
        x: f64,
        y: f64
    }

    named!(pointf<Point>,
        do_parse!(
            first: float64 >>
            tag_s!(",") >>
            second: float64
            >>
            (Point{x: first, y: second})
        )
    );

    println!("got {:?}", nom_res!(pointf,"20,52.2").unwrap());
// got Point { x: 20, y: 52.2 }
```

İlgilenmediğimiz değerleri (bu örnekte olduğu gibi virgül) bir isme bağlamıyoruz ve iki noktalı sayıyı bir yapı oluşturmak için geçici isimlere atıyoruz. Parantezler içinde kalan kısım ise bir Rust kodu olmalı.

## Aritmatik İfadeleri Ayrıştırmak
Gerekli bilgiler sayesinde basit aritmatik ifadeleri ayrıştırabiliriz. İşte regexlerle yapamayacağınız şeylere güzel bir örnek.

Aşağıda yapmaya çalıştığımız şey ifadelerimizi ayıklayacak şeyi basitten karmaşığa doğru inşa etmektir. İfadeler eklenip çıkartılabilir *terimlerden (term)* oluşur. Terimler ise çarpılıp bölünebilir *faktörlerden* oluşur. Ve (şimdilik), faktörler sadece noktalı sayılardır:
```rust
    named!(factor<f64>,
        ws!(float64)
    );

    named!(term<&str,f64>, do_parse!(
        init: factor >>
        res: fold_many0!(
            tuple!(
                alt!(tag_s!("*") | tag_s!("/")),
                factor
            ),
            init,
            |acc, v:(_,f64)| {
                if v.0 == "*" {acc * v.1} else {acc / v.1}
            }
        )
        >> (res)
    ));

    named!(expr<&str,f64>, do_parse!(
        init: term >>
        res: fold_many0!(
            tuple!(
                alt!(tag_s!("+") | tag_s!("-")),
                term
            ),
            init,
            |acc, v:(_,f64)| {
                if v.0 == "+" {acc + v.1} else {acc - v.1}
            }
        )
        >> (res)
    ));

```

İfadelerimiz daha net ifade edilmiş oldu - bir ifade bir terimden ve artılı eksili daha fazla terimden oluşur. Onları biriktirmiyoruz, fakat uygun operatör vasıtasıyla *işliyoruz. (fold)* (Bunun gibi durumlarda Rust, ifadenin türünü tam olarak anlayamadığından işin içinden çıkamaz ve bir ipucu ister). Bu sayede işlem önceliğini sağlamış oluruz - `*` her zaman `+` gibi şeyler.

Noktalı sayılar için test ifadelerine ihtiyacımız olacak, ve [bunun için bir sandık var.](http://brendanzab.github.io/approx/approx/index.html).

Cargo.toml dosyanıza `approx=0.1.1` satırını ekleyin ve işimize bakalım:

```rust
#[macro_use]
extern crate approx;
...
    assert_relative_eq!(fold_sum("1 2 3").to_result().unwrap(), 6.0);
```
Bir küçük bir test makrosu yazalım. `stringify!`, ifadeyi bir karakter dizisi ifadesine dönüştürür ve bunu `expr` içerisine argüman olarak iletebiliriz, sonra da sonucu Rust'ın bulacağı ifadenin sonucu ile kıyaslayalım:

```rust
    macro_rules! expr_eq {
        ($e:expr) => (assert_relative_eq!(
            expr(stringify!($e).to_result().unwrap(),
            $e)
        )
    }

    expr_eq!(2.3);
    expr_eq!(2.0 + 3.0 - 4.0);
    expr_eq!(2.0*3.0 - 4.0);
```

Şükela - sadece birkaç satırla ifade işleyicisi tanımladık! Daha iyi olabilir. `factor` içindeki numaralara bir alternatif ekleyebiliriz - parantez içindeki ifadeler için:

```rust
    named!(factor<&str,f64>,
        alt!(
            ws!(float64) |
            ws!(delimited!( tag_s!("("), expr, tag_s!(")") ))
        )
    );

    expr_eq!(2.2*(1.1 + 4.5)/3.4);
    expr_eq!((1.0 + 2.0)*(3.0 + 4.0*(5.0 + 6.0)));
```

Şükelalık ifadenin artık terimler açısından *özyinemeli (recursively)* olarak çağrılmasıdır!

`delimited!` makrosunun özel sırrı parantezlerin iç içe olabilmesidir - Nom parantezlerin kapandığından emin olacaktır.

Regexin yapabileceklerinin çok çok ötesindeki ve `strip` uygulanmış ikili dosyamız sadece 0.5Mb, ki  hâlen daha ekrana "Merhaba Dünya" yazdıran regex programımızın yarısı kadar ediyor bu.





