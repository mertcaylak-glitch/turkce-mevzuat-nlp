# Türkçe Mevzuat NLP Projesi — Katkı Yol Haritası

*Sabancı Üniversitesi, Inanc Arin danışmanlığında yürütülen "Domain-Adapted Turkish Language Model" projesi için hazırlanmıştır.*

## Nerede durduğun

Güçlü yanlar: Python, se-automation'da edinilen yapılandırılmış veri/pipeline tasarımı deneyimi (Cypher SQL'den farklı ama "yapılandırılmış depoyu düşünme" mantığı aynı), DSA 210'dan gelen toplama → temizleme → analiz döngüsü.

Muhtemelen yeni olacaklar: web scraping araçları, PDF parsing, Neo4j/Cypher, HuggingFace ekosistemi — hiçbiri zor değil, zaman istiyor.

**Genel prensip:** se-automation'da izlenen "önce dar kapsamda uçtan uca çalışan bir zincir kur, sonra genişlet" yaklaşımı ve "yapılandırılmış taslak → insan onayı → mekanik uygulama" deseni burada da uygulanacak (özellikle Faz 4'te KG'ye yazarken).

---

## Faz 0 — Kurulum & Kapsam (Hafta 1)

- Inanc Arin veya bir yüksek lisans öğrencisiyle 20-30 dk netleştirme görüşmesi:
  - Hangi mevzuat alt kümesiyle başlanacak (tek bir bakanlık mı, belirli bir kanun kategorisi mi)
  - Var olan bir ontoloji/şema taslağı var mı
  - 5 kişilik ekipte rol dağılımı nasıl olacak
  - Hedeflenen yayın türü ne (workshop paper / dataset paper / vb.)
- Ortam kurulumu: Python 3.11+, `venv`, Neo4j Desktop veya Docker ile local instance, Cypher temellerine 1-2 saat ayır
- Repo iskeleti henüz yoksa öner: `/scraping`, `/preprocessing`, `/kg`, `/notebooks`, `/docs`
- Aşağıdaki kaynak listesini ekiple paylaşacağın kısa bir nota dönüştür — ilk somut katkı bu olabilir

---

## Faz 1 — Veri Toplama (Hafta 1-4)

### Önemli bulgu: tekerleği yeniden icat etme

- **mevzuat-mcp** (GitHub: saidsurucu, MIT lisanslı): Adalet Bakanlığı'nın Mevzuat Bilgi Sistemi'ne (mevzuat.gov.tr) ve bedesten.adalet.gov.tr'ye programatik erişim sağlayan, 26 araçlı açık kaynak bir MCP sunucusu. Kanun, KHK, tüzük, yönetmelik türlerine özel arama, tam metin erişimi (Markdown formatında), OpenRouter tabanlı semantik arama, CB Kararı/Genelgesi gibi PDF-ağırlıklı belgeler için Mistral OCR entegrasyonu içeriyor.
- **muhammetakkurt/mevzuat-gov-dataset** (HuggingFace) + **Mevzuat-Gov-Scraper** (GitHub): mevzuat.gov.tr'den çekilmiş, kanun adı/numarası/kabul tarihi/madde metinleri içeren hazır bir veri seti ve eşlik eden scraper kodu (Scrapy + Selenium + regex/spaCy `tr_core_news_lg` ile ayrıştırma). 2024 Eylül'e kadarki mevzuatı kapsıyor; bazı alanlar eksik/tarih formatları tutarsız olabiliyor.
- **Resmi Gazete**: resmi bir API yok, standart yaklaşım `requests` + `BeautifulSoup` ile tarihe göre sayfa çekip ayrıştırmak.
- **Hukuki durum**: Resmî Gazete'de yayımlanan mevzuat metinleri, Yargıtay kararları ve TBMM tutanakları telif hakkı ve veri tabanı koruması kapsamı dışında — scraping burada gri alan değil, sadece kaba kullanım/rate-limit nezaketi bekleniyor.

### Ne yapmalısın

Staj tanımı "kendi parser'ını yaz" diyor — bu kaynakları referans implementasyon olarak oku, sonra tek bir doküman tipiyle (örn. tek bir yönetmelik kategorisi) kendi ucu-uca akışını kur. Var olan veri setini zaman kazanmak için tohum veri olarak kullan.

**Kütüphaneler:** `requests`/`httpx` + `BeautifulSoup`/`lxml` (HTML), `pdfplumber` veya `PyMuPDF` (PDF), JS-render sayfalar için `Playwright`.

**Depolama:** Ham veriyi kaynak + tarih + doküman no + versiyon metadata'sıyla sakla (JSON Lines veya SQLite).

---

## Faz 2 — Ön İşleme & Kalite Kontrol (Hafta 2-5, Faz 1 ile paralel)

