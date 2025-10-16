# ==============================
# AKILLI EV GÜVENLİK SİSTEMİ
# Ana Döngü (7/24)
# ==============================

BAŞLAT:
    yapılandırma ← YapılandırmayıYükle()                 # eşikler, modlar, tekrar deneme, hız sınırı
    sensörler ← SensörleriBaşlat(yapılandırma)           # hareket, kapı, pencere, kamera, duman, CO, su, sabotaj
    alarm ← AlarmDonanımınıBaşlat()                      # siren, ışık, kilit kontrolü
    bildirici ← BildiriciBaşlat(yapılandırma)            # SMS, push, e-posta, arama
    günlük ← GünlükleyiciBaşlat()
    sağlık ← SağlıkMonitörünüBaşlat()                    # pil, bağlantı, kalp atışı
    olayGeçmişi ← YeniKısaKaydırmaPenceresi(boyut=100)
    bildirimKuyruğu ← YeniEşzamanlıKuyruk()

    İşParçacığıBaşlat(BildirimGöndericiİP, bildirimKuyruğu, bildirici, yapılandırma, günlük)
    İşParçacığıBaşlat(GözetmenİP, sağlık, alarm, bildirimKuyruğu, yapılandırma, günlük)

# ------------------------------
# ANA DÖNGÜ
# ------------------------------
ANA_DÖNGÜ:
    SİSTEM_ÇALIŞIYOR iken:
        döngüBaşlangıcıZamanı ← Şimdi()

        # 1) Sensör Okuma (gerekirse paralel)
        hamOkumalar ← TümSensörleriOku(sensörler)        # [{id, tür, değer, zamanDamgası, durum}, ...]
        Günlükle(günlük, "sensör_okuma", hamOkumalar)

        # 2) Ön İşleme (gürültü temizleme, debounce)
        okumalar ← Önİşle(hamOkumalar, yapılandırma)

        # 3) Sensör Füzyonu (durum çıkarımı)
        birleşikDurum ← FüzyonDeğerlendir(okumalar)       # örn: hareket+kapı=olası izinsiz giriş

        # 4) Tehdit Algılama (kural + isteğe bağlı ML)
        tehdit ← TehditAlgıla(birleşikDurum, olayGeçmişi, yapılandırma)
        # tehdit = {seviye: 0..100, tür: "giriş"/"yangın"/"co"/"su"/"sabotaj", kanıt: [...]}

        # 5) Karar ve Tepki
        EĞER tehdit.seviye ≥ yapılandırma.alarmEşiği İSE:
            Günlükle(günlük, "tehdit_algılandı", tehdit)

            EĞER DeaktifVeyaUyumsuz(alarm, tehdit.tür) İSE:
                kip ← AlarmKipiSeç(tehdit.tür, yapılandırma)  # örn: yangında uzun siren + tüm ışıklar
                AlarmAktifleştir(alarm, kip)

            bildirim ← BildirimOluştur(tehdit, birleşikDurum, yapılandırma)

            EĞER BildirimKuyruğaAlınmalı(bildirim, bildirimKuyruğu, yapılandırma) İSE:
                KuyruğaEkle(bildirimKuyruğu, bildirim)

            olayGeçmişi.Ekle(tehdit)

            # Otomasyonlar: kapıları kilitle, ışıkları aç, kamerayı kayda başlat, akıllı kilit vb.
            OtomasyonlarıUygula(tehdit, alarm, sensörler, yapılandırma)

        DEĞİLSE:
            # Yanlış alarm kontrolü
            EĞER AlarmAktifMi(alarm) İSE:
                EĞER YanlışAlarmDoğrulandı(okumalar, olayGeçmişi, yapılandırma) İSE:
                    AlarmDevreDışıBırak(alarm)
                    sonOlay ← olayGeçmişi.Son()
                    KuyruğaEkle(bildirimKuyruğu, AlarmDüzeldiBildirimİnşaEt(sonOlay, yapılandırma))
                    Günlükle(günlük, "yanlış_alarm_temizlendi")
                DEĞİLSE:
                    Günlükle(günlük, "alarm_devam")

        # 6) Sağlık Kontrolü (periyodik)
        EĞER Şimdi() - sağlık.sonKontrolZamanı ≥ yapılandırma.sağlıkKontrolAralığı İSE:
            rapor ← SağlıkKontrolüYap(sağlık, sensörler, alarm, bildirici)
            Günlükle(günlük, "sağlık_raporu", rapor)
            EĞER KritikSorunVar(rapor) İSE:
                KuyruğaEkle(bildirimKuyruğu, SağlıkUyarısıOluştur(rapor, yapılandırma))

        # 7) Telemetri (opsiyonel)
        telemetri ← TelemetriHazırla(okumalar, birleşikDurum, tehdit)
        İzinliİseTelemetriGönder(telemetri, yapılandırma)

        # 8) Zamanlama
        SonrakiDöngüyeKadarUyku(döngüBaşlangıcıZamanı, yapılandırma.döngüAralığı)

