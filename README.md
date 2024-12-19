# LSTM ile Zaman Serisi Tahmini: Hava Kirliliği Öngörüsü

Bu proje, LSTM (Long Short-Term Memory) sinir ağlarını kullanarak hava kirliliği (“PM2.5 partikül”) tahmini üzerine geliştirilmiştir. Aşağıda proje detayları, metodoloji ve elde edilen sonuçlar detaylı bir şekilde sunulmaktadır.

---

## Proje Amacı

Hava kirliliği, insan sağlığını tehdit eden önemli çevresel sorunlardan biridir. Bu projede, geçmiş saatlere ait meteorolojik ve hava kirliliği verileri kullanılarak bir sonraki saatin PM2.5 kirlilik seviyesi tahmin edilmektedir.

- **Girdi Verileri:** Son 12 saate ait pollution, sıcaklık, basınç, rüzgar hızı ve diğer meteorolojik değişkenler.
- **Çıktı:** Bir sonraki saatin PM2.5 değeri.

---

## Veri Seti Bilgisi

Proje kapsamında kullanılan veri seti, Kaggle platformundan alınmıştır ve meteorolojik değişkenlerle birlikte hava kirliliği ölçümlerini içermektedir.

| Sütun Adı  | Açıklama                          | Tip      |
|------------|----------------------------------|----------|
| DATE       | Zaman bilgisi (saatlik)         | datetime |
| POLLUTION  | PM2.5 hava kirliliği ölçümü     | float64  |
| DEW        | Çiy noktası (°C)               | int64    |
| TEMP       | Sıcaklık (°C)                  | float64  |
| PRESS      | Basınç (hPa)                    | float64  |
| WND_DIR    | Rüzgar yönü (kategorik)         | object   |
| WND_SPD    | Rüzgar hızı (m/s)               | float64  |
| SNOW       | Kar yağış süresi (saat)         | int64    |
| RAIN       | Yağmur yağış süresi (saat)      | int64    |

- **Toplam Gözlem Sayısı:** 43,800 saatlik veri
- **Veri Kaynağı:** [Kaggle Dataset Link](https://www.kaggle.com/datasets/rupakroy/lstm-datasets-multivariate-univariate)

---

## Metodoloji

### 1. Veri Ön İşleme
- **Zaman Formatı:** DATE sütunu datetime formatına dönüştürülmüş ve indeks olarak atanmıştır.
- **Kategorik Kodlama:** WND_DIR sütunu, Label Encoding ile sayısallaştırılmıştır.
- **Ölçeklendirme:** MinMaxScaler ile sürekli değişkenler [0, 1] aralığına ölçeklendirilmiştir.

### 2. Gecikmeli (Lagged) Özellikler
- 12 saatlik gecikmeli özellikler oluşturularak modelin geçmiş verilere dayanarak tahmin yapması sağlanmıştır.

### 3. Modelleme

#### Temel LSTM Modeli
| Katman/Parametre        | Detaylar                                                                 |
|-------------------------|-------------------------------------------------------------------------|
| **Model Yapısı**        | Sequential                                                             |
| **Birinci LSTM Katmanı**| 128 nöron, ReLU aktivasyonu, return_sequences=True                     |
| **Dropout Katmanı**     | Oran: 0.1                                                             |
| **İkinci LSTM Katmanı** | 64 nöron, ReLU aktivasyonu, return_sequences=False                    |
| **Çıkış Katmanı**       | Dense, 1 nöron (PM2.5 tahmini için regresyon çıktısı)                 |

- **Loss Fonksiyonu:** Mean Squared Error (MSE)
- **Optimizer:** Adam
- **Batch Size:** 32
- **Epochs:** Maksimum 100 (early stopping ile durdurulabilir)
- **Early Stopping:** val_loss metriği ile izlenmiş ve patience=10 olarak ayarlanmıştır.

#### Hiperparametre Optimizasyonu
- **Yöntem:** GridSearchCV
- **Optimize Edilen Parametreler:**
  - Units: 64, 128 vb.
  - Dropout: 0.1, 0.2 vb.
  - Batch Size: 16, 32
  - Epochs: 50, 100

---

## Performans Değerlendirme

Modelin performansı aşağıdaki regresyon metrikleri ile değerlendirilmiştir:

#### Model Parametreleri

| Parametre   | LSTM Modeli (Öncesi) | LSTM Modeli (Sonrası) |
|-------------|-----------------------|-----------------------|
| Units       | 64.0                 | 128.0                |
| Dropout     | 0.1                  | 0.2                  |
| Batch Size  | 32.0                 | 16.0                 |
| Epochs      | 50.0                 | 100.0                |

#### Performans Metrikleri

| Metrik  | LSTM Modeli (Temel) | LSTM Modeli (Optimizasyonlu) |
|---------|---------------------|-----------------------------|
| RMSE    | 26.35              | 25.38                      |
| MAE     | 14.48              | 14.26                      |
| R² Skoru| 0.9288             | 0.9340                     |

- **RMSE:** Ortalama hata miktarını ifade eder. Optimize edilmiş modelde 25.38'e düşürülmüştür.
- **MAE:** Tahminlerin ortalama mutlak hatasıdır. Optimize edilmiş modelde 14.26 olarak hesaplanmıştır.
- **R² Skoru:** Modelin veri setindeki değişkenliği açıklama oranıdır. Optimize edilmiş model %93.4 başarı göstermiştir.

---

## Sonuçlar

- Optimizasyonlar sayesinde LSTM modeli zaman serisi tahmininde etkili bir performans sergilemiştir.
- **Düşük Hata Oranı:** Optimize edilmiş modelin hata metrikleri düşürülmüştür.
- **Genelleştirme Başarısı:** Model, hava kirliliği gibi karmaşık zaman serisi problemlerinde güçlü bir genelleştirme kapasitesine sahiptir.

Daha fazla bilgi için kodları inceleyebilirsiniz. 🚀