- **Büyük/küçük harf:** Python'un varsayılan `.lower()`'ı Türkçe İ/ı çiftinde kırılır — normalize etmeden önce özel bir çeviri tablosu kullan.
- **Encoding:** eski/taranmış belgelerde Windows-1254 kalıntıları olabilir, UTF-8'e normalize et.
- **Deduplication:** mevzuat metinleri sık değişiklik görüyor (aynı maddenin farklı versiyonları) — paragraf seviyesinde near-dup tespiti (MinHash) işine yarar.
- **Morfolojik analiz:** Zemberek makul bir seçenek (Apache 2.0, JPype ile Python'dan çağrılabiliyor) ama son sürümü 2019'dan beri güncellenmedi ve **NER modülü hiçbir zaman eğitilmiş model içermedi** — NER için Faz 4'teki alternatiflere bak.

---

## Faz 3 — Ontoloji & Neo4j Şeması (Hafta 3-5, paralel başlanabilir)

Minimal bir şema öner, elle test et, sonra otomatikleştir:

```cypher
MERGE (k:Kanun {no: "6698"})
MERGE (m:Madde {no: "5", kanun_no: "6698"})
MERGE (m)-[:PARCASI]->(k)
```

- **Düğümler:** `Kanun`, `Yönetmelik`, `Madde`, `Kurum`, `Değişiklik`
- **İlişkiler:** `DEĞİŞTİRİR`, `ATIF_YAPAR`, `YÜRÜRLÜĞE_KOYAR`, `PARCASI`
- Python tarafı için resmi `neo4j` sürücüsü yeterli.
- Kapsam belirlerken referans: 2026'da yayımlanan **Mecellem** çalışması, Türkçe hukuki metinler için sözleşme/mevzuat/içtihat alt görevlerinden oluşan bir MTEB-Türkçe hukuk değerlendirmesi kurmuş — "mevzuat" alt görevinin tanımına bakmak faydalı.

---

## Faz 4 — Çıkarım → KG Doldurma (Hafta 5-8)

- Önce regex/kural tabanlı git: "bu Kanunun 5 inci maddesi", "6698 sayılı Kanun" gibi atıf kalıpları Türkçe mevzuatta son derece tekrarlı — ML'e gerek kalmadan yüksek doğrulukla yakalanabilir.
- Genel varlıklar (kurum, kişi, tarih) için NER'e geçerken Zemberek'i atla; **HukukBERT** gibi (Yargıtay/Danıştay/İstinaf/AYM kararlarından 21GB+ veriyle önceden eğitilmiş) hukuk-alanına-özel bir temel model üstüne kurmayı değerlendir.
- **Kritik:** Çıkarılan triple'ları KG'ye direkt yazma. "Yapılandırılmış taslak → insan onayı → mekanik uygulama" deseni: çıkarımı önce bir ara CSV/JSON'a yaz, biri gözden geçirsin, sonra Neo4j'ye commit et.

---

## Faz 5 (Opsiyonel/İleri Seviye) — QLoRA Fine-tuning (Hafta 8-10+)

Sadece veri/KG katmanı stabil olduktan sonra başla.

- **Genel taban modeller (2026):** 7-8B sınıfı (Llama 3.1 8B, Mistral 7B v0.3, Qwen 2.5 7B) QLoRA ile tek bir 12GB GPU'ya sığıyor; `Unsloth` süreci kolaylaştırıyor.
- **Türkçe'ye özel taban seçenekleri:** `WiroAI/wiroai-turkish-llm-9b` (Gemma tabanlı, Türkçe SFT), `TURKCELL/Turkcell-LLM-7b-v1` (Mistral tabanlı, 5B token Türkçe korpus, DoRA+LoRA).
- **Özetleme/üretim görevleri için:** `VBART` (vngrs-ai) — Türkçe'ye özel tokenizer çok dilli modellerden 11 kata kadar daha verimli.
- **Doğrudan emsal:** `AlicanKiraz0/Mihenk-LLM-14B` — Qwen3-14B'nin Türkçe finans metinleriyle fine-tune edilmiş hali. Bu proje aynı deseni hukuk alanına uyguluyor; iyi bir "ilgili çalışma" referansı.

---

## Görünürlük ve Yazarlık İçin Pratik Not

- Kota 5, yazarlık katkıya bağlı — haftalık senkrona somut bir şey getir (çalışan bir scraper, 20 satır triple, bir şema taslağı).
- Haftalık log tut.
- Hiçbir mimari karar alınmamışken repo yapısını/ilk şemayı sen önerirsen, projenin şekillenmesinde söz sahibi olursun.

---

## Hızlı Kaynak Listesi

| Kaynak | Tür | Not |
|---|---|---|
| [mevzuat-mcp](https://github.com/saidsurucu/mevzuat-mcp) | Araç (GitHub) | mevzuat.gov.tr + bedesten.adalet.gov.tr için 26 araçlı MCP sunucusu, MIT |
| [muhammetakkurt/mevzuat-gov-dataset](https://huggingface.co/datasets/muhammetakkurt/mevzuat-gov-dataset) | Veri seti (HuggingFace) | Hazır mevzuat veri seti |
| [Mevzuat-Gov-Scraper](https://github.com/muhammetakkurtt/mevzuat-gov-scraper) | Kod (GitHub) | Yukarıdaki veri setini üreten scraper |
| HukukBERT (arXiv, 2026) | Model/makale | Türk hukuku için alan-özel BERT |
| Mecellem models (arXiv, 2026) | Model/makale | Hukuk alanında sürekli ön-eğitim + MTEB-Türkçe hukuk benchmark'ı |
| METU tezi, "Turkish Legal NLP" (2025) | Makale | 23K+ karar, 9K+ mevzuat metninden temiz korpus |
| [agmmnn/turkish-nlp-resources](https://github.com/agmmnn/turkish-nlp-resources) | Liste (GitHub) | Yargı MCP, TurkColBERT, TurkEmbed4Retrieval dahil güncel kaynak listesi |

---

*Bu doküman [Claude](https://claude.ai) ile hazırlanmıştır — Temmuz 2026.*
