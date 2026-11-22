# ZeroTwin ESP8266 — tombol fisik + monitor HP

Evil-twin + deauther dua-dalam-satu untuk ESP8266, dikendalikan **dari tombol
fisik maupun dari browser HP**. Berbasis [ZeroTwin v1.0](https://github.com/shinyxn)
— logika deauth dan evil-twin-nya dipertahankan, bagian UI/IO yang dirapikan.

> **Peringatan:** memalsukan AP, memutus koneksi WiFi, dan menangkap kata sandi
> milik jaringan orang lain adalah tindakan ilegal. Pakai hanya di perangkat /
> jaringan milik sendiri.

## Apa yang beda dari ZeroTwin asli

| Aspek | Asli | Versi ini |
| :--- | :--- | :--- |
| Tombol fisik | tidak ada | ADC `A0` (rangkain resistor 5 tombol) atau pin digital, dengan debounce |
| OLED | log teks ber-`delay()` | layar status, refresh 1 detik, non-blocking |
| Waktu siap | `setup()` nge-blok ±40 detik | langsung siap (`delay()` 33 → 2) |
| Monitoring | reload panel | `GET /api/status` + baris status auto-refresh 2 detik |
| Stop semua | harus dua tombol | satu tombol **Stop Semua** |

## Fitur

- Scan AP di sekitar, pilih target dari tabel panel.
- **Deauth**: frame `0xC0` (deauth) + `0xA0` (disassoc) tiap 1 detik ke BSSID target.
- **Evil-Twin**: bikin AP dengan SSID target, DNS wildcard, halaman login tiruan.
- Password yang diketik dicoba sungguhan lewat `WiFi.begin(ssid, pass, ch, bssid)`;
  hasilnya tampil di panel dan OLED.
- Captive portal: `onNotFound` membalas panel, jadi HP otomatis membuka halaman.

## Tombol fisik

Set `#define BTN_ADC` di sketch: `1` = ADC `A0` (rangkain resistor), `0` = digital
`INPUT_PULLUP` di `BTN_PIN`.

| Tombol | ADC (kasar) | Aksi |
| :--- | :--- | :--- |
| UP | > 750 | target berikutnya |
| DOWN | 550–750 | scan ulang |
| OK | 350–550 | start/stop deauth |
| BACK | 150–350 | start/stop evil-twin |
| EXTRA | < 150 | stop semua |

Nilai ADC mentah tampil di OLED (`raw=`) dan di panel HP, jadi ambang bisa
dicocokkan dengan rangkaian yang benar-benar terpasang. Debounce: nilai harus
stabil 5 loop (~25 ms) dan pin harus balik idle sebelum tekan berikutnya —
pin mengambang tidak akan memicu aksi.

## Build & flash

Butuh core `esp8266` (diuji 3.1.2) + library `Adafruit SSD1306`, `Adafruit GFX`.

```bash
cd zerotwin
arduino-cli compile --fqbn esp8266:esp8266:nodemcuv2:eesz=4M2M,xtal=160 --build-path ./build .
arduino-cli upload -p /dev/ttyUSB0 --fqbn esp8266:esp8266:nodemcuv2:eesz=4M2M,xtal=160 --input-dir ./build
```

## Pakai dari HP

1. Sambung ke AP `ZeroTwin v1.0` (WPA2, password `zero8888` — ganti di sketch).
2. Buka `http://192.168.4.1` (biasanya kebuka sendiri karena captive portal).
3. Pilih target → **Start Deauth** / **Start Evil-Twin**.
4. Baris status di atas menampilkan uptime, heap, target, state, paket terkirim,
   tombol terakhir + nilai ADC, dan password terakhir yang masuk.

## Endpoint

```
GET  /                panel (atau halaman login saat twin aktif)
GET  /result          hasil percobaan password
GET  /api/status      up/heap/target/deauth/twin/pkt/btn/btnRaw/pass
POST /?ap=<bssid>     pilih target
POST /?deauth=start|stop
POST /?hotspot=start|stop
POST /?stop=1         stop semua
POST /?password=...   percobaan password (saat twin aktif)
```

## Kredit & lisensi

Base: **ZeroTwin v1.0** oleh [shinyxn](https://github.com/shinyxn). Modifikasi di
repo ini mengikuti ketentuan lisensi proyek aslinya — sertakan kredit ini kalau
mendistribusikan ulang. Hasil kompilasi: RAM 40%, flash 28% (core 3.1.2).
