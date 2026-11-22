# EduKidsOS

**EduKidsOS** adalah distro Linux bertema *Kids Explorer* untuk anak Sekolah Dasar (6–12 tahun). Distro ini dibuat dengan me-remaster **Linux Mint 22 XFCE** memakai **Cubic**, berisi aplikasi edukasi, tampilan ramah anak, suara startup, serta pembatasan situs web.

Dokumen ini berisi seluruh perintah Linux yang dipakai selama proses remastering, berurutan dari awal sampai pengecekan akhir.

---

## Daftar Isi

1. [Gambaran Umum](#gambaran-umum)
2. [Persiapan Aset](#persiapan-aset)
3. [Langkah 1 - Instal Cubic](#langkah-1---instal-cubic)
4. [Langkah 2 - Salin Aset ke Proyek Cubic](#langkah-2---salin-aset-ke-proyek-cubic)
5. [Langkah 3 - Instal Aplikasi Edukasi](#langkah-3---instal-aplikasi-edukasi)
6. [Langkah 4 - Wallpaper Default](#langkah-4---wallpaper-default)
7. [Langkah 5 - Logo Menu Start](#langkah-5---logo-menu-start)
8. [Langkah 6 - Suara Startup](#langkah-6---suara-startup)
9. [Langkah 7 - Pembatasan Situs (DNS, SafeSearch, Blokir Sosmed)](#langkah-7---pembatasan-situs)
10. [Langkah 8 - Boot Logo Plymouth](#langkah-8---boot-logo-plymouth)
11. [Langkah 9 - Pengecekan Otomatis](#langkah-9---pengecekan-otomatis)
12. [Catatan Penting](#catatan-penting)

---

## Gambaran Umum

| Item | Keterangan |
|------|------------|
| Nama distro | EduKidsOS 1.0 |
| Basis | Linux Mint 22 (XFCE Edition) |
| Alat remastering | Cubic (Custom Ubuntu ISO Creator) |
| Aplikasi utama | Tux Paint, Tux Math, Tux Typing, GCompris |
| Filter konten | DNS Cloudflare for Families, SafeSearch, blokir domain via `/etc/hosts` |

### Di mana perintah dijalankan?

| Langkah | Tempat menjalankan |
|---------|--------------------|
| 1 dan 2 | **Terminal biasa** di komputer host (bukan jendela Cubic) |
| 3 sampai 9 | **Terminal virtual Cubic** (lingkungan chroot sebagai root, makanya tanpa `sudo`) |

---

## Persiapan Aset

Siapkan tiga file gambar di folder `~/bahan-edukidsos/`:

```
~/bahan-edukidsos/
├── wallpaper.png     # wallpaper desktop
├── logo.png          # logo menu Start dan sistem
└── boot-logo.png     # logo saat boot (Plymouth)
```

Proyek Cubic diasumsikan berada di `~/remaster-EdukidsOS-final/`.

---

## Langkah 1 - Instal Cubic

Jalankan di **terminal biasa**. Perintah ini melewati instalasi bila Cubic sudah ada.

```bash
which cubic || (sudo apt-add-repository -y universe && \
  sudo apt-add-repository -y ppa:cubic-wizard/release && \
  sudo apt-get update && \
  sudo apt-get install -y --no-install-recommends cubic)
```

---

## Langkah 2 - Salin Aset ke Proyek Cubic

Jalankan di **terminal biasa** (bukan jendela Cubic). Satu blok perintah ini menyalin `wallpaper.png`, `logo.png`, dan `boot-logo.png` langsung ke sistem proyek Cubic.

```bash
PROJ=~/remaster-EdukidsOS-final/custom-root && \
sudo mkdir -p "$PROJ/usr/share/backgrounds/edukidsos" \
              "$PROJ/usr/share/icons/edukidsos" \
              "$PROJ/usr/share/pixmaps" \
              "$PROJ/tmp/edukids-assets" && \
sudo cp -fv ~/bahan-edukidsos/wallpaper.png "$PROJ/usr/share/backgrounds/edukidsos/wallpaper.png" && \
sudo cp -fv ~/bahan-edukidsos/logo.png      "$PROJ/usr/share/icons/edukidsos/logo.png" && \
sudo cp -fv ~/bahan-edukidsos/logo.png      "$PROJ/usr/share/pixmaps/edukidsos-logo.png" && \
sudo cp -fv ~/bahan-edukidsos/boot-logo.png "$PROJ/tmp/edukids-assets/boot-logo.png"
```

| File sumber | Lokasi di dalam sistem |
|-------------|------------------------|
| `wallpaper.png` | `/usr/share/backgrounds/edukidsos/wallpaper.png` |
| `logo.png` | `/usr/share/icons/edukidsos/logo.png` |
| `logo.png` | `/usr/share/pixmaps/edukidsos-logo.png` |
| `boot-logo.png` | `/tmp/edukids-assets/boot-logo.png` (sementara) |

---

## Langkah 3 - Instal Aplikasi Edukasi

Mulai langkah ini, jalankan di **terminal virtual Cubic**.

```bash
apt-get update && apt-get install -y \
  tuxpaint tuxmath tuxtype gcompris-qt \
  imagemagick ffmpeg pulseaudio-utils pavucontrol
```

| Paket | Fungsi |
|-------|--------|
| `tuxpaint` | Aplikasi menggambar untuk anak |
| `tuxmath` | Game berhitung |
| `tuxtype` | Game mengetik |
| `gcompris-qt` | Kumpulan game edukasi |
| `imagemagick` | Mengubah ukuran gambar (perintah `convert`) |
| `ffmpeg` | Konversi audio ke format OGG |
| `pulseaudio-utils` | Memutar suara (perintah `paplay`) |
| `pavucontrol` | Pengatur volume |

---

## Langkah 4 - Wallpaper Default

Menjadikan `wallpaper.png` sebagai wallpaper default desktop, baik lewat penggantian file bawaan Mint maupun konfigurasi XFCE untuk user baru (`/etc/skel`).

```bash
cp -f /usr/share/backgrounds/edukidsos/wallpaper.png \
      /usr/share/backgrounds/linuxmint/default_background.jpg && \
mkdir -p /etc/skel/.config/xfce4/xfconf/xfce-perchannel-xml
```

```bash
cat << 'EOF' > /etc/skel/.config/xfce4/xfconf/xfce-perchannel-xml/xfce4-desktop.xml
<?xml version="1.0" encoding="UTF-8"?>
<channel name="xfce4-desktop" version="1.0">
  <property name="backdrop" type="empty">
    <property name="screen0" type="empty">
      <property name="monitorVirtual1" type="empty">
        <property name="workspace0" type="empty">
          <property name="color-style" type="int" value="0"/>
          <property name="image-style" type="int" value="5"/>
          <property name="last-image" type="string" value="/usr/share/backgrounds/edukidsos/wallpaper.png"/>
        </property>
      </property>
      <property name="monitoreDP-1" type="empty">
        <property name="workspace0" type="empty">
          <property name="color-style" type="int" value="0"/>
          <property name="image-style" type="int" value="5"/>
          <property name="last-image" type="string" value="/usr/share/backgrounds/edukidsos/wallpaper.png"/>
        </property>
      </property>
    </property>
  </property>
</channel>
EOF
```

> `monitorVirtual1` dipakai pada mesin virtual, sedangkan `monitoreDP-1` untuk layar laptop. `image-style` bernilai `5` berarti *Zoomed*.

---

## Langkah 5 - Logo Menu Start

Mengganti logo sistem dan ikon tombol menu Whisker.

```bash
[ -f /usr/share/linuxmint/logo.png ] && \
  cp -f /usr/share/pixmaps/edukidsos-logo.png /usr/share/linuxmint/logo.png

mkdir -p /etc/skel/.config/xfce4/panel
```

```bash
cat << 'EOF' > /etc/skel/.config/xfce4/panel/whiskermenu-1.rc
button-icon=/usr/share/pixmaps/edukidsos-logo.png
button-title=EduKidsOS
show-button-title=true
show-button-icon=true
EOF
```

Terapkan ikon ke semua file konfigurasi Whisker yang ada:

```bash
for rc in /etc/skel/.config/xfce4/panel/whiskermenu*.rc /etc/xdg/xfce4/panel/whiskermenu*.rc; do
  [ -f "$rc" ] && sed -i 's|^button-icon=.*|button-icon=/usr/share/pixmaps/edukidsos-logo.png|' "$rc"
done
```

---

## Langkah 6 - Suara Startup

### 6.1 Membuat nada startup (Python) dan mengonversi ke OGG

Skrip membuat melodi 4 nada (C5, E5, G5, C6) dalam format WAV, lalu `ffmpeg` mengonversinya menjadi `desktop-login.ogg`.

```bash
python3 -c '
import wave, struct, math

rate = 44100
notes = [(523.25, 0.18), (659.25, 0.18), (783.99, 0.18), (1046.50, 0.55)]
frames = []

for freq, dur in notes:
    n = int(rate * dur)
    for i in range(n):
        env = math.exp(-3.5 * (i / n))
        val = int(28000 * env * math.sin(2 * math.pi * freq * (i / rate)))
        frames.append(struct.pack("<h", val))

with wave.open("/tmp/edukids-startup.wav", "w") as f:
    f.setnchannels(1); f.setsampwidth(2); f.setframerate(rate)
    f.writeframes(b"".join(frames))
' && ffmpeg -y -i /tmp/edukids-startup.wav -c:a libvorbis -q:a 5 \
      /usr/share/mint-artwork/sounds/desktop-login.ogg && \
rm -f /tmp/edukids-startup.wav
```

### 6.2 Memutar suara otomatis saat login

```bash
mkdir -p /etc/skel/.config/autostart && \
cat << 'EOF' > /etc/skel/.config/autostart/edukidsos-startup-sound.desktop
[Desktop Entry]
Type=Application
Name=EduKidsOS Startup Sound
Exec=paplay /usr/share/mint-artwork/sounds/desktop-login.ogg
OnlyShowIn=XFCE;
NoDisplay=true
EOF
```

---

## Langkah 7 - Pembatasan Situs

Tiga lapis pembatasan: **DNS ramah anak**, **SafeSearch paksa**, dan **blokir media sosial**.

### 7.1 DNS ramah anak (Cloudflare for Families, `1.1.1.3` / `1.0.0.3`)

DNS ini memblokir malware dan konten dewasa.

```bash
mkdir -p /etc/NetworkManager/conf.d /etc/systemd/resolved.conf.d
```

Cegah NetworkManager menimpa DNS:

```bash
cat << 'EOF' > /etc/NetworkManager/conf.d/edukidsos-dns.conf
[main]
dns=none
EOF
```

Atur `systemd-resolved`:

```bash
cat << 'EOF' > /etc/systemd/resolved.conf.d/edukidsos.conf
[Resolve]
DNS=1.1.1.3 1.0.0.3
FallbackDNS=1.1.1.3
EOF
```

Tulis `resolv.conf` langsung:

```bash
rm -f /etc/resolv.conf && cat << 'EOF' > /etc/resolv.conf
nameserver 1.1.1.3
nameserver 1.0.0.3
EOF
```

### 7.2 SafeSearch Google dan Bing + blokir media sosial

Perintah memakai `>>` (menambah) sehingga isi `/etc/hosts` yang lama tetap aman.

```bash
cat << 'EOF' >> /etc/hosts
# EduKidsOS SafeSearch Enforcement
216.239.38.120 www.google.com google.com www.google.co.id google.co.id
204.79.197.220 www.bing.com bing.com

# EduKidsOS Blokir Sosial Media (Instagram, Facebook, TikTok, Reddit)
0.0.0.0 instagram.com www.instagram.com m.instagram.com i.instagram.com
0.0.0.0 facebook.com www.facebook.com m.facebook.com web.facebook.com mobile.facebook.com l.facebook.com fb.com www.fb.com
0.0.0.0 tiktok.com www.tiktok.com m.tiktok.com vm.tiktok.com vt.tiktok.com
0.0.0.0 reddit.com www.reddit.com old.reddit.com new.reddit.com m.reddit.com i.redd.it v.redd.it
EOF
```

| Situs | Status |
|-------|--------|
| Google dan Bing | Dipaksa SafeSearch |
| Instagram | Diblokir |
| Facebook | Diblokir |
| TikTok | Diblokir |
| Reddit | Diblokir |

---

## Langkah 8 - Boot Logo Plymouth

Mengubah ukuran `boot-logo.png` menjadi 250x250, mengganti logo di tema Plymouth (file asli dicadangkan sebagai `.bak`), lalu membangun ulang initramfs.

```bash
convert /tmp/edukids-assets/boot-logo.png -resize 250x250 /tmp/edukids-assets/boot-logo-clean.png && \
for dir in /usr/share/plymouth/themes/mint-logo /usr/share/plymouth/themes/spinner; do
  if [ -d "$dir" ]; then
    for img in logo.png watermark.png; do
      if [ -f "$dir/$img" ]; then
        cp -n "$dir/$img" "$dir/$img.bak"
        cp -f /tmp/edukids-assets/boot-logo-clean.png "$dir/$img"
        echo "[OK] Boot logo terpasang di: $dir/$img"
      fi
    done
  fi
done && rm -rf /tmp/edukids-assets && apt-get clean && update-initramfs -u -k all
```

---

## Langkah 9 - Pengecekan Otomatis

**Semua baris harus berstatus `[OK]` sebelum menekan tombol Next di Cubic.**

```bash
echo "=== HASIL PENGECEKAN EDUKIDSOS 1.0 ==="

[ -f /usr/share/backgrounds/edukidsos/wallpaper.png ] && echo "[OK] Wallpaper" || echo "[GAGAL] Wallpaper"

[ -f /usr/share/pixmaps/edukidsos-logo.png ] && echo "[OK] Logo Start Menu" || echo "[GAGAL] Logo Start Menu"

([ -f /usr/share/plymouth/themes/mint-logo/logo.png.bak ] || [ -f /usr/share/plymouth/themes/spinner/watermark.png.bak ]) && echo "[OK] Boot Logo Plymouth" || echo "[GAGAL] Boot Logo Plymouth"

[ -f /usr/share/mint-artwork/sounds/desktop-login.ogg ] && echo "[OK] Suara Startup" || echo "[GAGAL] Suara Startup"

grep -q "1.1.1.3" /etc/resolv.conf && echo "[OK] DNS Ramah Anak" || echo "[GAGAL] DNS Ramah Anak"

grep -q "instagram.com" /etc/hosts && echo "[OK] Blokir IG/FB/TikTok/Reddit" || echo "[GAGAL] Blokir Situs"

which tuxpaint tuxmath tuxtype gcompris-qt >/dev/null && echo "[OK] Aplikasi Edukasi" || echo "[GAGAL] Aplikasi Edukasi"
```

Contoh keluaran yang diharapkan:

```
=== HASIL PENGECEKAN EDUKIDSOS 1.0 ===
[OK] Wallpaper
[OK] Logo Start Menu
[OK] Boot Logo Plymouth
[OK] Suara Startup
[OK] DNS Ramah Anak
[OK] Blokir IG/FB/TikTok/Reddit
[OK] Aplikasi Edukasi
```

---
