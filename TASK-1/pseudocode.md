BAŞLA

Kartı yerleştir
Şifre ekranını bekle

deneme = 0

TEKRAR:
    4 haneli PIN gir
    EĞER PIN doğruysa
        İşlemler ekranını aç
        GİT → ISLEMLER
    DEĞİLSE
        deneme = deneme + 1
        EĞER deneme < 4 ise
            Tekrar şifre iste
            GİT → TEKRAR
        DEĞİLSE
            Bankamatik kartı yutar
            BİTİR
        SON
    SON

ISLEMLER:
    Para çekmeye tıkla
    EĞER yanlış tuşa basıldıysa
        İşlemler ekranına geri dön
        GİT → ISLEMLER
    DEĞİLSE
        Para çekme ekranına gir
    SON

PARA_CEKME:
    İstenen nakit miktarını gir
    EĞER istenen_nakit ≤ mevcut_bakiye ise
        Bankamatik parayı verir
        GİT → FIS_SOR
    DEĞİLSE
        “Yeterli para yok” uyarısı ver
        İşlemler ekranına geri dön
        GİT → ISLEMLER
    SON

FIS_SOR:
    Fiş isteniyor mu?
    EĞER cevap = HAYIR ise
        Kartı ve parayı ver
        BİTİR
    EĞER cevap = EVET ise
        Fişi ver
        Kartı ve parayı ver
        BİTİR
    SON

SON
