# 03 · TCO & Deployment · H8Mart Quick-Commerce Ops

**Nama:** Yusuf Baihaqi Amanda
**Tanggal verifikasi sumber harga:** 19 November 2026
**Status:** *estimasi untuk proposal Milestone*, bukan tagihan aktual AWS; tidak mencakup pajak dan biaya yang tergantung pemakaian berlebih.
**Horizon TCO:** 12 bulan (730 jam per bulan). **Kurs simulasi:** Rp16.000 per USD; bukan kurs pasar yang diklaim berlaku pada tanggal dokumen.

## 1. Deployment aktual dan kebutuhan sistem

H8Mart telah diuji pada **AWS EC2 `t3.small` (2 vCPU, 2 GiB RAM), region `ap-southeast-2` / Sydney, Ubuntu 24.04**, menggunakan Docker Compose. Service utama: **n8n, PostgreSQL, mock API, task runners, Caddy HTTPS, Uptime Kuma**. Domain HTTPS produksi: `https://h8mart-yusuf.duckdns.org`. Telegram digunakan untuk digest, approval refund, dan alert. WF2 menyajikan tiga MCP tools baca-saja dan WF3 memakai OpenAI untuk pertanyaan merchant.

Kontainer aplikasi dan PostgreSQL berjalan dalam satu VM, sehingga arsitektur ini sesuai **demonstrasi atau pilot dengan beban terbatas**, tetapi belum merupakan desain *high availability*. Kegagalan satu VM berpotensi memutus seluruh layanan. Ukuran dan jenis disk EBS yang benar-benar terpasang **belum dikonfirmasi dari AWS**, maka perhitungan berikut mengasumsikan **30 GiB gp3**. Perhatikan perbedaan antara **status Published** lima workflow dan **Schedule Trigger WF1 yang sengaja dinonaktifkan setelah pengambilan evidence** untuk mencegah banjir pesan selama penilaian; jangan mengklaim job 15 menit dan digest 07.00 sedang aktif bila memang belum diaktifkan kembali.

## 2. Dasar dan asumsi TCO (harga diperiksa 19 November 2026)

Perbandingan ini menggunakan **tiga model deployment berbeda**, bukan tiga penyedia VPS. Horizon 12 bulan, operasi 24/7, kurs simulasi **Rp16.000/USD** dan **Rp17.500/EUR** (keduanya asumsi, bukan kurs aktual). Pajak dan biaya tak terduga dikecualikan. LLM WF3 dipertahankan **US$0,048/bulan = Rp768/bulan** sesuai simulasi dokumen 02; bukan tagihan penggunaan sebenarnya.

**Kebutuhan kapasitas:** jadwal sinkronisasi tiap 15 menit menghasilkan `4 × 24 × 30 = 2.880` eksekusi/bulan; digest harian menambah sekitar 30, belum termasuk webhook order, WF3, WF4, maupun error WF5. Angka ini merupakan estimasi saat schedule aktif terus-menerus. Ketika schedule dinonaktifkan selama penilaian, pemakaian aktual lebih rendah. **n8n Cloud Starter tidak memenuhi kebutuhan 2.910+ eksekusi/bulan** karena kuotanya 2.500 eksekusi/bulan. Pilihan pembanding n8n Cloud adalah **Pro, 10.000 eksekusi/bulan**, dengan asumsi total beban H8Mart tetap di bawah kuota tersebut. Ini harus dikaji ulang jika volume order dan merchant bertambah.

