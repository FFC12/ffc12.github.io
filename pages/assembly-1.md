# x86 Assembly 1. Bölüm : Intel vs AT&T
Tarih:​ ​25 Şubat 2021​ 

Daha önce şu​ ​giriş​ yazısı ile bu yazı dizgesinin neler ihtiva edeceğinden
bahsetmiştik.Bu yazıda birinci bölümde ele alınacak olan Assembly diyalektlerinden,
Assembler-lardan ve İşletim Sistemine bağlı olarak yazacağımız kodun nasıl
farklılaştığından bahsedeceğiz.

Assembly yazarken karşımıza iki farklı sentaks kuralı çıkıyor: Intel ve AT&T. Bu iki farklı
sentaksın da ayrı ayrı kendisine göre avantajları ve tabi dezavantajları bulunuyor.Bu
yazımda bu konu üzerine derinlemesine bir inceleme yapacağız ve birbirleri üzerindeki
avantajlarından ve dezavantajlarından bahisler açacağız. Ayrıca bu yazıyı üç kısma bölersek
-o şekilde yazmayı kafamda tasarlamıştım- birinci kısmı bu oluşturacak iken – Intel vs AT&T
mevzuu – , ikinci kısımda Assembler-lardan ve üçüncü kısımda da üzerinde çalıştığımız bir
işletim sisteminin olup olmaması ve bu işletim sistemlerinin farklılıklarının Assembly
yazarken neden önem arz ettiğinden bahis edeceğiz.

>
>   Intel okunduğu gibi “Intel” olarak, AT&T ise “eyti end ti” şeklinde telaffuz edilir.
>

Bu yazıda aslında diğer bölümlerdeki pekçok meselenin temelini atacak ve en önemlisi
tarihsel olarak bir bakış kazandırmaya gayret edeceğiz. Tarihsel bir bakıştan neyi murad
ettiğimizi yazının son cümlelerine doğru iyi anlayacağımızı düşünerek geçiyorum.


>Sentaks Nedir?
>
>   Söz zincirinde birbirini izleyen ve belli bir birim oluşturan öğeler birleşimi,
>   sözcüklerin tümcede dilbilgisi kurallarına göre dizilişi, sıralanışı.
>
>
>   Google


Basitçe bunu bir versus’a indirgeyecek olursak aslında Unix dünyası vs Microsoft şeklinde
bir versusla da gösterilebilirdi. Unix dünyasında AT&T sentaksı yaygın olarak kullanılır(bknz.


PDP-11​) ve ortak Assembly sentaksı olarak kabul görür. Diğer taraftan Unix ve GNU demek
Linux demek olduğu için GNU’da Unix dünyasından gelen teamülleri devam ettirmiş ve
birçok diğer farklı tevarüste olduğu gibi Linux dünyasında da AT&T etkili olmuştur. Yani
Linux’un derlendiği GCC ve GCC’nin destekleyicisi olan Binutils araçları ve tabi bu araçların
en önemlilerinden birisi olan GAS (Gnu Assembler) AT&T sentaksını kullandığı için bu çokça
yaygın olmuş ve Linux için de ortak Assembler sentaksı olarak kalmıştır. Tabi bu demek
değildir ki Intel sentaksı Linux dünyasının içerisinde görmezden gelinir veya kullanılmaz.
GAS diyalekt olarak Intel sentaksını kabul eder ve binary formata dönüştürebilir.Yine tabi
GCC’de bu diyalekt için Assembly kodu oluşturabilir. Bu söylediklerimizi basit şekilde GCC
için ​ **-masm=Intel** ​ ​flag​-i veya kaynak kodun içerisinde ​ **“.intel noprefix”** ​ gibi direktiflerle
yapmak pekala kolay ve mümkündür.Ayrıca Linux’ta kullanılabilecek tek Assembler GAS
değildir; NASM,modern hali YASM, TASM ve sair birçok farklı Assembler ile de Intel
sentaksı ana diyalekt olarak kullanılabilir. Diğer tarafta ise, Microsoft dünyasında, Intel
sentaksı yaygın olmuştur. MS-DOS’la başlayan bu tercih günümüzde Microsoft’un
MASM’ına kadar gelmiştir.Bunları hususi araştırmanızda fayda olacaktır.Bu yazının ikinci
kısmında Assembler-lardan bahsederken bu konuya tekrardan geri döneceğiz.Şimdilik bu
kadarıyla yetinerek bu iki sentaks kuralına yakından gözatalım.

## My Old Friend, AT&T – Eski Dostum, AT&T

İlk olarak AT&T sentaksını ele alalım.Hatta şuan hiçbir şeyden anlamıyor olsak da sadece
gözümüzün ucuyla bakmak için basit bir “Hello world” uygulamasını inceleyelim.

```
.text
.global _start
_start:
movl ​ 13 ​,​%edx
movl ​$msg​,​%ecx
movl $​ 1 ​,​%ebx
movl $​ 4 ​,​%eax
int $​ 0 ​x
```
```
movl $​ 0 ​,​%ebx
movl $​ 1 ​,​%eax
int $​ 0 ​x
```
```
.data
msg:
.​ascii​ ​"Hello world!\n"
```

