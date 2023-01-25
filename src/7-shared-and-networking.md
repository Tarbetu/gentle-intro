# Sistem Süreçleri, Ağlar and Paylaşım

## Değişemez olanı Değiştirmek

Eğer biraz dik kafalıysanız (anladığım kadarıyla) ve ödünç alma kurallarını nasıl es geçebilmenin bir yolu olup olmadığını kara kara düşünüyor olabilirsiniz. 

Bu minik programı bir inceleyin, derlenecek ve hiçbir hata vermeyecektir.

```rust
// cell.rs
use std::cell::Cell;

fn main() {
    let answer = Cell::new(42);

    assert_eq!(answer.get(), 42);

    answer.set(77);

    assert_eq!(answer.get(), 77);
}
```
Evet, `answer` değişkeni değişebilir olarak belirtilmemesine rağmen içeriği değişti! 

Bu gayet emniyetli, çünkü içeriğindeki `Cell` içindeki veri yalnızca `set` veya `get` ile erişilebilir. Bunun adı *iç değişebilirliktir (interior mutability)*: Eğer bir `v` isminde bir yapı (struct) tanımladıysam `v` isimli yapı değiştirilebilirse `v.a` da değiştirilebilir olur. `Cell`, biraz bu kuralı rahatlatıyor çünkü `set` aracılığıyla içeriğindeki değeri değiştirebiliyoruz.

Fakat, `Cell` sadece `Copy` özelliğine sahip tiplerle çalışır. (Mesela kullanıcının tanımadığı `Copy` özelliğine sahip tiplerle veyahut ilkel tiplerle)

Farklı türden verilerle çalışmak için referansa ihtiyaç duyarız, değişebilir ya da değişemez olması fark etmeksizin. İşte bize `RefCell` bize bu konuda yarımcı olur - içerideki veriye açıkça bir şekilde referanslarla erişirsiniz.  

```rust
// refcell.rs
use std::cell::RefCell;

fn main() {
    let greeting = RefCell::new("hello".to_string());

    assert_eq!(*greeting.borrow(), "hello");
    assert_eq!(greeting.borrow().len(), 5);

    *greeting.borrow_mut() = "hola".to_string();

    assert_eq!(*greeting.borrow(), "hola");
}
```

Dikkat edin, `greeting` değişebilir bir değişken olarak bildirilmedi!

Rust'ın dereferans operatörü bir tık kafa karıştırıcı olabilir, çünkü çoğu zaman buna ihtiyaç duymazsınız - mesela `greeting.borrow().len()` diye çağırsaydık metot kendiliğinden dereferans edeceği için sorun olmazdı. Ancak içeriden gelen `greeting.borrow()` üzerinden gelen `&String` veya `greeting.borrow_mut()` üzerinden gelen `&mut String` ile çalışmaya devam edebilmek adına `*` eklemeniz gerekecektir. 

`RefCell` kullanmak her zaman emniyetli değildir, hâlen daha geri dönen metotların temel kurallara riayet etmesi gerekmektedir.   

```rust
    let mut gr = greeting.borrow_mut(); // gr is a mutable borrow
    *gr = "hola".to_string();

    assert_eq!(*greeting.borrow(), "hola"); // <== we blow up here!
....
thread 'main' panicked at 'already mutably borrowed: BorrowError'
```
Eğer değişebilir bir referansınız hâlihazırda varsa tekrardan değişebilir referans oluşturamazsınız. Fakat bu sefer bu kural ihlali derleme zamanında değil *çalışma zamanında* anlaşılabilir. Çözüm, her zaman olduğu gibi, değişebilir referansları mümkün olduğunca az sayıda tutmaktır - örneğimizdeki kod için değişebilir referansımız `gr`'ı kod blokları içerisinde tutmayı düşünebilirsiniz böylece referansımız tekrar ödünç almadan önce düşürülmüş olur. 

Bu özellik iyi bir sebebiniz olmadan kullanmak isteyeceğiniz bir şey değil çünkü hatalarınızı derleme zamanında al*ma*yacaksınız. Bu tipler, olağan kuralların sizin işinizi engellediği ama doğrusunu ypatığınız durumlarda size `dinamik ödünç alma`  sunmak için vardır.

## Paylaşılan Referanslar
Şimdiye dek, değer ve ödünç alınmış referanslar derleme zamanında açıkça biliniyordu. Veri sahiptir, referanslar ise onsuz var olamaz. Fakat her şey bu düzgün desen tasarımına uygun değildir, mesela düşünelim ki `Rol` ve `Oyuncu` diye iki yapımız (struct) var. `Oyuncu`, `Rol` objelerine referanslar bulunan bir vektör tutuyor. Burada veriler arasında net bir şekilde birebir eşleşme yok ve `rustc`'yi buna ikna etmek işleri epeyce karmaşıklaştıracaktır.

