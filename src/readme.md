# Neden yeni bir programlama dili öğrenmelisiniz?
![Rust hakkında bir karikatür](https://stevedonovan.github.io/rust-gentle-intro/PPrustS.png)

[David Marino'ya teşekkürlerle!](https://leftoversalad.com/c/015_programmingpeople/)

>Karikatürün çevirisi:
>
>Rust: Bu gece şöförlük yapacağım.
>
>Dış ses: Biliyoruz.
>
>Rust: Konuşurken yemek yeme çünkü boğazına takılırsa ölürsün. Haha.

Bu rehberin amacı [kutsal kitabımız gibi](https://doc.rust-lang.org/stable/book/) İnternet'te bulabileceğiniz çeşitli kaynakları anlayacak kadar Rust okuryazarlığı aşılamaktır. Rehberi, bu programlama dilinin gücünü tonla şeyin arasına girmeden anlamak ve denemek için bir fırsat olarak düşünebilirsiniz.

Einstein'in dediği gibi, "Her şeyi olabildiğince yumuşak yapın, ama yumuş yumuş olmasın." (Ç.N: Einstein'in "Her şeyi daha sade yapın, ama basit değil" sözüne gönderme.) Rust'a dair öğrenecek epey şey var ve pek çok şey iyice kafanızı bulandırabilir. Burada kastedilen "yumuşaklık", zorluğuna karşın Rust'ın çözümlerinin uygulamalı olarak sunulmasıdır. Sorunları anlamak, hemen çözümleri görmekten çok daha faydalıdır. Bunu kayaçların süreçlerini anlamak için coğrafya dersinden sonra hemen dağ bayır gezintiye çıkmak gibi düşünebilirsiniz. Bunun elbette zorluğu olacak ama neticesi oldukça hoş olacak. Katılanlar oldukça memnun olacak ve birbirlerine yardımcı olmaktan çekinmeyecekler. (Buna benzer olarak) [Rust kullanıcıları forumu (İngilizce)](https://users.rust-lang.org/) ve oldukça aktif bir [subreddit (Bu da İngilizce)](https://www.reddit.com/r/rust/) var. Aynı zamanda bazı sorularınız varsa [sıkça sorular sorular (Herhâlde İngilizce)](https://www.rust-lang.org/en-US/faq.html)'a da bakabilirsiniz.

Ama hepsinden önce, neden durup dururken yeni bir dil öğrenelim ki? Bu öyle ya da böyle epey vakit ve enerji alan, durup dururken yapılmayacak bir iş. Bu iş size çok süper, über bir iş buldurmayacak olsa bile beyin kaslarınızı çalıştıracak ve sizi daha iyi bir programcı yapacaktır. İyi bir yatırım mı? Tartışılır. Yine de *gerçek anlamda* bir şey öğrenmezseniz zaman içerisinde durgunlaşacak ve on yılı aşkın aynı şeylere bakan birisi olacaksınız. 

# Rust'ın Parladığı Nokta
Rust statik ve güçlü tiplenen bir sistem programlama dilidir. *Statik* bütün tiplerin derleme zamanında bilinmesi anlamına gelir, *güçlü* ise programın çalışma mantığının tip mantığının dışına çıkmaması demektir. Başarılı bir derleme aynı zamanda C gibi tekinsiz bir dile göre çok daha fazla şeyi garanti ettiğiniz anlamına gelir. *Sistem (Programlama)*'dan kasıt makine için en uygun kodun sıkı bir bellek denetimi ile oluşturulmasıdır. Kullanım alanları biraz ilginç: işletim sistemleri, donanım sürücüleri ve bir işletim sistemi bile olmayan gömülü sistemler. Ancak Rust normal uygulamaları programlamak için de gayet uygundur. 

C ve C++ ile Rust arasındaki en büyük fark *doğal olarak emniyetli olmasıdır.* Bütün bellek erişimleri denetlendiğinden kazara belleği darmaduman etmeniz mümkün değildir.

Rust'ın ana prensipleri şunlardır:
- Verinin *emniyetli olarak ödünç alınmasını* zorlamak
- Fonksiyon, metot ve kapamalar (closure) ile veriye müdahale etmek
- Verileri çokuzlu (tuple), yapılar (struct) ve numaralandırmalar (enum) ile bir araya toplamak
- Verileri örüntü eşleştirme (pattern matching) ile seçmek ve parçalarına ayırmak
- Verilerin *görevini* özellikler (trait) aracılığı ile tanımlamak 

Cargo sayesinde oldukça hızlı büyüyen bir ekosistem olsa da biz çoğunlukla dilin temel özelliklerini anlamaya ve standart kütüphaneyi kullanmaya eğileceğiz. Tavsiyem *çok sayıda ufak programlar* yazmanızdır, bundan dolayı `rustc` kullanmayı öğrenmek en önemli beceri olacaktır hâliyle. Bu rehberde bulacağınız pek çok örneği çalıştırmak için derleyen ve programı yürüten `rrun` diye minik bir betik hazırladım:
```
rustc $1.rs && ./$1
```

# Kurulum
Bu rehber makinenize Rust'ı kurmayı gerektirir. Neyse ki bunu yapmak [çok basit](https://www.rust-lang.org/en-US/downloads.html).
```
$ curl https://sh.rustup.rs -sSf | sh
$ rustup component add rust-docs
```
Kararlı sürümü kullanmanızı tavsiye ediyorum; sonradan kararsız sürümü kurup aralarında geçiş yapmak da kolaydır.

Bu komut derleyiciyi Cargo paket yöneticisini, API belgelendirmesini ve Rust kitabını indirir. Uzun yolculukların hepsi bir adımla başlar ve bu adımı atmak zahmetsizdir.

`rustup` komutu Rust kurulumunuzu kontrol eden komuttur. Yeni bir kararlı sürüm yayınlandığında yalnızca  `rustup` yazarak güncelleyebilirsiniz. `rustup doc` ise resmi Rust belgelerini çevrimdışı olarak tarayıcınızda açar.

Muhtemelen beğendiğiniz bir editor vardır ve bu editörün [temel Rust desteği](https://areweideyet.com/) varsa o editörü istediğiniz gibi kullanabilirsiniz. Öncelikle sözdiziminin renklendirilmesi ile yetinmenizi öneririm, programlarınız büyüdükçe daha ötesine geçersiniz.

Şahsen varsayılan olarak Rust desteği ile gelen nadir editörlerden birisi olan [Geany'i](https://www.geany.org/download/releases/) beğeniyorum; paket yöneticisiyle kurulabildiği için Linux üzerinde kullanmak gayet kolaydır, diğer platformlarda da pekâlâ kullanılabilir.

Esas nokta Rust programlarını düzenleyebilmek, derleyebilmek ve çalıştırabilmektir. Programlamayı *parmaklarınızla* anlayın, kodu kendiniz yazın ve editörün kodu nasıl düzelteceğini öğrenin.

Zed Shaw'ın programlamayı Python ile öğrenme [tavsiyesi](https://learnpythonthehardway.org/book/intro.html) geçen yıllara rağmen dil fark etmeksizin dikkate değerdir. Ona göre programlamayı öğrenmenin bir müzik enstürmanı öğrenmek gibidir - esas sır pratik ve sabırdır. Taiciçüen gibi yumuşak dövüş sanatlarının ve Yoga'nın bir güzel öğüdü vardır; gerilimi hissedin, ama aşırı gerilmeyin. Karanlık bir spor salonunda sert bir disiplini sadakatle zayıflamaya çalışmıyorsunuz; rahatlayın.

Kötü İngilizcemi veya yetersiz Rust bilgimi yakalayarak beni uyaran destekçilere teşekkürler etmek isterim, aynı zamanda David Marino'ya Rust'ı parlayan zırhı içerisinde sempatik fakat mantığına sıkı sıkıya bağlı bir şovalye olarak çizdiği için de teşekkürlerimi sunarım.

Steve Donovan © 2017-2018 MIT Lisans Versiyonu 0.4.0

Çeviri: Emrecan Şuşter

## Bazı Çeviri Notları

Bu çeviriyi motamot çevirmek yerine, kaldı ki bunu makineler de yapabiliyor, içeriğinde de kültüre özgü değişiklikler yaptım. Maksat rehberin ana dilinde yazılmış gibi bir akıcılıkla okunabilmesi idi. Çoğu yerin birebir çevrilmediğini, bazı şeylerin eklendiğini de göreceksiniz. Bu İngilizce ile Türkçe arasındaki hem kültürel hem de anlamsal farklardan dolayıdır.

Epey yerleşmiş, Türkçesi de (göreceli olarak) oldukça anlamsız gelen bazı kavramları doğrudan İngilizce hâliyle metin içinde kullandım. (Heap ve Stack gibi) Ancak bunun dışında çoğu yerde Türkçe karşılıkları kullanmaya özen gösterdim ve yanların parantez içinde veya "/" işareti ile İngilizce karşılıklarını bildirdim. Terimler için ["Rust Dili" projesinin sözlüğünden](https://github.com/RustDili/dokuman/tree/master/sozluk) faydalandım.

Bunun dışında kod parçalarına dokunmadım, içindeki yorumları bile olduğu gibi bıraktım zira bu kod parçaları değiştiği zaman çıkacak ekstra iş gücünden çekindim.

Çeviriyi geliştirmek için her zaman [çevirinin Github deposuna](https://github.com/Tarbetu/gentle-intro) uğrayabilirsiniz.