# ------------------------------
# YARDIMCI FONKSİYONLAR
# ------------------------------
FONKSİYON TümSensörleriOku(sensörler):
    DÖN ParalelHaritala(sensörler, s → s.Oku())

FONKSİYON Önİşle(hamOkumalar, yapılandırma):
    # debounce, median filtre, eşik ön-uygulama, sensör kalibrasyonu
    DÖN FiltreMotoru(hamOkumalar, yapılandırma)

FONKSİYON FüzyonDeğerlendir(okumalar):
    # örn: (hareket∧kapıAçık)→izinsiz_giriş_kanıtı; (duman∧sıcaklık↑)→yangın_kanıtı
    DÖN FüzyonMotoru.Çıkar(okumalar)

FONKSİYON TehditAlgıla(birleşikDurum, geçmiş, yapılandırma):
    kuralPuanı ← KurallarıDeğerlendir(birleşikDurum, yapılandırma.kurallar)
    modelPuanı ← MLMevcutMu()? MLModeli.Tahmin(birleşikDurum) : 0
    toplam ← AğırlıklıBirlestir(kuralPuanı, modelPuanı, yapılandırma.ağırlıklar)
    tür ← TürÇıkar(birleşikDurum)
    kanıt ← Özetle(birleşikDurum)
    DÖN { seviye: toplam, tür: tür, kanıt: kanıt }

FONKSİYON BildirimKuyruğaAlınmalı(bildirim, kuyruk, yapılandırma):
    EĞER Yinelenen(bildirim, kuyruk, pencere=yapılandırma.tekilleştirmePenceresi) İSE DÖN YANLIŞ
    EĞER HızSınırıAşıldı(bildirim.alıcı, yapılandırma) İSE DÖN YANLIŞ
    DÖN DOĞRU

# ------------------------------
# İŞ PARÇACIKLARI
# ------------------------------
İŞPARÇACIĞI BildirimGöndericiİP(kuyruk, bildirici, yapılandırma, günlük):
    SİSTEM_ÇALIŞIYOR iken:
        bildirim ← KuyruktanÇek(kuyruk, zamanAşımı=yapılandırma.bildirimBekleme)
        EĞER bildirim = BOŞ İSE DEVAM
        DENE:
            sonuç ← bildirici.Gönder(bildirim)
            EĞER sonuç.başarılı:
                Günlükle(günlük, "bildirim_gönderildi", bildirim.id)
            DEĞİLSE:
                EĞER bildirim.tekrar < yapılandırma.maksTekrar:
                    bildirim.tekrar ← bildirim.tekrar + 1
                    bildirim.sonrakiZaman ← Şimdi() + ÜstelGecikme(bildirim.tekrar)
                    KuyruğaTekrarEkle(kuyruk, bildirim)
                DEĞİLSE:
                    Günlükle(günlük, "bildirim_başarısız", bildirim.id)
                    AlternatifKanalaYönlendir(bildirim, bildirici, yapılandırma)
        YAKALA Hata e:
            GünlükleHata(günlük, "bildirim_istisna", e)
            bildirim.tekrar ← bildirim.tekrar + 1
            KuyruğaTekrarEkle(kuyruk, bildirim)

İŞPARÇACIĞI GözetmenİP(sağlık, alarm, kuyruk, yapılandırma, günlük):
    SİSTEM_ÇALIŞIYOR iken:
        EĞER DeğişkenleriPingle(sağlık) = YANLIŞ İSE:
            GünlükleKritik(günlük, "gözetmen_ihlali")
            GüvenliModuEtkinleştir(alarm)                 # kritik cihazları güvenli duruma al
            KuyruğaEkle(kuyruk, GözetmenUyarısıOluştur())
        Uyku(yapılandırma.gözetmenAralığı)
