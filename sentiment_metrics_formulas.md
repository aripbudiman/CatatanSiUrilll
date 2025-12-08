# Rumus Perhitungan Volume Komentar & Puncak Kemarahan

## 1. Volume Komentar

### A. Volume Total
```
Volume Total = Σ semua komentar dalam periode tertentu
```

### B. Volume per Satuan Waktu
```
Volume Rate = Jumlah komentar / Satuan waktu (jam/hari/minggu)
```

**Contoh:**
- Hari 1: 1,500 komentar
- Hari 2: 3,200 komentar  
- Hari 3: 8,500 komentar ← **Puncak volume**
- Hari 4: 2,100 komentar

```
Average Daily Volume = (1500 + 3200 + 8500 + 2100) / 4 = 3,825 komentar/hari
```

### C. Growth Rate (Tingkat Pertumbuhan)
```
Growth Rate (%) = ((Volume_hari_ini - Volume_hari_kemarin) / Volume_hari_kemarin) × 100
```

**Contoh:**
```
Hari 2 ke Hari 3 = ((8500 - 3200) / 3200) × 100 = 165.6% increase
```

### D. Volume Anomaly Detection
Untuk deteksi **lonjakan tidak normal**:

```
Z-Score = (Volume_hari_ini - Mean_volume) / Standard_Deviation

Jika Z-Score > 2 atau < -2 → Anomali (puncak/penurunan drastis)
```

## 2. Puncak Kemarahan (Anger Peak)

### A. Sentiment Polarity Score

**Metode 1: Simple Polarity (-1 hingga +1)**
```
Polarity = (Positive_words - Negative_words) / Total_words
```

- **-1.0 hingga -0.5** → Sangat negatif / Marah
- **-0.5 hingga -0.1** → Negatif
- **-0.1 hingga +0.1** → Netral
- **+0.1 hingga +0.5** → Positif
- **+0.5 hingga +1.0** → Sangat positif

**Metode 2: Categorical (0-4 scale)**
```
0 = Sangat Negatif
1 = Negatif
2 = Netral
3 = Positif
4 = Sangat Positif
```

### B. Anger Intensity Score

Untuk mengukur **intensitas kemarahan**, bukan hanya negatif/positif:

```
Anger Intensity = (Jumlah kata marah × Bobot) / Total kata dalam komentar

Kategori:
- 0.0 - 0.2 → Tenang/Netral
- 0.2 - 0.4 → Kecewa ringan
- 0.4 - 0.6 → Marah sedang
- 0.6 - 0.8 → Sangat marah
- 0.8 - 1.0 → Rage/Murka
```

**Kata-kata marah dan bobotnya (untuk Bahasa Indonesia):**

| Kata/Frasa | Bobot |
|------------|-------|
| kecewa, kurang puas | 0.3 |
| kesal, dongkol | 0.5 |
| marah, geram | 0.7 |
| murka, benci, jijik | 0.9 |
| menyedihkan, memalukan | 0.6 |
| bodoh, tolol, goblok | 0.8 |
| !!! (tanda seru berlebihan) | +0.2 |
| CAPSLOCK SEMUA | +0.3 |

### C. Average Sentiment per Time Period

```
Avg_Sentiment_Score_per_hari = Σ(Sentiment_score semua komentar) / Jumlah komentar hari itu
```

**Contoh:**
```
Hari 1: Avg sentiment = -0.25 (negatif ringan)
Hari 2: Avg sentiment = -0.48 (negatif)
Hari 3: Avg sentiment = -0.76 (sangat negatif) ← **Puncak kemarahan**
Hari 4: Avg sentiment = -0.52 (negatif, mulai mereda)
```

### D. Weighted Anger Score (Kombinasi Volume + Intensity)

Ini metode **paling powerful** karena gabungkan volume dan intensitas:

```
Weighted Anger Score = Volume_komentar × Avg_Anger_Intensity

atau

Weighted Anger Score = (Volume × Avg_Sentiment × Engagement_factor)
```

**Contoh:**
```
Hari 1: 1,500 komentar × 0.35 intensity = 525 anger points
Hari 2: 3,200 komentar × 0.52 intensity = 1,664 anger points
Hari 3: 8,500 komentar × 0.76 intensity = 6,460 anger points ← **PUNCAK!**
Hari 4: 2,100 komentar × 0.48 intensity = 1,008 anger points
```

### E. Engagement-Weighted Sentiment

Komentar dengan engagement tinggi (likes, replies) = lebih berpengaruh:

```
Weighted_Sentiment = Σ(Sentiment_score × (Likes + Replies + 1)) / Σ(Likes + Replies + 1)
```

## 3. Deteksi Puncak (Peak Detection)

### A. Simple Peak Detection
```python
# Peak = hari dengan nilai tertinggi
Peak_day = max(daily_anger_scores)
```