`Rc` tıpkı `Box` gibi çalışır - heap belleği tahsis edilir ve veri bunun içine taşınır. Eğer bir `Box` klonlarsanız, içindeki klonlanmış veriyi de beraberinde tutan bir bellek alanı tahsis eder. Fakat bir `Rc` klonlamak bilgisayar için daha kolaydır, çünkü her klonlama yapacağınız zaman sadece verinin referans sayısını arttırır. Bu, bellek yönetimi için eski ve bilindik bir yöntemdir; mesela iOS ve MacOSlarda kullanılan Objective C'nin çalışma zamanında kullanılır. (Ç.N: Eğer ilgiliyseniz "Automatic Referance Counting" diye bir araştırma yapabilirsiniz.) Modern C++'da bu özellike `str::shared_ptr` aracılığıyla bulunabilir.

Bir `Rc`nin içinde bulunduğu kapsam sona erdiği zaman referans sayımı da bir azaltılır. Eğer bu sayı sıfır olursa bellek boşaltılır ve sahiplenilmiş olan veri de düşürülür.

```rust
// rc1.rs
use std::rc::Rc;

fn main() {
    let s = "hello dolly".to_string();
    let rs1 = Rc::new(s); // s moves to heap; ref count 1
    let rs2 = rs1.clone(); // ref count 2

    println!("len {}, {}", rs1.len(), rs2.len());
} // both rs1 and rs2 drop, string dies.
```
Orijinal veriye istediğiniz kadar çok referans alabilirsiniz - bu daha önce bahsettiğimiz `dinamik ödünç alma`dır. `T` verisi ve onun referansları olan `&T`'yi dikkatlice takip etmek zorunda değilsiniz. Bunun karşılığında makine çalışma zamanında biraz daha yorulmuş olur, bu yüzden `ilk` seçiminiz bu olmamalıdır; fakat bu şekilde ödünç alma mekanizmasının sert kurallarına ters düşebilecek paylaşım tasarıları kurgulayabilirsiniz. Dikkat edin ki `Rc` size değiştirilemez referanslar sağlayacaktır, aksi taktirde en basit ödünç alma kuralını ihlal ediyor olurduk. Bilirsiniz, huylu huyundan vazgeçmez.

`Oyuncu` örneğinde, rollerimizi `Vec<Rc<Rol>>` olarak tutabiliriz ve böylece rol ekleyip çıkartabiliriz ancak oluşturulduktan sonra rolleri değiştiremeyiz.

Fakat, ya `Oyuncu` başka bir takıma referanslar bulunduruyorsa ve takım oyuncu referanslarını tek bir vektör içinde tutuyorsa? Her şey değiştirilemez olur çünkü `Oyuncu` verilerinin `Rc` olarak depolanması gerekir! Bu durumda `RefCell` gerekli olur. Bu sefer bütün takım `Vec<Rc<RefCell<Oyuncu>>>` içinde tutulabilir. Oyuncuyu `borrow_mut`, aynı anda birden çok değişebilir referans tutmayacağımızdan emin olarak değiştirebiliriz. Mesela oyuncuya özel bir şeyler olursa bütün takımın güçleneceğine dair güçlü bir kuralımız var:

```rust
    for p in &self.team {
        p.borrow_mut().make_stronger();
    }
```
Kod fena değil, ancak tipler biraz ürkünç görünebilir. Her zaman `type` ile onlara daha basit isimler atayabiliriz:  
```rust
type PlayerRef = Rc<RefCell<Player>>;
```

## Çoklu Sistem Süreçleri
Yaklaşık bir yirmi yıldır, saf işlem hızından çoklu çekirdeklere bir geçiş var. Son teknoloji bilgisayarlarımızdan tamamen verim almak için bütün çekirdekleri kullanmalıyız. Bunun en iyi yolu arkaplanda alt süreçler oluşturmaktır, tıpkı daha önceden gördüğümüz `Command` ancak bu sefer bir senkranizasyon sorunumuz var; bu alt süreçleri beklemeden onların tamamlanıp tamamlanmadığından emin değiliz.

Ayrı iş süreçlerine ihtiyaç duymamızın tek sebebi bu değil elbette, bütün programı sırf bir girdi almak için bekletemezsiniz, mesela. 

Alt süreçler üretmek oldukça basit, sadece arkaplanda işletilecek `spawn` için bir kapama hazırlayın. 

```rust
// thread1.rs
use std::thread;
use std::time;

fn main() {
    thread::spawn(|| println!("hello"));
    thread::spawn(|| println!("dolly"));

    println!("so fine");
    // wait a little bit
    thread::sleep(time::Duration::from_millis(100));
}
// so fine
// hello
// dolly
```

Kod satırında gördüğünüz "wait a little bit", "az bekle" demektir ve pek de mantıklı bir çözüme benzemiyor. Bize dönen nesnelerin üzerinde `join` çağırmak biraz daha mantıklı, ana süreç bu süreçlerin tamamlanmasını bekleyecektir.

