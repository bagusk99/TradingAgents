# IDX Backtest — Rencana & Aturan Wajib

Proyek sampingan di fork TradingAgents. **Terpisah dari kode TradingAgents** (folder `idx/` saja).
Tujuan: strategi saham IDX berbasis rumus (hardcode), diuji jujur ke belakang sebelum pakai uang asli.

## Konteks user
- Pemula di trading & strategi. Jelaskan pakai bahasa sederhana + contoh, jangan asumsikan istilah dipahami.
- Prioritas utama user: **tidak ada salah hitung / data diam-diam salah**. Akurasi > kecepatan.
- Broker: IPOT (eksekusi manual, tidak ada API trading publik untuk ritel).

## Keputusan yang sudah diambil
| Hal | Keputusan | Alasan |
|---|---|---|
| Pasar | Saham IDX, long-only | Ritel praktis tidak bisa short |
| LLM | **Tidak dipakai** di tahap 1. Semua logika hardcode | Deterministik, bisa diulang, bebas bocoran pengetahuan LLM |
| Data | yfinance (`.JK`), benchmark `^JKSE` | Gratis |
| Timeframe | Data harian, rebalance mingguan | Fee IDX besar, sering trading = profit habis |
| Strategi awal | Momentum 3–6 bulan (skip 1 bulan terakhir) + filter tren IHSG (di bawah MA200 → kas) | Sederhana, banyak diteliti |
| Modal (virtual) | Dua skenario: **Rp 100.000** dan **Rp 10.000.000** | User ingin uji modal ekstrem 100k; 10jt sebagai pembanding |
| Periode | 1–3 tahun ke belakang | |
| Fee | Beli 0.19%, jual 0.29% (dibulatkan ke atas, konservatif) | Mendekati IPOT + pajak jual 0.1% |

## Output yang diminta
Satu halaman HTML (plotly):
1. Kurva saldo program vs beli-IHSG-lalu-diam vs kas.
2. Chart harga asli per saham + penanda ▲ beli / ▼ jual dari program.
3. Tabel: total return, max drawdown, akurasi sinyal, total fee, **daftar data yang dikecualikan + alasannya**.

## ATURAN WAJIB anti salah hitung (jangan dilonggarkan)
### A. Tanpa look-ahead
1. **Universe point-in-time**: JANGAN pakai daftar LQ45 hari ini (survivorship bias). Tiap minggu saring saham likuid pakai data volume/nilai transaksi s/d minggu itu saja, dari daftar ticker IDX selebar mungkin.
2. Sinyal dari close hari keputusan; eksekusi di **open hari bursa berikutnya**.
3. Tes otomatis: potong data di tanggal X → sinyal tanggal X harus identik dengan run data penuh.

### B. Hitungan uang realistis
4. Lot 100 lembar, pembulatan ke bawah, fee dipotong, sisa kas dicatat.
5. ARA/ARB: buy di ARA = gagal, sell di ARB = gagal. **Tabel aturan ARA/ARB per periode harus diverifikasi dari sumber resmi, bukan ingatan.**
6. Suspensi: posisi terkunci, tidak bisa dijual.
7. Aksi korporasi (split, reverse split, rights, dividen): sinyal pakai harga adjusted; jumlah lembar & dividen sebagai kejadian nyata.

### C. Validasi data
8. Cek sebelum pakai: lonjakan > batas ARA/ARB tanpa aksi korporasi, harga datar + volume 0, hari hilang, harga ≤ 0.
9. Data bermasalah → dikeluarkan untuk periode itu **dan dicatat di laporan**. Tidak ada drop diam-diam.
10. Cek silang sampel harga yfinance dengan data resmi IDX.

### Keterbatasan yang diketahui (tulis di laporan)
- Saham delisting mungkin hilang dari yfinance (sisa survivorship bias).
- Kualitas yfinance IDX tidak sempurna.
- Apa pun yang tidak terverifikasi ditulis "tidak terverifikasi", bukan dianggap benar.

## Peta jalan
1. **[sekarang]** Backtest rumus murni + laporan HTML.
2. Jika lolos (kalahkan IHSG setelah fee, drawdown wajar): tambah filter LLM untuk Keterbukaan Informasi IDX — LLM hanya parser teks → JSON, keputusan tetap rumus.
3. Paper trading beberapa bulan.
4. Uang kecil.

## Ekspektasi jujur (sudah disampaikan ke user)
- Hasil paling mungkin: mirip IHSG, lebih baik saat trending, lebih buruk saat sideways.
- Modal 100k: hanya muat 1–2 saham harga Rp 200–500, tanpa diversifikasi; saham ≤ Rp 100 = gorengan/illiquid.

## Status
- [ ] Verifikasi tabel ARA/ARB historis
- [ ] Kumpulkan daftar ticker IDX lebar
- [ ] Loader data + validasi
- [ ] Engine backtest + tes anti-look-ahead
- [ ] Laporan HTML
