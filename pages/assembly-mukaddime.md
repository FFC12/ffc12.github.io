# x86 Assembly Yazılarının Mukaddimesi 
İlk Yayınlanma Tarihi:​​ 13 Şubat 2021​  
Son Güncellenme Tarihi: 3 Şubat 2022


x86 mimarisine ait bir Assembly yazı serisi hazırlamaya karar verdim.Her yazıda x86 mimarisinin assembly diline dair birçok farklı konuyu ele alacağım (çok da detaylı değil) yazılara başlamadan evvel bir mukaddime yazısı yazmayı uygun gördüm. Bu mukaddime yazısında hızlı bir giriş yaparak genel bir bakış kazandırmaya çalışacağım.Hem de bu yazı serisinden istifade etmek isteyenlerin ön-araştırma yapmalarına yardımcı olacak kapsamda da bahislere yer vereceğim.Ayrıca bu yazı serisinden azami ölçüde istifade etmek isteyenlere bazı tavsiyelerim de şu şekilde olabilir:

- Aynı anda kapsamlı araştırmalar yaparak, paralel olarak takip edin.
- Kavramları etimolojik olarak da inceleyin.
- Not alın.
- Okuduğunuz iki yazı arasında uzun bir müddet varsa aldığınızı varsaydığım notlarınızı tekrar edin.

Başlamadan önce bazı uyarılar: 
- Bu yazıda “Intel x86 mimarisi için Assembly” den kısaca “Assembly” diye bahsedilecektir.Yani x64'e dair hiçbir şeyden bahsedilmeyecektir​.
- Ayrıca okunmasını tavsiye ettiğim bazı referansları yazının en alt kısmında paylaştım. Çok fazla şey koymadım ama araştırmaya başlamak için ideal yazılar olduğunu söylemekte fayda var.
- Yazı ağır bir analoji metodolojisi izlemektedir.Benzetme sanatının anlama ve öğrenmede en etkili yaklaşım olduğuna inandığım için faydasının olacağı kanaatindeyim.

Her Problemi Çözen Tek Bir Problem Vardır
-----------------------------------------

Hemen her programlama dilini öğrenmeye başladığımızda yazdığımız ilk program bir “Hello,world!” programı olur.Bu teamülü devam ettirerek biz de banal bir “Hello, world!” programını Assembly ile yazalım hemen.
 
```
global​ main
extern​ printf
section​ .data​
	fmt:  db  '%s' , 0x0a, 0​
	msg:  db  'Hello world!'​,​0
section​ .text
main:​
	mov esi​, msg​
	mov edi​, fmt​
	mov eax​, ​0​
	call printf​
	ret 
```


Bu sıradan "Hello, world!” programının daha önce hiç “Assembly nedir?” diye merak edip bakmamış olan birisi için anlamsız yahud korkutucu gelebileceğini tahmin edebiliyorum.Ancak bu basit program tek başına size Assembly’yi öğretebilecek kadar farklı özelliği ve sentaks kurallarını ihtiva ediyor. Tabi bu mübalağa bir kod parçası gördüğünüzde en azından “Burada neler yapılmış?” sorusuna cevap verebilecek bir noktaya kadar anlamlı.Gerçek dünyada bir program üzerinde reverse engineering (tersine mühendislik)yaparken veya compiler’ın ürettiği bir optimize (-02 veya -03 flagleri gibi ) Assembly kodunu incelerken, kafanız karışmadan ve belki anlamak için uzun süreler harcamadan bu pek mümkün olmayacaktır. Ancak bu korkulacak veya endişe edilecek bir şey değildir.Assembly öğrenerek bir nevi hiç yoktan aslında tarihsel olarak geçmişe doğru bir yolculuk yaptığınızı düşünebilirsiniz. Nerelerde ve nasıl işinize yarayacağı konusuyla ilgili burada bir bahis olmayacak. O konuda da sanırım araştırma yaparak pek çok sonuca ulaşmanız mümkün.Ama değerli bir bakış açısı kazandıracağı kesin olarak, yaptığınız veya yapmakta olduğunuz rutin programlama işlerinize daha felsefi ve derinlikli bir anlam katmanıza olanak sağlayacaktır.

