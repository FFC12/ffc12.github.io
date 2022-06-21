# Giriş

Şu [müzik](https://www.youtube.com/watch?v=VgR0O3klPhw) eşliğinde "bad ass" bir giriş yapalım. Bugün konumuz neredeyse her programlama dilini öğrenmeye başlarken konsola/terminale "Hello, world!" yazdırdığımız bir program olacak. Bunu Assembly yazarak yapacağız elbette. Tabi bu programın üzerinde de genel olarak önceki yazılarda konuştuğumuz Intel diyalektiğini kullanan ve NASM ismindeki assembler-a uygun sentaks kurallarıyla yazılmış olacak programın yapısını inceleyeceğiz. Böylelikle de Assembly (bu yazılar boyunca x86 Assembly için Assembly diye bahsedeceğiz diye belirtmiştim ilk yazıda) öğrenmeye başlamış olacağız.

Ancak programı yazmaya başlamadan önce bazı önbilgiler vermem gerekiyor. Öncelikle fazla kapsamlı olmayacak şekilde işletim sistemlerinin (OS) nasıl çalıştığını hatırlayalım. Bilgisayar birçok elektronik devrenin aslında belli bir organizasyon içerisinde çalıştığı oldukça karmaşık bir elektronik cihazdır ve bir bütün olarak CPU, RAM, mouse, klavye ve diğer birçok farklı dahili veya harici donanımdan oluşmaktadır. İşletim sistemleri ise sahip oldukları kernel (çekirdek) ile donanımlarla konuşarak üst seviyede genelde sunduğu bir arayüz yardımı ile karmaşık pek çok operasyonu bizden gizleyerek yapmak istediğimiz işlemleri yapmamıza olanak sağlarlarlar.

İşletim sistemleri işlemcinin korunumlu modunda (protected mode) çalışırlar. Yani kernel-ları ile aralarında soyut bir katman bulunur ve genel olarak işletim sistemi üzerinde çalışan programlar kernel-a ait sistem çağrılarıyla (system calls) donanımın veya bu soyut katmanın izin verdiği ölçüde kaynaklar üzerinde tasarrufta bulunabilirler. Ayrıca belirtmekte fayda vardır ki; virtual memory ile fiziksel olmayan bir memory üzerinden kaynak kullanımı söz konusudur. Bu ayrıca uzun bir konu olduğu için işletim istemlerinin memory-i nasıl kullandığını ve user space'deki uygulamaların da memory'de nasıl allocation yaptığını hususi araştırmanızda fayda var.

Günümüzdeki bilgisayarların mimarisi genellikle Von Neumann ya da Princeton mimarisi olarak adlandırılır. Bu mimaride özetle birincil ve ikincil bir hafıza (storage) vardır. İkincil hafızada programın çalışmadan önceki sahip olduğu makine kodu ve kullanacağı dosyalar bulunur. Ne zaman ki programın execute edilmesi istenirse o zaman memory-e (RAM) yani birincil hafızaya makine kodu kopyalanarak, çalıştırılır. Tabi bu uçucu (volatile memory) bir memory-dir ki elektrik kesintisi durumunda tüm memory-deki bilgiler kaybolur. Ancak ikincil hafızadaki her şey korunur.

Programlarımızın birincil hafızaya düzgün şekilde yerleştirilebilmesi için (memory-e (RAM) execute edildikten sonra) verilerin belli boyutlarda olması iktiza eder. Genellikle 2'nin katları olarak 8, 16, 32 ve x64 mimarisinde 64 ve hatta 128 bitlik boyutlardan bahsedebiliriz. Genellikle C, C++ ve benzeri programlama dillerinde de bu veri tipleri (data types) vardır; int, short, long vb. gibi.  Bunlar aslında memory'de tutulacak verinin boyutuna dair derleyicinin (compiler) bilgi sahibi olmasına olanak sağlarlar. Yine belirttiğim gibi Assembly'de de aslında memory-e yerleştirmeyi düşündüğümüz verinin boyutunu belirtmemiz gerekiyor. Dolayısıyla veri saklama boyutları (storage sizes) üst seviye programlama dillerindeki (high-level programming languagues) gibi bulunur diyebiliriz. Bunlar Assembly seviyesinde genelde aşağıdaki tablodaki gibi isimlendirilirler.

|     |     |     |
| --- | --- | --- |
| Veri Saklama Tipi | Boyut (Bit) | Boyut (Byte) |
| Byte | 8 bit | 1 byte |
| Word | 16 bit | 2 byte |
| Double-word | 32 bit | 4 byte |
| Quad-word (x64 Sistemler) | 64 bit | 8 byte |
| Double quad-word (x64 Sistemler) | 128 bit | 16 byte |

Tablodaki saklama boyutları (storage sizes) NASM'ın sentaksı için de aşağıdaki tabloda gösterdiğim pseudo instruction-lar ile tanımlanacaklar/kullanılacaklardır. Ancak belirtmekte fayda var; genel olarak diğer assembler-lar veya diyalektikler için de aşağı yukarı bu ve benzeri isimler söz konusu olsa da farklı sentakslarla karşılaşmanız fevkalade mümkündür.

|     |     |     |
| --- | --- | --- |
| NASM Veri Tipi | Boyut (Bit) | Boyut (Byte) |
| db  | 8 bit | 1 byte |
| dw  | 16 bit | 2 byte |
| dd  | 32 bit | 4 byte |
| dq (x64 Sistemler) | 64 bit | 8 byte |
| ddq (x64 Sistemler) | 128 bit | 16 byte |

Bu veri tiplerini kullanarak çok değişik tiplerde veriyi memory-de saklama imkanımız olur. Şanslıyız ki assembler-lar zannedildiğinin aksine çok da low-level değiller görüldüğü üzere. Yani hala bize anlamlı gelecek şekilde verileri değişken tanımlar gibi tanımlayabiliyoruz. Tabi assembly yazarken high-level dillerde olduğu gibi akıllı compiler özellikleri (type-system ya da type inference) söz konusu değildir ve verilerimizin assembler tarafından yorumlanmasını bekleyemeyiz. Demem o ki her tip için opcode-larımızı veya pointer işlemlerimizi ne yaptığımızın farkında olarak yapmamız önem arz ediyor.

Diğer taraftan nihayetinde noktalı sayı (float) da aynı tam sayı (int) gibi birkaç bit içerisinde tutuluyor. Daha sonra işlem yaparken ya da ekrana bunu yazdırmak istediğimizde sadece nasıl yorumlamamız gerektiğine karar veriyoruz. Yani en azından high-level dillerdeki standart kütüphane fonksiyonları çoğunlukla bizim yerimize bunu yapıyor. Ancak assembly yazarken burada veriyi nasıl yorumlayacağımız bize kalmış ve bunu programda kendimiz yapmamız gerekebilir. Elbette hala C standart kütüphanesindeki fonksiyonları kullanma şansımız var fonksiyonların calling convention-larını bildiğimiz için. Neyse fazla uzatmadan birkaç örnek verelim. Bu örnekleri [şuradan](https://stackoverflow.com/a/22554838) almıştım:

```
    db      0x55                ; just the byte 0x55
    db      0x55,0x56,0x57      ; three bytes in succession
    db      'a',0x55            ; character constants are OK
    db      'hello',13,10,'$'   ; so are string constants
    dw      0x1234              ; 0x34 0x12
    dw      'A'                 ; 0x41 0x00 (it's just a number)
    dw      'AB'                ; 0x41 0x42 (character constant)
    dw      'ABC'               ; 0x41 0x42 0x43 0x00 (string)
    dd      0x12345678          ; 0x78 0x56 0x34 0x12
    dq      0x1122334455667788  ; 0x88 0x77 0x66 0x55 0x44 0x33 0x22 0x11
    ddq     0x112233445566778899aabbccddeeff00
    ; 0x00 0xff 0xee 0xdd 0xcc 0xbb 0xaa 0x99
    ; 0x88 0x77 0x66 0x55 0x44 0x33 0x22 0x11
    do      0x112233445566778899aabbccddeeff00 ; same as previous
    dd      1.234567e20         ; floating-point constant
    dq      1.234567e20         ; double-precision float
    dt      1.234567e20         ; extended-precision float
```

Tabi biz x86 Assembly yazacağımız için dq ve ddq gibi data type-ları kullanmayacağız. Ancak bilmekte ve görmekte fayda var. Bunlara yukarıda değindiğimiz gibi ayrıca sözde talimatlar (pseudo instructions) denilmektedir. Çünkü bunlar assembler tarafından basitçe boyutunun ne olduğuna dair yorumlanarak memory'e uygun şekilde yerleştirileceklerdir. Özetle yukarıda yazının ilerleyen kısımlarında göreceğimiz .data section'ında aslında assembly içerisinde uygun boyutta değişkenler tanımlamış oluyoruz. Buraya geri dönerek daha ayrıntılı irdeleyeceğiz. Şuanlık ilerliyoruz.

# Register-lar

Register-lar, CPU'nun geçici ve genellikle akümülatör görevi gören hafızalarıdır (temporary storage). Memory-den ayrı olarak doğrudan CPU'nun içerisinde bulunurlar ve tipik olarak tüm hesaplamalar (computations) bunlarla yapılır. Memory-den veya ikincil hafızadan (hard disk) veri almaya kıyasla daha hızlı oldukları için hesaplamaların daha hızlı olarak yapılmasına olanak sağlarlar diyebiliriz. Ayrıca tüm bunların dışında da önbellek (cache memory) bulunur. Bu da yine memory ve ikincil hafızaya göre register-lar kadar hızlı olmasa da veriye ulaşma imkanı sağlar ve yine CPU'da bulunurlar (L1 Cache, L2 Cache vs.). Ancak şuan için konumuzla ilgili değil. Dolayısıyla detaylarına girmiyorum.

Genel olarak register-lar şu şekilde ayrıştırılabilirler (farklı kaynaklarda farklı şekillerde de olabilir):

- **General Purpose Register-lar** (Genel amaçlı register-lar) x86 mimarisi için konuşacak olursak toplamda 8 tane (eax, ebx, ecx, edx, edi, esi, ebp, esp) bulunur; yine x64 mimarisinde toplamda 16 tane (rax, rbx, rcx, rdx, rsi, rdi, rbp, rsp, r8, r9, r10, r11, r12, r13, r14, r15) bulunur.
- **Stack Pointer Register** (ESP/RSP) aslında ESP veya RSP de genel amaçlı register-lar kategorisine girse de özel olarak kullanıldığı durumlar vardır. Bu genellikle memory-deki stack üzerinde veriyi saklamak için ve çoğunlukla da fonksiyon çağrılarında argümanların ve geri dönüş değerlerinin yerleştirilip, fonksiyon çağrısı tamamlandıktan sonra stack-ten temizlenerek yine programın çalışma akışında kaldığı yerden devam etmesine olanak sağlamak için kullanılırlar.
- **Base Pointer Register** (EBP/RBP) ESP veya RSP ile beraber kullanılarak Stack Frame-leri oluşturmamıza olanak sağlarlar. Bunlar ileride üzerinde detaylı duracağımız register-lar ve memory management teknikleri, o yüzden kafanızı karıştırdıysa veya anlamadıysanız endişelenmenize şuanlık gerek yok.
- **Instruction Pointer Register** (EIP/RIP) EIP veya RIP ise yine genel amaçlı bir register olarak listelense de özel olarak memory-de tutulan makine kodlarının/programın execute edilecek bir sonraki talimatını (instruction) CPU'ya göstermek için kullanılır. Debugger-larda genellikle eğer ki EIP veya RIP ile bir instruction gösteriliyorsa, o daha execute edilmemiştir. Genelde bu debugger araçlarında kafa karışıklığına sebep olan önemli bir bilgidir. Çoğunlukla da çok istisnai durumlar dışında EIP veya RIP gibi özel maksatlar için kullanılan register-lar programlarda hesaplamalar için kullanılmazlar.
- **Flag Register** (rFlags) CF, PF, AF,ZF,SF,DF, OF gibi status flag register-ları bulunur. Ayrıca control ve system flag-leri de bulunur. Bu register-ların genel olarak kullanımı yapılan matematiksel ve mantıksal işlemlerin sonuçlarına veya işlemcinin durumuna dair flag görevleri görmeleridir. Basit bir örnek vermek gerekirse, mesela 32 bitlik iki tane sayı toplandığında eğer ki sonuç ancak 33 bite sığabilecek büyüklükte bir sayı ise bu durumda CF set edilecek. Nihayetinde program akışı boyunca bu flag-lerin durumuna göre algoritma kurma imkanımız olacak. Bunlarla ilgili bilmemiz gerekenleri konusu geldikçe bu ve sonraki yazılar boyunca aktaracağım.
- **Segment Register-lar** CS, DS, SS, ES, FS, GS ismindeki segment register-ları sadece 32 bit işlemcilerin olduğu dönemlerde özellikle segmented memory modeli için geliştirilmiş register-lar olsa da şuan x64 mimarisinde de bulunuyorlar. Genel olarak işlemcinin real mode ve protected mode'unda işlemci mimarisine göre farklı maksatlarla kullanılabilmektedirler. Ancak özetle x86'da (ve x64'te de) memory güvenliği için bilgisayarların protected mode'unda doğrudan fiziksel memory (RAM) kaynakları tüketmek mümkün değildir. Ring-3 seviyesinde çalışan işletim sistemleri bize user space'de sadece çalıştırdığımız programın yine işletim sisteminin izin verdiği ölçüde sistem çağrıları yaparak virtual memory üzerinden kaynakları tüketmemize olanak sağlar. Bu güvenlik için gerekli olan prosedürü elbette atlatmak mümkündür. Ancak bunu bir işletim sistemi üzerinde yapmanız mümkün değildir. Kendimizin bir bootloader ile programımızı boot ettirmesi gerekir ki aslında bu da bir nevi işletim sistemlerinin yaptığı şeydir. Bu konular ayrı ve hususi başlıklar içerdiği için kısaca bahsettim ama şimdilik geçiyorum. Dolayısıyla segmentation modelinde, programların birbirinden soyutlanması ve en önemlisi memory-e alınan verilerin sınıflarına göre yine soyutlanması amaçlanmıştır. CS (code segment) register'ı code segment'ine yani memory-e execute edilmek için alınan makine kodlarına (instuction-ları) ulaşmaya imkan sağlarken DS (data segment) register-ı ise programın tanımlanan verilerini birbirileri üzerine overlap olmayacak şekilde yerleştirilmesine ve ulaşılmasına imkan sağlamaktadır. x86 Segmentation konusu biraz karışık bir konu olduğu için çok özet ve basit şekilde aktarmaya çalıştım. Ancak "Hello world" için bu register-lara zaten ihtiyacımız olmayacak.

Dikkat ederseniz "E" ile başlayan register-lar x86 mimarisi için, "R" ile başlayanlar ise x64 mimarisi için kullanılmaktadır. İsimlendirmedeki bu farklılık bizim için önemli. Ayrıca x64 mimarisinde "E" ile başlayan registerlar da kullanılabilir. Çünkü x64 mimarisi ayrıca x86 (yani 32-bit) mimarisiyle uyumludur ki aslında x64 mimarisindeki tüm register-lar alt parçalar olarak x86 mimarisine ait register-ları gösterebilmektedirler. Bunun için aşağıdaki tabloyu inceleyebilirsiniz.

![659de28eddb8a842ce33d4366a8bc68e.png](:/7a95d599be084fd58cdd5d7dc55e3219)

# Memory'e Yerleşim

Assembly kodumuzun assembler ile bulunduğumuz sistemin linklenebilir dosya formatı ne ise (linux için ELF, windows için PE) ona dönüştürüldüğünü sonra da linker yardımıyla executable dosyamızın oluşturulduğunu söylemiştim. Elbette bu executable programı çalıştırdığımızda, bunun memory'e yüklenerek execute edilmeye başlanacağını da ayrıca belirtmiştim. Tüm bunlar olurken aslında programımızın içerisinde tanımladığımız ifadeler (değişkenler diyelim) değerleri önceden tanımlanmış (initialized) ve değerleri önceden tanımlanmamış (uninitialized) veriler olarak, niteliklerine göre memory'de farklı kısımlara yerleştirilir. Bunu assembly kodumuzu yazarken özellikle belirtmemiz gerekecek. Section ismini verdiğimiz ve bölümler olarak ifade edebileceğimiz ve yazacağımız assembly kodunun bölümlendirilmesine imkan sağlayacak olan direktifler assembler tarafından yorumlanarak verinin initialized ya da uninitialized data olarak tanımlanmasına göre linklenebilir dosyayı oluşturacak. Daha sonra bu linklenebilir dosya linklendiğinde verilerin memory'e yerleştirilirken uygun adreslere koyulabilmesi için uygun şekilde encode edilecek.

Bu section-lardan toplamda üç tane bulunur. Sırasıyla "data", "bss" ve "text" section-ları. Bunlardan "data" ve "bss" değişkenlerimizi tanımlayacağımız bölümlerdir. "text" section-ı ise kodlarımızın yer alacağı kısmı teşkil eder. Bunlardan "data" section-ı değerlerini başlangıçta belirtmek istediğimiz (ki bu bir anlamda statik değerlerdir) değişkenlerin yer alacağı kısımdır. Ayrıca yazının başında ifade ettiğimiz "db, dd, ..." gibi veri saklama tiplerini bu kısımda kullanacağız. "bss" section-ı ise "data" section-ından farklıdır. Ayrıca kullanılması gereken direktifler "resb","resw", "resd","resq","resdq" olarak değişiklik gösterir. Bunlar memory-e yerleştirilirken ne kadarlık boyutta yer tahsis edildiyse tüm bu adresler 0 olarak tanımlanır.

```asm
section .data
// değerleri öncesinde belirlenmiş değişkenler (initialized data)

section .bss
// değerleri öncesinde belirlenmemiş değişkenler (uniniatilized data)
// genellikle burada array veya bir dizi baytın 0 olarak tanımlandığı 
// tipte değişkenler tanımlanacaktır.

section .text
// kodlar
```

# "Hello world"

Şuana kadar ihtiyacımız olan bilgileri öğrendik/tekrar ettik. Artık programı yazabilir ve satır satır inceleyebiliriz. Tabi bu noktada yazdığımız programı nasıl assembler ile derleyeceğimizi bir önceki bölümde NASM için gösterdik ama tekrardan hatırlatalım. Öncelikle benim kullandığım sistem Linux olduğu için kullanacağım sistem çağrılarının Linux sistem çağrıları olacağını belirteyim. Doğrusu Windows için sistem çağrılarını da sizin araştırıp kendinizin uyarlaması gerekir. Bu noktada size de GNU/Linux tabanlı bir işletim sistemi kullanmanızı tavsiye edebilirim. Tabi özellikle ticari olmayan işletim sistemi seçerseniz, açık kaynak kodlu ve istediğiniz gibi konfigüre edebileceğinizi ve daha yaratıcı düşünme olanağına sahip olacağınızı garanti ederim. Başta öğrenmesi biraz zor ve belki yorucu gelebilir ama bir süre sonra kendinizi daha konforlu ve ne yaptığınızın farkına varır bulacaksınız. Öncelikle sahip olmanız gereken "nasm" ve "binutils" paketlerini sisteminize yüklemeniz gerekebilir. Nasm'ı ve GNU Linker'ı kullanacağız nitekim. Daha sonra basit bir editör yardımıyla şu aşağıdaki kodları "hello_world.asm" şeklinde bir dosya oluşturup ona kaydedebilirsiniz:

```asm
SECTION .data
msg     db      'Hello world', 0Ah     ; "Hello world" bir dizi byte datası olarak tanımlıyoruz. "0ah" ise "\n"
 
SECTION .text
global  _start		

; entry point (giriş noktası)
_start:
    mov     edx, 12     ; terminale yazılacak toplam bayt sayısı (line feed yani "\n" karakteri de dahil olarak)
    mov     ecx, msg    ; bu msg data-sının memorydeki adresini ecx register-ına taşıyorum
    mov     ebx, 1      ; ebx register-ı ile SYS_WRITE sistem çağrısının file descriptor-ını STDOUT olarak belirtiyorum
    mov     eax, 4      ; eax register-ı ile linux için SYS_WRITE sistem çağrısının numarasını belirtiyorum
    int     80h		; kernel-ı çağırıyorum ve öncesindeki sistem çağrımın gerçekleştirilmesini sağlıyorum.
```

Daha sonra "build.sh" isminde bir dosya aynı dizinde oluşturarak içerisine bunu yazın.

```bash
#!/usr/bin/bash
nasm -f elf "$1.asm" && ld -m elf_i386 "$1.o" -o $1 && ./$1 && rm "$1.o" $1
```

Burada özetle "nasm" ile elf linklenebilir formatına assembly kodumuzu derliyor ardından da "ld" yani GNU Linker ile elf dosyasını linkleyerek executable bir dosya oluşturuyor, nihayetinde de bunu çalıştırıp, oluşturulan dosyaları da en sonunda siliyoruz. Böylelikle tekrar tekrar uzun uzuna yamamız gerekmeyecek. "build.sh" dosyasına çalışma izni verdikten sonra "chmod + build.sh" yapmanız gereken şey "hello_world.asm" derlemek için şu:

```bash
./build.sh hello_world
```

Bunu terminalde çalıştırdığınızda konsola "Hello world" yazdığını görmeniz gerekir. Şimdi adım adım kodumuzu inceleyerek syscall-dan da bahsederek bu yazıyı bitirelim.

Öncelikle dikkat ederseniz iki farklı section olarak kodu ayırdık ve öncesinde de söylediğim gibi değişkenimizi ".data" section-ına yazdık.

```asm
msg db 'Hello world!', 0Ah
```

Burada "msg" değişken ismi olurken "db" hangi boyut ile tanımlanacak bunu belirtmiş olduk. "db" yani bayt, yani toplamda her karakteri 8-bitten oluşacak bir string ifadeyi dizi olarak tanımlamış olduk aslında. Yani 1 baytlık bir veri değil nihayetinde "Hello world". Toplamda boşluk karakteri de dahil (0x20) 11 bayttan oluşuyor ve virgülden sonra 0ah (0xA) yani "\\n" karakteri de eklenerek 12 baytlık char\[12\] tipinde bir değişken tanımlamış olduk diyebiliriz. Ancak burada özellikle 12 tane olduğunu göstermedik. Nasm bu konuda oldukça akıllı olduğu için bu string'i memory-e "H" den başlayarak "\\n" e kadar yerleştirecek. "msg" ismindeki değişken de aslında artık "H" nin bulunduğu memory-deki adresi tutacak. Dolayısıla (msg + n) ile sanki C'de pointer aritmetiği yapar gibi istediğimiz karaktere gidebiliriz. Burası biraz kafanızı karıştışmış olabilir. Bir de şöyle bakın:

![image.png](:/39ebab8e16af4953aeb750c241c6c6ab)

Devam edersek 0ah karakteri aslında hexadecimal kullanımın değişik bir varyasyonu Nasm'da. Bu karakter aşina olduğumuz hexadecimal şekilde 0xA olarak gösterilebilir. 0ah ile sondaki "h" bu karakterin hexadecimal olduğunu gösteriyor sadece. "0ah" yerine "0xa" veya "0xA" da yazabiliriz. Hatta bunu kendiniz dosyada değiştirip tekrar derlerseniz herhangi bir şeyin değişmediğini göreceksiniz. Peki bunu neden "Hello world\\n" şeklinde değil de "Hello world", 0ah şeklinde gösterdik? Bunun sebebi Nasm'ın yani Assembler-ın stringleri formatlıyor olmamasıdır. Yani high-level dillerde genelde bu compiler tarafında yorumlanarak bunun bir "escape sequence" olduğu anlaşılır. Ancak burada biz virgül yardımıyla farklı karakterleri ayrıca string'den bağımsız olarak sonuna ASCII değerine bakarak byte olarak (ASCII karakterlerinin hepsi 1 bayt zaten) koyabiliriz. Yani "\\n" ifadesi eğer ki "Hello world\\n" bu şekilde yazılmış olsaydı "\\" ve "\\n" sanki "H" veya "w" gibi karakterlermişcesine memory-e yerleştirilecek ilen 0ah ile biz doğrudan "line feed" yani ASCII karşılığı 10 olan karakteri de eklemiş olduk. 

"section .text" kısmına gelirsek, burası daha önce de söylediğimiz gibi assembly kodlarımızın yer alacağı bölümdü. Ancak burada şuana kadar bahsetmediğim global \_start gibi bir direktif bulunuyor. "global" direktifi Nasm spesifik bir direktif olmakla beraber farklı assembler-ların sentakslarında da buna muadil direktifler bulunur. Amacı linklenirken belirlenen label-ın (burada \_start) sembol olarak dikkate alınması ve bu noktadan programın çalışmaya başlmasının gerekeceğini bildiriyor. Yine "\_start:" da dediğimiz gibi bir label olmakla beraber programın başlayacağı yeri temsil ediyor diyebiliriz. Sistem seviyesinde çalışan programlama dillerindeki "main" fonksiyonu gibi düşünebilirsiniz. "\_start" linkerların varsayılan olarak aradığı bir adres yani eğer ki "global \_nostart" yazıp "\_start" ı da "\_nostart" yazsaydınız linker programı linklerken sembolü sembol tablosuna eklese de başlangıç noktası olarak "\_start" ı bulmaya çalışacaktı. Bunun da tabiki linker-a bir flag yardımıyla bildirilmesi durumunda varsayılan olarak "_start" alınma durumunu değiştirmek mümkün olabilmektedir:

```bash
ld -e _nostart -o out a.o
```

Aslında tüm programımız özetle şu kısımdan ibaret:

```asm
; entry point (giriş noktası)
_start:
    mov     edx, 12     ; terminale yazılacak toplam bayt sayısı (line feed yani "\n" karakteri de dahil olarak)
    mov     ecx, msg    ; bu msg data-sının memorydeki adresini ecx register-ına taşıyorum
    mov     ebx, 1      ; ebx register-ı ile SYS_WRITE sistem çağrısının file descriptor-ını STDOUT olarak belirtiyorum
    mov     eax, 4      ; eax register-ı ile linux için SYS_WRITE sistem çağrısının numarasını belirtiyorum
    int     80h		; kernel-ı çağırıyorum ve öncesindeki sistem çağrımın gerçekleştirilmesini sağlıyorum.
```

Burada daha önceki yazılarda bahsettiğim instruction ve opcode kavramlarına geri dönelim. Ayrıca sistem çağrılarından da bahsedeceğim. Burada size şuan için tanıdık gelmesi gereken tek şey olan register-ları farketmişsinizdir. Burada her satır bir instuction-ındır. Yani "mov edx, 12" bir instruction-dır. Ancak dikkat ederseniz başta kullanılan keyword-lar, ki bunlara opcode diyeceğiz (mov, int, ...), yapılacak işlemin/operasyonun adını taşımaktadır. "mov" aslında "movE" kelimesinin kısaltmasıdır. Yani data transfer etme ilgili bir opcode-dan bahsediyoruz. Bunu yaparken de iki tane şeye ihtiyacı var bir instruction oluşturabilmesi için. Bunlardan ilki hedef (destination) ikincisi de kaynak (source). Hedefler veya kaynaklar birer register olabilirler. Ancak sadece kaynak bir değer (immediate value) olabilir. Bu özelliği dikkate almak mecburiyetindeyiz. Yoksa assembler derlerken hata verecektir. Diğer taraftan "int" diye bir opcode görüyoruz en son satırda. Bu ise "intERRUPT" yani kesme yapmaktan gelen bir isimlendirmeye sahip. Birçok interrupt kodu bulunur. Bunlar real mode ve protected modda değişmektedir. Ancak karşığındaki hexadecimal sayılar faraza sayılar değil. "0x80" Linux kernel-ı için, syscall çağrısı yaparken kullanılan ve kernel-ı çağıran bir interrupt-tır. Bunu interrupt yapıldığında ve kernel çağırdığında, kernel öncesinde belirlenmiş olan standartlarda registerlar-a bakarak hangi sistem çağrısını yaptığımızı, hangi parametreleri sağladığımızı alır ve sistem çağrısını gerçekleştirir. Bir nevi fonksiyon çağırarak bu fonksiyona parametreleri registerlar ile veriyormuşuz gibi düşünebilirsiniz. Standart dedim ki aslında bunlar kernel geliştirilirken her sistem çağrısı için önceden belirlenmiş. Yani hangi çağrıyı yaparsanız hangi register-a hangi değerleri vermeniz gerektiği belirlidir. Bu sistem çağrılarının aynısını C standart kütüphanesi de kullanır. Tabi o biraz daha wrapleyerek kullanır ama temelde o da bizim yaptığımızı C ile yapar. 

Dolayısıyla "mov eax, 4" yani "mov eax, 0x4" ile yapacağımız sistem çağrısının hangisi olduğunu belirtiyoruz. "0x4" SYS_WRITE sistem çağrısının kodudur. Bu sistem çağrılarının hepsini görmek için [şuraya](https://chromium.googlesource.com/chromiumos/docs/+/master/constants/syscalls.md#x86-32_bit) bakabilirsiniz. Tablo şeklinde listelenmiş zaten C standart kütüphanesinden veya Unix sistem çağrılarından aşina olabileceğiniz Linux sistem çağrıları bu şekildedir. Bizi şuan için ilgilendiren "write" 5 satırdaki 4 numaralı fonksiyondur:

![7d26385aec0428698dea6ef35060be68.png](:/3b0409248c304204ac018d323e95c0a6)

Görüldüğü gibi bu sistem çağrısının yapılması için "eax" register-ında sistem çağrısının numarası, ebx register-ı ile birinci argümanı (write için file descriptor olarak STDOUT yani 1'i, bunun için de [şuraya](https://en.wikipedia.org/wiki/File_descriptor) bakabilirsiniz), ecx register-ı ile ikinci argümanı (buffer'ın yani konsola yazılacak string ifadenin adresini ki bizim durumda zaten msg "H"yi yani string ifademizin başlangıcındaki adresini tutuyordu) ve edx register-ı ile de string ifadenin başlangıç adresinden kaç karakter kadarının konsola yazılacağını belirtmek için bayt cinsinden bir boyut bilgisini argüman olarak vermemiz gerekiyor. Dolayısıyla tekrar baktığımız zaman **edx** register-ına 12 yani string ifademizin boyutunu "\\n" karakteri de dahil olarak argüman olarak **mov** opcode-uyla taşıdık (transfer ettik). Ardından **ecx** register-ına **msg** string datamızın başlangıç adresini verdik ki aslında varsayılan olarak assembly-de tüm değişkenler bir adresi temsil ediyor (tabi deferans da edebiliriz ilerde bahsedeceğiz). Sonra da **ebx** register-ı ile SYS_WRITE sistem çağrısı için gerekli olan file descriptor-ı (Linux'ta her şey bir dosyadır ve o şekilde muamele edilir; bunun için de file descriptoprlar kullanılır) argüman olarak verdik. Ve nihayet **eax** register-ı ile de hangi sistem çağrısını yapacaksak, ki biz SYS_WRITE sistem çağrısını yapacağız, bunu da sağladıktan sonra, kernel-ı "**int 0x80"** interrupt-ı ile çağırdık ve konsolumuza kernel "Hello world"-u yazdırmış oldu.

Basit şekilde **"Hello world" **programımız yazdık ve nasıl çalıştığını da anladık. Umuyorum ki... Bir sonraki yazıda "Addressing Modes" a bakacağız ve Assembly öğrenmek için bir adım daha ileriye gideceğiz.