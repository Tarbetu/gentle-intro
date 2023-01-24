# Standart Kütüphane Konteynırları

# Belgeleri Anlamak
Bu kısımda kabaca size Rust'ın standart kütüphanesinin bilindik bazı kısımlarını tanıtacağım. Belgelendirme gayet iyi ancak bağlamı tanıtmak ve biraz örneğin kimseye zararı olmaz.

Hepsinden önce Rust belgelerini okumak biraz yorucu gelebilir, bundan dolayı bir örneği inceleyeceğiz ki bu örnek `Vec` olacak. Kullanışlı bir tavsiye verelim, "\[-\]" belgeleri açıp kapamaya yarar. (Eğer `rustup component add rust-src` ile belgeleri indirmişseniz yanında bir de "\[src\]" bağlantısını göreceksiniz. Metotların bir krokisine buradan ulaşabilirsiniz. 

Dikkat etmeniz gereken ilk detay, bütün ilişkili metotların `Vec`'in kendisinde tanımlanmadığıdır. Bunlar (çoğunlukla) `push` gibi vektörü değiştiren metotlardır. Bazı metotlar ise sadece vektörlerin içinde tuttuğu tiplere göre değişkenlik gösterir. Mesela, `dedup`'ı (kopyaları kaldır) sadece eşitliği denetlenebilir tipler üzerinde çalışır. `Vec` tipinde kullanılan birden fazla `impl` bloğu vardır ki bunlar içinde bulunduğu tiplerin çeşitliliğine göre şekillenmiştir.

`Vec<T>` ile `&[T]` arasında da özel bir ilişki olduğunu biliyoruz. Dilimler üzerinde çlışan her bir metot vektörler üzerinde doğrudan çalışacaktır, fazladan `as_slice` gibi metotlar kullanmanıza hiç gerek yoktur. Bu ilişki `Deref<Target=[T]>` ile gösterilir. Ayrıca bir vektörü referans olarak göstermek onu bir dilime çevirir - tip dönüşümlerinin nadiren gerçekleştiği nadir yerlerden birisidir. İlk öğeyi geri dönen `first` gibi dilim metotları, ya da bunun tersini yapan `last`, vektörler için de kullanılabilir. Metotların pek ciddi bir kısmı karakter dizilerini çağrıştırabilir, mesela `split_at` dilimi belirli bir indekse göre ayırır, `starts_with` bir vektörün belirli bir veri silsilesi ile başlayıp başlamadığını belirtir, `contains` bir vektörün belirli bir veriyi içerip içermediğini belirtir.

Belirli bir verinin indeksini bulmak için Rust'ta `search` metotu yoktur. Şimdi size size esas olayı anlatayım; eğer konteynırda metotu bulamazsanız, döngüleyici metotlarına bakın:

```rust
    let v = vec![10,20,30,40,50];
    assert_eq!(v.iter().position(|&i| i == 30).unwrap(), 2);
```

( `&` kullanmamızın sebebi döngüleyicinin referanslar üzerinde çalışmasıdır - alternatif olarak kıyaslamak için `*i == 30` kullanabilirsiniz.)

Benzer şekilde vektörler üzerinde `map` metotu yoktur çünkü `iter().map(...).collect` ile aynı pekâlâ işi yapabilirsiniz. Rust, gerekmedikçe bellek tahsis etmeyi sevmez - çoğu zaman hâlihazırda bellekte yer tutan `map`'ın bütün sonuçlarına ihtiyacınız olmaz. 

Döngüleyici (`iterator`) metotlarına aşina olmanızı tavsiye ederim çünkü iç içe girmiş döngülerle boğuşmadığınız iyi bir Rust kodu yazmak için elzemdirler. Her zaman olduğu gibi, büyük bir program yazarken bir anda onlarla güreşmek yerine döngüleyici metotlarını keşfetmek için minik programlar yazın.

`Vec<T>` ve `&[T]` metotları birbirleriyle ortak özellikleri (trait) paylaşırlar: vektörler kendi hata ayıklama bilgilerinin nasıl gösteirlebilirler. (Eğer bütün öğeler `Debug` özelliğine sahipse.) Aynı şekilde, eğer bütün öğeleri klonlanabilirlerse kendileri de klonlanabilirler. `Drop` özelliğine sahiptirler, bir vektör düşürüldüğü zaman bellekteki yerleri boşaltılır ve tek tek bütün öğeleri de düşürülür.

`Extend` özelliği döngüleyicilerdeki değerlerin bir döngü içerisine herhangi bir döngü kurmadan eklenebileceğini ifade eder.

```rust
v.extend([60,70,80].iter());
let mut strings = vec!["hello".to_string(), "dolly".to_string()];
strings.extend(["you","are","fine"].iter().map(|s| s.to_string()));
```

Aynı zamanda `FromIterator` özelliği de vektörlerin döngüleyicilerden *inşa edilebileceğini* ifade eder. (Döngülerin `collect` metotu bunu kullanır.)

Her konteynır döngülenebilir olmalıdır. [Üç tarz-ı döngüleyiciyi](#) hatırlayın.

```rust
for x in v {...} // returns T, consumes v
for x in &v {...} // returns &T
for x in &mut v {...} // returns &mut T
```

`for` deyimi `IntoIterator` üzerinde iş yapar ve buna bağlı olarak üç farklı kullanımı vardır.

Bir de `Index` (Bir vektörden okurken çalışan) bir de `IndexMut` (Bir vektörü düzenlerken çalışan) ile kontrol edilen indekslememiz vardır. Pek çok şey yapabiliriz çünkü `v[0..2]` gibi ifadelerle dilimlere indeksleyebilir ve dönebiliriz ya da sadece `v[0]` ile ilk elemana referans alabiliriz.

`From` özelliğinin de birtakım kullanımları vardır. Mesela `Vec::from("hello".to_string())` size karakter dizelerinin özündeki `Vec<u8>` tipindeki vektörü verecektir. Ancak şunu düşünebilirsiniz, zaten `String` tipi için `into_bytes` diye bir vektör varken bunun ne özelliği var? Bir işi yapmanın birden çok yolu olması saçma değil mi? Ancak bu, özelliklerin (traits) genellenen metotlar oluşturması için gerekliliktir. 

Bazen Rust'ın tip sisteminin kısıtlamalarından illallah edebilirsiniz. Mesela `PartialEq` boyutu 32'den az olan diziler için *ayrıca* tanımlanmıştır. (Bunu iyileştirecekler.) Bu vektörlerle dizileri doğrudan rahatça kıyaslamanızı sağlar ancak boyut sınırına dikkat etmelisiniz. 

Belgelendirmenin diplerinde bazı [gizli hazinelerle](https://www.youtube.com/watch?v=j6K5IblbBzE) karşılaşabilirsiniz. Tıpkı Karol Kuczmarski'nin dediği gibi; "Kimse bu kadar arayıp taramaz.". Bir döngüleyicideki hataları nasıl yönetmelisiniz? Mesela bir döngüleyici üzerinde `map` kullandığınızda bazı öğeler sorun çıkarabilir ve size `Result` dönebilirler, böyle bir döngüleyici ile çalışacağınızı düşünün:

```rust
fn main() {
    let nums = ["5","52","65"];
    let iter = nums.iter().map(|s| s.parse::<i32>());
    let converted: Vec<_> = iter.collect();
    println!("{:?}",converted);
}
//[Ok(5), Ok(52), Ok(65)]
```

Yeterince iyi, ama tek tek bütün hataları kontrol etmeniz gerekiyor - dikkatlice! Ancak Rust bu işin doğrusunu yapar, eğer vektörün `Result` içerisinde barındırılmasını isterseniz - hepsi bu, eğer bir hata varsa bütün vektörü hatalı kabul edebiliriz.

```rust
    let converted: Result<Vec<_>,_> = iter.collect();
//Ok([5, 52, 65])
```

Ya dönüşüm başarısız olursa? İlk hatada işi fazla uzatmadan hemen `Err` döner. `collect`'in nasıl da esnek olduğuna dair iyi bir örnek olduğunu düşünebiliriz. (Tip bildirimini tuhaf bulabilirsiniz. `Vec<_>` kabaca bu bir vektör, `Result<Vec<_>,_>` herhangi bir vektörün `Result` tipi demektir. Siz ne istediğini belirttikten sonra Rust sizin yerinize işi çözer.)

Belgelendirmede *epeyce* detay var ancak ne olursa olsun C++'ın `std::vector` hakkındaki bilgilendirmesinden çok daha anlaşılır ve net.

> Öğelerin gerektiği gereksinimler konteynırın üzerinde yapılan işlemlere dayanır. Çoğunlukla elemanın tipinin karşılanması ve düşürülebilir olması (drop) yeterlidir ancak bazı fonksiyonların katı gereksinimleri vardır.

C++'da kendi başınızın çaresine bakmanız gerekir. Rust'ın ilk başta her şeyi aleni olarak beklemesi sizi ürkütebilir ancak kısıtlamaları anlarken herhangi bir `Vec` metotunun gereksinimlerini de anlayacaksınız. 

Kaynak kodlarını `rustup component add rust-src` ile okumanızı tavsiye ederim, standart kütüphanenin kodları oldukça okunaklıdır ve metotların içeriği tanımlarından çok daha anlaşılırdır.

# Sözlükler (Maps)

*Sözlükler (HashMap)* dilediğiniz veriye bir *anahtar* ile ulaşabilmenizi sağlar. Aman aman bir fikir değil ve dilerseniz aynı şeyi demet dizisi ile yapabilirsiniz:

```rust
    let entries = [("one","eins"),("two","zwei"),("three","drei")];

    if let Some(val) = entries.iter().find(|t| t.0 == "two") {
        assert_eq!(val.1,"zwei");
    }
```

Küçük sözlükler ve sadece anahtar denkliği gerektiren durumlar için üstteki örnek iş görür, ancak içerisinde bir şey aramanın süresi doğru orantıya tabidir - sözlüğün büyüklüğü ile doğru orantılı. 

Pek çok *anahtar/veri çifti* gerektiği zaman bir `HashMap` ile çalışmak çok çok daha verimlidir.

```rust
use std::collections::HashMap;

let mut map = HashMap::new();
map.insert("one","eins");
map.insert("two","zwei");
map.insert("three","drei");

assert_eq! (map.contains_key("two"), true);
assert_eq! (map.get("two"), Some(&"zwei"));
```

`&"zwei"` mı? `get` ile verinin kendisini değil de *referansını* döndüğü için böyle bir şey görüyoruz. Eğer verinin tipi `&str` ise pekâlâ `&&str` alabiliriz. Alacağımız verinin *referans* olması gerekir çünkü çoğu zaman sahipli tiplerin değerlerini *taşımak* istemeyiz.

`get_mut` tıpkı `get` gibi çalışır ancak değişebilir bir referans döner. Şimdi karakter dizilerini sayılara çeviren bir sözlüğü inceleyelim ve "two" değerini güncellemeye çalışalım.

```rust
let mut map = HashMap::new();
map.insert("one",1);
map.insert("two",2);
map.insert("three",3);

println!("before {}", map.get("two").unwrap());

{
    let mut mref = map.get_mut("two").unwrap();
    *mref = 20;
}

println!("after {}", map.get("two").unwrap());
// before 2
// after 20
```

Referansı farklı bir bloğa aldığımıza dikkat edin - aksi taktirde sonuna kadar değişebilir bir referansımız olurdu ve Rust `map`, `map.get("two")` ile hiçbir şeyi ödünç almamıza izin vermezdi; değişebilir bir referans varken değişmez referanslara izin verilmez. (Eğer izin verilseydi, değişmez referansların geçerliliği şaibeli olurdu.) Bundan dolayı değişebilir referansı erkenden aradan çıkararak işi çözmüş oluyoruz.

Elbette bunun çok zarif bir API olduğunu söyleyemeyiz ama hatalara karşı daha dikkatli davranırız. Python olsa ters bir durumda hemen ekrana hata mesajları dizer ve C++ ise bize varsayılan veri dönerdi. (Aslında güzel bir çözüm ancak bazı sorunları var. Mesela `a_map["two"]` 0 döndüğü zaman "bulunamadı" mesajı ile gerçek sıfırın arasındaki farkı anlayamayız. *Üstüne de* fazladan bir girdi atanmış olur.)

Kimse `unwrap` kullanmaz, örneklerde öyle değil tabii. Gördüğünüz çoğu Rust kodu da bağımsız örneklerden oluştuğu için yaygın olarak kullanıldığı kanısına kapılabilirsiniz. Ancak çoğu zaman bir eşleşmenin kullanılması daha olasıdır:

```rust
if let Some(v) = map.get("two") {
    let res = v + 1;
    assert_eq!(res, 3);
}
...
match map.get_mut("two") {
    Some(mref) => *mref = 20,
    None => panic!("_now_ we can panic!")
}
```

Dilenirse anahtar/veri ikilileri üzerinde döngü kurabilirsiniz ancak belli bir sırası yoktur.

```rust
for (k,v) in map.iter() {
    println!("key {} value {}", k,v);
}
// key one value eins
// key three value drei
// key two value zwei
```

Ek olarak `keys` ve `values`'un döngüleyici dönen metotları vardır ki bu değerlerden vektör kullanmayı epeyce kolaylaştırır.

# Örnek: Kelimeleri saymak
Metinleri anlamak için yapabileceğiniz keyifli işlerden birisi bir metinde kaç farklı kelime olduğunu sayabilmektir. Bir metni kelimelere bölmek `split_whitespace`  ile oldukça kolaydır ancak noktalama işaretlerine özen göstermemiz gerekir. Bundan dolayı kelimeler sadece alfabetik karakterden oluşacak şekilde bölünmelidir. Üstelik kelimeler işleme tamamen küçük harfli olarak alınmalıdır.

Bir sözlükte içeriği değiştirecek tarzdan bir şey aramak kolaydır ancak arama başarısız olduğu zaman ne yapacağını belirtmek biraz tuhaf kaçabilir. Neyse ki hata koşulunu kontrol etmek için gayet zarif bir çözümümüz var:

```rust
let mut map = HashMap::new();

for s in text.split(|c: char| !c.is_alphabetic()) {
    let word = s.to_lowercase();
    let mut count = map.entry(word).or_insert(0);
    *count += 1;
}
```

Eğer aradığımız kelime sözlükte yoksa sözlüğe sıfır içeren yeni bir girdi yaratıyoruz ve onu sözlüğe sokuyoruz (*insert*). C++'daki sözlükler de aynen böyle çalışır tek fark burada varsayılan veri kendiliğinden gelmez ve net bir şekilde belirtilir. 


Bu kapamada (*closure*) net bir tip belirttik ve tip de `char` oluyor. Bunun nedeni `split` tarafından kullanılan karakter dizilerinin `Pattern` özelliğinin tuhaflığıdır. Ancak Rust burada sözlüğün anahtar tipinin `String`, sözlüğün veri tipinin de `i32` olduğunu çıkarabilir. 

Gutenberg projesinden [Sherlock Holmes'un maceraları'nı (The Adventures of Sherlock Holmes)](http://www.gutenberg.org/cache/epub/1661/pg1661.txt) kullanarak bunu güzelce test edebiliriz. (`map.len()` ile) Öğreniyoruz ki birbirinden farklı toplam 8071 kelime kullanılmış. 

Peki ya en çok kullanılan yirmi kelimeyi nasıl öğrenebiliriz? Öncelikle sözlüğümüzü bir (anahtar, veri) formatında demetlerle dolu bir vektöre çevirebiliriz. (Bu `map`ı yok edecektir, çünkü `into_iter` kullandık)

```rust
let mut entries: Vec<_> = map.into_iter().collect();
```

Sonra bunları azalacak şekilde dizelim. `sort_by`, `cmp` metotunun sonuçlarını bekleyecektir ki bu metot sayı tiplerinde bulunur. 

```rust
    entries.sort_by(|a,b| b.1.cmp(&a.1));
```

Ve bu sayıları ilk yirmi çıktıyı ekrana yazdıralım:

```rust
    for e in entries.iter().take(20) {
        println!("{} {}", e.0, e.1);
    }
```

(Sadece `0..20` üzerinde bir döngü *kurabilirdiniz* - bu kabul edilebilir ancak Rust'ın kendisine özgü tarzının dışına çıkmış olurduk - üstelik büyük döngüler için daha maliyetli olurdu.)

```
 38765
the 5810
and 3088
i 3038
to 2823
of 2778
a 2701
in 1823
that 1767
it 1749
you 1572
he 1486
was 1411
his 1159
is 1150
my 1007
have 929
with 877
as 863
had 830
```

Listenin başında bir tuhaflık sezdiniz mi? O aslında boş bir kelime. `split` metotu tek karaktere göre parçaladığı için iki noktalama işaretinin arasındaki boşlukklar da kelimeden sayılmış oldu.

# Kümeler (Sets/HashSets)

Kümeleri sadece anahtarlarını umursadığınız sözlükler olarak düşünebilirsiniz, anahtarların karşılığı yoktur. Bundan dolayı `insert` sadece tek bir veri alır ve dilerseniz `contains` kullabilirsiniz. 

Ç.N: Teknik olarak doğru olsa da buradaki tanımı karmaşık buldum. Kümeleri basitçe her verisi özgün olan, aynı veriyi ikinci kez almayan sırasız bir vektör gibi düşünebilirsiniz.

Diğer konteynırlar gibi bir döngüleyiciden `HashSet` oluşturabilirsiniz. `collect` ile bu işi yapabilirsiniz, tipi bildirdiğiniz sürece.

```rust
// set1.rs
use std::collections::HashSet;

fn make_set(words: &str) -> HashSet<&str> {
    words.split_whitespace().collect()
}

fn main() {
    let fruit = make_set("apple orange pear orange");

    println!("{:?}", fruit);
}
// {"orange", "pear", "apple"}
```

Aynı anahtarın tekrar girmiş olmanız (beklenildiği gibi) hiçbir etki oluşturmaz ve bir verideki sıralaması önemli değildir. 

Matematikteki setlerle yaptığınız işlemleri pekâlâ Rust ile de yapabilirsiniz:

```rust
let fruit = make_set("apple orange pear");
let colours = make_set("brown purple orange yellow");

for c in fruit.intersection(&colours) {
    println!("{:?}",c);
}
// "orange"
```

Bütün işlemler döngüleyici döner ve `collect` kullanarak onları tekrardan sete çevirebilirsiniz.

İşte bir kısayol, vektörleri nasıl kullanıyorsak aynı şekilde kullanabiliriz.

```rust
use std::hash::Hash;

trait ToSet<T> {
    fn to_set(self) -> HashSet<T>;
}

impl <T,I> ToSet<T> for I
where T: Eq + Hash, I: Iterator<Item=T> {

    fn to_set(self) -> HashSet<T> {
       self.collect()
    }
}

...

let intersect = fruit.intersection(&colours).to_set();
```

Bütün Rust jeneriklerinde olduğu gibi burada da tipleri özelliklerle kısıtlamanız gereklidir - yukarıdaki kod sadece eşitliği (`Eq`) ve "hash fonksiyonu" (`Hash`) bulunan tipler için çalışır. `Iterator` diye bir tip bulunmadığını ve `I`'nın `Iterator` özelliğine sahip bir tip olması gerektiğini belirtiyoruz.

Standart kütüphane tiplerinine kendi metotlarımızı eklemek gözünüze biraz abartılı görünebilir ancak unutmayın ki kurallar var. Bunu sadece kendi özelliklerimize (*trait*) uygulayabiliriz. Eğer özelliğin ve yapının (*struct*) ikisi de aynı sandıktan geliyorsa (Mesela ki standart kütüphaneyi sunan "stdlib") bu tarz bir kullanıma izin verilmeyecektir. Bu şekilde bir dikkat dağınıklığından kurtulabiliyoruz.

Kendimizi bu zekice ve uygun kısayolu bulduğumuz için övmeye başlamadan önce yaratabileceği sonuçlara dikkat etmelisiniz. Eğer `make_set` aşağıdaki gibi kullanılırsa, ki burada sahipli bir tip olan `String`'in kümesi vardır, `intersect`'in tipi sizi epeyce bir şaşırtabilir:

```rust
fn make_set(words: &str) -> HashSet<String> {
    words.split_whitespace().map(|s| s.to_string()).collect()
}
...
// intersect is HashSet<&String>!
let intersect = fruit.intersection(&colours).to_set();
```

Rust sahipli karakter dizilerinin kopyalarını oluşturmadığı için aksi olamaz. `intersect`'in içerisinde `fruit`ten ödünç alınmış tek bir `&String` bulunmakta. Bunun daha sonra size zorluk çıkaracağına yemin edebilirim, mesela ki yaşam sürelerini belirtmeye başalrken. Daha iyi bir çözüm, döngüleyicinin `cloned` metotunu kullanarak kesişim için kendi sahipli tiplerinizi üretmenizdir. 

```rust
// intersect is HashSet<String> - much better
let intersect = fruit.intersection(&colours).cloned().to_set();
```

`to_set`'in daha iyi bir tanımı, `self.cloned().collect()` ile hazırlanabilir ki bir de bunu böyle denemenizi tavsiye ediyorum.

# Örnek: İnteraktif Olarak Komut İşleme

Bir programın interaktif bir oturumu olması oldukça kullanışlı olabilir. Her bir satır kendi başına işleme alınır ve içindeki kelimelere bölünür; komut ilk bölümde yer alır ve geri kalan kelimeler ise komutun argümanları olur.

Bunun en akla yatan çözümlerinden birisi komut isimlerinden kapamalara (closure) ulaşılabilen bir sözlük inşa etmek olur. Peki ya nasıl kapamaları bir yerde barındıracağız? Hepsinin farklı boyutları olduğunu düşününce kulağa daha zor geliyor. En uygun çözüm, onların kopyalarını `heap`'a kutulamaktır (box):

Hadi deneyelim:

```rust
    let mut v = Vec::new();
    v.push(Box::new(|x| x * x));
    v.push(Box::new(|x| x / 2.0));

    for f in v.iter() {
        let res = f(1.0);
        println!("res {}", res);
    }
```

İkinci `push` kullanımında çok net bir hata alacağız:

```
  = note: expected type `[closure@closure4.rs:4:21: 4:28]`
  = note:    found type `[closure@closure4.rs:5:21: 5:28]`
note: no two closures, even if identical, have the same type
```

Ç.N: Aynı görünseler bile iki kapama asla aynı tipte olmayacaktır. 

`rustc` gereğinden fazla spesifik bir tip çıkarımında bulundu, bundan dolayı vektörün içindeki tipi kendimiz *kutulanmış özellik tipi (boxed trait type)* olarak belirtmeliyiz:

```rust
    let mut v: Vec<Box<Fn(f64)->f64>> = Vec::new();
```

Şimdi kutulanmış kapamaları `HashMap` (sözlük) tipi için de kullanabiliriz. Kapamalar bulundukları ortamlardan veri çekebildikleri için yaşam sürelerini takip etmeliyiz.

`FnMut`u kullanmayı düşünebilirsiniz - çünkü yakaladıkları her türlü değişkenleri düzenleyebilirler. Ancak bir kapamaya tekabül eden birden fazla komutumuz bulunacağı için tekrar tekrar değişebilir referanslar alamazsınız.

Böylece kapamalar argümanlara *değişebilir referanslar* olarak erişir, karakter dizilerinin dilimleri de (`&[&str]`) satırdaki argümanları alır. Tasarladığımız yapıda geri dönüşü `Result` ile paketleyeceğiz - hata olarak en önce `String` kullanacağız.

`D` boyutu belli olan herhangi bir tipi gösterir.

```rust
type CliResult = Result<String,String>;

struct Cli<'a,D> {
    data: D,
    callbacks: HashMap<String, Box<Fn(&mut D,&[&str])->CliResult + 'a>>
}

impl<'a,D: Sized> Cli<'a,D> {
    fn new(data: D) -> Cli<'a,D> {
        Cli{data: data, callbacks: HashMap::new()}
    }

    fn cmd<F>(&mut self, name: &str, callback: F)
    where F: Fn(&mut D, &[&str])->CliResult + 'a {
        self.callbacks.insert(name.to_string(),Box::new(callback));
    }
```

`cmd` imzaya göre bir isim ve bir kapama alır, kapama kutulanmış ve sözlüğe girmiş olmalıdır. `Fn` ise çevreden verileri ödünç alabilir ancak düzenleyemez demektir. Bu tarz genelleme metotları en kötüsüdür, imzasına bakarken kafanız karışık ancak içeriği pirüpak anlaşılırdır! Yaşam ömrünü belirtmeyi unutmak burada en sık yapılan hatalardandır - Rust, çevresine kısıtlanmış kapamaların yaşam ömürlerini unutmanızı hoş görmeyecektir!

Şimdi komutları inceleyelim ve çalıştıralım:

```rust
    fn process(&mut self,line: &str) -> CliResult {
        let parts: Vec<_> = line.split_whitespace().collect();
        if parts.len() == 0 {
            return Ok("".to_string());
        }
        match self.callbacks.get(parts[0]) {
            Some(callback) => callback(&mut self.data,&parts[1..]),
            None => Err("no such command".to_string())
        }
    }

    fn go(&mut self) {
        let mut buff = String::new();
        while io::stdin().read_line(&mut buff).expect("error") > 0 {
            {
                let line = buff.trim_left();
                let res = self.process(line);
                println!("{:?}", res);

            }
            buff.clear();
        }
    }
```

Gayet anlaşılır - satırları kelimelere ayırıp bir vektörde topluyoruz, ardından sözlükte ilk kelimeyi aratıyoruz ve sözlüğün döndüğü kapamayı değişebilir verilerimizle ve kelimenin geri kalanlarıyla çağırıyoruz. Boş satırlar görmezden gelinir ve hata olarak değerlendirilmez.

Şimdi kapamalarımızın olumlu ve olumsuz sonuçlar dönmesini kolaylaştırmak için yardımcı fonksiyonlar tanımlayalım. Burada zekice *ufak* bir detay var; 
tanımladığımız genellenen fonksiyonların çalıştığı tipleri "`String`"e çevirebilir. 

 ```rust
fn ok<T: ToString>(s: T) -> CliResult {
    Ok(s.to_string())
}

fn err<T: ToString>(s: T) -> CliResult {
    Err(s.to_string())
}
```

İşte karşımızda ana programımız var. "`ok(answer)`"ın nasıl çalıştığına dikkat edin - çünkü sayılar kendilerini nasıl karakter dizilerine çevrileceğini iyi bilirler!

```rust
use std::error::Error;

fn main() {
    println!("Welcome to the Interactive Prompt! ");

    struct Data {
        answer: i32
    }

    let mut cli = Cli::new(Data{answer: 42});

    cli.cmd("go",|data,args| {
        if args.len() == 0 { return err("need 1 argument"); }
        data.answer = match args[0].parse::<i32>() {
            Ok(n) => n,
            Err(e) => return err(e.description())
        };
        println!("got {:?}", args);
        ok(data.answer)
    });

    cli.cmd("show",|data,_| {
        ok(data.answer)
    });

    cli.go();
}
```

Hataları biraz uyduruk bir yoldan ele aldık ve bu tarz durumlarda soru işareti operatörünün nasıl çalıştığının inceleyeceğiz. Basitçe `std::num::ParseIntError` hatası `std::errror::Errır` özelliğini (*trait*) içeriyor ki bu bulunduğumuz bloğa `description` metotunu getiriyor - Rust özellikler erişilebilir olmadan üzerinde işlem yapmamıza izin vermez.

Ve çalıştıralım:

```
Welcome to the Interactive Prompt!
go 32
got ["32"]
Ok("32")
show
Ok("32")
goop one two three
Err("no such command")
go 42 one two three
got ["42", "one", "two", "three"]
Ok("42")
go boo!
Err("invalid digit found in string")
```

Denemek isteyeceğiniz pek çok iyileştirme olabilir. Mesela `cmd` komutuna yardım satırını içeren üçüncü bir argüman ekleyebilir, `help` komutuna bu üçüncü argümanla cevap verebilirdik. Ya da Cargo ile [rustyline](https://crates.io/crates/rustyline) sandığını kullanarak komut düzenleme ve geçmişe dönmek konularını daha akılcı bir yoldan halledebiliriz.