| Sumber input | Nilai untuk model | Status / catatan |
| --- | --- | --- |
| n8n Cloud Pro | **€50/bulan, ditagih tahunan**; 10.000 executions/bulan | Tarif publik **n8n resmi** pada opsi penagihan tahunan, bukan harga kontrak aktual |
| n8n Cloud Starter | €20/bulan, ditagih tahunan; 2.500 executions/bulan | **Tidak cocok** untuk 2.910+ execution/bulan dari schedule saja |
| Self-hosted n8n Community | Biaya lisensi langsung **US$0** | Community tersedia gratis untuk self-hosting, tunduk Sustainable Use License; tidak identik dengan fitur Business/Enterprise |
| AWS EC2 `t3.small` Sydney | US$0,0264/jam × 730 = US$19,272/bulan | **Estimasi regional** dari versi TCO sebelumnya; verifikasi angka aktual melalui AWS Pricing Calculator sebelum pengadaan |
| AWS EBS gp3 Sydney | 30 GiB × US$0,096 = US$2,880/bulan | **Asumsi** volume 30 GiB; ukuran disk nyata belum dikonfirmasi |
| Public IPv4 AWS | US$0,005/jam × 730 = US$3,650/bulan | Mengacu pada daftar harga **AWS resmi** |
| AWS backup + trafik/overhead | US$2,50 + US$2,00/bulan | **Allowance anggaran**, bukan invoice AWS |
| On-premise hardware | **Rp6.000.000 pembelian di awal** | **Skenario asumsi**, belum ada penawaran vendor atau spesifikasi pengadaan resmi; mencakup unit host sederhana untuk pilot |
| On-premise listrik | 50 W × 730 jam × Rp1.500/kWh = **Rp54.750/bulan** | Konsumsi dan tarif listrik **asumsi**, belum diukur atau disesuaikan golongan tarif |
| On-premise koneksi, admin, backup | Rp150.000 + Rp100.000 + Rp50.000/bulan | **Alokasi biaya tambahan ilustratif**; bukan quotation resmi; belum mencakup ruang, UPS, pendingin, atau dukungan enterprise |

**Sumber harga dan produk resmi (diakses 19 November 2026):**