Hello, world!” ten Öncesine Geri Dönelim (Flashback)
----------------------------------------------------

Muhtemelen daha o yazıyı okumadınız ama yazı serisinin ikinci bölümünde ele alacağımız “Hello, world!” ün öncesine burada geri dönüyoruz(Eğer okuduysanız burayı da baştan aşağı okumanızda fayda vardır).Bu geriye dönüş “Bir bilgisayar nasıl çalışır?” sorusuna kadar gidecektir. İşte bu noktada karşımıza birkaç tane de son derece önemli ara soru çıkmaktadır.Bize güzel cevaplar verecek ve bilgisayar bilimine dair telakkimizi geliştirecek bu özdeş sorular bir silsileyle şu şekilde gösterilebilir:

1. "Bir bilgisayar nasıl çalışır?” ​**ın bizi götürdüğü** ​“Bir işlemci/mikroişlemci nasıl çalışır? ”sorusu
2. "Birtakım harflerden oluşan (öyle görüyoruz) metinler (programların kaynak kodları) nasıl compile ediliyor? ” ​**un götürdüğü klişe olan** ​“Her şey 0 ve 1’lerden ibarettir.” , “Nasıl oluyor da çok karmaşık gelen hesap işlerimizi kolayca yapabiliyoruz? ” ​**soruları**

Baştan söyleyeyim bu iki soru silsilesine cevaplar vermek pek kolay olmayacak.Bunlar farklı farklı şekillerde sorulabilir.Ancak burada bir kitap ele almadığım, blog yazısı yazdığım için oldukça basit ve detaydan kaçınarak, sade ve anlaşılır bir şekilde açıklamaya gayret edeceğim. O yüzden bu soruları genel olarak iki bölümde ele alacağım. Bunlardan birincisi “CPU” merkezli olacak ama “ram” de bahislerin yer alacağı bir bölüm olacaktır. Diğer bölümde ise yine “CPU” merkezli bahisler olacak ancak bu sefer tabir yerindeyse kitabın kapağına bakmak yerine açıp sayfalarına bakacağız yani donanım ile yazılımın kesiştiği noktayı anlamaya çalışacağız.

