#🔑 IDOR: Dijital Dünyanın "Yanlış Kapı" Anahtarı
Hayal et ki bir oteldesin. Elinde 101 numaralı odanın kartı var. Kapıya gidiyorsun, kartı okutuyorsun ve içeridesin. Buraya kadar her şey normal. Ancak, koridorda yürürken 102 numaralı odanın kapısına geliyorsun ve kendi kartını orada deniyorsun. Tak! Kapı açılıyor.

İşte IDOR (Insecure Direct Object Reference) tam olarak budur: Sistem senin kim olduğunu biliyor (otel misafirisin) ama hangi odaya girme yetkin olduğunu kontrol etmeyi unutuyor.

🛠️ Senaryo: Basit Bir Manipülasyon
Sistem, senin verilerine ulaşmak için bir "nesne referansı" (ID) kullanır. Eğer bu referans korumasızsa, sadece bir rakamı değiştirerek başkasının dünyasına sızabilirsin.

Senin Verin: https://banka.com/islem?id=1234 (Kendi dekontunu görüyorsun.)

Saldırı Anı: URL'deki 1234 kısmını 1235 yapıyorsun.

Sonuç: Eğer karşına tanımadığın birinin hesap dökümü çıkıyorsa, tebrikler (veya eyvah!); bir IDOR buldun.

🛠️ IDOR Zafiyeti Mutfağı: Nasıl Oluşur?
IDOR bir yazılım hatasından ziyade, bir mantık hatasıdır. Temelinde yatan sebep, uygulamanın kullanıcıya sunduğu "anahtar" ile arka taraftaki "kilit" arasındaki ilişkiyi sorgulamamasıdır.

1. Tahmin Edilebilir Kimlikler (ID)
Çoğu web uygulaması, veritabanındaki verileri yönetmek için Artan Tamsayı (Auto-Increment Integer) kullanır.

Örnek: Sizin profil sayfanız site.com/user/105 ise, bir sonraki kullanıcının 106 olduğunu bilmek dahi gerekmez; tahmin etmek çocuk oyuncağıdır.

