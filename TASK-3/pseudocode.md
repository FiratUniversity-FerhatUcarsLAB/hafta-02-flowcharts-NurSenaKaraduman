#modul1 pseudocode'u                      HASTANE RANDEVU SİSTEMİ
BAŞLA

// 1. Kimlik Doğrulama
EKRANA "T.C. Kimlik Numaranızı giriniz:" YAZ
TC = GİRİŞ_AL()
EKRANA "Şifrenizi giriniz:" YAZ
SIFRE = GİRİŞ_AL()

EĞER KimlikDogrula(TC, SIFRE) = DOĞRU İSE
    EKRANA "Giriş başarılı. Hoş geldiniz!" YAZ
AKSİ HALDE
    EKRANA "Kimlik doğrulama başarısız. Tekrar deneyin." YAZ
    DUR

// 2. Poliklinik Seçimi
POLIKLINIK_LISTESI = PoliklinikleriGetir()
EKRANA "Lütfen bir poliklinik seçiniz:" YAZ
PoliklinikListesiniGöster(POLIKLINIK_LISTESI)

SECILEN_POLIKLINIK = GİRİŞ_AL()

// 3. Doktor Listesi Görüntüleme
DOKTOR_LISTESI = DoktorlariGetir(SECILEN_POLIKLINIK)
EKRANA "Bu poliklinikteki doktorlar:" YAZ
DoktorListesiniGöster(DOKTOR_LISTESI)

EKRANA "Randevu almak istediğiniz doktoru seçiniz:" YAZ
SECILEN_DOKTOR = GİRİŞ_AL()

// 4. Uygun Saatleri Gösterme
UYGUN_SAATLER = UygunSaatleriGetir(SECILEN_DOKTOR)
EĞER UYGUN_SAATLER BOŞSA
    EKRANA "Bu doktorda şu anda uygun saat yok. Lütfen başka tarih seçiniz." YAZ
    DUR
AKSİ HALDE
    EKRANA "Uygun saatler:" YAZ
    SaatleriGöster(UYGUN_SAATLER)

// 5. Randevu Saati Seçimi
EKRANA "Randevu almak istediğiniz saati seçiniz:" YAZ
SECILEN_SAAT = GİRİŞ_AL()

// 6. Randevu Onaylama
EKRANA "Randevunuzu onaylıyor musunuz? (E/H)" YAZ
ONAY = GİRİŞ_AL()

EĞER ONAY = "E" İSE
    RandevuKaydet(TC, SECILEN_POLIKLINIK, SECILEN_DOKTOR, SECILEN_SAAT)
    SMS_Gonder(TC, "Randevunuz başarıyla oluşturuldu: " + SECILEN_SAAT)
    EKRANA "Randevunuz onaylandı ve SMS gönderildi." YAZ
AKSİ HALDE
    EKRANA "Randevu iptal edildi." YAZ
SON

BİTİR



#modul2 pseudocode'U              TAHLİL SONUCU GÖRÜNTÜLEME SİSTEM

BAŞLA