### B. Moving Average untuk Smooth Peak
```python
# 3-day moving average untuk menghilangkan noise
MA_3day = (Day_t-1 + Day_t + Day_t+1) / 3

# Peak = nilai tertinggi setelah smoothing
```

### C. Rate of Change (Laju Perubahan)
```
ROC = (Anger_today - Anger_yesterday) / Anger_yesterday × 100

Jika ROC > 50% → Lonjakan signifikan (mendekati puncak)
Jika ROC < -30% → Penurunan signifikan (setelah puncak)
```

## 4. Contoh Implementasi Python

```python
import pandas as pd
import numpy as np

# Contoh data
data = {
    'date': ['2024-12-01', '2024-12-02', '2024-12-03', '2024-12-04', '2024-12-05'],
    'volume': [1200, 2800, 9500, 3200, 1800],
    'avg_sentiment': [-0.25, -0.48, -0.82, -0.56, -0.38]
}

df = pd.DataFrame(data)
df['date'] = pd.to_datetime(df['date'])

# 1. Hitung Anger Score (Volume × Intensity)
df['anger_score'] = df['volume'] * abs(df['avg_sentiment'])

# 2. Hitung Growth Rate
df['growth_rate'] = df['volume'].pct_change() * 100

# 3. Hitung Z-Score untuk deteksi anomali
df['z_score'] = (df['volume'] - df['volume'].mean()) / df['volume'].std()

# 4. Identifikasi puncak
peak_day = df.loc[df['anger_score'].idxmax()]

print("=== ANALISIS PUNCAK KEMARAHAN ===")
print(f"Tanggal Puncak: {peak_day['date'].strftime('%Y-%m-%d')}")
print(f"Volume Komentar: {peak_day['volume']:,}")
print(f"Avg Sentiment: {peak_day['avg_sentiment']:.2f}")
print(f"Anger Score: {peak_day['anger_score']:.2f}")
print(f"Growth Rate: {peak_day['growth_rate']:.1f}%")
print(f"Z-Score: {peak_day['z_score']:.2f}")

print("\n=== DAILY BREAKDOWN ===")
print(df.to_string(index=False))
```

## 5. Interpretasi Hasil

### Volume Threshold
- **< 1,000 komentar/hari** → Low engagement
- **1,000 - 5,000** → Medium engagement  
- **5,000 - 10,000** → High engagement
- **> 10,000** → Viral / Crisis level

### Anger Intensity Threshold
- **< 0.3** → Kritik ringan
- **0.3 - 0.5** → Kekecewaan
- **0.5 - 0.7** → Kemarahan
- **> 0.7** → Kemarahan masif / Outrage

### Combined (Volume × Intensity)
- **< 1,000** → Manageable
- **1,000 - 3,000** → Perlu perhatian
- **3,000 - 5,000** → Urgent response needed
- **> 5,000** → Crisis mode

## 6. Visualisasi Puncak

```python
import matplotlib.pyplot as plt

fig, ax1 = plt.subplots(figsize=(12, 6))

# Plot volume (bar)
ax1.bar(df['date'], df['volume'], alpha=0.6, label='Volume', color='steelblue')
ax1.set_ylabel('Volume Komentar', fontsize=12)
ax1.set_xlabel('Tanggal', fontsize=12)

# Plot anger score (line)
ax2 = ax1.twinx()
ax2.plot(df['date'], df['anger_score'], color='red', marker='o', 
         linewidth=2, markersize=8, label='Anger Score')
ax2.set_ylabel('Anger Score', fontsize=12, color='red')

# Tandai puncak
peak_idx = df['anger_score'].idxmax()
ax2.scatter(df.loc[peak_idx, 'date'], df.loc[peak_idx, 'anger_score'], 
            s=200, color='darkred', marker='*', zorder=5, label='PUNCAK')

plt.title('Volume Komentar vs Anger Score Over Time', fontsize=14, fontweight='bold')
ax1.legend(loc='upper left')
ax2.legend(loc='upper right')
plt.grid(alpha=0.3)
plt.tight_layout()
plt.show()
```

## 7. Early Warning System

Untuk prediksi **kapan akan mencapai puncak**:

```python
# Hitung momentum (percepatan)
df['momentum'] = df['anger_score'].diff()

# Jika momentum > 0 dan terus naik → belum puncak
# Jika momentum mulai menurun → mendekati/melewati puncak

if df['momentum'].iloc[-1] > 0 and df['momentum'].iloc[-2] > 0:
    print("⚠️ WARNING: Anger masih meningkat, belum mencapai puncak!")
elif df['momentum'].iloc[-1] < 0:
    print("✓ Peak sudah terlewati, sentiment mulai mereda")
```