2. Doğrudan Referansın İfşası
Bu kimlik belirleyicileri (ID'ler) gizli tutulmak yerine;

URL parametrelerinde: ?fatura_id=2024

HTTP isteklerinde (Body/Header): {"account_id": "550"}

Çerezlerde (Cookies): user_session=12
şeklinde dış dünyaya, yani saldırganın görüş alanına sunulur.

3. Eksik Yetki Kontrolü (Zurnanın Zırt Dediği Yer)
Saldırgan 105 olan ID değerini 106 ile değiştirdiğinde, sunucu tarafındaki kod şu hatayı yapar:

"106 numaralı veriyi getirmemi istedin, al işte burada!" Oysa yapması gereken şudur:
"106 numaralı veriyi istedin ama sen 105 numaralı kullanıcısın. Bu veriyi görmeye yetkin yok!"

🛠️ IDOR Sahada: En Yaygın Saldırı Senaryoları
IDOR sadece profil bilgilerini röntgenlemek değildir; bazen tüm sistemin anahtarını saldırgana teslim etmektir. İşte en tehlikeli iki senaryo:

1. Kritik İşlemlerde IDOR: Şifre Değiştirme 🔑
En korkutucu senaryolardan biridir. Uygulama, hangi kullanıcının şifresinin değişeceğine karar verirken oturum bilgilerine (Session) bakmak yerine URL'deki parametreye güvenirse felaket kaçınılmaz olur.

Normal İstek: https://site.com/sifre_degistir.php?userid=1701

Saldırı: Saldırgan userid değerini 1702 (kurbanın ID'si) yapar.

Sonuç: Saldırgan kendi belirlediği yeni şifre ile başkasının hesabına tam erişim sağlar.

2. Dosya Sistemine Sızma: Path Traversal ile Dans 📂
Bazı IDOR türleri, veritabanı ID'leri yerine doğrudan dosya isimlerini hedef alır. Eğer uygulama dosya yolunu doğrulamıyorsa, saldırgan "dizin atlama" (Directory Traversal) tekniklerini IDOR ile birleştirir.

Normal İstek: https://site.com/indir.php?dosya=fatura_01.pdf

Saldırı: https://site.com/indir.php?dosya=../../../../etc/passwd

Sonuç: Saldırgan, sunucudaki hassas sistem dosyalarını (kullanıcı listeleri, yapılandırma dosyaları vb.) bilgisayarına indirir.

IDOR Zafiyetlerini Tespit Etme
IDOR, mantıksal bir zafiyet olduğu için her zaman güvenlik tarama araçlarıyla tespit edilemeyebilir. Bu nedenle manuel sızma testi ve güvenlik odaklı kod incelemeleri gerekir.

🔍 İz Peşinde: IDOR Zafiyeti Nasıl Tespit Edilir?
IDOR, otomatik araçların (Scanner) en çok zorlandığı zafiyettir; çünkü bu bir mantık hatasıdır. Bir robot, 101 numaralı odanın size mi yoksa başkasına mı ait olduğunu her zaman anlayamaz. Bu yüzden manuel testler hayati önem taşır.

1. Parametre Avcılığı (Manipülasyon)
Saldırgan gibi düşünün: URL veya veri paketindeki (POST/JSON) rakamları kurcalayın.

Değiştir ve Gör: id=1001 değerini 1002 yapın. Eğer sayfa hata vermeden başkasının verisini getiriyorsa bingo!

JSON & XML İnceleme: Sadece URL'ye bakmayın; arka planda giden API isteklerini (Burp Suite gibi araçlarla) yakalayıp içindeki transaction_id gibi değerleri manipüle edin.

2. Yetki Çaprazlaması (RBAC Testleri)
Sisteme iki farklı kullanıcıyla (Kullanıcı A ve Kullanıcı B) giriş yapın.

Kullanıcı A'ya ait bir linki kopyalayın.

Bu linki Kullanıcı B'nin tarayıcısında açmayı deneyin.

Erişim sağlanıyorsa, yetkilendirme katmanı çökmüş demektir.

3. Araçların Gücü: Burp Suite & ZAP
Otomatik tarayıcılar tek başına yetmez ama yardımcı olur:

Fuzzing: Belirli bir parametreye (örneğin fileId) binlerce farklı rakam göndererek hangilerinin "200 OK" (başarılı) döndüğünü test edin.

AuthMatrix / Autorize: Burp Suite eklentileriyle yetki testlerini otomatize edin.

4. Kod ve Log Analizi (Beyaz Kutu Testi)
Eğer sistemin mutfağına (kodlarına) erişiminiz varsa:

Kod İnceleme: Veritabanı sorgularında WHERE id = ? ifadesinin yanında AND user_id = ? kontrolü var mı? Yoksa IDOR kapıdadır.

Loglar: Kısa sürede binlerce farklı ID deneyen bir IP adresi varsa, o bir IDOR taramasıdır.

Örnek Uygulamalar
Siteye gittiğimizde bu ekran ile karşılaşıyoruz.
<img width="913" height="659" alt="Ekran görüntüsü 2026-03-09 021554" src="https://github.com/user-attachments/assets/78bc4cc4-9da1-4651-a827-27143a9efbbc" />

View butonuna tıklıyarak ne var ne yok bakıyoruz.

<img width="1919" height="1021" alt="Ekran görüntüsü 2026-03-09 022016" src="https://github.com/user-attachments/assets/74ca09a8-f8e3-4f57-8dbc-9decde94a1c6" />

Butona tıkladıktan sonra açılan sayfada URL'de bulunan invoice_id parametresi dikkatimizi çekiyor.

Bu değeri değiştirerek yetkimiz olmadan, farklı kullanıcıların faturalarına erişebilecek miyiz bunu test edelim. 



<img width="1918" height="964" alt="Ekran görüntüsü 2026-03-09 022004" src="https://github.com/user-attachments/assets/d049ba6a-24a1-4126-a245-69089ad4e511" />

Evet gördüğümüz gibi farklı birinin faturası karşımıza çıkıyor.

Burada tespit etmiş olduğumuz IDOR zafiyeti ile invoice_id parametresinin değerini manipüle ederek farklı kullanıcıların faturalarına ulaşabiliyoruz.

Örnek Uygulamalar 2 

<img width="1914" height="1025" alt="image" src="https://github.com/user-attachments/assets/ef0ca2d7-bee5-4170-8911-3d3af471715e" />

Sitemiz bu şekilde açılıyor.

Laboratuvarda bizden bilet satın almamız isteniyor ancak görüldüğü üzere hesabımızdaki para tutarı bilet almak için yeterli değil.

<img width="1917" height="924" alt="image" src="https://github.com/user-attachments/assets/e2caf6ab-fafb-4cb9-8a5a-7b3ea2a1fe1b" />

Bilet sayısı olarak 1 girip Buy butonuna bastığımızda yukarıdaki gibi bir hata ile karşılaşıyoruz.

Sayfayı biraz inceleyerek bir zafiyet olup olmadığını araştıralım.

<img width="1919" height="1013" alt="image" src="https://github.com/user-attachments/assets/dc96407e-e7cb-402e-a125-a23607f70567" />

Sayfanın kaynak koduna baktığımızda bilet tutarının form verisi olarak istemciden gönderildiğini görüyoruz. Bu da demek oluyorki biz istemci olarak bilet fiyatını istediğimiz gibi değiştirebiliri
Chrome DevTools'u açmak için sayfaya Sağ tıklayıp açılır menüden İncele sekmesini seçelim. 

<img width="827" height="551" alt="Ekran görüntüsü 2026-03-09 023614" src="https://github.com/user-attachments/assets/2ce9dc3b-1da2-4f37-bb75-05e4ee06b798" />

1. DevTools açtıktan sonra ilk olarak sol üst köşedeki Seçme butonu aktifleştirilmelidir.

2. Ardından bilet adedi girme alanını seçelim.

3. Sağ tarafta bulunan DOM'da hidden özniteliğine sahip ticket_money form verisini bulalım.

Şimdi bu form verisinin value alanında yazan 300 değerini üzerine çift tıklayarak düzenleyelim. Bakiyemiz 50$ olduğu için bu değeri 50'den düşük yapmamız gerekiyor.
<img width="1919" height="1028" alt="image" src="https://github.com/user-attachments/assets/a51cdf9d-0eda-41e5-91e8-ea584c8db9e8" />

ticket_money verisinin value alanını 5 yaptık, bilet sayısı olarak 3 yazıp ve ardından Buy butonuna tıklayalım.
Gördüğümüz üzere bilet satın almayı başardık.