1. [n8n — Plans and Pricing](https://n8n.io/pricing/) — Cloud Starter/Pro, batas execution, pilihan penagihan tahunan, dan Community Edition.
2. [n8n — Which license do I need?](https://support.n8n.io/article/can-i-use-your-license-for-my-use-case) — Sustainable Use License pada Community self-hosted.
3. [AWS — EC2 On-Demand Pricing](https://aws.amazon.com/ec2/pricing/on-demand/) — struktur harga VM On-Demand; angka regional perlu dikalkulasi sesuai konfigurasi.
4. [AWS — VPC Pricing](https://aws.amazon.com/vpc/pricing/) — US$0,005/jam per alamat IPv4 publik.
5. [AWS Pricing Calculator](https://calculator.aws/) — verifikasi kuotasi AWS regional yang sebenarnya sebelum keputusan pengadaan.

**Keterbatasan pembandingan:** n8n Cloud adalah layanan terkelola di lingkungan penyedia; pada self-hosted AWS kita mengoperasikan n8n, DB, reverse proxy dan monitoring dalam VM sendiri; on-premise membutuhkan perangkat dan operasi lokal. Perbedaan tanggung jawab, ketersediaan, support, latensi dan fitur tidak dinilai hanya dengan harga. Kuota Cloud harus mencakup *semua* eksekusi yang dihitung paket, bukan hanya lima workflow utama.

## 3. TCO tiga model deployment selama 12 bulan

| Komponen | **A. n8n Cloud Pro** | **B. Self-hosted AWS — aktual H8Mart** | **C. On-premise — simulasi** |
| --- | ---: | ---: | ---: |
| Platform/lisensi | €50 × 12 = **€600/tahun** | Community: Rp0 lisensi | Community: Rp0 lisensi |
| Komputasi & storage | Tercakup langganan Cloud n8n | US$19,272 + US$2,880 per bulan | Hardware Rp6.000.000 di awal |
| IPv4, backup, trafik tambahan | Dalam paket n8n Cloud untuk layanan n8n; integrasi/DB eksternal **belum dianggarkan** | US$3,650 + US$2,50 + US$2,00 per bulan | Listrik Rp54.750/bulan; jaringan Rp150.000/bulan; admin Rp100.000/bulan; backup Rp50.000/bulan |
| LLM WF3 (simulasi) | Rp768/bulan | US$0,048/bulan | Rp768/bulan |
| **Biaya berulang per bulan dalam IDR** | **Rp875.768** | **Rp485.600** | **Rp355.518** |
| **Biaya awal yang dimodelkan** | Rp0 | Rp0 | **Rp6.000.000** |
| **TCO 12 bulan** | **Rp10.509.216** | **Rp5.827.200** | **Rp10.266.216** |
| Kelengkapan perbandingan | **Parsial:** DB, mock API, Kuma, integrasi pihak ketiga serta kebutuhan hosting pelengkap belum diberi anggaran | **Baseline deployment yang diuji:** tetap estimasi, bukan invoice | **Parsial:** hardware dan biaya operasi hipotetis; belum dihitung UPS, penggantian perangkat, redundansi/DR |

**Rumus yang dapat direproduksi:**

- **Cloud Pro:** `(€50 × 12 × Rp17.500/EUR) + (12 × Rp768) = Rp10.509.216`. Tarif €50/bulan merupakan harga *annual billing*; tidak boleh diklaim sama dengan harga pembayaran bulanan tanpa komitmen.
- **Self-hosted AWS:** `[(0,0264 × 730) + (30 × 0,096) + (0,005 × 730) + 2,50 + 2,00 + 0,048] × 12 × Rp16.000/USD = Rp5.827.200`. Nilai **Rp484.832/bulan sebelum LLM** dan **Rp768/bulan LLM** mempertahankan dasar perhitungan ROI dokumen 02.
- **On-premise:** `Rp6.000.000 + 12 × (Rp54.750 + Rp150.000 + Rp100.000 + Rp50.000 + Rp768) = Rp10.266.216`. Harga perangkat dan operasional semuanya **asumsi skenario**, bukan penawaran resmi. Biaya listrik, internet dan SDM harus disesuaikan dengan kapasitas dan lokasi nyata.

**Interpretasi:** Perbandingan angka di atas **bukan hasil tender** dan tidak sepenuhnya *apple-to-apple*. Cloud Pro mungkin memerlukan layanan PostgreSQL/Mock API/Uptime Kuma yang dikelola terpisah agar H8Mart tetap setara; jika belum dihitung, TCO Cloud cenderung **underestimate**. Perbandingan yang lebih lengkap memerlukan kuotasi layanan terkelola tambahan dan pengukuran beban. Pada on-premise, amortisasi hardware dalam jangka lebih panjang akan mengubah perbandingan; 12 bulan sengaja dipakai sesuai tugas. Untuk milestone, **pertahankan opsi B** karena sudah berjalan dan pengujian tersedia.

### Risiko dan sensitivitas

- Jika eksekusi workflow naik melewati 10.000/bulan, n8n Cloud Pro dapat memerlukan upgrade atau biaya tambahan sesuai paket yang berlaku; kapasitas memori/CPU AWS juga mungkin perlu ditambah.
- EC2 T3 bersifat *burstable*: kredit CPU, transfer data, snapshot, log, pajak, dan biaya pemulihan bisa membuat tagihan aktual melampaui baseline.
- On-premise memiliki risiko listrik, ISP, kerusakan perangkat, keamanan fisik, backup dan recovery. Nilai overhead SDM dalam contoh sangat rendah sehingga **tidak layak menjadi proyeksi komersial tanpa quotation**.
- Kurs EUR dan USD sama-sama memengaruhi perbandingan; gunakan kurs aktual serta tanggal transaksi untuk proposal pengadaan nyata.

## 4. Pemilihan solusi dan keputusan deployment

**Pilihan implementasi Milestone: opsi B — self-hosted AWS EC2 `t3.small`.** Pilihan ini didasarkan pada **deployment yang sudah berfungsi dan sudah memiliki evidence**, bukan karena AWS termurah. Untuk penilaian, risiko migrasi dan konfigurasi ulang lebih besar daripada potensi penghematan VPS pada horizon singkat. Pada tahap produksi sebenarnya, keputusan perlu dibandingkan ulang dengan kebutuhan kapasitas, lokasi data, ketahanan, support, dan SLA yang disepakati.

**Pemetaan layanan:**

| Komponen | Implementasi | Pertimbangan |
| --- | --- | --- |
| Orkestrasi | n8n dalam Docker Compose | Lima workflow tersedia di instance self-hosted |
| Penyimpanan | PostgreSQL dalam container/volume | Seed merchant/order serta tabel Ops; belum HA |
| Reverse proxy & HTTPS | Caddy + DuckDNS | HTTPS publik untuk webhook dan callback approval |
| Integrasi eksternal | Mock Merchant/Payment/Notification API, Telegram, OpenAI | Kredensial dibuat di n8n / env, bukan dalam repo publik |
| Monitoring | Uptime Kuma push heartbeat + WF5 | Dapat memantau gangguan eksplisit dan silent failure; jadwal WF1 sementara nonaktif sesuai prosedur evidence |
| Akses operasional | SSH key untuk Ubuntu dan antarmuka n8n terproteksi | Batasi Security Group, rotasi token setelah kebocoran, hindari publikasi SSH key |
| MCP | WF2 read-only via PostgreSQL role `mcp_reader` | MCP sebaiknya hanya diakses dari n8n/internal network; wajib audit exposure jaringan |

## 5. Deployment, kontrol, dan prosedur verifikasi

1. **Environment:** gunakan `compose.yaml` dan `compose.vps.yaml`, set variabel dalam `.env` lokal yang tidak di-commit. Jangan menuliskan kunci API, token bot, password PostgreSQL, SSH private key, atau token penilaian dalam GitHub maupun evidence.
2. **Persistent state:** jaga volume PostgreSQL dan n8n agar data tetap ada setelah container di-recreate. Simpan backup terenkripsi di lokasi terpisah bila akan digunakan lebih lama.
3. **Ingress:** hanya HTTPS reverse proxy Caddy yang dibuka ke internet untuk endpoint yang dibutuhkan; batasi SSH (22) ke alamat administrator; Uptime Kuma UI diakses melalui port-forward SSH, bukan membuka port 3001 ke publik.
4. **Workflow:** lima WF dalam status Published; konfirmasi trigger webhook dapat menerima event penilaian. **Schedule 15 menit dan 07.00 belum aktif pada kondisi terakhir**, sengaja untuk mencegah banjir notifikasi; jelaskan status kepada penilai dan rencanakan aktivasi ulang sesuai arahan mentor.
5. **Smoke test terarah:** gunakan webhook produksi dengan event dari `fixtures/`, periksa hasil status HTTP dan jejak execution; jangan men-trigger refund nyata berulang. Verifikasi koneksi MCP internal, Telegram, dan Error Workflow dengan evidence yang sudah didokumentasikan.
6. **Uji pemulihan (belum dibuktikan):** backup SQL/volume, restore di lingkungan terpisah, audit output; catat waktu pulih (RTO/RPO) sebelum dinyatakan production-ready.
7. **Audit pasca-deploy:** cek masa berlaku domain/HTTPS, penggunaan RAM, disk, billing alert, konsumsi token LLM, error rate, dan risiko akses antarmerchant.

### Status bukti saat penyusunan

| Kriteria | Status berdasarkan pengujian sebelumnya | Bukti / gap |
| --- | --- | --- |
| AWS EC2 + Compose + Caddy HTTPS | **Terbukti** | `evidence/deploy/` memuat instance AWS, service Docker, HTTPS n8n |
| Bot Telegram dan approval | **Terbukti untuk jalur yang diuji** | `evidence/wf1/`, `evidence/wf4/`, `evidence/wf5/` |
| Workflow webhook 5 WF Published | **Terbukti Published** | Screenshot/eksport n8n; pengujian per workflow di `evidence/wf*/` |
| Monitoring Uptime Kuma | **Terpasang; UP/DOWN telah disimulasikan secara terkontrol.** Expression WF1 direvisi agar 11/12 merchant tidak otomatis DOWN; bukti end-to-end ekspresi versi baru masih perlu diverifikasi | `evidence/wf1/13-kuma-heartbeat-up.png`, `14-kuma-heartbeat-down.png`; jangan klaim uptime 100% |
| Backup restorasi / HA / DR | **Belum diuji / tidak termasuk lingkup Milestone** | Risiko dan rencana tindak lanjut |
| Biaya AWS faktual | **Belum diverifikasi terhadap AWS Billing** | TCO adalah proyeksi daftar harga/allowance, bukan invoice |
| Otorisasi merchant per tenant | **Belum dipenuhi end-to-end** | `merchant_id` webhook perlu diikat ke identitas user terautentikasi sebelum production real |

## 6. Keterkaitan dengan estimasi ROI dokumen 02

Dokumen `02-desain-solusi-roi.md` **telah diselaraskan** menggunakan biaya platform **self-hosted AWS** (opsi B) **US$30,302/bulan**, yaitu Rp484.832/bulan sebelum LLM, dan biaya LLM ilustratif Rp768/bulan. Perubahan perbandingan menjadi Cloud vs self-hosted vs on-premise **tidak mengubah baseline AWS untuk ROI** karena deployment yang benar-benar diuji tetap AWS.
Dengan asumsi manfaat bruto dokumen 02 Rp1.683.333/bulan dan implementasi awal Rp2.000.000:

- **Net benefit diperbarui** ≈ Rp1.683.333 − Rp484.832 − Rp768 = **Rp1.197.733/bulan**.
- **Simple payback** ≈ Rp2.000.000 / Rp1.197.733 = **1,67 bulan**.
- **ROI 12 bulan** ≈ `((12 × Rp1.197.733 − Rp2.000.000) / Rp2.000.000) × 100%` = **618,64%** (hasil sedikit berbeda jika menggunakan angka pecahan penuh).

Nilai tersebut tetap **simulasi**, bukan bukti realisasi ROI.

## 7. Regulasi dan kontrol kepatuhan Indonesia

H8Mart adalah **skenario fiktif dengan data sintetis**. Rujukan berikut menjelaskan kewajiban **yang harus dinilai bila solusi diterapkan pada bisnis nyata**, bukan klaim bahwa pilot saat ini sudah patuh secara hukum.

| Regulasi / pasal resmi | Kewajiban atau isu relevan | Penerapan / gap pada H8Mart |
| --- | --- | --- |
| **UU No. 27 Tahun 2022 Pasal 35** (Pelindungan Data Pribadi) | Pengendali wajib menerapkan langkah teknis-operasional dan tingkat keamanan sesuai risiko data pribadi | HTTPS, pembatasan akses, kontrol credential, enkripsi backup dan audit hak akses diperlukan; bukti pilot belum mencakup audit komprehensif |
| **UU No. 27 Tahun 2022 Pasal 46** | Jika terjadi kegagalan pelindungan data pribadi, pengendali wajib memberi pemberitahuan tertulis paling lambat **3 × 24 jam** kepada subjek data dan lembaga, dengan isi minimum yang ditentukan | WF5 Telegram alert adalah notifikasi operasional **bukan pengganti proses pelaporan insiden hukum**; perlu prosedur respons insiden dan penanggung jawab |
| **UU No. 27 Tahun 2022 Pasal 56** | Transfer data pribadi ke luar wilayah Indonesia memerlukan pemenuhan kondisi/tingkatan perlindungan yang diatur UU | AWS Sydney dan n8n Cloud (EU) perlu **analisis transfer lintas negara** bila menyimpan data pribadi nyata; data pilot masih sintetis |
| **PP No. 71 Tahun 2019 Pasal 21** | PSE Lingkup Privat boleh mengelola/memroses/menyimpan data di dalam atau di luar Indonesia dengan kewajiban memastikan efektivitas pengawasan dan penegakan hukum | Lokasi AWS Sydney harus dicatat dalam data map dan diuji kesesuaiannya untuk konteks organisasi; jangan menyimpulkan semua data wajib berada di Indonesia |
| **PP No. 71 Tahun 2019 Pasal 22–24** | Rekam jejak audit dan pengamanan sistem elektronik, termasuk prosedur untuk menghindari gangguan/kegagalan | Execution log, dead-letter, WF5 dan Kuma membantu; masih perlu retensi log, pembatasan akses, pengujian backup/restore, dan kontrol perubahan |

**Sumber hukum resmi (diakses 19 November 2026):**

- [JDIH Kemkomdigi — UU Nomor 27 Tahun 2022](https://jdih.komdigi.go.id/produk_hukum/view/id/832/t/undangundang%20nomor%2027%20tahun%202022)
- [JDIH Kemkomdigi — PP Nomor 71 Tahun 2019](https://jdih.komdigi.go.id/produk_hukum/view/id/695/t/peraturan%20pemerintah%20nomor%2071%20tahun%202019)
- [Pendaftaran PSE Lingkup Privat — FAQ resmi](https://pse.komdigi.go.id/pertanyaan-umum) (periksa apakah entitas dan model operasinya masuk cakupan pendaftaran; tidak otomatis disimpulkan dari simulasi)

**Kepatuhan belum dinyatakan PASS.** Perlu data inventory, penetapan pengendali/prosesor, dasar pemrosesan, penilaian risiko, perjanjian penyedia/transfer, retensi, prosedur breach, dan tinjauan aturan sektoral saat migrasi ke produksi nyata.

## 8. Rekomendasi tahap lanjut

Pertahankan AWS deployment untuk penilaian dan dokumentasikan URL, jalur webhook, serta token penilaian **hanya dalam DM pribadi kepada instruktur**. Sebelum penggunaan bisnis nyata, prioritaskan autentikasi tenant, restore test, observabilitas biaya, least privilege, HTTPS/ingress audit, dan evaluasi kapasitas 2 GiB. Jangan menampilkan credential, token, IP privat, maupun password dalam repo publik.
