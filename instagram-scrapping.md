# Tutorial Instagram Comment Scraper dengan Python

## Metode 1: Menggunakan Instaloader (Recommended)

### Instalasi
```bash
pip install instaloader
```

### Scraping Komentar dari Post
```python
import instaloader
import pandas as pd
from datetime import datetime

# Inisialisasi Instaloader
L = instaloader.Instaloader()

# Login (opsional tapi direkomendasikan untuk menghindari rate limit)
USERNAME = 'your_username'
PASSWORD = 'your_password'

try:
    L.login(USERNAME, PASSWORD)
    print("Login berhasil!")
except Exception as e:
    print(f"Login gagal: {e}")

# Ambil post dari shortcode (kode di URL Instagram)
# Contoh URL: https://www.instagram.com/p/ABC123xyz/
# Shortcode: ABC123xyz
shortcode = 'ABC123xyz'  # Ganti dengan shortcode post yang ingin di-scrape

post = instaloader.Post.from_shortcode(L.context, shortcode)

# Scrape semua komentar
comments_data = []

print("Mengambil komentar...")
for comment in post.get_comments():
    comment_info = {
        'username': comment.owner.username,
        'text': comment.text,
        'created_at': comment.created_at_utc,
        'likes': comment.likes_count,
        'comment_id': comment.id
    }
    comments_data.append(comment_info)
    print(f"Komentar dari @{comment.owner.username}: {comment.text[:50]}...")

# Simpan ke DataFrame
df = pd.DataFrame(comments_data)

# Ekspor ke CSV
df.to_csv('instagram_comments.csv', index=False, encoding='utf-8-sig')
print(f"\nTotal {len(comments_data)} komentar berhasil di-scrape!")
print("Data disimpan ke instagram_comments.csv")
```

## Metode 2: Menggunakan instagrapi (Lebih Lengkap)

### Instalasi
```bash
pip install instagrapi
```

### Scraping Komentar dengan instagrapi
```python
from instagrapi import Client
import pandas as pd
import time

# Inisialisasi client
cl = Client()

# Login
USERNAME = 'your_username'
PASSWORD = 'your_password'

try:
    cl.login(USERNAME, PASSWORD)
    print("Login berhasil!")
except Exception as e:
    print(f"Login gagal: {e}")
    exit()

# Ambil komentar dari post
# Bisa pakai URL atau media_id
post_url = 'https://www.instagram.com/p/ABC123xyz/'
media_id = cl.media_pk_from_url(post_url)

# Scrape komentar
print("Mengambil komentar...")
comments = cl.media_comments(media_id, amount=0)  # amount=0 berarti ambil semua

# Proses data
comments_data = []
for comment in comments:
    comment_info = {
        'username': comment.user.username,
        'full_name': comment.user.full_name,
        'text': comment.text,
        'created_at': comment.created_at_utc,
        'likes': comment.like_count,
        'comment_id': comment.pk,
        'user_id': comment.user.pk
    }
    comments_data.append(comment_info)

# Simpan ke DataFrame dan CSV
df = pd.DataFrame(comments_data)
df.to_csv('instagram_comments_instagrapi.csv', index=False, encoding='utf-8-sig')

print(f"\nTotal {len(comments_data)} komentar berhasil di-scrape!")
print("Data disimpan ke instagram_comments_instagrapi.csv")
```

## Metode 3: Scraping Multiple Posts

```python
from instagrapi import Client
import pandas as pd
import time

cl = Client()
cl.login('your_username', 'your_password')

# List URL posts yang ingin di-scrape
post_urls = [
    'https://www.instagram.com/p/ABC123xyz/',
    'https://www.instagram.com/p/DEF456abc/',
    'https://www.instagram.com/p/GHI789def/'
]

all_comments = []

for url in post_urls:
    try:
        media_id = cl.media_pk_from_url(url)
        comments = cl.media_comments(media_id, amount=0)
        
        for comment in comments:
            comment_info = {
                'post_url': url,
                'username': comment.user.username,
                'text': comment.text,
                'created_at': comment.created_at_utc,
                'likes': comment.like_count
            }
            all_comments.append(comment_info)
        
        print(f"Selesai scraping {len(comments)} komentar dari {url}")
        time.sleep(5)  # Jeda untuk menghindari rate limit
        
    except Exception as e:
        print(f"Error pada {url}: {e}")
        continue

# Simpan semua data
df = pd.DataFrame(all_comments)
df.to_csv('all_instagram_comments.csv', index=False, encoding='utf-8-sig')
print(f"\nTotal {len(all_comments)} komentar dari {len(post_urls)} posts!")
```

## Tips Penting

### 1. Rate Limiting
```python
import time

# Tambahkan delay antar request
time.sleep(2)  # Tunggu 2 detik
```

### 2. Error Handling
```python
try:
    comments = cl.media_comments(media_id, amount=0)
except Exception as e:
    print(f"Error: {e}")
    # Tunggu lebih lama jika kena rate limit
    time.sleep(60)
```

### 3. Simpan Session untuk Login
```python
# Simpan session setelah login
cl.dump_settings('session.json')

# Load session (tidak perlu login lagi)
cl.load_settings('session.json')
cl.login(USERNAME, PASSWORD)
```

## Analisis Sentiment Sederhana

```python
from textblob import TextBlob
import pandas as pd

df = pd.read_csv('instagram_comments.csv')

def get_sentiment(text):
    analysis = TextBlob(str(text))
    if analysis.sentiment.polarity > 0:
        return 'Positive'
    elif analysis.sentiment.polarity < 0:
        return 'Negative'
    else:
        return 'Neutral'

df['sentiment'] = df['text'].apply(get_sentiment)
df['polarity_score'] = df['text'].apply(lambda x: TextBlob(str(x)).sentiment.polarity)

# Lihat distribusi sentiment
print(df['sentiment'].value_counts())

# Simpan hasil analisis
df.to_csv('instagram_comments_with_sentiment.csv', index=False, encoding='utf-8-sig')
```

## ⚠️ Peringatan & Best Practices

1. **Jangan scrape terlalu agresif** - Instagram punya rate limit yang ketat
2. **Gunakan akun dummy** - jangan pakai akun utama Anda
3. **Tambahkan delay** - minimal 2-5 detik antar request
4. **Respect ToS** - Instagram melarang scraping di Terms of Service mereka
5. **Data pribadi** - hati-hati dengan data pengguna, jangan disalahgunakan
6. **Gunakan proxy** - jika scraping dalam jumlah besar
7. **Simpan session** - untuk menghindari login berulang

## Troubleshooting

**Login Error / Challenge Required:**
- Instagram mungkin meminta verifikasi
- Gunakan akun yang sudah lama dan aktif
- Hindari login dari IP yang berbeda-beda

**Rate Limit:**
- Tambah delay lebih lama (10-30 detik)
- Kurangi jumlah request per hari
- Gunakan multiple accounts dengan rotation

**No Comments Retrieved:**
- Pastikan post memiliki komentar
- Akun mungkin kena limit, tunggu beberapa jam
- Coba gunakan library alternatif