```rust
// thread2.rs
use std::thread;

fn main() {
    let t = thread::spawn(|| {
        println!("hello");
    });
    println!("wait {:?}", t.join());
}
// hello
// wait Ok(())
```
İşte başka bir tür acayiplik: Alt süreci paniklemeye zorlayalım:

```rust
    let t = thread::spawn(|| {
        println!("hello");
        panic!("I give up!");
    });
    println!("wait {:?}", t.join());
```
Beklediğimiz gibi panikledi, ancak sadece panikleyen süreç öldü! Ekrana hata mesajını `join` ile yazabiliyoruz, yani evet her paniğin sonu programın kapatılması değildir; ancak süreçler bilgisayar için yorucu bir işlemdir ve bu süreçleri kontrol etmenin bir yolu olarak görülmelidir.   
```
hello
thread '<unnamed>' panicked at 'I give up!', thread2.rs:7
note: Run with `RUST_BACKTRACE=1` for a backtrace.
wait Err(Any)
```

Dönen objeler çeşitli alt süreçlerini takip için etmek için kullanılabilir.

```rust
// thread4.rs
use std::thread;

fn main() {
    let mut threads = Vec::new();

    for i in 0..5 {
        let t = thread::spawn(move || {
            println!("hello {}", i);
        });
        threads.push(t);
    }

    for t in threads {
        t.join().expect("thread failed");
    }
}
// hello 0
// hello 2
// hello 4
// hello 3
// hello 1

```
Rust `join`den dönen sonucu ele almamız için bize ısrar eder, mesela alt süreç panikleyebilir. (Bu gerçekleştiği zaman genelde programın tamamını durdurmazsınız, sadece hataları not edersiniz, yeniden denersiniz vs.)

