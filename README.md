Zeynep Yücel - 243405096
# Türkçe Yemek Tarifleri Üzerinde Doğal Dil İşleme Projesi

Bu proje, Doğal Dil İşleme dersi kapsamında işlenen temel metin ön işleme, vektörleştirme, benzerlik ölçümü ve denetimli metin sınıflandırma yöntemlerinin Türkçe yemek tarifleri veri seti üzerinde uygulanması amacıyla geliştirilmiştir.

## Veri Seti ve Format Dönüşümü
* **Veri Seti Kaynağı:** [Turkish Recipes Structured (Kaggle)](https://www.kaggle.com/datasets/bit104/turkish-recipes-structured)
* **Format Dönüşümü:** Veri seti orijinal kaynağından **JSON** formatında (`tarifler.json`) temin edilmiş, ders içi uygulamalarla tutarlılık sağlamak amacıyla proje defteri içerisinde `pandas` kullanılarak **CSV** formatına (`tarifler.csv`, `utf-8-sig`) dönüştürülmüştür.

Veri seti **3.320 adet** Türkçe yemek tarifinden ve 8 farklı kategoriden oluşmaktadır. Kategoriler arasında belirgin bir **dengesiz veri dağılımı** bulunmaktadır:

| Kategori | Tarif Sayısı |
| :--- | :---: |
| Ana Yemek | 1256 |
| Tatlı | 832 |
| Salata | 337 |
| Çorba | 293 |
| Kahvaltı | 285 |
| Atıştırmalık | 264 |
| İçecek | 36 |
| Turşu | 17 |
| **Toplam** | **3320** |

Veri setindeki `malzemeler` ve `yapilis_adimlari` sütunları `ast.literal_eval` ile ayrıştırılarak birleştirilmiştir. Modelin kategoriyi başlıktan doğrudan ezberlemesini önlemek amacıyla `tarif_adi` sütunu metin girdisine dahil edilmemiştir.

## Uygulanan Yöntemler

### 1. Metin Ön İşleme
* Türkçe karakter normalizasyonu (`Â/â -> a`, `İ -> i`, `I -> ı` vb.) ve noktalama/sayı temizliği (`re`) uygulanmıştır.
* `stopwords.txt` kullanılarak Türkçe dolgu sözcükleri temizlenmiştir. Yemek tarifleri alanında kritik malzeme isimleri olan **"et"** ve **"un"** kelimeleri dolgu sözcüğü kümesinden çıkarılmıştır.
* `simplemma` (`text_lemmatizer`) ile Türkçe kök bulma (lemmatization) uygulanmıştır. `-ma/-me` yapım ekinin olumsuzluk eki gibi kesilmesini önlemek için *kıyma, sarma, dolma, kavurma, haşlama, kızartma, dondurma, kızarma, közleme* kelimeleri kök bulma aşamasında korunmuştur.

### 2. Benzer Tarif Öneri Sistemi (TF-IDF + Kosinüs Benzerliği)
* Temizlenmiş 3.320 tarif üzerinden `word2idx` sözlüğü (**2.350 benzersiz kelime**) oluşturulmuştur.
* `numpy` kullanılarak **TF**, **IDF** ve **TF-IDF** (`3320 x 2350`) matrisleri sıfırdan hesaplanmıştır.
* Tarif vektörleri arasındaki benzerlik **Kosinüs Benzerliği** ile ölçülerek seçilen bir tarife en benzer yemekler listelenmiştir.

### 3. Denetimli Öğrenme ile Kategori Sınıflandırması
Dengesiz veri dağılımının sınıflandırma performansına etkisini incelemek amacıyla **ana yemek (`0`: 1256)** ve **çorba (`1`: 293)** kategorileri seçilmiş, veri seti `%80 eğitim (1239)` ve `%20 test (310)` olarak ayrılmıştır. Sözlükler yalnızca eğitim kümesi üzerinden oluşturulmuştur.

| Vektörleştirme + Model | Eğitim Doğruluğu | Test Doğruluğu | Eğitim F1 | Test F1 |
| :--- | :---: | :---: | :---: | :---: |
| **CountVectorizer + MultinomialNB** | `0.9435` | `0.9194` | `0.8622` | `0.8000` |
| **TfidfVectorizer + MultinomialNB** | `0.9362` | `0.9194` | `0.7990` | `0.7312` |
| **TfidfVectorizer + DecisionTreeClassifier** | `1.0000` | `0.9548` | `1.0000` | `0.8772` |

* Sınıflar arasındaki ~4:1 oranındaki dengesizlik nedeniyle tüm modellerde doğruluk skoru F1 skorundan daha yüksek ölçülmüştür.
* En yüksek test başarısı **TF-IDF + Karar Ağacı** modelinde elde edilmiştir; ancak eğitim setindeki `%100` skor modelin ezberlemeye daha yatkın olduğunu göstermektedir.
* **CountVectorizer + MultinomialNB** modeli, azınlık sınıfı olan çorba tariflerini **TF-IDF + MultinomialNB** modeline kıyasla daha dengeli sınıflandırmıştır.
* Eğitim setinde yer almayan iki yeni örnek tarif cümlesiyle yapılan canlı testte her iki model de sınıfları doğru tahmin etmiştir.

## Dosya Yapısı
* `proje.ipynb` — Tüm kodları, açıklamaları ve hücre çıktılarını içeren Jupyter Notebook dosyası
* `tarifler.json` — Orijinal kaynak veri seti (JSON formatı)
* `tarifler.csv` — JSON dosyasından dönüştürülen veri seti (CSV formatı)
* `stopwords.txt` — Türkçe dolgu sözcükleri listesi
* `README.md` — Proje dokümantasyonu

## Kurulum ve Çalıştırma
Gerekli kütüphaneleri yükleyin:
```bash
pip install numpy pandas matplotlib scikit-learn simplemma
