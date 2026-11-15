# 03 · TCO & Deployment · H8Mart Quick-Commerce Ops

**Nama:** Yusuf Baihaqi Amanda
**Tanggal estimasi harga:** 15 November 2026
**Status:** *estimasi untuk proposal Milestone*, bukan tagihan aktual AWS; tidak mencakup pajak dan biaya yang tergantung pemakaian berlebih.
**Horizon TCO:** 12 bulan (730 jam per bulan). **Kurs simulasi:** Rp16.000 per USD; bukan kurs pasar yang diklaim berlaku pada tanggal dokumen.

## 1. Deployment aktual dan kebutuhan sistem

H8Mart telah diuji pada **AWS EC2 `t3.small` (2 vCPU, 2 GiB RAM), region `ap-southeast-2` / Sydney, Ubuntu 24.04**, menggunakan Docker Compose. Service utama: **n8n, PostgreSQL, mock API, task runners, Caddy HTTPS, Uptime Kuma**. Domain HTTPS produksi: `https://h8mart-yusuf.duckdns.org`. Telegram digunakan untuk digest, approval refund, dan alert. WF2 menyajikan tiga MCP tools baca-saja dan WF3 memakai OpenAI untuk pertanyaan merchant.

Kontainer aplikasi dan PostgreSQL berjalan dalam satu VM, sehingga arsitektur ini sesuai **demonstrasi atau pilot dengan beban terbatas**, tetapi belum merupakan desain *high availability*. Kegagalan satu VM berpotensi memutus seluruh layanan. Ukuran dan jenis disk EBS yang benar-benar terpasang **belum dikonfirmasi dari AWS**, maka perhitungan berikut mengasumsikan **30 GiB gp3**. Perhatikan perbedaan antara **status Published** lima workflow dan **Schedule Trigger WF1 yang sengaja dinonaktifkan setelah pengambilan evidence** untuk mencegah banjir pesan selama penilaian; jangan mengklaim job 15 menit dan digest 07.00 sedang aktif bila memang belum diaktifkan kembali.

## 2. Parameter perhitungan dan sumber

| Parameter | Angka | Dasar dan jenis angka |
| --- | ---: | --- |
| Jam komputasi bulanan | 730 jam | Asumsi standar pemodelan TCO: 24/7 sepanjang tahun, rata-rata 730 jam/bulan |
| Kurs konversi | Rp16.000/USD | **Asumsi simulasi**, agar konsisten dengan `docs/02-desain-solusi-roi.md` |
| AWS EC2 `t3.small`, Sydney | **US$0,0264/jam** | Harga On-Demand Linux regional yang dipublikasikan oleh DoiT Compute dan AWS-pricing; perlu verifikasi di AWS Pricing Calculator untuk tagihan riil |
| AWS EBS `gp3`, Sydney | **US$0,096/GiB-bulan** | Harga indikatif regional dari AWS-pricing; asumsi ukuran volume **30 GiB**, belum dikonfirmasi |
| AWS public IPv4 | **US$0,005/jam** | Daftar harga resmi Amazon VPC untuk alamat IPv4 publik |
| DigitalOcean Basic Droplet 1 vCPU, 2 GiB RAM | **US$12/bulan**, 50 GiB SSD | Harga publik resmi Droplet; kapasitas CPU tidak setara dengan AWS 2 vCPU, menjadi risiko perbandingan |
| Hetzner Cloud CX23, 2 vCPU, 4 GiB RAM | **US$6,49/bulan**, belum termasuk IPv4 | Tarif USD regional Eropa dari pengumuman perubahan harga efektif 15 Juni 2026; ketersediaan dan lokasi bisa berbeda |
| LLM untuk WF3 | **US$0,048/bulan** | **Asumsi perhitungan dokumen 02**, bukan tagihan atau token usage aktual: 120 pertanyaan, 180 ribu input dan 60 ribu output token/bulan; tarif model harus diverifikasi terpisah |
| Backup, trafik tambahan, IPv4 Hetzner | Tercantum per opsi | **Allowance/asumsi anggaran**, bukan kutipan harga layanan atau invoice |

Sumber yang ditinjau pada 15 November 2026:

1. AWS EC2 On-Demand: https://aws.amazon.com/ec2/pricing/on-demand/
2. AWS EC2 `t3.small` regional: https://www.doit.com/compute/compute/aws/ap-southeast-2/t3.small
3. AWS EBS regional: https://aws-pricing.com/ap-southeast-2.html
4. AWS IPv4: https://aws.amazon.com/vpc/pricing/
5. DigitalOcean Droplet: https://www.digitalocean.com/pricing/droplets
6. Hetzner, penyesuaian harga 15 Juni 2026: https://docs.hetzner.com/general/infrastructure-and-availability/price-adjustment/

## 3. Perbandingan tiga opsi TCO (12 bulan)

Semua opsi diasumsikan menjalankan n8n, PostgreSQL, Caddy, task runners, mock API, dan Uptime Kuma dalam **satu VPS**. Domain DuckDNS, n8n Community self-hosted, PostgreSQL, serta Caddy diasumsikan tanpa biaya lisensi langsung. Angka tidak memasukkan biaya SDM/operasional, PPN, kemungkinan biaya transfer tambahan yang melampaui allowance, biaya pemulihan insiden, atau biaya implementasi awal. **Biaya LLM yang sama** dimasukkan pada ketiga opsi agar perbandingannya konsisten.