Alt süreçlerin işleyişi için belirli bir sıra yoktur (Program her çalışmada farklı bir sıra verir), ve buna esas noktadır - onlar gerçekten *bağımsız yürütme süreçleridir (independent threads of execution*. Çoklu süreçler kolaydır, esas olay *eşzamanlılıktır* (concurrency) - birden çok sürecin süreci senkronize etmek ve yönetmek. 

## Süreçler Ödünç Almaz
Süreç kapamaları dışarıdan veri alabilir, ancak *taşıyarak*, *ödünç alarak* değil!

```rust
// thread3.rs
use std::thread;

fn main() {
    let name = "dolly".to_string();
    let t = thread::spawn(|| {
        println!("hello {}", name);
    });
    println!("wait {:?}", t.join());
}
```

Ve işte karşınızda size yardımcı olan hata mesajı: 

```
error[E0373]: closure may outlive the current function, but it borrows `name`, which is owned by the current function
 --> thread3.rs:6:27
  |
6 |     let t = thread::spawn(|| {
  |                           ^^ may outlive borrowed value `name`
7 |         println!("hello {}", name);
  |                             ---- `name` is borrowed here
  |
help: to force the closure to take ownership of `name` (and any other referenced variables), use the `move` keyword, as shown:
  |     let t = thread::spawn(move || {
```
Anlaşılabilir! Bir fonksiyon içerisinde üretilen süreci düşünün - fonksiyon çağrısı sona erdikten sonra bile çalışmaya devam eder ve `name` dürüşürülebilir. Kapamamıza `move` eklemek bu sorunu çözecektir.

Fakat bu bir *taşımadır*, yani `name` sadece bir süreçte var olabilr! Referansları paylaşmanın mümkün olduğunu vurgulamak istiyorum, ancak `static` ömre sahip olmalıdırlar:

```rust
let name = "dolly";
let t1 = thread::spawn(move || {
    println!("hello {}", name);
});
let t2 = thread::spawn(move || {
    println!("goodbye {}", name);
});
```
`name`, bütün programın çalışma süresince var olacaktır (`static`), bu yüzden `rustc` kapama çalıştığı sürece `name` değerinin de varlığından emin olacaktır. Ancak, havalı referansların `static` ömürleri yoktur!

Alt süreçler ortak bir ortamı paylaşamazlar - bu Rust'ın kendi tarzıdır. Biraz detay girersek, olağan referansları paylaşamazlar çünkü kapamalar yakaladıkları verileri taşırlar.

Yine de *Paylaşılan referanslar* fena değil çünkü yaşam ömürleri "gerektiği kadardır" - ama bunun için `Rc` kullanamazsınız. Çünkü `Rc` *alt süreçler arası emniyete* sahip değildir (*thread safe*) - sadece süreçlerin olmadığı anlar için hızlı olmaya optimize edilmiştir. Neyse ki `Rc` kullanırsanız derleme zamanında bir hata alırsınız, derleyici sizin arkanızı kollar.

Alt süreçler için `std::sync::Arc` kullanmalısınız - "Arc"ın açılımı "Atomic Referance Counting" yani "Atomik Referans Sayımıdır". Hepsi bu, bu dost her şeyin tek bir işlemde değiştirileceğini garanti eder. Bu garantiyi sağlamak için de işlem gerçekleştirilirken kilitlenir ve o an sadece bir sürecin erişimine izin verilir. `clone` kullanmak, bir kopya üretmekten bilgisayar için daha zahmetsizdir. (Ç.N: Buradaki `clone`, baştan aşağıya bellekte veri kopyalayan `std::clone` değil, referans klonlayan `std::sync::Arc::clone` metotudur.)

```rust
// thread5.rs
use std::thread;
use std::sync::Arc;

struct MyString(String);

impl MyString {
    fn new(s: &str) -> MyString {
        MyString(s.to_string())
    }
}

fn main() {
    let mut threads = Vec::new();
    let name = Arc::new(MyString::new("dolly"));

    for i in 0..5 {
        let tname = name.clone();
        let t = thread::spawn(move || {
            println!("hello {} count {}", tname.0, i);
        });
        threads.push(t);
    }

    for t in threads {
        t.join().expect("thread failed");
    }
}
```

Bilinçli olarak `String` barındıran bir tip oluşturdum, ("newtype" ya da yenitür) çünkü `MyString`, `Clone` özelliğini taşımayacak. Fakat paylaşılan referanslar klonlanabilir!

`name`'e ait paylaşılan referanslar her yeni alt sürece `clone` ile oluşturulan yeni referanslar olarak iletilir ve kapama içerisine taşınır. Biraz fazla kod yazdırıyor, ancak bu emniyetli bir örüntüdür. Emniyet, eşzamanlılık için epey önemlidir çünkü sorunlar pek öngörülebilir değildir. Program sizin bilgisayarınızda düzgünce çalışsa bile bir sunucuda patlayabilir, mesela haftasonu keyif yaparken. Daha da kötüsü, problemi semptomları bir tanı koymanıza yardımcı olmayabilir. 

## Kanallar
Süreçler arasında veri paylaşmanın çeşitli yolları var. Rust içerisinde, bunlardan birisi *kanalları* kullanmaktır. `std::sync::mpsc::channel()`, *alıcı* kanal ve *verici* kanal olmak üzere bize iki veri tutan bir demet sunar. Her bir alt sürece verici `clone` ile yollanır, ve referans üzerinde `send` çağrılır. Ana süreç ise bu esnada alıcı üzerinde `recv`i çağırır.

`MPSC`'nin açılımı "Multiple Producer Single Consumer"dır, yani "Çoklu üretici, tek tüketici". Kanala veri yollamaya teşebbüs eden birden çok altsüreç oluşturacağız, ana sürecimiz de kanalı "tüketecek".
```rust
// thread9.rs
use std::thread;
use std::sync::mpsc;

fn main() {
    let nthreads = 5;
    let (tx, rx) = mpsc::channel();

    for i in 0..nthreads {
        let tx = tx.clone();
        thread::spawn(move || {
            let response = format!("hello {}", i);
            tx.send(response).unwrap();
        });
    }

    for _ in 0..nthreads {
        println!("got {:?}", rx.recv());
    }
}
// got Ok("hello 0")
// got Ok("hello 1")
// got Ok("hello 3")
// got Ok("hello 4")
// got Ok("hello 2")
```
Bu sefer `join` kullanmaya ihtiyacımız yok çünkü alt süreçler kendilerini sonlandırmasından sonra cevaplarını dönecektir, fakat bu her an gerçekleşebilir. `recv` süreci kilitleyecektir ve eğer yollayıcı kanal devredışı kalacaksa bir hata dönecektir. `recv_timeout` ise belli bir süre boyunca bloklayacaktır ve ek olarak bir de zamanaşımı hatası dönebilecektir.

`send` ise süreci kilitlemeyecektir, bu faydalıdır çünkü süreçler mesajın alınmasını beklemeye gerek duymaksızın bir veriyi kanala atıp devam edecektir. Ek olarak, kanalın bir belleği vardır, böylece birden çok `send` metodu çalışabilir ve alıcıya sırasıyla mesaj iletilecektir. 

Fakat, sürecin engellenmemesi aynı zamanda `Ok` değerinin mesajın başarıyla iletildiği anlamına gelmediğini de işaret eder.

`sync_channel` ise kanala veri yollarken süreci kilitler. Argüman sıfır olursa, alıcı mesajı `recv` ile alana kadar süreç kilitlenir. Süreçler ya buluşmalıdır ya da randevulaşmalıdır. (Her zaman yabancı kökenli kelimeler kulağa bir tık daha teknik ve doğru gelir.)

```rust
    let (tx, rx) = mpsc::sync_channel(0);

    let t1 = thread::spawn(move || {
        for i in 0..5 {
            tx.send(i).unwrap();
        }
    });

    for _ in 0..5 {
        let res = rx.recv().unwrap();
        println!("{}",res);
    }
    t1.join().unwrap();
```
Burada `send` kullanılmamışken `recv` kullanarak bir hataya kolayca sebep olabilir, mesela döngüyü `for i in 0..5` ile kurmak yerine `for i in 0..4` kullanarak. Süreç sona erer, `tx` düşer ve `recv` başarısız olur. Bu aynı zamanda bir süreç paniklediği zaman da gerçekleşir, stack yavaşça çözülür ve bütün veriler düşürülür.

Eğer `sync_channel` sıfır olmayan bir argümanla oluşturulursa, buna `n` diyelim, bu sefer en fazla `n` değeri alan bir sıra (queue) gibi davranır, `send` sadece sırada bekleyen `n`den fazla değer varsa süreci kilitleyecektir.

Kanallar güçlü tip (strongly type) mantığına uygundur, bu örnekte kanalın tipi `i32`dir, ancak tip çıkarımı bunu biraz gizler. Eğer farklı türden verilere ihtiyacınız varsa, numaralandırmalar (enum) bunu ifade etmek için uygundur. 

## Senkronizasyon
Senkranizasyona bakalım. `join` oldukça basit, tek işi bir iş parçacığı bitene kadar beklemek. `sync_channel` ise iki kanalı birbirine senkronize ediyor - son örneğimizde üretilen alt süreç ve ana süreç tamamen birbirine kilitlenmişti.

Bariyer senkronizasyonu, bütün süreçlerin bir noktaya geldiği zaman diğer süreçlerin beklemesini içerir, sonra yollarına devam ederler. Bariyer, beklemesini istediğimiz süreçlerin toplam sayısıyla oluşturulur. Daha önce olduğu gibi `Arc` aracılığıyla bu bariyeri diğer altsüreçlerle paylaşabilirsiniz.


```rust
// thread7.rs
use std::thread;
use std::sync::Arc;
use std::sync::Barrier;

fn main() {
    let nthreads = 5;
    let mut threads = Vec::new();
    let barrier = Arc::new(Barrier::new(nthreads));

    for i in 0..nthreads {
        let barrier = barrier.clone();
        let t = thread::spawn(move || {
            println!("before wait {}", i);
            barrier.wait();
            println!("after wait {}", i);
        });
        threads.push(t);
    }

    for t in threads {
        t.join().unwrap();
    }
}
// before wait 2
// before wait 0
// before wait 1
// before wait 3
// before wait 4
// after wait 4
// after wait 2
// after wait 3
// after wait 0
// after wait 1
```
Süreçler yine yarı-rastgele çalışırken bir anda birleşiyorlar ve sonra tekrar devam ediyorlar. Bu, devam ettirilebilir bir `join` gibidir ve bütün süreçlerin belli bir işi yaptıktan sonra o işle devam etmesini istediğinizde kullanışlı olabilir. 

## Paylaşılmış Durumlar
Süreçler, kendi paylaşılmış durum bilgisini nasıl *düzenler*?

Aklınıza *dinamik* olarak paylaşılan değişebilir referans almak için kullandığımız  `Rc<RefCell<T>>` stratejisini getirin. `RefCell`in süreçlerde kullanılan muadili ise `Mutex`  - değişken referansı `lock` kullanarak alabilirsiniz. Referans var olduğu müddetçe diğer süreçler veriye erişemeyecektir. `mutex`'in açılımı "Mutual Exclusion" yani "Karşılıklı Hariciyet" - bir sürecin erişmesi için kodun ilgili kısmını kilitliyoruz ve ardından kilidini açıyoruz. `lock` ile kilitlersiniz ve referans düşünce de kilit kalkar.  

```rust
// thread9.rs
use std::thread;
use std::sync::Arc;
use std::sync::Mutex;

fn main() {
    let answer = Arc::new(Mutex::new(42));

    let answer_ref = answer.clone();
    let t = thread::spawn(move || {
        let mut answer = answer_ref.lock().unwrap();
        *answer = 55;
    });

    t.join().unwrap();

    let ar = answer.lock().unwrap();
    assert_eq!(*ar, 55);

}
```
`RefCell` kadar kolay değil çünkü eğer kilidin olduğu bir süreç paniklerse `mutex` daima kilitli kalabilir. (Böyle bir durumda, dokümentasyon açıkça süreci `unwrap` ile terk etmeniz gerektiğini söyler çünkü bir şeyler çok yanlış gitmiştir.)

Bu sefer değişebilir referansları mümkün olduğunca az tutmak çok daha önemli; çünkü bu `mutex` kapalı kaldıkça diğer süreçler de bloklanacaktır. Bu, kilitleyip bilgisayar için zor hesaplamaların yapmanın yeri bu değil! Yani, muhtemelen kodunuz şuna benzeyecek:

```rust
// ... do something in the thread
// get a locked reference and use it briefly!
{
    let mut data = data_ref.lock().unwrap();
    // modify data
}
//... continue with the thread
```
## Yüksek Seviyeli İşlem
Belki de süreçleri yönetmenin daha yüksek seviyeli bir yolunu bulmak, süreçleri tek tek elle kontrol etmekten daha iyidir. Bunun bir örneği hesaplamaları paralel olarak yaptırmak ve sonuçları toplamak olabilir. Epey enteresan bir sandık olarak [pipeliner](https://docs.rs/pipeliner/0.1.1/pipeliner/)[^old] sandığına bakabilirsiniz ki çok anlaşılır bir API'ya sahiptir. Deneysel bir "Merhaba Dünya"ye ne dersiniz? - bize çıktılar veren bir döngüleyici kurgulayalım ve `n` adet işlemi paralel olarak çalıştıralım:
[^old]: Görünüşe göre pipeliner sandığı Şubat 2020'den beri güncelleme almamış

```rust
extern crate pipeliner;
use pipeliner::Pipeline;

fn main() {
    for result in (0..10).with_threads(4).map(|x| x + 1) {
        println!("result: {}", result);
    }
}
// result: 1
// result: 2
// result: 5
// result: 3
// result: 6
// result: 7
// result: 8
// result: 9
// result: 10
// result: 4
```

Salakça bir örnek olduğunun farkındayız, çünkü ilgili operasyon zaten bilgisayar için zahmetsiz ancak paralel işlemler gerçekleştirmenin ne derece kolay olabileceğini göstermiş oldum.

Daha kullanışlı bir şey yapalım. Ağ işlemlerini paralel olarak gerçekleştirmek faydalı olabilir, çünkü genellikle uzun zaman alırlar ve işe başlamak için hepsinin tamamlanmasını beklemeyi istemezsiniz.

Örneğimiz biraz kötü (emin olun yapmanın çok daha iyi yolları var) ancak ne işe odaklanın. 4. Bölümde tanımladığımız `shell` fonksiyonunu belli bir aralıktaki IP4 adreslerine `ping` atmak için tekrar kullanacağız:

```rust
extern crate pipeliner;
use pipeliner::Pipeline;

use std::process::Command;

fn shell(cmd: &str) -> (String,bool) {
    let cmd = format!("{} 2>&1",cmd);
    let output = Command::new("/bin/sh")
        .arg("-c")
        .arg(&cmd)
        .output()
        .expect("no shell?");
    (
        String::from_utf8_lossy(&output.stdout).trim_right().to_string(),
        output.status.success()
    )
}

fn main() {
    let addresses: Vec<_> = (1..40).map(|n| format!("ping -c1 192.168.0.{}",n)).collect();
    let n = addresses.len();

    for result in addresses.with_threads(n).map(|s| shell(&s)) {
        if result.1 {
            println!("got: {}", result.0);
        }
    }

}
```

Kendi ev ağımda sonuç şöyle bir şey:

```
got: PING 192.168.0.1 (192.168.0.1) 56(84) bytes of data.
64 bytes from 192.168.0.1: icmp_seq=1 ttl=64 time=43.2 ms

--- 192.168.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 43.284/43.284/43.284/0.000 ms
got: PING 192.168.0.18 (192.168.0.18) 56(84) bytes of data.
64 bytes from 192.168.0.18: icmp_seq=1 ttl=64 time=0.029 ms

--- 192.168.0.18 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.029/0.029/0.029/0.000 ms
got: PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
64 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=110 ms

--- 192.168.0.3 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 110.008/110.008/110.008/0.000 ms
got: PING 192.168.0.5 (192.168.0.5) 56(84) bytes of data.
64 bytes from 192.168.0.5: icmp_seq=1 ttl=64 time=207 ms
...
```

Aktif adresler hızlıca bize bir saniyenin yarısı kadar bir sürede bize cevap verebiliyor, gerisi de olumsuz cevapları beklemek oluyor. Eğer paralel olarak çalıştırmasaydık, yaklaşık bir dakika boyunca sonuçları beklemek zorunda kalırdık! Ping zamanı gibi çıktıdan analiz ederek bulmayı düşünebilirsiniz, ancak bu sadece Linux üzerinde işe yarabilirdi. `ping` her sistemde vardır ancak çıktı her platform için değişiklik gösterebilir. Eğer Rust ile çalışacak evrensel br ağ API hizmeti tasarlamak isterseniz, ağlar kısmına geçiş yapabiliriz. 

## Adresleri Çözümlemenin Daha İyi Bir Yolu
Eğer *sadece* neyin aktif olduğu bilmek istiyorsanız ve gelişmiş ping istatistikleri ilginizi çekmiyorsa, `std::net::ToSocketAddrs` özelliği sizin için DNS çözümlemesi yapacaktır.

```rust
use std::net::*;

fn main() {
    for res in "google.com:80".to_socket_addrs().expect("bad") {
        println!("got {:?}", res);
    }
}
// got V4(216.58.223.14:80)
// got V6([2c0f:fb50:4002:803::200e]:80)
```

Bu bir döngüleyicidir çünkü genellikle bir alan adına birden çok arayüz bağlıdır, ikisi de Google'un arayüzleridir; birisi IPv4 ve diğeri IPv6 olmak üzere. 

Şimdi `pipeliner` örneğimizi masumca bu metotu yeniden yazmak için kullanabiliriz. Çoğu ağ protokolü hem bir adres hem de bir port kullanır:
```rust
extern crate pipeliner;
use pipeliner::Pipeline;

use std::net::*;

fn main() {
    let addresses: Vec<_> = (1..40).map(|n| format!("192.168.0.{}:0",n)).collect();
    let n = addresses.len();

    for result in addresses.with_threads(n).map(|s| s.to_socket_addrs()) {
        println!("got: {:?}", result);
    }
}
// got: Ok(IntoIter([V4(192.168.0.1:0)]))
// got: Ok(IntoIter([V4(192.168.0.39:0)]))
// got: Ok(IntoIter([V4(192.168.0.2:0)]))
// got: Ok(IntoIter([V4(192.168.0.3:0)]))
// got: Ok(IntoIter([V4(192.168.0.5:0)]))
// ....
```
Bu, `ping` yollamaktan çok daha hızlıdır çünkü sadece bir IP adresinin geçerli olup olmadığını ölçüyoruz, eğer bir gerçek alan adlarının bir listesini kullansaydık DNS araştırması epey vakit alabilirdi ve bu paralleliğin nasıl önemli olabileceğini bize gösteriyor.

İlginç bir şekilde, bu "çalıştır ve unut" mantığında işliyor. Standart kütüphanede bulunan ve `Debug` barındıran her şey, hata ayıklamak için de müthiş keşifler sunar. Dönngüleyici `Result` (hâliyle `Ok`) dönüyor ve bu `Result`, IPv4 veyahut IPv6 varyantları olan bir numalandırma olan `SocketAddr` barındıran bir `IntoIter` içeriyor. Peki neden `IntoIter`? Çünkü bir soketin birden çok adresi olabilir, (hem IPv4 hem de IPv6 adresi olması gibi.) 

```rust
    for result in addresses.with_threads(n)
        .map(|s| s.to_socket_addrs().unwrap().next().unwrap())
    {
        println!("got: {:?}", result);
    }
// got: V4(192.168.0.1:0)
// got: V4(192.168.0.39:0)
// got: V4(192.168.0.3:0)
```
Bu da çalışıyor, ilginç olarak, bizim basit örneğimiz kadar işe yarıyor. İlk `unwrap` `Result`'tan kurtuluyor ve döngüleyiciden ilk çıkan değeri dışarı çıkartıyor. `Result` genellikle bu durumda anlamsız adreslerde tetiklenir. (Mesela portu olmayan adres isimleri gibi.)

## TCP İstemci Sunucusu
Rust, en çok kullanılan ve en yaygın ağ protokolü için gayet makul bir atayüz de sunar; TCP. TCP, hatalara karşı oldukça dayanıklıdır ve ağlarla örülü dünyamızın temel taşıdır - *paketler* onaylanarak gönderilir ve onaylanarak alınır. Bunun tersi olarak UDP ise paketleri hiçbir onay olmadan yollar. Bunun hakkında şöyle garabet bir espri vardır: "Sana UDP hakkında bir fıkra anlatabilirim ama muhtemelen kafan almayacak." (Ağlar hakkındaki espriler komiktir, sizin komikten ne anladığınıza göre değişir tabii.)

Fakat, hata kontrolü ağlarla uğraşırken *çok* önemlidir çünkü her an her şey bir anlığına oluşabilir.

TCP, bir istemci/sunucu modeliyle çalışır; sunucu adresi belli bir *ağ portundan* dinler ve istemci de sunucuya bağlanır. Bağlantı oluşturulduğunda ise istemci ve sunucu bir soket üzerinden haberleşebilir.

`TcpStream::connect`, `SoccetAddr`'a dönüştürülebilecek her şeyi kabul eder, kullandığımız düz karakter dizilerini de.

Rust'ta basit bir TCP istemcisi yazmak oldukça kolaydır - `TcpStream` yapısı hem yazılabilir hem de okunabilirdir. Her zaman olduğu gibi, özellikleri kullanmak için, `Read`, `Write` ve diğer `std::io` özelliklerini kapsamda görünür kılmalıyız.

```rust
// client.rs
use std::net::TcpStream;
use std::io::prelude::*;

fn main() {
    let mut stream = TcpStream::connect("127.0.0.1:8000").expect("connection failed");

    write!(stream,"hello from the client!\n").expect("write failed");
 }
```
Sunucumuz pek karmaşık değil, bir dinleyici kuruyoruz ve bağlantıları bekliyoruz. Eğer bir istemci bağlanırsa, sunucu tarafında `TcpStream` elde ederiz. Bu örnekte sunucuya gelen her şey bir karakter dizisine yazılmış olur.

```rust
// server.rs
use std::net::TcpListener;
use std::io::prelude::*;

fn main() {

    let listener = TcpListener::bind("127.0.0.1:8000").expect("could not start server");

    // accept connections and get a TcpStream
    for connection in listener.incoming() {
        match connection {
            Ok(mut stream) => {
                let mut text = String::new();
                stream.read_to_string(&mut text).expect("read failed");
                println!("got '{}'", text);
            }
            Err(e) => { println!("connection failed {}", e); }
        }
    }
}
```

Port numarasını aşağı yukarı rastgele geçtim, ancak pek çok [port](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers) özel anlamlar barındırır.

İki tarafında protokol üzerinde uzlaştığına dikkat edin. İstemci akışa yazı yazabileceğini düşünür ve sunucu da akıştan yazıyı okuyabileceğini umar. Eğer oyunu aynı kurallarla oynamazlarsa, bir tarafın engellendiği ve hiç gelmeyecek bir cevabı beklediği durumlar oluşur. 

Hataların kontrolü önemlidir - ağ girdi çıktıları çeşitli sebeplerden aksayabilir ve hatalar dolunayda dosya sisteminin kurtadama dönüşmesi gibi acayip sebeplerlerl de geçerkleşebilir. Birileri kablolarla ip atlayabilir, öbür tarafın bilgisayarı çökebilir, falan fistan. Yazdığımız ufak sunucu çok da sağlam değil, çünkü ilk hata çökecektir.

Hataları düzgünce ele alan güçlü bir sunucu aşağıda bulunmaktadır. Akıştan bir satırı bir `io::BufRead` üreten bir `io::BufReader` aracılığıyla okur ve böylece çıktı üzerinde `read_line` çağırabiliriz.

```rust
// server2.rs
use std::net::{TcpListener, TcpStream};
use std::io::prelude::*;
use std::io;

fn handle_connection(stream: TcpStream) -> io::Result<()>{
    let mut rdr = io::BufReader::new(stream);
    let mut text = String::new();
    rdr.read_line(&mut text)?;
    println!("got '{}'", text.trim_right());
    Ok(())
}

fn main() {

    let listener = TcpListener::bind("127.0.0.1:8000").expect("could not start server");

    // accept connections and get a TcpStream
    for connection in listener.incoming() {
        match connection {
            Ok(stream) => {
                if let Err(e) = handle_connection(stream) {
                    println!("error {:?}", e);
                }
            }
            Err(e) => { print!("connection failed {}\n", e); }
        }
    }
}
```

`handle_connection` içindeki `read_line` çökebilir ancak sonuçta hata emniyetli bir şekilde kontrol edilmiş olur.

Bu tarz tek yönlü iletişimler bazen kullanışlı olabilir - bu örnekte olduğu gibi. Ağda bulunan servislerin durumlarını, merkezi bir yerde toplamış oluyoruz. Fakat kibarca bir şekilde geri dönüş yapsak iyi olur, en azından "ok" deyip geçelim.

İşte bir "eko" sunucusu. İstemci, sonunda satır sonu işareti olan bir metni sunucuya yollar ve sunucuda ek bir satır sonu ekleyerek geri yollar - akış okunabilir ve yazılabilirdir. 
```rust
// client_echo.rs
use std::io::prelude::*;
use std::net::TcpStream;

fn main() {
    let mut stream = TcpStream::connect("127.0.0.1:8000").expect("connection failed");
    let msg = "hello from the client!";

    write!(stream,"{}\n", msg).expect("write failed");

    let mut resp = String::new();
    stream.read_to_string(&mut resp).expect("read failed");
    let text = resp.trim_right();
    assert_eq!(msg,text);
}
```
Sunucu şimdi ilginç bir hâl aldı. Sadece `handle_connection`u değiştirelim:

```rust
fn handle_connection(stream: TcpStream) -> io::Result<()>{
    let mut ostream = stream.try_clone()?;
    let mut rdr = io::BufReader::new(stream);
    let mut text = String::new();
    rdr.read_line(&mut text)?;
    ostream.write_all(text.as_bytes())?;
    Ok(())
}
```

Bu yaygın olarak kullanılan yaygın br çift taraflı soket iletişimidir; `BufReader`'a yollamak için bir satır istiyoruz - fakat "BufReader" akışı *tüketiyor*! Bu yüzden akışı klonlamalıyız ve aynı soketi işaret eden yeni bir yapı oluşturmalıyız. Böylece nihayet huzuru elde ediyoruz.