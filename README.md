# TIMER MAXDIS — Java Native Android

Project Android Studio berbasis **Java native**. Tidak menggunakan WebView.

## Fitur

- Alur enam lorong: L01–L06.
- Input Qty dan mulai dengan tombol ENTER pada keyboard angka.
- Rumus timer per lorong: `(Qty / 12) × 60 detik` = **Qty × 5 detik** (dihitung dengan bilangan bulat agar tepat). Qty valid: 1–100.000.
- Stepper Input → Proses → Selesai.
- Popup timer, ring countdown, video display (dibisukan), dan tombol LANJUT setelah waktu habis.
- Alarm berulang dari `res/raw/notifikasi.mp3` sampai pengguna menekan LANJUT.
- Rekap total Qty, shift, waktu standar, waktu kerja, gap, waktu aktual, deviasi, dan waktu selesai.
- Foreground Service dengan notifikasi berjalan dan logo aplikasi selama countdown / alarm aktif.
- Alarm sistem exact (bila pengguna mengizinkan) untuk membangunkan perangkat saat layar mati / Doze.
- Deadline tersimpan di SharedPreferences. Jika proses aplikasi dihentikan oleh sistem, service bisa dipulihkan. Jika perangkat reboot, receiver membaca deadline dan melanjutkan atau menandai timer selesai.

## Batasan penting

- **Layar mati / aplikasi ditinggalkan:** timer tetap berjalan karena deadline dihitung dari jam sistem dan dijaga Foreground Service + AlarmManager.
- **Perangkat benar-benar dimatikan:** tidak ada kode aplikasi yang bisa berjalan saat perangkat kehilangan daya. Setelah perangkat dinyalakan kembali, `BOOT_COMPLETED` memulihkan status. Jika deadline sudah lewat ketika perangkat mati, aplikasi menandai lorong selesai saat boot dan memulai alarm setelah sistem mengizinkan aplikasi berjalan.
- Android dapat membatasi aplikasi jika pengguna memilih **Paksa berhenti / Force stop**, mematikan izin notifikasi, mencabut Alarms & reminders, atau kebijakan penghemat baterai vendor membatasi aplikasi. Force stop tidak dapat dilewati oleh aplikasi.
- Android 12+ meminta izin khusus **Alarms & reminders** untuk alarm presisi. Jika ditolak, aplikasi memakai fallback alarm non-presisi dan sistem bisa menundanya ketika Doze.
- Izin notifikasi diminta di Android 13+. Notifikasi foreground service tetap wajib dibuat; agar terlihat di panel notifikasi, pengguna harus mengizinkan notifikasi aplikasi.
- Service menggunakan tipe foreground `specialUse` karena countdown pengguna yang berjalan lama tidak masuk tipe data-sync/media. Jika kelak dipublikasikan ke Google Play, jelaskan use-case ini di deklarasi foreground service Play Console dan ikuti kebijakan yang berlaku.

## Membuka project

1. Ekstrak ZIP.
2. Android Studio → **Open** → pilih folder `TIMER_MAXDIS_JAVA` (folder yang berisi `settings.gradle`).
3. Gunakan JDK 17.
4. Pastikan Android SDK Platform 35 dan Build Tools 35 terpasang.
5. Tunggu Gradle Sync selesai.
6. Jalankan ke HP Android atau pilih **Build → Build APK(s)**.

## Build melalui GitHub Actions

Workflow tersedia di `.github/workflows/android.yml`. Upload seluruh project ke repository GitHub, lalu jalankan workflow **Build TIMER MAXDIS APK**. APK debug diunggah sebagai artifact `TIMER-MAXDIS-debug-apk`.

## Izin yang disarankan pada HP

1. Izinkan notifikasi TIMER MAXDIS.
2. Izinkan **Alarms & reminders** saat diminta.
3. Pada Samsung, setelan baterai aplikasi dapat diubah ke **Unrestricted / Tidak dibatasi** bila perangkat tetap menunda proses latar belakang. Nama menu bisa berbeda sesuai versi One UI.
4. Jangan gunakan **Paksa berhenti** ketika timer sedang berjalan.

## Struktur inti

- `MainActivity.java` — tampilan native dan alur input/rekap.
- `TimerService.java` — Foreground Service, notifikasi berjalan, countdown dan alarm MP3.
- `TimerState.java` — penyimpanan sesi dan deadline.
- `TimerLogic.java` — rumus timer, validasi input Qty, dan kalkulasi rekap.
- `AlarmReceiver.java` — alarm tepat waktu.
- `BootReceiver.java` — pemulihan setelah reboot.
- `NextActionReceiver.java` — tombol LANJUT dari notifikasi.
- `PANDUAN_UPLOAD_GITHUB.md` — langkah upload ke GitHub dan build APK otomatis.
- `LAPORAN_PERBAIKAN.md` — daftar bug yang diperbaiki dan alasannya.

## Catatan pengujian

Unit test ada di `app/src/test/java/id/maxdis/timer/`:

- `TimerLogicTest` — rumus durasi (semua Qty 1–100.000), validasi input, target, format waktu, batas shift, rekap dan deviasi.
- `TimerStateTest` — mesin status sesi (start, selesai, lanjut, sesi 6 lorong penuh, klik ganda, alarm terlambat).

Jalankan: `gradle :app:testDebugUnitTest` (atau `./gradlew ...` bila wrapper sudah dibuat) atau lewat GitHub Actions. Lihat `HASIL_PENGUJIAN.txt` untuk apa yang sudah dan belum diverifikasi. APK belum pernah di-build di lingkungan pengembangan source ini (tanpa Android SDK); lakukan build lewat Android Studio / GitHub Actions dan uji di perangkat Samsung/Android target.
