# Türkçe Mevzuat NLP Projesi

Türkçe hukuki/mevzuat metinleri üzerinde alan-uyarlamalı bir dil modeli ve bilgi grafiği geliştirme projesi.

## Klasör Yapısı

```
turkce-mevzuat-nlp/
├── scraping/         # Veri toplama scriptleri (Faz 1)
├── preprocessing/    # Temizlik, normalizasyon, deduplication (Faz 2)
├── kg/               # Ontoloji, Neo4j entegrasyon kodu, Cypher sorguları (Faz 3-4)
├── notebooks/        # Keşif/analiz için Jupyter notebook'ları
├── docs/             # Literatür taraması, kapsam notları, haftalık log
├── requirements.txt
├── docker-compose.yml
└── .gitignore
```

## Kurulum

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Neo4j'yi local'de ayağa kaldır
docker compose up -d
# Neo4j Browser: http://localhost:7474 (kullanıcı: neo4j, şifre: .env dosyasına bak)
```

## Durum

- [ ] Faz 0: Kurulum & kapsam netleştirme
- [ ] Faz 1: Veri toplama
- [ ] Faz 2: Ön işleme & kalite kontrol
- [ ] Faz 3: Ontoloji & Neo4j şeması
- [ ] Faz 4: Çıkarım → KG doldurma
- [ ] Faz 5 (opsiyonel): QLoRA fine-tuning

Detaylı yol haritası için `docs/yol-haritasi.md` dosyasına bak.
