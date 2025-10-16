BASLA SİSTEM

    BASLA KULLANICI_GİRİŞİ
        EĞER kullanıcı_oturumu_yoksa İSE
            GİRİŞ_EKRANI_GÖSTER()
            EĞER kullanıcı_giriş_yaparsa İSE
                OTURUM_OLUŞTUR()
            DEĞİLSE
                MİSAFİR_SEPETİ_OLUŞTUR()
            BİTİR EĞER
        DEĞİLSE
            VAR_OLAN_OTURUMU_KULLAN()
        BİTİR EĞER
    BİTİR KULLANICI_GİRİŞİ


    BASLA ÜRÜN_EKLEME
        ÜRÜN_ID, ADET, VARYANT = KULLANICIDAN_AL()
        
        BASLA STOK_KONTROLÜ
            STOK = STOK_SORGULA(ÜRÜN_ID)
            EĞER STOK >= ADET İSE
                SEPETE_EKLE(ÜRÜN_ID, ADET, VARYANT)
            DEĞİLSE
                UYARI_GÖSTER("Yetersiz stok. Maksimum: " + STOK)
                ADET = STOK
                SEPETE_EKLE(ÜRÜN_ID, ADET, VARYANT)
            BİTİR EĞER
        BİTİR STOK_KONTROLÜ
    BİTİR ÜRÜN_EKLEME


    BASLA SEPET_GÖRÜNTÜLEME
        SEPETİ_GÖSTER()
        DÖNGÜ kullanıcı_sepeti_değiştirene_kadar
            KULLANICI_EYLEM = BEKLE()
            EĞER KULLANICI_EYLEM == "ÜRÜN_SİL" İSE
                SEPETTEN_SİL(ÜRÜN_ID)
            EĞER KULLANICI_EYLEM == "ADET_GÜNCELLE" İSE
                ADET_GÜNCELLE(ÜRÜN_ID, YENİ_ADET)
            EĞER KULLANICI_EYLEM == "DEVAM_ET" İSE
                ÇIK_DÖNGÜ
            BİTİR EĞER
        BİTİR DÖNGÜ
    BİTİR SEPET_GÖRÜNTÜLEME


    BASLA İNDİRİM_KODU_KONTROLÜ
        EĞER kullanıcı_kodu_girdiyse İSE
            KOD = KULLANICIDAN_AL()
            EĞER KOD_GECERLI(KOD) İSE
                İNDİRİM_TUTARI = HESAPLA_İNDİRİM(KOD)
                SEPET_TOPLAM -= İNDİRİM_TUTARI
                BİLGİ_GÖSTER("İndirim uygulandı: " + İNDİRİM_TUTARI)
            DEĞİLSE
                UYARI_GÖSTER("Geçersiz indirim kodu")
            BİTİR EĞER
        DEĞİLSE
            DEVAM_ET
        BİTİR EĞER
    BİTİR İNDİRİM_KODU_KONTROLÜ


    BASLA KARGO_HESAPLAMA
        ADRES = KULLANICIDAN_ADRES_AL()
        KARGO_UCRETİ = HESAPLA_KARGO(ADRES, SEPET_TOPLAM)
        
        EĞER SEPET_TOPLAM >= ÜCRETSİZ_KARGO_LİMİTİ İSE
            KARGO_UCRETİ = 0
            BİLGİ_GÖSTER("Ücretsiz kargo kazandınız!")
        BİTİR EĞER

        GENEL_TOPLAM = SEPET_TOPLAM + KARGO_UCRETİ
        TUTAR_GÖSTER(GENEL_TOPLAM)
    BİTİR KARGO_HESAPLAMA


    BASLA ÖDEME_AŞAMASI
        SİPARİŞ_ÖZETİNİ_GÖSTER(SEPET, GENEL_TOPLAM)
        ÖDEME_YÖNTEMİ = KULLANICIDAN_SEÇİM_AL()

        EĞER ÖDEME_YÖNTEMİ == "KART" İSE
            KART_BİLGİLERİ = AL_KART_BİLGİLERİ()
            TOKEN = TOKENIZE(KART_BİLGİLERİ)
            ÖDEME_SONUCU = ÖDEME_SAĞLAYICIYA_GÖNDER(TOKEN, GENEL_TOPLAM)
        EĞER ÖDEME_YÖNTEMİ == "HAVALE" İSE
            ÖDEME_SONUCU = BEKLE_HAVALE_ONAYI()
        EĞER ÖDEME_YÖNTEMİ == "KAPIDA" İSE
            ÖDEME_SONUCU = "ONAYLANDI"
        BİTİR EĞER

        EĞER ÖDEME_SONUCU == "ONAYLANDI" İSE
            BASLA SİPARİŞ_OLUŞTURMA
                SİPARİŞ_ID = YENİ_SİPARİŞ_OLUŞTUR(SEPET, ADRES, GENEL_TOPLAM)
                STOK_GÜNCELLE(SEPET)
                SEPET_TEMİZLE()
                EPOSTA_GÖNDER("Siparişiniz alındı", SİPARİŞ_ID)
                MESAJ_GÖSTER("Siparişiniz başarıyla tamamlandı.")
            BİTİR SİPARİŞ_OLUŞTURMA
        DEĞİLSE
            UYARI_GÖSTER("Ödeme başarısız! Lütfen tekrar deneyin.")
        BİTİR EĞER
    BİTİR ÖDEME_AŞAMASI


    BASLA SİPARİŞ_YÖNETİMİ
        DÖNGÜ kullanıcı_siparişi_takip_ederken
            DURUM = SİPARİŞ_DURUMU_SORGULA(SİPARİŞ_ID)
            DURUM_GÖSTER(DURUM)
            EĞER DURUM == "TESLİM_EDİLDİ" İSE
                ÇIK_DÖNGÜ
            BİTİR EĞER
        BİTİR DÖNGÜ
    BİTİR SİPARİŞ_YÖNETİMİ

BİTİR SİSTEM
