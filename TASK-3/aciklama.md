

Bu çalışmada, hastane bilgi sistemi için iki modül içeren bir akış diyagramı (flowchart) tasarlandı.
Diyagramın en üstünde ANA MENÜ yer almakta ve kullanıcıya iki seçenek sunmaktadır:
Randevu Sistemi (Modül 1): Kimlik doğrulama, poliklinik ve doktor seçimi, uygun saatleri gösterme, randevu onaylama ve SMS gönderme adımlarını içerir.
Tahlil Sonucu Görüntüleme Sistemi (Modül 2): Kimlik doğrulama, tahlil kaydı kontrolü, sonuç durumu kontrolü, hazırsa sonucu görüntüleme ve PDF indirme, değilse bekleme mesajı gösterme işlemlerini kapsar.
Bu iki modül, Graphviz DOT diliyle oluşturulan subgraph yapılarıyla görsel olarak ayrılmıştır ve kullanıcı seçiminden sonra ilgili modül akışına yönlendirilir.
