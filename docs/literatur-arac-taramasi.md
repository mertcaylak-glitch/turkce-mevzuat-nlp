# Literatür & Araç Taraması

*Amaç: Sıfırdan başlamadan önce var olanı görmek, tekerleği yeniden icat etmemek, ve "ilgili çalışmalar" bölümü için erken bir temel oluşturmak.*

## Veri Kaynakları ve Hazır Araçlar

| Kaynak | Ne sunuyor | Kullanım önerisi |
|---|---|---|
| [mevzuat-mcp](https://github.com/saidsurucu/mevzuat-mcp) | mevzuat.gov.tr + bedesten.adalet.gov.tr için 26 araçlı, MIT lisanslı MCP sunucusu. Tam metin erişimi, semantik arama, PDF OCR desteği. | Referans implementasyon olarak incele; API tasarımı ve edge-case'leri (rate limit, hata yönetimi) görmek için değerli. |
| [muhammetakkurt/mevzuat-gov-dataset](https://huggingface.co/datasets/muhammetakkurt/mevzuat-gov-dataset) | Hazır mevzuat veri seti (2024 Eylül'e kadar). | Tohum veri olarak kullanılabilir; eksik alanlar/tarih tutarsızlıkları var, doğrulama gerekir. |
| [Mevzuat-Gov-Scraper](https://github.com/muhammetakkurtt/mevzuat-gov-scraper) | Yukarıdaki veri setini üreten scraper kodu (Scrapy + Selenium + regex/spaCy). | Kendi scraper'ımızı yazarken mimari referans. |
| Resmi Gazete | Resmi API yok; `requests` + `BeautifulSoup` ile standart scraping. | Kendi scraper'ımız gerekiyor. |

**Hukuki not:** Resmî Gazete'de yayımlanan mevzuat metinleri, Yargıtay kararları ve TBMM tutanakları telif hakkı/veri tabanı koruması kapsamı dışında — scraping burada hukuki bir gri alan değil.

## İlgili Akademik Çalışmalar (2025-2026)

| Çalışma | İçerik | Bizim için önemi |
|---|---|---|
| HukukBERT (arXiv, 2026) | Yargıtay/Danıştay/İstinaf/AYM kararlarından 21GB+ veriyle önceden eğitilmiş hukuk-alanına-özel BERT. | Faz 4'te NER/embedding temeli olarak kullanılabilir. |
| Mecellem models (arXiv, 2026) | Hukuk alanında sürekli ön-eğitim + sözleşme/mevzuat/içtihat alt görevlerinden oluşan MTEB-Türkçe hukuk benchmark'ı. | "Mevzuat" alt görev tanımı ontoloji kapsamımız için referans olabilir. |
| METU tezi, "Turkish Legal NLP" (2025) | 23.035 yargı kararı + 9.277 mevzuat/düzenleyici metinden oluşan temizlenmiş korpus. | "İlgili çalışmalar" bölümü + olası karşılaştırma verisi. |
| [agmmnn/turkish-nlp-resources](https://github.com/agmmnn/turkish-nlp-resources) | Yargı MCP, TurkColBERT, TurkEmbed4Retrieval dahil güncel Türkçe NLP kaynak listesi. | Genel referans, düzenli kontrol edilmeli. |

## Dil Araçları — Durum Kontrolü

- **Zemberek**: Apache 2.0, morfolojik analiz/tokenization için hâlâ kullanılabilir. Son sürüm 2019'dan beri değişmedi. **NER modülü hiçbir zaman eğitilmiş model içermedi** — bu konuda başka bir kaynağa (Türkçe BERT tabanlı NER, HukukBERT) yönelmek gerekiyor.

## Sonuç: Nerede Boşluk Var?

Mevcut kaynaklar veri toplama ve genel-amaçlı temsil öğrenimi tarafını büyük ölçüde kapsıyor. Bu projenin katkı sağlayabileceği asıl boşluklar:
1. Mevzuat metinlerine özel, doğrulanmış bir bilgi grafiği / ontoloji (KANUN-MADDE-DEĞİŞİKLİK ilişkileri) — mevcut çalışmaların hiçbiri buna odaklanmıyor.
2. Regex+insan-onaylı çıkarım hattı ile yüksek kaliteli triple üretimi (var olan veri setlerindeki tutarsızlık sorunlarını çözen bir yaklaşım).
