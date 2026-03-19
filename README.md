# TRex EMU — 802.1X (dot1x) + FreeRADIUS Test Lab

Bu repo, **FreeRADIUS** sunucusu ve **TRex v3.08 EMU** kullanılarak gerçekleştirilen 802.1X (dot1x) kimlik doğrulama testini belgeler.

---

## 🖥️ Topoloji

```
[TRex EMU İstemciler] <---> [Switch/NAS] <---> [FreeRADIUS Sunucu]
  dot1x supplicant            802.1X               192.168.124.x
  (Port 0/1, 5 istemci)    Authenticator           RADIUS Server
```

---

## ⚙️ Ortam Gereksinimleri

| Bileşen | Açıklama |
|---------|----------|
| TRex | v3.08 |
| FreeRADIUS | v3.x |
| Ubuntu/Debian | FreeRADIUS kurulum ortamı |
| Python | Python 3 |

---

## 🔧 FreeRADIUS Kurulumu

### 1. Paket Kurulumu

```bash
apt update
apt install freeradius freeradius-utils -y
```

### 2. Versiyon Doğrulama

```bash
freeradius -v
```

---

## 📁 FreeRADIUS Yapılandırması

### `/etc/freeradius/3.0/clients.conf` — NAS / İstemci Tanımları

```conf
# Custom clients
client switch1 {
    ipv4addr = 192.168.124.100
    secret   = switch1
}

client tacgui {
    ipaddr = 192.168.124.67
    secret = testing123
}
```

| İstemci | IP | Secret |
|---------|----|--------|
| switch1 | 192.168.124.100 | switch1 |
| tacgui | 192.168.124.67 | testing123 |

---

### `/etc/freeradius/3.0/users` — Kullanıcı Tanımları

```
test1 Cleartext-Password := "test1"
test2 Cleartext-Password := "test2"
test3 Cleartext-Password := "test3"
test4 Cleartext-Password := "test4"
test5 Cleartext-Password := "test5"
test6 Cleartext-Password := "test6"
test7 Cleartext-Password := "test7"
test8 Cleartext-Password := "test8"
test9 Cleartext-Password := "test9"
```

---

## 🧪 FreeRADIUS Testi

### Debug Modunda Çalıştırma

```bash
freeradius -X
```

### radtest ile EAP-MD5 Doğrulama

```bash
radtest -t eap-md5 test1 test1 1.1.1.2 0 switch1
```

| Parametre | Değer | Açıklama |
|-----------|-------|----------|
| `-t eap-md5` | EAP-MD5 | Kimlik doğrulama yöntemi |
| `test1` | Kullanıcı adı | |
| `test1` | Parola | |
| `1.1.1.2` | RADIUS sunucu IP | |
| `0` | NAS port | |
| `switch1` | Shared secret | |

---

## 🚀 TRex Başlatma

### Terminal 1 — TRex Çalıştırma

```bash
./t-rex-64 -i -c 1 --software --emu
```

### Terminal 2 — TRex Console

```bash
./trex-console --emu
```

---

## 📋 TRex dot1x Test Adımları

### 1. dot1x Profili Yükleme

2 namespace, her birinde 5 istemci:

```
trex> emu_load_profile -f emu/simple_dot1x.py -t --ns 2 --clients 5
```

### 2. Tüm İstemcileri Görüntüleme

```
trex> emu_show_all
```

## 📋 TRex Switch Test Adımları

### 1. ARP Tablosunu Görüntüleme

```
VM-IOSvL2-1> show arp
```

### 1. dot1x İstatistiklerini Görüntüleme

```
VM-IOSvL2-1> show dot1x statistics
```

### 3. dot1x Oturumlarını Görüntüleme

```
VM-IOSvL2-1> show dot1x all
```

---

## 📡 802.1X EAP Akışı

```
TRex (Supplicant)        Switch (Authenticator)      FreeRADIUS (Auth Server)
        |                         |                           |
        |<--- EAPOL Start --------|                           |
        |--- EAP Identity Req --->|                           |
        |<-- EAP Identity Resp ---|                           |
        |                         |--- RADIUS Access-Req ---->|
        |                         |<-- RADIUS Access-Chall ---|
        |<-- EAP Challenge -------|                           |
        |--- EAP Response ------->|                           |
        |                         |--- RADIUS Access-Req ---->|
        |                         |<-- RADIUS Access-Accept --|
        |<-- EAP Success ---------|                           |
        |   [ Port Authorized ]   |                           |
```

---

## 🔍 Faydalı Komutlar

| Komut | Açıklama |
|-------|----------|
| `freeradius -X` | FreeRADIUS'u debug modunda başlat |
| `radtest -t eap-md5 <user> <pass> <ip> <port> <secret>` | EAP-MD5 test isteği gönder |
| `emu_show_all` | Tüm namespace ve istemci bilgilerini göster |
| `show arp` | ARP tablosunu görüntüle |
| `show dot1x statistics` | dot1x istatistiklerini göster |
| `show dot1x all` | Tüm dot1x oturumlarını listele |

---

## 📝 Notlar

- TRex EMU `simple_dot1x.py` profili **802.1X supplicant** rolünü simüle etmektedir.
- `--ns 2 --clients 5` ile 2 namespace × 5 istemci = **toplam 10 dot1x oturumu** oluşturulur.
- FreeRADIUS shared secret, switch `clients.conf` tanımıyla eşleşmelidir.
- EAP-MD5 yöntemi test amaçlıdır; üretim ortamında EAP-TLS veya PEAP önerilir.
- TRex'i `--software` moduyla başlatmak, desteklenmeyen NIC'lerde yazılım tabanlı paket işlemeyi etkinleştirir.
