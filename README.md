# Titanic Veri Setinde Koşullu Eksik Değer Doldurma (Class-Based Imputation)

Veri madenciliği ve modelleme süreçlerinde en çok karşılaşılan sorunlardan biri eksik verilerdir. Bu çalışmada, Titanic veri setindeki eksik yaş (`Age`) değerlerini doğrudan tüm veri setinin ortalamasıyla doldurmak yerine, verinin alt gruplarındaki örüntüyü koruyarak bilet sınıflarına (`Pclass`) göre koşullu olarak doldurdum.

## Problem ve Karşılaşılan Durum:
Veri setini yükleyip eksik değer kontrolü yaptığımızda:
- `Age` değişkeninde **177 adet** eksik değer (NaN) bulunuyor.
- `Cabin` değişkeninde **687**, `Embarked` değişkeninde ise **2** eksik değer mevcut.

Genelde yapılan ilk refleks `df['Age'].fillna(df['Age'].mean())` yazıp geçmektir. Ancak bu yaklaşım veriyi bozar. 

Veriyi bilet sınıflarına (`Pclass`) göre gruplayıp yaş ortalamalarına baktığımızda tablo oldukça netleşiyor:
- **1. Sınıf Yolcular:** ~38.23 yaş
- **2. Sınıf Yolcular:** ~29.88 yaş
- **3. Sınıf Yolcular:** ~25.14 yaş

1. sınıfta seyahat eden yolcular ile 3. sınıfta seyahat eden yolcular arasında yaklaşık 13 yıllık belirgin bir yaş farkı var. Eğer 3. sınıftaki 20'li yaşlarındaki bir yolcuya genel ortalama olan ~29.7 değerini atarsak, verideki sosyoekonomik dağılımı ve varyansı yapay olarak bozmuş oluruz. Bu yüzden her yolcuya kendi bilet sınıfının yaş ortalamasını atamak çok daha mantıklı bir mühendislik yaklaşımıdır.

---

## Uygulama ve Kodun Mantığı

Bu işlemi döngülerle (for loop) uğraşmadan, Pandas'ın vektörize operasyonlarıyla tek satırda çözüyoruz:

```python
import pandas as pd
import numpy as np

# 1. Veriyi okuma
df = pd.read_csv('train.csv')

# 2. Eksik değer kontrolü
print(df.isnull().sum())

# 3. Sınıflara göre yaş ortalamasını inceleme
print(df.groupby('Pclass')['Age'].mean().round(2))

# 4. Koşullu doldurma (Transform ile grup ortalamasını eşleme)
df['Age'] = df['Age'].fillna(df.groupby('Pclass')['Age'].transform('mean'))

# 5. Kontrol
print(f"Kalan eksik yaş sayısı: {df['Age'].isnull().sum()}")


Burada transform Tam Olarak Ne Yapıyor?

groupby('Pclass')['Age'].mean() dediğimizde bize sadece 3 satırlık bir özet serisi döner (1, 2 ve 3 için ortalamalar).

Ancak transform('mean') kullandığımızda, hesaplanan bu 3 ortalama değeri alıp orijinal tablodaki her satırın Pclass değerine göre ilgili satıra hizalar (yani tablonun satır boyutuna genişletir).

fillna() fonksiyonu da bu genişletilmiş seriyi referans alarak sadece boş olan satırlara doğru sınıfın ortalamasını yazar.


Sonuç

Age sütunundaki eksik değer sayısı veri kaybı yaşamadan 0'a indirildi.

Sınıflar arası yaş farkı korunduğu için veri seti modelleme ve özellik mühendisliği (feature engineering) aşamasına daha sağlıklı bir şekilde hazırlandı.