## 8. Faktor Pengali (Multiplier)

Beberapa faktor yang memperkuat anger:

```
Final Anger Score = Base_Anger × Viral_Factor × Celebrity_Factor × Timing_Factor

Viral_Factor:
- Share/Retweet > 1000 → multiply by 1.5
- Media mainstream pickup → multiply by 2.0

Celebrity_Factor:
- Influencer >100k followers → multiply by 1.3
- Public figure involvement → multiply by 1.8

Timing_Factor:
- Weekend/malam → multiply by 1.2
- Event nasional → multiply by 1.5
```

---

## Tips Praktis

1. **Update setiap 6-12 jam** untuk kasus viral/crisis
2. **Bandingkan dengan baseline** (rata-rata 7 hari sebelumnya)
3. **Perhatikan trigger events** (video viral, statement pejabat, dll)
4. **Track bukan hanya puncak, tapi juga "long tail"** - apakah sentiment mereda atau persisten
5. **Cross-reference dengan platform lain** - apakah puncaknya sama di Twitter, IG, TikTok?

# convert tanggal IG
```py
import pandas as pd
from datetime import datetime, timedelta
import re

def parse_instagram_time(time_str, scrape_date=None):
    """
    Convert Instagram relative time format (e.g., '6d', '22h', '3w') to actual datetime
    
    Args:
        time_str (str): Instagram time format (e.g., '6d', '22h', '5m', '3w')
        scrape_date (datetime): Tanggal scraping dilakukan. Default: sekarang
    
    Returns:
        datetime: Tanggal komentar yang sebenarnya
    """
    if scrape_date is None:
        scrape_date = datetime.now()
    
    # Clean the string
    time_str = str(time_str).strip().lower()
    
    # Extract number and unit using regex
    match = re.match(r'(\d+)([smhdw])', time_str)
    
    if not match:
        # Jika format tidak dikenali, return scrape_date
        return scrape_date
    
    value = int(match.group(1))
    unit = match.group(2)
    
    # Convert based on unit
    if unit == 's':  # seconds
        delta = timedelta(seconds=value)
    elif unit == 'm':  # minutes
        delta = timedelta(minutes=value)
    elif unit == 'h':  # hours
        delta = timedelta(hours=value)
    elif unit == 'd':  # days
        delta = timedelta(days=value)
    elif unit == 'w':  # weeks
        delta = timedelta(weeks=value)
    else:
        delta = timedelta(0)
    
    # Subtract delta from scrape date
    actual_date = scrape_date - delta
    
    return actual_date


def convert_instagram_dates(df, time_column='created_at', scrape_date=None):
    """
    Convert entire DataFrame column from Instagram time format to datetime
    
    Args:
        df (DataFrame): DataFrame dengan kolom waktu Instagram
        time_column (str): Nama kolom yang berisi waktu Instagram
        scrape_date (datetime): Tanggal scraping dilakukan
    
    Returns:
        DataFrame: DataFrame dengan kolom waktu yang sudah dikonversi
    """
    if scrape_date is None:
        scrape_date = datetime.now()
    
    # Buat kolom baru untuk tanggal yang sudah dikonversi
    df['comment_datetime'] = df[time_column].apply(
        lambda x: parse_instagram_time(x, scrape_date)
    )
    
    # Tambah kolom tanggal saja (tanpa waktu)
    df['comment_date'] = df['comment_datetime'].dt.date
    
    # Tambah kolom waktu saja
    df['comment_time'] = df['comment_datetime'].dt.time
    
    return df


# ============ CONTOH PENGGUNAAN ============

# Contoh 1: Data dengan berbagai format waktu Instagram
sample_data = {
    'username': ['user1', 'user2', 'user3', 'user4', 'user5', 'user6'],
    'text': ['Komentar 1', 'Komentar 2', 'Komentar 3', 'Komentar 4', 'Komentar 5', 'Komentar 6'],
    'created_at': ['6d', '22h', '3w', '45m', '2d', '1w']
}

df = pd.DataFrame(sample_data)

# Tanggal scraping (misal: 8 Desember 2024, 10:00)
scrape_date = datetime(2024, 12, 8, 10, 0, 0)

print("=" * 70)
print("DATA SEBELUM KONVERSI")
print("=" * 70)
print(df)
print()

# Convert
df_converted = convert_instagram_dates(df, time_column='created_at', scrape_date=scrape_date)

print("=" * 70)
print("DATA SETELAH KONVERSI")
print("=" * 70)
print(df_converted[['username', 'text', 'created_at', 'comment_datetime', 'comment_date']])
print()

# ============ CONTOH 2: Load dari CSV hasil scraping ============

def process_scraped_data(csv_file, scrape_date_str=None):
    """
    Process CSV hasil scraping Instagram dengan konversi tanggal
    
    Args:
        csv_file (str): Path ke file CSV
        scrape_date_str (str): Tanggal scraping dalam format 'YYYY-MM-DD HH:MM:SS'
                               Jika None, akan pakai datetime sekarang
    """
    # Load CSV
    df = pd.read_csv(csv_file)
    
    # Parse scrape date
    if scrape_date_str:
        scrape_date = datetime.strptime(scrape_date_str, '%Y-%m-%d %H:%M:%S')
    else:
        scrape_date = datetime.now()
    
    print(f"Tanggal Scraping: {scrape_date.strftime('%Y-%m-%d %H:%M:%S')}")
    print(f"Total komentar: {len(df)}")
    print()
    
    # Convert dates
    df = convert_instagram_dates(df, time_column='created_at', scrape_date=scrape_date)
    
    # Sorting by datetime (terbaru ke terlama)
    df = df.sort_values('comment_datetime', ascending=False)
    
    # Save hasil konversi
    output_file = csv_file.replace('.csv', '_converted.csv')
    df.to_csv(output_file, index=False, encoding='utf-8-sig')
    
    print(f"File berhasil dikonversi dan disimpan ke: {output_file}")
    print()
    print("Sample data:")
    print(df[['username', 'text', 'created_at', 'comment_datetime']].head(10))
    
    return df

# ============ CONTOH 3: Untuk analisis time-based ============

def analyze_by_time(df):
    """
    Analisis komentar berdasarkan waktu
    """
    df_analysis = df.copy()
    
    # Group by date
    daily_counts = df_analysis.groupby('comment_date').size().reset_index(name='volume')
    daily_counts = daily_counts.sort_values('comment_date')
    
    print("=" * 70)
    print("VOLUME KOMENTAR PER HARI")
    print("=" * 70)
    print(daily_counts)
    print()
    
    # Find peak
    peak_day = daily_counts.loc[daily_counts['volume'].idxmax()]
    print(f"📊 PUNCAK VOLUME: {peak_day['comment_date']} dengan {peak_day['volume']} komentar")
    print()
    
    return daily_counts

# Uncomment untuk test dengan data real
# df_real = process_scraped_data('instagram_comments.csv', scrape_date_str='2024-12-08 10:00:00')
# daily_analysis = analyze_by_time(df_real)


# ============ BONUS: Handle edge cases ============

def advanced_parse_instagram_time(time_str, scrape_date=None):
    """
    Advanced parser yang handle lebih banyak format Instagram
    Termasuk: 'just now', '1 day ago', 'yesterday', dll
    """
    if scrape_date is None:
        scrape_date = datetime.now()
    
    time_str = str(time_str).strip().lower()
    
    # Handle special cases
    if time_str in ['just now', 'baru saja']:
        return scrape_date
    
    if time_str in ['yesterday', 'kemarin']:
        return scrape_date - timedelta(days=1)
    
    # Handle "X time ago" format
    if 'ago' in time_str or 'yang lalu' in time_str:
        # Extract number and unit
        match = re.search(r'(\d+)\s*(second|minute|hour|day|week|month|year|detik|menit|jam|hari|minggu|bulan|tahun)', time_str)
        if match:
            value = int(match.group(1))
            unit = match.group(2)
            
            unit_mapping = {
                'second': 'seconds', 'detik': 'seconds',
                'minute': 'minutes', 'menit': 'minutes',
                'hour': 'hours', 'jam': 'hours',
                'day': 'days', 'hari': 'days',
                'week': 'weeks', 'minggu': 'weeks',
                'month': 'days', 'bulan': 'days',  # Approximate 1 month = 30 days
                'year': 'days', 'tahun': 'days'     # Approximate 1 year = 365 days
            }
            
            unit_normalized = unit_mapping.get(unit, 'days')
            
            if unit_normalized == 'days' and unit in ['month', 'bulan']:
                value = value * 30
            elif unit_normalized == 'days' and unit in ['year', 'tahun']:
                value = value * 365
            
            delta = timedelta(**{unit_normalized: value})
            return scrape_date - delta
    
    # Fallback to simple parser
    return parse_instagram_time(time_str, scrape_date)


print("=" * 70)
print("TEST ADVANCED PARSER")
print("=" * 70)

test_cases = [
    'just now',
    '5m',
    '2h',
    '1d',
    '3w',
    '2 hours ago',
    'yesterday',
    '1 week ago'
]

scrape_time = datetime(2024, 12, 8, 15, 30, 0)
print(f"Scrape time: {scrape_time.strftime('%Y-%m-%d %H:%M:%S')}\n")

for test in test_cases:
    result = advanced_parse_instagram_time(test, scrape_time)
    print(f"{test:20} → {result.strftime('%Y-%m-%d %H:%M:%S')}")
```