| Komponen (USD/bulan) | **A. AWS EC2 — aktual** | **B. DigitalOcean — alternatif** | **C. Hetzner CX23 — alternatif** |
| --- | ---: | ---: | ---: |
| VM / komputasi | 0,0264 × 730 = **19,272** | **12,000** | **6,490** |
| Disk utama | 30 × 0,096 = **2,880** | **0** (50 GiB SSD termasuk) | **0** (disk bawaan paket; pastikan detail paket saat pembelian) |
| IPv4 publik | 0,005 × 730 = **3,650** | **0** (diasumsikan termasuk) | **0,600** (*allowance*, verifikasi di checkout) |
| Backup / snapshot | **2,500** (*allowance*) | **2,400** (*allowance* 20% paket) | **1,300** (*allowance* 20% paket) |
| Trafik / operasional tambahan | **2,000** (*allowance*) | **1,000** (*allowance*) | **1,000** (*allowance*) |
| Pemakaian LLM WF3 | **0,048** (*asumsi*) | **0,048** (*asumsi*) | **0,048** (*asumsi*) |
| **Total estimasi/bulan** | **30,350** | **15,448** | **9,438** |
| **TCO 12 bulan (USD)** | **364,200** | **185,376** | **113,256** |
| **TCO 12 bulan (IDR, Rp16.000/USD)** | **Rp5.827.200** | **Rp2.966.016** | **Rp1.812.096** |

**Rumus umum:** `Biaya bulanan = komputasi + disk + IPv4 + backup + trafik/overhead + LLM`.
`TCO 12 bulan (USD) = 12 × biaya bulanan (USD)`.
`TCO 12 bulan (IDR) = TCO USD × Rp16.000`.

**Contoh AWS:** `(730 × 0,0264) + (30 × 0,096) + (730 × 0,005) + 2,50 + 2,00 + 0,048 = US$30,350/bulan` → `12 × 30,350 = US$364,20/tahun` → `364,20 × Rp16.000 = Rp5.827.200/tahun`.

### Kesetaraan dan sensitivitas biaya

- **Tidak apple-to-apple secara mutlak**: DigitalOcean 2 GiB pada harga US$12 memiliki **1 vCPU**, sedangkan AWS t3.small memiliki 2 vCPU *burstable*; Hetzner CX23 memiliki 4 GiB RAM tetapi berlokasi di Eropa pada tarif rujukan. Uji kapasitas, latensi API/Telegram, ketersediaan instance, dan kebijakan data sebelum migrasi.
- **Harga AWS dapat naik** akibat kredit CPU T3 Unlimited, EBS snapshot/IOPS tambahan, outbound transfer, perubahan disk, dan pajak. Estimasi tidak mengklaim mencakup seluruh biaya aktual.
- **Harga LLM hanya ilustratif**. Pantau token usage dan harga model yang dipakai melalui billing API/provider; besaran ini harus direvisi bila tarif model atau volume panggilan berbeda.
- **Biaya backup** dihitung sebagai allowance, bukan jaminan pemulihan. Backup PostgreSQL harus diuji melalui proses restore.

## 4. Pemilihan solusi dan keputusan deployment

**Pilihan implementasi Milestone: opsi A — AWS EC2 `t3.small`.** Pilihan ini didasarkan pada **deployment yang sudah berfungsi dan sudah memiliki evidence**, bukan karena AWS termurah. Untuk penilaian, risiko migrasi dan konfigurasi ulang lebih besar daripada potensi penghematan VPS pada horizon singkat. Pada tahap produksi sebenarnya, keputusan perlu dibandingkan ulang dengan kebutuhan kapasitas, lokasi data, ketahanan, support, dan SLA yang disepakati.

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
| Monitoring Uptime Kuma | **Terbukti dipasang, tetapi heartbeat bisa DOWN saat jadwal dimatikan atau merchant M009 gagal** | `evidence/deploy/04-uptime-kuma.png`; jangan klaim uptime 100% |
| Backup restorasi / HA / DR | **Belum diuji / tidak termasuk lingkup Milestone** | Risiko dan rencana tindak lanjut |
| Biaya AWS faktual | **Belum diverifikasi terhadap AWS Billing** | TCO adalah proyeksi daftar harga/allowance, bukan invoice |
| Otorisasi merchant per tenant | **Belum dipenuhi end-to-end** | `merchant_id` webhook perlu diikat ke identitas user terautentikasi sebelum production real |

## 6. Keterkaitan dengan estimasi ROI dokumen 02

Dokumen `02-desain-solusi-roi.md` telah diselaraskan dengan estimasi TCO AWS pada dokumen ini. Biaya platform yang digunakan adalah **US$30,302/bulan** atau **Rp484.832/bulan** pada kurs asumsi Rp16.000/USD, tidak termasuk biaya LLM sebesar **Rp768/bulan**. Dengan demikian, perhitungan ROI dan TCO menggunakan dasar biaya yang konsisten.

Dengan asumsi manfaat bruto dokumen 02 Rp1.683.333/bulan dan implementasi awal Rp2.000.000:

- **Net benefit diperbarui** ≈ Rp1.683.333 − Rp484.832 − Rp768 = **Rp1.197.733/bulan**.
- **Simple payback** ≈ Rp2.000.000 / Rp1.197.733 = **1,67 bulan**.
- **ROI 12 bulan** ≈ `((12 × Rp1.197.733 − Rp2.000.000) / Rp2.000.000) × 100%` = **618,64%** (hasil sedikit berbeda jika menggunakan angka pecahan penuh).

Nilai tersebut tetap **simulasi**, bukan bukti realisasi ROI.

## 7. Rekomendasi tahap lanjut

Pertahankan AWS deployment untuk penilaian dan dokumentasikan URL, jalur webhook, serta token penilaian **hanya dalam DM pribadi kepada instruktur**. Sebelum penggunaan bisnis nyata, prioritaskan autentikasi tenant, restore test, observabilitas biaya, least privilege, HTTPS/ingress audit, dan evaluasi kapasitas 2 GiB. Jangan menampilkan credential, token, IP privat, maupun password dalam repo publik.
