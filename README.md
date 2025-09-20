
<div align="center">
	<h1> ft_irc — Modern C++98 IRC Sunucusu </h1>
	<p>🚀 <b>Gerçek zamanlı, çoklu istemci destekli, minimal ve RFC uyumlu IRC sunucusu!</b> </p>
</div>

---

## 📁 Dosya Düzeni

```
├── src/           # Sunucu kaynak kodları (Server, Client, Channel, Poller, Parser, Router...)
├── tests/         # Manuel ve otomasyon testleri
├── config/        # Örnek konfigürasyon dosyaları
├── USAGE.md       # Ayrıntılı kullanım kılavuzu
├── TESTS.md       # Test planı ve senaryoları
├── DESIGN.md      # Yüksek seviye mimari
├── README.md      # Bu dosya
└── Makefile       # Derleme talimatları
```

---

## 🛠️ Kurulum & Çalıştırma

1. **Projeyi klonla ve dizine gir:**
	 ```bash
	 git clone <repo-url>
	 cd ft_irc
	 ```
2. **Derle:**
	 ```bash
	 make
	 ```
3. **Sunucuyu başlat:**
	 ```bash
	 ./ircserv <port> <password>
	 # Örnek:
	 ./ircserv 6667 secret
	 ```

---

## 🤝 Bağlantı Yöntemleri

### 🔹 Netcat (nc) ile Hızlı Test
```bash
nc -C 127.0.0.1 6667
PASS secret
NICK alice
USER alice 0 * :Alice
JOIN #test
PRIVMSG #test :merhaba dünya
```

### 🔹 KVIrc veya Diğer IRC İstemcileri
- KVIrc ile hızlı bağlantı:
	```
	/server -p=6667 -w=secret 127.0.0.1
	```
- GUI ile sunucu ekleyip parola ve portu girerek de bağlanabilirsiniz.

---

## 🌟 Temel Özellikler

✅ Tek poll() döngüsü, tüm soketler non-blocking
✅ Çoklu istemci desteği, IPv6/IPv4 uyumlu
✅ PASS/NICK/USER, PING/PONG, QUIT, JOIN/PART/PRIVMSG/NOTICE
✅ Kanal yönetimi: operator, modlar (+i, +t, +k, +l, +o), davet, kick, topic
✅ Basit numeric yanıtlar (RFC uyumlu)
✅ Hata ve yetki kontrolleri, güvenli bağlantı yönetimi

---

## 🚦 Kullanım Aşamaları

### 1️⃣ Sunucuyu Başlat
```bash
./ircserv 6667 secret
```

### 2️⃣ Bir İstemciyle Bağlan
Netcat, KVIrc veya başka bir IRC istemcisi kullanabilirsin.

### 3️⃣ Kimlik Doğrulama
Sırasıyla şu komutları gönder:
```
PASS secret
NICK alice
USER alice 0 * :Alice
```

### 4️⃣ Kanal Oluştur/Katıl
```
JOIN #kanal
```

### 5️⃣ Mesaj Gönder
```
PRIVMSG #kanal :herkese selam!
```

### 6️⃣ Kanal Modları ve Yetkiler
```
MODE #kanal +i
MODE #kanal +k s3cr3t
MODE #kanal +l 10
MODE #kanal +o bob
```

### 7️⃣ Operatör İşlemleri
- Davet: `INVITE <nick> #kanal`
- Atma: `KICK #kanal <nick> :sebep`
- Başlık: `TOPIC #kanal :yeni başlık`

---

## 🧪 Test Planı (Özet)

- ✅ Doğru/yanlış parola ile giriş ve hata mesajları
- ✅ Aynı nick ile çakışma testi
- ✅ Parçalı veri gönderimi (partial send)
- ✅ Çoklu istemciyle yayın ve mesajlaşma
- ✅ Kanal modları ve yetki testleri (+i, +k, +l, +t, +o)
- ✅ KICK, INVITE, TOPIC komutları ve kısıtlamalar
- ✅ QUIT/PART ile temiz çıkış ve kaynak yönetimi
- ✅ Hatalı komutlar ve numerik yanıtlar (421, 464, 475, 482, vb.)
- ✅ IPv4 ve IPv6 bağlantı testleri

Daha fazla detay için: [TESTS.md](./TESTS.md)

---

## 🏗️ Yüksek Seviyede Tasarım

- **Server:** Socket açma, accept, poll döngüsü, komut yönlendirme
- **Poller:** poll() sarmalayıcı, non-blocking I/O yönetimi
- **Client:** Bağlantı başına kimlik, buffer ve kanal listesi
- **Channel:** Üyelik, operator, modlar, topic, davet
- **Parser:** IRC satırlarını prefix, komut ve parametrelere ayırma
- **Router:** Komut handler’ları ve iş akışı

Daha fazla mimari detay için: [DESIGN.md](./DESIGN.md)

---

## ❓ Sıkça Sorulanlar & Sorun Giderme

- 🔸 **Bağlanamıyorum:** Sunucu çalışıyor mu, port ve parola doğru mu?
- 🔸 **Parçalı veri:** `-C` bayrağı ile nc kullanın, satırların `\r\n` ile bittiğinden emin olun.
- 🔸 **Hatalı komutlar:** RFC’ye uygun numerik hata mesajları döner.
- 🔸 **nc Komutunda -C Bayrağı Neden Kullanılır?**  
	`-C` bayrağı, netcat’in (nc) satır sonlarında otomatik olarak CRLF (`\r\n`) karakter çifti göndermesini sağlar. IRC protokolü, satırların kesinlikle CRLF ile bitmesini ister. Sadece LF (`\n`) ile biterse komutlar işlenmez veya hatalı çalışır. Yani, `-C` olmadan gönderilen komutlar IRC sunucusu tarafından doğru algılanmayabilir.
- 🔸 **USER Komutunda 0 * Ne Anlama Geliyor?**  
	IRC protokolünde `USER <username> <mode> <unused> :<realname>` formatı kullanılır. `<mode>` genellikle 0 girilir, modern sunucularda anlamı yoktur (eski RFC’de bazı özel anlamları vardı). `<unused>` yıldız (*) veya başka bir şey olabilir, çoğu sunucu bunu dikkate almaz. Yani, `USER alice 0 * :Alice` yazmak bir standarttır ve IRC istemcileri/sunucuları bu şekilde bekler. Kısacası: “0 *” kısmı protokol gereği doldurulması gereken, ama pratikte işlevi olmayan alanlardır.

---

## 📚 Ekstra Bilgiler

- C++98 standardı, dış kütüphane yok, `-Wall -Wextra -Werror` ile derlenir.
- Tek poll() döngüsü, tüm FD’ler O_NONBLOCK.
- Eğitim ve değerlendirme amaçlıdır, prodüksiyon için uygun değildir.

---

<div align="center">
	<b>Daha fazla detay ve örnekler için <code>USAGE.md</code>, test senaryoları için <code>TESTS.md</code>, mimari için <code>DESIGN.md</code> dosyalarına göz atabilirsin!</b>
</div>