Bu sentaks oldukça basit bir işi, Linux sistem çağrılarını (syscall) kullanarak “Hello, world”
yazmaktadır. Sistem çağrısı veya Linux nedir diye endişe etmeyin.İşletim sistemlerinden ve
sistem çağrılarından (syscall) son kısımda -üçüncü- bahsedeceğiz.Daha önce Assembly ile
uğraşmadıysanız bu kodu anlamakla uğraşmanıza da gerek yok.Bu yazı dizgesiyle birlikte
aşama aşama birçok meseleyi zaten inceleyeceğiz ve umuyorum ki, bu kodu, yazı dizgesini
tamamıyla okuyup takip edip en nihayetinde bitirdikten sonra ezberden yazabilecek kadar
Assembly ile uğraşmış olacaksınız.Şuan sadece teorik denilebilecek ve hem çalışacağımız
ortamı oluşturabilmek için hem de yazacağımız kodu neden ve niçin o şekilde veya bu
şekilde yazdığımızı anlayabilmek adına bir önbilgi kazanıyoruz. Ayrıca tarihsellik problemini
de bir şekilde çözmüş olacağız: “16-bit,32-bit ve 64-bit”. (Şuan böyle bir problem yaşamıyor
olabilirsiniz ama her alanda,branşta veya konuda ihtisas yaparken tarihsellik – uğraştığınız
işin gelişimi ve metodolojilerinin evrilmesini belli bir kronolojide anlamakta- problemi yaşar ve
bir şekilde öğrendiklerinizi ilişkisel olarak bir kronolojik sıraya sokmak zorunda kalırsınız.)

AT&T sentaksı ilk bakışta göze oldukça karmaşık ve korkunç bir görünüme sahipmiş gibi
gelebilir.Ancak bu yaklaşımla aslında biraz daha işlevsel ve uzun satırlar kod yazarken
bulunduğu bağlama bağlı olmaksızın (okunurken) okunurluluğu uzun vadede artırdığına
ileride tanıklık edeceksinizdir.Ayrıca belki Intel sentaksına göre daha cool gözüktüğü de
söylenebilir.

## Intel Syntax the Handsome Guy – Yakışıklı Herif Intel

AT&T sentaksına kabaca bakarken yaptığımız gibi hemen Intel sentaksı ile yazılmış bir
“Hello world” uygulamasını inceleyelim.

Intel sentaksı sade ve anlaşılır olması bakımından çok fazla sevilir. Öğrenmeye yeni
başlarken genelde tercih edilen sentaks da Intel’in ki olur. Ama AT&T’nin aksine kod satır

```
;intel syntax
global​ _start
section​ .data
message:
db​ ​'Hello world!'​,​ 0
section​ .text
_start:
​mov​ ​eax​, ​0x
​mov​ ​ebx​, ​0x
​mov​ ​ecx​, message
​mov​ ​edx​, ​ 12
​int​ ​0x
```
```
​mov​ ​eax​, ​0x
​mov​ ​ebx​, ​0x
​int​ ​0x
​ret
```

sayısı artıkça yeterli tecrübeyle bile anlaşılırlık açısından bunun bir ilüzyon olabileceğine
tanıklık edeceksinizdir.Tabi buradaki ufak kod parçacıklarıyla tüm iyi taraflarına ve kötü
taraflarına tanıklık etmek mümkün değildir.Ama şuan öğrenme aşamasındaki birileri için kısa
vadede doğru ve anlaşılır kod yazıp okumak açısından son derece anlamlı bir tercih
olabileceğini söyleyebiliriz.

Aslında karşılaştırmalı şekilde herhangi bir avantaj veya dezavantajdan bahsetmetik. Çünkü
gerçekten bunu yapacak kadar başta ifade ettiğimiz gibi avantaj ve dezavantaj
sağlayabilecek (modern halleriyle) bir fark bulunmuyor.Sadece okunabilirliği açısından AT&T
ilk bakışta daha karmaşık gelmekte.Ve sonraki yazılarda Assembly yazmaya ve anlamaya
başladığımızda karşımıza çıkacak olan “opcode” kavramı açısından AT&T’de “data”nın
boyutuna göre isimlendirilmesine bağlı olarak daha fazla “opcode” olması da belki burada
zikredilebilir.Ancak bu da tam olarak bir dezavantaj sağlayacak mıdır, tartışılır.

Velhasıl bu yazı dizgesi boyunca Intel sentaksını kullanarak kodlama yapacak ve
Assembly’nin özelliklerinden bahsedeceğiz. Genel olarak hem konsepti hem de işleyişi
anladıktan sonra zaten AT&T ‘yle ya da Intel ile istediğiniz gibi çalışmanız ve diyalekten
agnostik düşünmeniz pekala mümkün olabilmektedir.

## Assembler – Her Yiğidin Bir Yoğurt Yiyişi Vardır.

Devam Edecek ...

