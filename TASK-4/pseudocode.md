FONKSIYON DersKayitAkisi(ogrenci, tumDersler):

    GIRIS_BASARILI <- GirisYap(ogrenci.id, ogrenci.sifre)

    EĞER GIRIS_BASARILI = FALSE İSE
        Yazdir("Giriş başarısız. Bilgileri kontrol edin.")
        CIK
    İSE
        Yazdir("Giriş başarılı. Ana panele yönlendiriliyor...")
    SON

    // Ana döngü: Öğrenci seçim yapar -> Sistem kontrol eder -> Danışman onayı -> (gerekirse) düzelt ve tekrar
    KAYIT_TAMAMLANDI <- FALSE

    TEKRAR ET
        // 1) Ders listesini hazırla
        uygunDersler <- FiltreleUygunDersler(tumDersler, ogrenci.donem)

        // 2) Öğrenci dersleri seçer (UI tarafında). Burada seçimi dışarıdan alındı varsayalım
        secimler <- AlOgrenciSecimleri(uygunDersler)

        // 3) Sistem Kontrolleri
        HATALAR <- BOS_LISTE()
        UYARILAR <- BOS_LISTE()

        // 3.1) Önkoşul kontrolü
        TOPLAM_KREDI <- 0
        i <- 0
        IKILI_CAKISMA_VAR <- FALSE

        // Kontenjan / Önkoşul / Kredi kümülatif hesap
        FOR i 0..UZUNLUK(secimler)-1
            ders <- DersBul(tumDersler, secimler[i])

            // (A) Kontenjan
            EĞER ders.kontenjanDoluluk >= ders.kontenjanMax İSE
                HATALAR.Ekle("Kontenjan dolu: " + ders.kod)
            SON

            // (B) Önkoşul
            EĞER UZUNLUK(ders.onKosullar) > 0 İSE
                j <- 0
                TUM_ONKOSULLAR_TAMAM <- TRUE

                FOR j 0..UZUNLUK(ders.onKosullar)-1
                    onk <- ders.onKosullar[j]
                    EĞER TamamladiMi(ogrenci, onk) = FALSE İSE
                        TUM_ONKOSULLAR_TAMAM <- FALSE
                        HATALAR.Ekle("Önkoşul sağlanmadı: " + ders.kod + " için " + onk)
                    SON
                SON

                // Ek: eş-zamanlı alma (coreq) desteklenecekse burada ayrı kural yazılır
            SON

            // (C) Kredi biriktirme
            TOPLAM_KREDI <- TOPLAM_KREDI + ders.kredi
        SON

        // (D) Kredi limitleri
        EĞER TOPLAM_KREDI > ogrenci.maxKredi İSE
            HATALAR.Ekle("Kredi limiti aşıldı. Toplam: " + TOPLAM_KREDI + ", Maks: " + ogrenci.maxKredi)
        İSE EĞER TOPLAM_KREDI < ogrenci.minKredi İSE
            UYARILAR.Ekle("Minimum dönem kredisi altında. Toplam: " + TOPLAM_KREDI + ", Minimum: " + ogrenci.minKredi)
        SON

        // (E) Zaman çakışması kontrolü (çift döngü ile tüm çiftleri karşılaştır)
        x <- 0
        WHILE x < UZUNLUK(secimler)
            dersX <- DersBul(tumDersler, secimler[x])

            y <- x + 1
            WHILE y < UZUNLUK(secimler)
                dersY <- DersBul(tumDersler, secimler[y])

                // Her iki dersin tüm slotlarını karşılaştır
                sx <- 0
                EĞER IKILI_CAKISMA_VAR = FALSE İSE
                    WHILE sx < UZUNLUK(dersX.slotlar)
                        sy <- 0
                        WHILE sy < UZUNLUK(dersY.slotlar)
                            EĞER dersX.slotlar[sx].gun = dersY.slotlar[sy].gun İSE
                                EĞER KesisirMi(dersX.slotlar[sx], dersY.slotlar[sy]) = TRUE İSE
                                    HATALAR.Ekle("Zaman çakışması: " + dersX.kod + " ↔ " + dersY.kod)
                                    IKILI_CAKISMA_VAR <- TRUE
                                SON
                            SON
                            sy <- sy + 1
                        END WHILE
                        sx <- sx + 1
                    END WHILE
                SON

                y <- y + 1
            END WHILE
            x <- x + 1
        END WHILE

        // 4) Hata/uyarı değerlendirmesi
        EĞER UZUNLUK(HATALAR) > 0 İSE
            Yazdir("Hatalar bulundu, kayıt gönderilemez:")
            YazdirListesi(HATALAR)

            // Öğrenci düzeltme yapar: hatalı dersleri çıkar, alternatif seçer vs.
            secimler <- DuzeltSecimlerUI(uygunDersler, secimler, HATALAR)

            // Düzeltme sonrası başa dön (kontrolleri tekrar et)
            DEVAM_ET // (TEKRAR ET bloğunun başına döner)
        SON

        // 5) Uyarılar varsa öğrenciye onay sor
        EĞER UZUNLUK(UYARILAR) > 0 İSE
            Yazdir("Uyarılar var:")
            YazdirListesi(UYARILAR)
            KULLANICI_ONAYI <- AlKullaniciOnayi("Uyarılara rağmen danışmana gönderilsin mi?")
            EĞER KULLANICI_ONAYI = FALSE İSE
                // Öğrenci düzenlemek istiyor
                secimler <- DuzeltSecimlerUI(uygunDersler, secimler, UYARILAR)
                DEVAM_ET
            SON
        SON

        // 6) Kaydet ve Danışman Onayı
        KaydetGecici(ogrenci, secimler, TOPLAM_KREDI)
        onaySonucu <- DanismanOnayiIste(ogrenci, secimler)

        EĞER onaySonucu = "ONAY" İSE
            KaydiKesinlestir(ogrenci, secimler)
            Yazdir("Ders kaydı kesinleşti. Toplam kredi: " + TOPLAM_KREDI)
            KAYIT_TAMAMLANDI <- TRUE

        İSE EĞER onaySonucu = "RED" İSE
            Yazdir("Danışman reddetti. Açıklamaları inceleyin ve düzeltin.")
            aciklama <- GetDanismanAciklamasi(ogrenci)
            Yazdir(aciklama)
            // Öğrenci düzeltir ve döngü tekrarlar
            secimler <- DuzeltSecimlerUI(uygunDersler, secimler, [aciklama])

        İSE EĞER onaySonucu = "DUZELTME_ISTEGI" İSE
            Yazdir("Danışman düzeltme istedi. Lütfen seçimleri güncelleyin.")
            detay <- GetDanismanDetayi(ogrenci)
            Yazdir(detay)
            secimler <- DuzeltSecimlerUI(uygunDersler, secimler, [detay])
        SON

    // Danışman onayı çıkana kadar ya da kullanıcı vazgeçene kadar döner
    SUREC_SONA_ERDI MI <- KayitDonemSuresiBittiMi()

    UNTIL KAYIT_TAMAMLANDI = TRUE VEYA SUREC_SONA_ERDI MI = TRUE

    EĞER KAYIT_TAMAMLANDI = FALSE İSE
        Yazdir("Kayıt dönemi sona erdi veya işlem iptal edildi. Kesin kayıt yapılamadı.")
    SON

SON FONKSIYON
