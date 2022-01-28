# Neden yeni bir programlama dili öğrenmelisiniz?
![Rust hakkında bir karikatür](https://stevedonovan.github.io/rust-gentle-intro/PPrustS.png)
[David Marino'ya teşekkürlerle!](https://leftoversalad.com/c/015_programmingpeople/)

>Karikatürün çevirisi:
  Rust: Bu gece şöförlük yapacağım.
>Dış ses: Biliyoruz.
>Rust: Konuşurken yemek yeme çünkü boğazına takılıp ölebilirsin. Haha

Bu rehberin amacı [kutsal kitabımız gibi](https://doc.rust-lang.org/stable/book/) İnternet'te bulabileceğiniz çeşitli kaynakları anlayacak kadar Rust okuryazarlığı aşılamaktır. Rehberi dilin gücünü tonla şeyin arasına girmeden gücünü anlamak ve denemek için köprüden önceki son çıkış olarak düşünebilirsiniz.

Einstein'in dediği gibi, "Her şeyi olabildiğince yumuşak yapın, ama yumuş yumuş olmasın." (Ç.N: Einstein'in "Her şeyi daha sade yapın, ama basit değil" sözüne gönderme.) Burada öğrenecek epey şey var ve iyice kafanızı iyice allak bullak edebilir. Burada kastedilen "yumuşaklık", çeşitli zorluklara karşın Rust'ın çözümlerini uygulamalı olarak sunulmasıdır. Sorunları anlamak, hemen çözümleri görmekten daha faydalıdır. Bunu kayaçların süreçlerini anlatmak için birkaç coğrafya dersinden sonra hemen dağ bayır gezintiye çıkmak gibi düşünebilirsiniz. Elbette biraz zoruklar olacak ama neticesi oldukça hoş olacak. Katılanlar oldukça memnun olacak ve birbirlerine yardımcı olmaktan çekinmeyecekler. (Buna benzer olarak) [Rust kullanıcıları forumu (İngilizce)](https://users.rust-lang.org/) ve oldukça aktif bir [subreddit (Bu da İngilizce)](https://www.reddit.com/r/rust/) var. Aynı zamanda bazı sorularınız varsa [sıkça sorular sorular (Herhâlde İngilizce)](https://www.rust-lang.org/en-US/faq.html)'a da bakabilirsiniz.

Ama hepsinden önce, neden durup dururken yeni bir dil öğrenelim ki? Bu öyle ya da böyle epey vakit ve enerji alan, durup duruken yapılmayacak bir iş. Bu iş size çok süper, über bir iş buldurmayacak olsa bile beyin kaslarınızı çalıştıracak ve sizi daha iyi bir programcı yapacaktır. Bu pekâlâ çok iyi bir yatırım gibi görünmeyebilir ama *gerçek anlamda* bir şey öğrenmezseniz zaman içerisinde durgunlaşacak ve on yılı aşkın aynı şeyleri yapan birisi olacaksınız. 

# Rust'ın Parladığı Nokta
Rust statik ve güçlü tiplenen bir sistem programlama dilidir. *Statik* bütün tiplerin derleme zamanında bilinmesi anlamına gelir, *güçlü* ise bütün tiplerin yanlış bir program yazmayı zorlaştırmasıdır. Başarılı bir derleme aynı zamanda C gibi maceracı bir dile göre çok daha fazla şeyi garanti ettiğiniz anlamına gelir. *Sistem (Programlama)*'dan kasıt makine için en uygun kodun tam bir bellek kontrolü ile oluşturulmasıdır. Kullanım alanları biraz zorlayıcı olabilir: işletim sistemleri, donanım sürücüleri ve bir işletim sistemi bile olmayan gömülü sistemler. Ancak Rust normal uygulamaları programlamak için de gayet uygundur. 

C ve C++ ile Rust arasındaki en büyük fark *doğal olarak güvenli olmasıdır.* Bütün bellek erişimleri kontrol edilir, kazara belleği darmaduman etmeniz mümkün değildir.

Rust'ın arkasındaki prensipler şunlardır:
- Verinin *güvenli olarak ödünç alınmasını* zorlamak
- Fonksiyon, metot ve kapamalar (closure) ile veriye müdahale etmek
- Verileri çokuzlu (tuple), yapılar (struct) ve numaralandırmalar (enum) ile bir araya toplamak
- Verileri örüntü eşleştirme (pattern matching) ile seçmek ve parçalarına ayırmak
- Verilerin işlevini özellikler (trait) aracılığı ile sağlamak

Cargo vasıtasıyla oldukça hızlı büyüyen bir ekosistem olsa da biz çoğunlukla dilin temel özelliklerini anlamaya ve standart kütüphaneyi kullanmaya eğileceğiz. Tavsiyem, *çok sayıda ufak programlar* yazmanızdır, bundan dolayı `rustc` kullanabilmek en temel gerekliliktir. Bu rehberde bulacağınız pek çok örneği çalıştırmak derlemeyi yapıp çalıştıran `rrun` diye minik bir betik hazırladım:
```
rustc $1.rs && ./$1
```

# Kurulum
Bu rehber makinenize Rust'ı kurduğunuzu varsayar. Neyse ki bunu yapmak [çok basit](https://www.rust-lang.org/en-US/downloads.html).
```
$ curl https://sh.rustup.rs -sSf | sh
$ rustup component add rust-docs
```
Kararlı sürümü kullanmanızı tavsiye ediyorum; sonradan kararsız sürümü kurup aralarında geçiş yapmak da kolaydır.

Bu komut derleyiciy, Cargo paket yöneticisini, API belgelendirmesini ve Rust kitabını indirir. Çok uzun yolculukların hepsi bir adımla başlar ve bu adımı atmak zahmetsizdir.

`rustup` komutu Rust kurulumunuzu kontrol eden komuttur. Yeni bir kararlı sürüm yayınlandığında yalnızca  `rustup` yazarak güncelleyebilirsiniz. `rustup doc` ise belgelendirmeyi çevrimdışı olarak tarayıcınızda açar.

Muhtemelen beğendiğiniz bir editor vardır ve [temel Rust desteği](https://areweideyet.com/) yeterlidir. Öncelikle temel sözdizimi renklendirmesi ile başlamanızı öneririm, programlarınız büyüdükçe daha ötesine geçersiniz.

Şahsen varsayılan olarak Rust desteği ile gelen nadir editörlerden birisi olan [Geany'i](https://www.geany.org/download/releases/) beğeniyorum; paket yöneticisiyle kurulabildiği için Linux üzerinde çalıştırmak gayet kolaydır, diğer platformlarda da pekâlâ kullanılabilir.

Esas nokta Rust programlarını düzenleyebilmek, derleyebilmek ve çalıştırabilmektir. Programlamayı *parmaklarınızla* anlayın, kodu kendiniz yazın ve editörün kodu nasıl düzelteceğini öğrenin.

Zed Shaw'ın programlamayı Python ile öğrenme [tavsiyesi](https://learnpythonthehardway.org/book/intro.html) hâlen daha kabul edilebilir, dil fark etmeksizin. O, programlamayı öğrenmenin bir müzik enstürmanı öğrenmek gibidir diyor -  esas sır pratik ve sırdır. Taiciçüen gibi yumuşak dövüş sanatlarının ve Yoga'nın bir güzel öğüdü vardır; gerilimi hissedin, ama aşırı gerilmeyin. Burada gerzekçe bir şekilde kas yapmaya çalışmıyoruz. 

Kötü İngilizcemi veya yetersiz Rust bilgimi yakalayarak beni uyaran destekçilere teşekkürler etmek isterim, aynı zamanda David Marino'ya Rust'ı parlayan zırhı içerisinde sempatik ama sağlam bir yersiz şovalye olarak çizdiği için de teşekkürlerimi sunarım.

Steve Donovan © 2017-2018 MIT Lisans Versiyonu 0.4.0

Çeviri: Emrecan Şuşter

## Bazı Çeviri Notları

Bu çeviriyi motamot çevirmek yerine, kaldı ki bunu makineler de yapabiliyor, içeriğinde de kültüre özgü değişiklikler yaptım. Maksat rehberin ana dilinde yazılmış gibi bir akıcılıkla okunabilmesi idi. Çoğu yerin birebir çevrilmediğini, bazı şeylerin eklendiğini de göreceksiniz. Bu İngilizce ile Türkçe arasındaki hem kültürel hem de anlamsal farklardan dolayıdır.

Epey yerleşmiş, Türkçesi de (göreceli olarak) oldukça anlamsız gelen bazı kavramları doğrudan İngilizce hâliyle metin içinde kullandım. (Heap ve Stack gibi) Ancak bunun dışında çoğu yerde Türkçe karşılıkları kullanmaya özen gösterdim ve yanların parantez içinde veya "/" işareti ile İngilizce karşılıklarını bildirdim. Terimler için ["Rust Dili" projesinin sözlüğünden](https://github.com/RustDili/dokuman/tree/master/sozluk) faydalandım.

Bunun dışında kod parçalarına dokunmadım, içindeki yorumları bile olduğu gibi bıraktım zira bu kod parçaları değiştiği zaman çıkacak ekstra iş gücünden çekindim.

Çeviriyi geliştirmek için her zaman [çevirinin Github deposuna](https://github.com/Tarbetu/gentle-intro) uğrayabilirsiniz.