CPU vs Learner – CPU vs Öğrenen
-------------------------------
![cpu vs learner](https://ffc12.github.io/docs/assets/images/cpu.png)


CPU yani merkezi işlem birimi için bir ad aktarması yapacak olsaydık doğrudan bilgisayar olarak ifade edilebilirdi; öyle de oluyor çoğu zaman. Yani bir anlamda bilgisayarın beyni olan bu donanım bulunmadığı sürece bilgisayardan söz etmek mümkün değildir. Tanım olarak, bilgisayarın içerisinde yer alabilen veya harici olarak da (microprocessor-microcontroller) gömülü sistemlerde kullanılabilen, görevi kendisine verilen instruction-ları işlemek olan bir elektronik devredir. Bu tanımı biraz daha açmamız gerekiyor.

Öncelikle “instruction” kendi başına bir başlıkta incelenmesi gereken bir konu ihtiva ediyor.Bunu başka ve mustakil bir yazıda ele alacağım. Ama özetle bu instruction-lar bilgisayarların işlemcilerinin elektronik devrelerinin sahip olduğu mantıksal işlemleri kullanarak, “programlamak” dediğimiz eylemi yapabilmemiz için benzersiz numaralarla soyutlanmış ve standartlaştırılmış bir modelin birer kuralı olarak düşünülebilir; bu instructionları daha iyi anlamak ve üzerinde çalışabilmek için bu benzersiz numaralarının bizlere anlamlı gelen kelimelerle ilişkilendirilmesiyle de Assembly dediğimiz dili elde etmiş oluruz.

Bu mantıksal işlemler bir noktada aritmetik işlemleri de yapabilecek şekilde gruplandırıldığı için en temelde bir veya iki adet bildiğimiz,anladığımız sayıları (1,2,1002323 vb. gibi)kullanarak önceden belirlenmiş veya devre üzerinde mapped edilmiş (haritalanmış) yollar üzerinde elektrik akışını(elektrik yolu) sağlamaktadır diyebiliriz.Yani belli bir sayı aralığında tüm ihtimalleri düşünülmüş mantıksal ve aritmetik işlemleri yapabilecek şekilde tasarlanmış bu devrenin, kendisi için belirlenmiş benzersiz numaralara göre farklı kombinasyonlarda devre üzerinde birer elektrik akış yolu bulunur.Bu kısım elektrik-elektronik bilgisi olmayan birileri için anlaması güç olabilecek bir kısım olmuş olabilir.

Biraz daha açmaya ve açıklamaya çalışalım.Düşünün ki 1,2,3 veya 4 gibi sayılar gerçek dünyada, matematikte aslında soyutturlar; 1 ile el sıkışamazsınız mesela , somut şekilde bu mümkün değil.Ancak parmaklarınızı ele alalım(sizin zaten elinizde biliyorum).1 sayısını göstermek için sadece işaret parmağınızı havada, parmak boğumlarınızı aynı doğrultuda tutarak, göstermeniz pekala mümkündür.Böylelikle evrensel olarak 1 sayısını somut şekilde göstermiş olabilirsiniz. Bu elektriğe veya elektrik devrelerine aslında işaret parmağında gerçekleşmiş mekanik hareketlere benzer bir şekilde atom altı etkileşimleri (daha genel olarak elektrik akımını) kullanarak ona 1’i göstertmek ve evrensel olarak bizim kabul ettiğimiz (standart model) somut bir işi yaptırmaktır.Dolayısıyla bir elimizdeki soldan ilk üç parmağın parmak boğumlarınının herbirisini kendi üç-boyutlu koordinat sistemi içinde aynı doğrultuda tutarsak (açarsak) ve kalan parmaklarımızın parmak boğumlarını aynı doğrultuda tutmazsak (kapatırsak) neticede 3 sayısını bir elimizi kullanarak göstermiş oluruz.

![one with finger](https://thumbs.dreamstime.com/b/1-2-3-4-5-hand-15977839.jpg)

Elektrik için de buradaki gibi mekanik olmasada aslında parmakları açıp kapatarak farklı sayıları göstermesine benzer bir mantıkla çalışacak devre kurabiliriz.Nitekim öyle de olmuş; CPU ve binlerce farklı donanım gündeme gelmiştir. Ve yine belki iki elimizi kullanırsak farklı şekillerle de aynı işaret dilinde olduğu gibi belli kelimeleri parmaklarımızla ifade edebiliriz. Neticede işte bu analojideki parmaklarımızı göstererek bu kelimeleri anlamamız veya anlatmamız Assembly dili soyutlaması olurken,parmaklardaki mekanik hareketlerin farklı şekillerde ifade ettikleri benzersiz durumları da göstermiş olmak önceden bizim belirlediğimiz standartlar olmuş oluyor diyebiliriz.Bu standartlara da bilgisayar için ISA (Instruction set architecture) ismi verilmektedir.Buna mimari ya da bilgisayar mimarisi de denilebilir.Bu ISA ile işlemcinin kullanabileceği data tipleri, registerlar, bir memory-i nasıl yöneteceği ve I/O (input-output) iletişiminin nasıl gerçekleşeceği -bu kavramları yazılarımızda açıklayacağız- binlerce farklı senaryo için numaralandırılmış (yukarıda uzunca anlattık, *benzersiz numaraların parmaklarla gösterilmesi*) ve devre bu şekilde tasarlanarak üretilmiştir.En nihayetinde de biz insanların anlamasını kolaylaştırmak, "programlamak" adınız verdiğimiz eylemi yaparken programları kolay ifade edebilmek için, sonra Assembly dili adı altında bu numaralar çeşitli kelimelerle soyutlanmıştır.Bu noktada fazla derine inmeden son olarak günümüzde iki tane ISA modelinin olduğundan bahsederek bir sonraki başlığa geçelim.Bu standart modellerden birisi CISC (Complex instruction set computer) ve diğeri de RISC’dir(Reduced instruction set computer). Bunların farkları bu yazının konusu değil ama özetle CISC daha donanım merkezci, karmaşık instruction-lar içerirken ve RAM dostu bir mimariyken, RISC daha yazılım merkezci,basit instruction-lar içeren ve ağır RAM kullanımına yol açabilen bir mimaridir.

Okus Pokus Compilus (Düzenleme yapılacak..)
-------------------------------------------

Compiler-lar günümüzde, belki de, biz farkında olmasak da bilgisayar bilimlerindeki en önemli konulardan birisidir - hatta en önemlisi -.Compiler-lar, Assembly hariç, programlama dillerini kullanarak yazılım geliştiren biz yazılımcılar için olmazsa olmaz programlardır. Düşünün ki şuan hiçbir hedef dili compile eden bir compiler olmasa kimse en sevdiği programlama dilinde bir yazılım geliştiremeyecektir.

Temelde compiler dediğimiz program yazdığınız kaynak kodlarını (bir grup sözcük ve karakter) işleyerek-belli dil kurallarına (syntax) uyup,uymadığını kontrol ederek, bunları neticede bir platformun Assembly diline çevirmeyi amaçlayan programlardır. Günümüzde her programlama dili elbette doğrudan platforma Assembly kodu üretecek şekilde tasarlanmamıştır. Şuan güneşli dünyada, Java, C# ve JavaScript gibi birçok dil sanal makineler üzerinde (bir nevi işlemci ve ram-i taklit eden programlar) çalışarak bunu sağlamaktadırlar. Tabi bu dilleri böyle tasarlamış olmalarının birçok sebepleri, avantajları ve tabiki dezavantajları var.Bu konuları araştırmanızda sizin için fayda vardır.Ayrıca bu tip dillere literatürde high-level programlama dilleri deniliyor.Buna sebep de bu bahsettiğimiz sanal makine yaklaşımıyla çalışmaları gösterilebilir. Diğer taraftan C,C++ ve Rust gibi diller platforma Assembly kodu üreterek çalışırlar.

Herneyse bir işlemcinin nasıl çalıştığına cevap vermiştik.Hatta Assembly dili seviyesine kadar açıklamaya çalışmıştık.Önceden belirlenmiş benzersiz numaralar vardı.Bunlar aslında kabaca farklı şekillerde elektriğin devreden geçmesiyle ortaya çıkan benzersiz durumlarla ilişkilendirilmiş benzersiz numaralardı. Bu numara daha sonrasında yine bizlerce bu sefer bir kelime ile ilişkilendiriliyordu: 

```
10 ‘a “taşı”
11 ‘e “artır”
```

şeklinde birer kelime ile “10” ve “11” numaralarının ilişkilendirildiklerini varsayalım.Dolayısıyla ben günün sonunda bu iki “instruction” için gerekli kelimeyi birleştirerek bir “instruction” –komut verebilirim.Bunu anlamlı hale getirmek için -çünkü bir hesap yapmasını bekleyeceğim- bildiğim ve anladığım sayıları da kullanabilmeliyim.Dolayısıyla bunu biraz geliştiriyorum ve bir sayıdan önce mesela 19 geliyorsa, her zaman ondan (19) sonra gelenin bir sayı olduğunu bilecek şekilde devremi tasarlamış olduğumu varsayıyorum. Yani:

```
10 ​'a "taşı"
11 ​'e "artır"
19 ​'a "sayı öneki
```

şeklinde kelimelerimle devremin üzerinde birer yolu temsil edecek benzersiz numaraları ilişkilendirdim(Bunu nasıl yaptığımı anlatabilmem şu noktada oldukça detaya girmek olacak.O aradaki elektrik-elektronik alanına giren bilgi katmanını şu videoya havale ederek işin içinden sıyrılmak istiyorum: [link](https://www.youtube.com/watch?v=cNN_tTXABUA)).Dolayısıyla artık mesela elimde 200 sayısı varsa ve eğer ben bunu klavyeden alıp, daha sonra 202’ye kadar arttırmak istiyorsam kabaca şöyle bir şekilde kodlamam gerektiğini düşünebilirim:

`10 19​​ 200 ​​11 ​​11`

bunu kelimelerle ifade edersem

```
ta​şı 200
artır 
artır
```

Bu pratikte çok doğru olmayabilir.Hatta aklınıza gelen “taşı” neden sayıdan önce neden sonra değil gibi bir sürü soru da olabilir. Bunları şimdilik sormayı bırakmanızı istiyorum. Çünkü bu örnek tamamen anlatabilmem için pseudo-instruction ve opcode-lardan oluşuyor.Yani mesele öğrenmek ve anlamak olarak düşünülebilir bu örnek için.Ancak burada “opcode” diye yeni bir kavramla tanışacağız.

Instruction Altı Parçacıklar *Opcodes
-------------------------------------

Başta anlattığımız, devrenin bir durumunun benzersiz bir numara ile ilişkisinin olmasıyla benzersiz numaranın da yine aynı şekilde bir kelimeye karşılık gelmesi ilişkilerini biraz daha açmam bu noktada çok anlamlı olacaktır. Devre durumu-benzersiz numara ilişkisinde bir farklılık yok. Ancak benzersiz numara ve kelime ilişkisini kurarken işlemcinin aynı anda kaç bitlik data-yı işleyebileceğini düşünerek ve bu kapasitesiyle en fazla ne kadarlık bir numara veya numaralar dizisini alabilir bunu da düşünmemiz gerekiyor. Sözgelimi mesela 8-bitlik bir işlemci için düşünecek olursak aynı anda tek bir ​​cycle​-da 8-bitten oluşan “instruction” yani numara veya numaralar dizisi alabilir, işleyebilir.Şuana kadar hep numara veya numaralar diye bahsettiğimiz bu kodların literatürdeki ismi ​binary forma/bytecode ​olarak geçer.Binary format isminden de anlaşılacağı üzere buradaki “data” aslında ikilik tabanda ifade edilmesinden dolayı kullanılan bir isim. Bytecode ise daha çok bu binary ifadelerin 8 ile bölündüğünde elde edilecek byte'ın dışında gruplandırmaya dair bir farklılık arz eder.Hani bildiğimiz klasik bilgisayarlar 0 ve 1-lerden anlardaki binary işte. Tabi diğer taraftan bu tamamen matematiksel soyut bir gösterim olduğundan binary şeklinde ifade edilebilir olduğu gibi ayrıca onaltılık ve onluk tabanda da ifade edilebilir.Bu seçim ve gösterim tamamen soyut.Anlama, kısa gösterim (okunabilirlik) ve standart oluşturma durumlarına bağlı data-nın gösterimi değişiklik gösterebilir. Ama bunun devreyle bir ilgisi yoktur.

Neticede bundan sonra biz de yeterince anladığımız bu kavramları gerçek isimleriyle zikredebiliriz.Ayrıca buradaki ifade ettiğimiz cycle, CPU’nun nasıl çalışacağını bildiren/gösteren “instruction”ların her birisinin bilumum kapasitesi ya da bu işlemi gerçekleştirme süresi için kullanılan bir kavramdır.Genel tanım, “instruction cycle” diye zikredilebilecek olan ve CPU’nun bilgisayarı boot edip kapatmasına kadarlık süreçte üç aşamadan(fetch-decode and execute / çek-decode et-çalıştır) oluşan bir yaklaşımla “instruction”ları işlemesi olarak yapılabilir. Bu işleme sürecinin birim süresi de CPU’nun kapasitesine (8-bit mi,16-bit mi,32-bit mi? ) bağlı olarak her 8-bitlik (16-bit veya 32-bit de olabilir) tam kapasite (yani 8-bitlik bir data) bir iş için cycle-larla ifade edilmektedir.Bu bir cycle-da üç aşamadan meydana gelir; ​fetch​, ​decode​ ve ​execute (birkaç cümle önce parantez içinde de Türkçe olarak ifade ettim)​.Bunu somut bir örneği ele alarak daha iyi anlamaya çalışalım.Mesela 8-bitlik bir işlemciyle 4 cycle-da yapabileceğim bir işi ihtiva eden toplam 32-bitlik bir “instruction” için bu işi 32-bitlik bir işlemci ile tek 1 cycle-da yapabilirim. Bu da aynı zamanda: “Neden 8-bit,16-bit veya 32-bit olması işlemci cihetiyle önemlidir sorusuna”, işte bu, sebeplerden bir tanesi olarak gösterilebilir; çünkü işlem yapabilme kapasitesi olan birim cycle-da 8-bit ve 32-bitlik işlemciler arasında kapasite olarak da 1/4 oran bulunacaktır.Bu noktada karşımıza şöyle bir önem arz eden durum çıkacaktır: ​Öyleyse işlemcinin daha performanslı çalışabilmesi için mümkün mertebe işlerimi en az sayıda cycle-la yaptırmalıyım​. Yani yapacağım iş için yazacağım “instruction”ının iyi şekilde ​encode edilmesini sağlamalıyım.Böylelikle işlemciye en kısa sürede yaptırmak istediğim işi yaptırabilirim. Bu genelde compiler geliştiricilerini (optimizasyondan ötürü) ve yine performans kritik işlem yapmak isteyen sair herkesi alakadar eden önemli bir konu. Velhasıl elbette herkes eğer mümkünse 8-bit bir işlemci yerine elbette 32-bit bir işlemci tercih eder ancak diğer yazılarda da karşımıza çıkacak ki bazı işleri yapmanın onlarca farklı yolu ve farklı instruction zincirleri ve kombinasyonları olabilir. Bunlar birbirinden farklı olarak, farklı sayıda cycle ile çalışabilir.Dolayısıyla yazılımsal olarak da optimizasyon yapmak daha doğrusu en iyi ihtimali bulmaya çalışmak gerekir.

Ayrıca instruction-ın encode edilmesinden maksat burada, yukarıda yaptığımız basit pseudo-opcode ve örnek instruction setlerinin dışında gerçek dünya için oldukça önemli. Cycle sayısı ile instruction'da encode edilecek bilginin dengesini bulmak gerekiyor. Öyle bir sistematik hazırlamalıyız (elementlerin periyodik tablosunu düşünün) ki mesela 3 ayrı cycle-da yapılan bir işi, instruction-ları, biraz daha karmaşıklaştırarak ama tek bir “instruction” şeklinde işlemciye vererek, yani benzersiz numaralandırmaları mümkün olduğunca sıkıştırarak ama sonradan decode edebilecek bir kural veya gösterim kullanarak tutabilmeliyim.Genelde sanıldığının aksine bunu yaparken burada kelimeler ile yaptığımız soyutlamayı değiştirmek zorunda değiliz. Nitekim işlemci için anlamlı olacak olan şey bir tane ISA ile belirlenmiş benzersiz numara yani “instruction” olacak.Senin ona Assembly seviyesinde “ali” veya “veli” ismini vermiş olman Assembler onu "instruction"-ı doğru şekilde encode ettiği sürece çok da umrunda değil; olamaz.Yani, unutmayın, bizim ne dediğimizin hiçbir önemi işlemci nazarında yok aslında, Assembler yani çevirmenin doğru çevirip çevirmemesi önemli.Zaten compiler teknolojileri bu yüzden önemli bir mahiyete sahip.

Yukarıdaki örnek pseudo-instruction-lara geri dönüyorum:

```   
10 'a "taşı"
11 'e "artır"
19 'a "sayı öneki" 
```

buradaki benzersiz numaraları kullanarak,

```
taşı 200
artır
artır
```

soyutladığım kelimelerle bir program yazmıştım ve aslında bu program şu şekilde dönüştürülüyordu.

`10 19 200 11 11`

Aslında bu tam olarak yaygın kullanımda bu şekilde ifade edilmez.Genelde bahsettiğimiz gösterimler arasında hexadecimal yani onaltılık taban kullanılarak “0x” prefix’i ile gösterilir:

`0A  13 C8  0B  0B`
(Hexadecimal gösterime dair bir bilginiz yoksa, bu konuyu en kısa sürede öğrenmenizde fayda var.)

Bu pseudo-instruction-ı oluştururken demiştik ki bir sayıyı işlemek istiyorsak onun önüne “19”sayısını getirmiş olalım; yani ​19 (0x13)​ burada ardınca gelenlerin işlenecek data olduğunu gösteren bir özel numaraydı. Bunun gibi bir data-nın olduğunun gösterilmesi ile ilgili numaralar ve data-nın kendisi haricindeki tüm ifadelere “​opcode​” denilmektedir.Yani 0x0A ve 0x0B birer ​opcode​‘tur.Peki data-nın kendisinin hiçbir ismi bulunmuyor mu? Hayır,bulunuyor: “​operand​” şeklinde ifade edilmektedir.Ve tabi işlenecek bir data var mı yok mu veya hangi boyutta (32 bit mi 8 bit mi vs.) gösterebilmek yahud anlatabilmek için de kullandığımız numaralara "prefix" ismini veriyoruz.Özetle, bir ​instruction​ işlemci ​kapasitesi​ne göre (8-bit mi , 32-bit mi ?) ve o işlemcinin ​ISA standart​ına göre belirlenmiş (kelimelerle soyutlanmış) ​opcode​-lar ve bunların belirttiği işlemlerin yapılabilmesi için gereken ​operand​-lardan (data) ve yine data-nın boyutu ya da mahiyeti ile ilgili bilgi veren prefixlerden oluşmaktadır. Bunlar belli kuralla instruction-ın CPU'nun decoder'ı ile decode edilebilecek şekilde encode edilir. Bu “instruction”ın oluşturulması işlemine de “​instruction encoding​” denilmektedir.Bu encode işlemi de işlemcinin kapasitesine göre ( bir cycle-da kaç bitlik data işleyebilir ) genellikle tasarlanmaktadır. Bu konuyu müstakil bir yazıda teferruatlı bir şekilde ele alacağım. Muhtemelen başlığı da“​Instruction Anatomisi​” olacaktır. Ancak burada elimden geldiğince açıklayıcı şekilde aktarmaya çalıştım.

 

Sonuç
-----

Yukarıda oldukça baside indirgeyerek cevaplamaya çalıştığımız “Bir işlemci nasıl çalışır?” ve “Compiler 0 ve 1-lere dönüştürüyor ama peki bu CPU'ya nasıl aktarılıyor ve daha sonra nasıl çalışıyor?” gibi sorular elbette oldukça detaylı cevaplara sahiptir. Filhakika bizim buradaki anlattıklarımız çok genel ve son derece yüzeysel(yine biraz detaylı ama) olmakla beraber Assembly ile programlamak ve Assembly’nin nasıl çalıştığını anlatmak için aslında temel bir bakış kazandırabilmekti. Buradan hareketle aslında çeşitli konulara dair iplerin ucunu sizlerin ellerinize verdik. Takip edip daha tafsilatlı bir şekilde konular üzerinde ihtisas yapmak sizlere kalmıştır.Buna en altta paylaşttığım linklerdeki okumaları yaparak başlayabilirsiniz.Toplamda bu mukaddime hariç 6 yazıdan oluşacağını tahmin ettiğim Assembly yazı serisini sonuna kadar okumanızı tavsiye ederim:

1. x86 Assembly 1. Bölüm : Intel vs AT&T
2. 2.x86 Assembly 2. Bölüm : “Hello, world”
3. 3.x86 Assembly 3. Bölüm : Köprüden Önceki Son Çıkış (Addressing Modes)
4. 4.x86 Assembly 4. Bölüm : Stack ve Avanesi
5. 5.x86 Assembly 5. Bölüm : Macro-lar Dost Mudur Düşman Mıdır?
6. 6.x86 Assembly 6. Bölüm : C İthalat-İhracat AŞ.

Okuma Listesi
- https://stackoverflow.com/a/6463981/15258319
- https://en.wikipedia.org/wiki/Central_processing_unit#:~:text=The%20actual%20mathematical%20operation%20for,storing%20the%20result%20to%20memory.
- https://en.wikipedia.org/wiki/Instruction_set_architecture
- https://www.microcontrollertips.com/risc-vs-cisc-architectures-one-better/
- https://www.techopedia.com/definition/938/binary-format
- https://en.wikipedia.org/wiki/Instruction_cycle
- https://en.wikipedia.org/wiki/Opcode
- Operating Systems: From 0 to 1'ın 4. bölümü