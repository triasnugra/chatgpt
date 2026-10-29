# Forward test CryptoEliz

Tujuan: menilai mesin pada data yang **tidak mungkin dicemari** oleh keputusan desain, yaitu sinyal yang dicatat
sebelum hasilnya ada. Backtest historis sudah terlanjur "terlihat"; ini satu-satunya uji yang bersih.

## Cara kerja (otomatis, tidak perlu melakukan apa pun)
1. Saat `scripts/jurnal.mjs` pertama kali jalan dengan kode ini, ia membuat `data/forward-protocol.json`:
   tanggal mulai, **hash mesin** (core.mjs + trade-sim.mjs), aturan, syarat evaluasi, dan kriteria lulus.
   File ini tidak pernah ditimpa.
2. Setiap sinyal baru disegel dengan **rantai hash** dan dicap versi mesinnya.
3. Setelah jatuh tempo, hasil dinilai otomatis dengan aturan yang sama dengan backtester, lengkap dengan waktu keluar
   dan dua **baseline** (MARKET dan MIRROR) pada saat yang sama.
4. `scripts/forward.mjs report` (dijalankan tiap run) menulis `data/forward-report.md` dan `.json`.
   Angka kinerja **disembunyikan** sampai syarat terpenuhi.

## Syarat evaluasi dan kriteria lulus (tetap)
- Minimal **150 trade terisi** (setelah satu order aktif per koin) dan **20 minggu berbeda** berisi trade.
  Dimulai 14 Okt 2026, 20 minggu paling cepat tercapai sekitar awal Maret 2027 (ditambah 7 hari untuk matang).
- Lulus bila batas bawah CI 95% (bootstrap per minggu) rata-rata R per trade, setelah biaya, **lebih dari 0**.
- Selisih vs baseline MARKET dilaporkan sebagai informasi nilai tambah logika, bukan syarat lulus.

## Aturan yang harus dipatuhi
- Jangan mengubah `core.mjs` atau `trade-sim.mjs`. Hash berubah dan baris baru tidak dihitung. Untuk uji ide baru:
  hapus `data/forward-protocol.json` (protokol baru dimulai dari nol) dan tulis hipotesisnya dulu.
- Jangan mengubah syarat evaluasi setelah dimulai. Jangan mengedit atau menghapus baris jurnal.
- Jangan mengevaluasi sebelum syarat terpenuhi. `node scripts/forward.mjs report --peek` ada untuk keingintahuan,
  tetapi hasilnya diberi label **TIDAK SAH**. Catatan: panel statistik lama di halaman web tetap menampilkan
  rata-rata berjalan. Abaikan panel itu untuk keputusan.

## Verifikasi
```
node scripts/forward.mjs verify               # cek rantai hash; cetak hash ujung ke ringkasan Actions
node scripts/forward.mjs verify --regrade     # + nilai ulang semua hasil dari data Binance
node scripts/test-forward.mjs                 # 56 uji otomatis (tanpa jaringan)
```
Atau lewat GitHub: Actions > "Forward test - verifikasi penuh" > Run workflow.

## Yang dijamin dan batasnya
- Mengedit sinyal (entry/sl/tp/alasan/waktu/arah/mesin) atau menghapus baris di tengah: **terdeteksi** rantai hash.
- Mengedit hasil: terdeteksi oleh `--regrade` (bukan rantai, karena hasil diisi belakangan).
- Menghapus baris **terakhir**: tidak terdeteksi rantai. Ditutup oleh hash ujung yang dicetak ke ringkasan tiap run
  Actions (stempel waktu di luar repo). Bandingkan sesekali.
- Pemilik repo secara teknis bisa menghitung ulang seluruh rantai. Bukti waktu yang kuat adalah riwayat commit GitHub
  dan hash ujung di ringkasan Actions. Yang dilindungi di sini terutama **kamu dari dirimu sendiri**.
- Lulus bukan jaminan untung di masa depan: pasar berubah dan sampelnya terbatas. Naikkan ukuran posisi bertahap.


## Catatan upgrade kandidat Mesin 2 (29 Okt 2026)

Perubahan pada `core.mjs` mengubah hash mesin. Jika upgrade ini diterapkan ke repo aktif, protokol forward test yang lama akan tetap terkunci pada hash lama dan sinyal versi baru **tidak dihitung** di evaluasi lama. Jangan hapus atau timpa `data/forward-protocol.json` secara otomatis: simpan jurnal lama sebagai bukti historis, lalu mulai protokol baru secara eksplisit setelah menyetujui perubahan. Versi baru harus diuji terpisah.

Filter kandidat baru: (1) sinyal memerlukan pola plus konfirmasi independen tren/divergence atau beberapa alasan pola; (2) validasi simbol dan level harga; (3) tolak setup jika target sudah tercapai sebelum entry; (4) tolak entry yang lebih dari 5% dari harga acuan, SL di atas 15%, atau RR di bawah 1.5R. Penolakan dicatat sebagai `NO_TRADE` agar tidak mencemari win rate TP/SL.

Filter tersebut adalah hipotesis untuk mengurangi sinyal berkualitas rendah, bukan klaim bahwa win rate akan naik. Jalankan backtest dan forward test baru sebelum digunakan dengan uang nyata.
