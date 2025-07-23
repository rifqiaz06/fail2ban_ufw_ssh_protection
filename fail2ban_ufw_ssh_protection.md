# Fail2Ban: Panduan Dasar (Ubuntu, SSH Protection)

## Apa Itu & Kenapa Pakai?

Fail2Ban merupakan tools log-based security yang otomatis mendeteksi multiple failed login attempts (brute force), dan blok IP via firewall (UFW/iptables).

---

## Instalasi di Ubuntu 22.04

```bash
sudo apt update
sudo apt install fail2ban ufw -y
```

---

## Konfigurasi Dasar (SSH)

Bikin file `/etc/fail2ban/jail.local`:

```ini
[sshd]
enabled   = yes
port      = ssh
logpath   = %(sshd_log)s
backend   = systemd
maxretry  = 3
findtime  = 10m
bantime   = 1h
ignoreip  = 127.0.0.1/8 192.168.0.0/24
```

---

## Jalankan & Monitoring

```bash
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
```

---

## Simulasi Brute Force

1. Login salah 3x via SSH
2. Cek block dengan `fail2ban-client status sshd`
3. Akhirnya muncul IP yang diblok, login ditolak (“Connection refused”)

---

## Tips & Best Practice

- Gunakan key-based auth (bukan password)
- Tambahin email alert opsional di config
- Pantau `fail2ban.log`:
  ```bash
  sudo tail -f /var/log/fail2ban.log
  ```
- Untuk unblock IP:
  ```bash
  sudo fail2ban-client set sshd unbanip 192.168.x.x
  ```
- Untuk block IP:
  ```bash
  sudo fail2ban-client set sshd banip 192.168.x.x
  ```

---

## Recheck: UFW vs UFW + Fail2Ban

| Skenario              | UFW               |        Fail2Ban        |
| --------------------- | ----------------  | ---------------------  |
| Buka port SSH         | ✅ Siapa pun bisa | ✅ Dipantau otomatis  |
| Brute force           | ❌ Nggak dicegah  | ✅ Auto ban IP        |
| Lihat percobaan login | ❌ Nggak bisa     | ✅ Bisa via log       |
| Whitelist IP          | ✅ Manual         | ✅ Via `ignoreip`     |
| Lihat IP penyerang    | ❌ Nggak ada log  | ✅ Dapat insight/log  |
| Botnet IP berubah     | ❌ Susah dicegah  | ✅ Tetap keblok       |
| Deteksi log layanan   | ❌ Gak bisa       | ✅ Bisa (SSH/FTP/etc) |
| Proteksi log-based    | ❌ Tidak ada      | ✅ Fitur utama        |
| Notifikasi email      | ❌ Nggak bisa     | ✅ Bisa diset         |

## Lanjut Ke…

- Menambahkan filter untuk nginx, postfix, dll
- Integrasi email / Slack alert
- Visualisasi log & alerting (Grafana, Prometheus)
