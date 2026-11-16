# TI4006_GA — Eksplorasi & Peningkatan Algoritma Genetika untuk Traveling Salesman Problem (TSP)

**Mata Kuliah:** TI4006 Algoritma Optimisasi dan Metaheuristik  
**Tugas:** Pertemuan 12 (Tugas Individu — Genetic Algorithm pada TSPLIB)  
**NIM:** `14423011`

Repositori ini berisi pengembangan dan evaluasi eksperimental **Algoritma Genetika (Genetic Algorithm / GA)** untuk menyelesaikan **Traveling Salesman Problem (TSP)** pada tiga instance benchmark standar **TSPLIB** (`eil51`, `berlin52`, dan `kroA100`), berbasiskan kerangka kode materi hands-on Pertemuan 11 (`P11_Hands_on_GA_TSP.ipynb`).

---

## Struktur Berkas di Repositori

| Nama Berkas | Deskripsi |
| :--- | :--- |
| `P11_Hands_on_GA_TSP.ipynb` | Notebook dasar materi hands-on Pertemuan 11 (`genetic_algorithm_tsp_v2`, `baca_tsplib`, operator dasar). |
| `AOM_GA_14423011.ipynb` | **Luaran Wajib 1:** Notebook hasil modifikasi (`use_M1`, `use_M2`, `use_M3`) yang telah dieksekusi penuh dari awal hingga akhir beserta tabel dan visualisasi pada ketiga instance. |
| `AOM_GA_Hasil_14423011.csv` | **Luaran Wajib 2:** Rekaman lengkap hasil eksperimen (`132` baris eksekusi) dengan format kolom `instance,seed,konfigurasi,jarak,waktu`. |
| `AOM_GA_Slide_14423011.pdf` | **Luaran Wajib 3:** Deck presentasi (`8` slide) berisi ringkasan masalah, modifikasi M1–M3 & rujukan, pseudocode, rancangan eksperimen, hasil tuning, studi ablasi 3 instance, tabel utama, kurva konvergensi, dan kesimpulan. |
| `eil51.tsp`, `berlin52.tsp`, `kroA100.tsp` | Berkas koordinat simpul benchmark TSPLIB (`EUC_2D`). |
| `berlin52.opt.tour`, `kroA100.opt.tour` | Berkas rute optimal referensi TSPLIB untuk verifikasi fungsi evaluasi jarak (`7542` dan `21282`). |

---

## Spesifikasi Lingkungan Komputasi

- **Sistem Operasi:** `Windows-11-10.0.26200-SP0`
- **Prosesor (CPU):** `12th Gen Intel(R) Core(TM) i9-12900H` (`20 logical cores`, `AMD64`)
- **Versi Python:** `Python 3.13.5` (64-bit)
- **Pustaka Utama:** `NumPy 2.5.2`, `Pandas 2.3.3`, `Matplotlib 3.11.1`
- **Standar Evaluasi Jarak:** Jarak Euclidean 2D (`EUC_2D`) dibulatkan ke bilangan bulat terdekat (`np.floor(d + 0.5)`) sesuai standar TSPLIB, dengan kota pertama pada berkas berperan sebagai depot (`simpul 0`).

| Instance | Jumlah Kota ($N$) | Nilai Optimal TSPLIB | Jenis Jarak | Status Verifikasi `.opt.tour` |
| :--- | :---: | :---: | :---: | :--- |
| `eil51` | 51 | `426` | `EUC_2D` | Referensi TSPLIB (`426`) |
| `berlin52` | 52 | `7542` | `EUC_2D` | Terverifikasi eksak dengan `berlin52.opt.tour` (`7542`) |
| `kroA100` | 100 | `21282` | `EUC_2D` | Terverifikasi eksak dengan `kroA100.opt.tour` (`21282`) |

---

## Rancangan Konfigurasi Baseline & Tiga Modifikasi (`use_M1`, `use_M2`, `use_M3`)

Fungsi utama `genetic_algorithm_tsp_v2` mempertahankan seluruh parameter asli dari notebook hands-on dan dilengkapi tiga sakelar boolean (`use_M1`, `use_M2`, `use_M3`) serta parameter `batas_waktu`:

1. **Konfigurasi Baseline (`use_M1=False, use_M2=False, use_M3=False`)**
   - Parameter asli hands-on: `ukuran_populasi = 200`, `max_generation = 300` (**60.000 evaluasi** per eksekusi), `crossover = position_based_crossover_cepat` (`crossover_rate = 0.7`), `mutasi = swap_mutation` (`mutation_rate = 0.05`), `ukuran_turnamen = 20`, `jumlah_elit = 0`.
2. **Modifikasi M1 — Paket Operator Struktur Rute & Memetic (`use_M1=True`)**
   - Mengaktifkan **Order Crossover (OX)** (Davis, 1985), **Inversion Mutation** (Croes, 1958), **Elitisme (`jumlah_elit = 2`)**, serta pemolesan lokal **Memetic 2-Opt terbatas** (`dua_opt`, *first-improvement* maks. 30 langkah) pada solusi terbaik generasi (Moscato, 1989). Sesuai ketentuan tugas, seluruh komponen dari materi hands-on ini dihitung sebagai **satu modifikasi (M1)**.
3. **Modifikasi M2 — Inisialisasi Heuristik Nearest Neighbor & Penolakan Klon Duplikat (`use_M2=True`)**
   - **Multi-Start Nearest Neighbor Seeding (10%):** Menginisialisasi 10% populasi awal (20 individu) menggunakan heuristik `bangun_tur_nearest_neighbor` (1 individu *greedy* NN dari depot 0 dan 19 individu *randomized 2-nearest neighbor* dari simpul awal berbeda), sedangkan 90% sisanya dibangkitkan secara acak (Talbi, 2009).
   - **Duplicate Rejection / Re-mutation:** Melacak himpunan kromosom unik (`himpunan_unik`) pada setiap generasi; anak yang identik dengan individu yang sudah ada di populasi baru dimutasi ulang (maks. 3 kali) sebelum diterima guna menjaga keragaman genetik (Talbi, 2009).
4. **Modifikasi M3 — Laju Mutasi Adaptif & Cataclysmic Double-Bridge Restart (`use_M3=True`)**
   - **Adaptive Mutation Rate:** Meningkatkan peluang mutasi secara proporsional terhadap jumlah generasi stagnan:
     $$\text{mr}_{\text{aktual}} = \min\left(0.65,\; \text{mutation\_rate} \times \left(1 + 1.5 \times \frac{\text{stagnasi}}{\text{ambang\_stagnasi}}\right)\right)$$
   - **Cataclysmic Restart dengan Double-Bridge Kick:** Saat solusi terbaik tidak membaik selama `ambang_stagnasi = 20` generasi berturut-turut, individu elit tetap dipertahankan sementara 60% populasi non-elit diinjeksi dengan hasil perturbasi 4-Opt non-sekuensial (`double_bridge_kick`) dari solusi terbaik (+ pemolesan `dua_opt` terbatas pada maksimal 6 kandidat) untuk melompati jebakan lokal 2-Opt (Eshelman, 1991 — CHC; Martin, Otto & Felten, 1991; Talbi, 2009).

---

## Ringkasan Hasil Eksperimen

### 1. Parameter Tuning pada `eil51` (Langkah 5 — Grid Search $3 \times 3$, Seed Tuning `[11, 22, 33]`, 27 Eksekusi)

Pengujian kisi (*grid search*) dilakukan pada konfigurasi `Lengkap (M1+M2+M3)` di instance `eil51` menggunakan 3 seed terpisah (`[11, 22, 33]`):
- `mutation_rate` $\in \{0.05, 0.15, 0.30\}$
- `ukuran_turnamen` $\in \{3, 5, 10\}$

| `mutation_rate` | `ukuran_turnamen` ($k$) | Terbaik | Rata-rata | Terburuk | Simpangan Baku | Gap Rata-rata (%) | Waktu Rata-rata (s) |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| `0.05` | `3` | 428 | 429.67 | 431 | 1.53 | 0.86% | 0.86s |
| `0.05` | `5` | 429 | 430.33 | 431 | 1.15 | 1.02% | 0.96s |
| `0.05` | `10` | 429 | 430.67 | 432 | 1.53 | 1.10% | 1.07s |
| `0.15` | `3` | 428 | 429.00 | 430 | 1.00 | 0.70% | 0.84s |
| `0.15` | `5` | 430 | 430.33 | 431 | 0.58 | 1.02% | 0.91s |
| `0.15` | `10` | 428 | 429.67 | 431 | 1.53 | 0.86% | 1.04s |
| **`0.30` (Terbaik)** | **`3` (Terbaik)** | **427** | **429.00** | **432** | **2.65** | **0.70%** | **0.88s** |
| `0.30` | `5` | 429 | 429.00 | 429 | 0.00 | 0.70% | 0.92s |
| `0.30` | `10` | 428 | 430.00 | 433 | 2.65 | 0.94% | 1.04s |

- **Kombinasi Parameter Terpilih:** `mutation_rate = 0.30` dan `ukuran_turnamen = 3` (rata-rata jarak terendah `429.00` dan solusi terbaik `427`).

---

### 2. Studi Ablasi dengan Parameter Terbaik (`mr = 0.30, k = 3`) pada Ketiga Instance (Langkah 4 — 5 Seed Utama `[42, 101, 202, 303, 404]`)

Studi ablasi menguji kelima konfigurasi (`Baseline (Tuned)`, `Lengkap`, `Tanpa M1`, `Tanpa M2`, `Tanpa M3`) dengan parameter terbaik hasil tuning (`mr = 0.30, k = 3`), serta menyertakan `Baseline (Default)` (`mr = 0.05, k = 20`) sebagai pembanding awal:

#### A. Instance `eil51` (Optimal = `426`)
| Konfigurasi | Parameter | M1 | M2 | M3 | Terbaik | Rata-rata | Terburuk | SD | Gap Terbaik (%) | Gap Rata-rata (%) | $\Delta$ Rata-rata vs Lengkap | Waktu (s) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Baseline (Default)** | `mr=0.05, k=20` | tidak | tidak | tidak | 548 | 601.00 | 635 | 32.73 | 28.64% | 41.08% | $+171.40$ ($+40.23\%$) | 1.02s |
| **Baseline (Tuned)** | `mr=0.30, k=3` | tidak | tidak | tidak | 551 | 604.20 | 658 | 40.15 | 29.34% | 41.83% | $+174.60$ ($+40.99\%$) | 0.68s |
| **Lengkap (M1+M2+M3)** | **`mr=0.30, k=3`** | **ya** | **ya** | **ya** | 428 | **429.60** | **431** | **1.52** | 0.47% | **0.85%** | **0.00 (Acuan)** | **0.88s** |
| **Tanpa M1** | `mr=0.30, k=3` | tidak | ya | ya | 472 | 485.80 | 506 | 13.63 | 10.80% | 14.04% | $+56.20$ ($+13.19\%$) | 0.72s |
| **Tanpa M2** | `mr=0.30, k=3` | ya | tidak | ya | **427** | 430.60 | 433 | 2.30 | **0.23%** | 1.08% | $+1.00$ ($+0.23\%$) | 0.79s |
| **Tanpa M3** | `mr=0.30, k=3` | ya | ya | tidak | 430 | 433.20 | 436 | 2.77 | 0.94% | 1.69% | $+3.60$ ($+0.85\%$) | 0.75s |

#### B. Instance `berlin52` (Optimal = `7542`)
| Konfigurasi | Parameter | M1 | M2 | M3 | Terbaik | Rata-rata | Terburuk | SD | Gap Terbaik (%) | Gap Rata-rata (%) | $\Delta$ Rata-rata vs Lengkap | Waktu (s) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Baseline (Default)** | `mr=0.05, k=20` | tidak | tidak | tidak | 10591 | 11262.00 | 11770 | 474.13 | 40.43% | 49.32% | $+3720.00$ ($+49.32\%$) | 1.10s |
| **Baseline (Tuned)** | `mr=0.30, k=3` | tidak | tidak | tidak | 9308 | 10054.40 | 11016 | 719.98 | 23.42% | 33.31% | $+2512.40$ ($+33.31\%$) | 0.67s |
| **Lengkap (M1+M2+M3)** | **`mr=0.30, k=3`** | **ya** | **ya** | **ya** | **7542** | **7542.00** | **7542** | **0.00** | **0.00%** | **0.00%** | **0.00 (Acuan)** | **0.93s** |
| **Tanpa M1** | `mr=0.30, k=3` | tidak | ya | ya | 8521 | 8810.60 | 8980 | 196.23 | 12.98% | 16.82% | $+1268.60$ ($+16.82\%$) | 0.74s |
| **Tanpa M2** | `mr=0.30, k=3` | ya | tidak | ya | **7542** | 7645.80 | 7828 | 143.36 | **0.00%** | 1.38% | $+103.80$ ($+1.38\%$) | 0.82s |
| **Tanpa M3** | `mr=0.30, k=3` | ya | ya | tidak | **7542** | 7712.00 | 7967 | 187.56 | **0.00%** | 2.25% | $+170.00$ ($+2.25\%$) | 0.83s |

#### C. Instance `kroA100` (Optimal = `21282`)
| Konfigurasi | Parameter | M1 | M2 | M3 | Terbaik | Rata-rata | Terburuk | SD | Gap Terbaik (%) | Gap Rata-rata (%) | $\Delta$ Rata-rata vs Lengkap | Waktu (s) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Baseline (Default)** | `mr=0.05, k=20` | tidak | tidak | tidak | 59295 | 61988.60 | 64677 | 2333.32 | 178.62% | 191.27% | $+40624.60$ ($+190.89\%$) | 1.45s |
| **Baseline (Tuned)** | `mr=0.30, k=3` | tidak | tidak | tidak | 56986 | 62759.60 | 67213 | 4683.94 | 167.77% | 194.90% | $+41395.60$ ($+194.51\%$) | 1.08s |
| **Lengkap (M1+M2+M3)** | **`mr=0.30, k=3`** | **ya** | **ya** | **ya** | **21282** | **21364.00** | **21446** | **80.82** | **0.00%** | **0.39%** | **0.00 (Acuan)** | **2.07s** |
| **Tanpa M1** | `mr=0.30, k=3` | tidak | ya | ya | 26605 | 27160.00 | 27484 | 416.24 | 25.01% | 27.62% | $+5796.00$ ($+27.23\%$) | 2.44s |
| **Tanpa M2** | `mr=0.30, k=3` | ya | tidak | ya | **21282** | 21485.00 | 21735 | 163.53 | **0.00%** | 0.95% | $+121.00$ ($+0.57\%$) | 2.99s |
| **Tanpa M3** | `mr=0.30, k=3` | ya | ya | tidak | 21634 | 21994.20 | 22252 | 252.65 | 1.65% | 3.35% | $+630.20$ ($+2.96\%$) | 2.76s |

---

### 3. Tabel Utama Perbandingan Baseline, Equal-Time Budget, dan Konfigurasi Terbaik pada 3 Instance (5 Seed Utama)

| Instance | Optimal | Konfigurasi | Terbaik | Rata-rata | Terburuk | Simpangan Baku | Gap Terbaik (%) | Gap Rata-rata (%) | Waktu Rata-rata (s) |
| :--- | :---: | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **`eil51`** | 426 | Baseline (Default `mr=0.05, k=20`) | 548 | 601.00 | 635 | 32.73 | 28.64% | 41.08% | 1.02s |
| **`eil51`** | 426 | Baseline (Tuned `mr=0.30, k=3`) | 551 | 604.20 | 658 | 40.15 | 29.34% | 41.83% | 0.68s |
| **`eil51`** | 426 | Baseline_EqualTime (53.1k eval) | 548 | 602.80 | 635 | 32.71 | 28.64% | 41.50% | 0.88s |
| **`eil51`** | 426 | **Lengkap (M1+M2+M3, `mr=0.30, k=3`)** | **428** | **429.60** | **431** | **1.52** | **0.47%** | **0.85%** | **0.88s** |
| **`berlin52`** | 7542 | Baseline (Default `mr=0.05, k=20`) | 10591 | 11262.00 | 11770 | 474.13 | 40.43% | 49.32% | 1.10s |
| **`berlin52`** | 7542 | Baseline (Tuned `mr=0.30, k=3`) | 9308 | 10054.40 | 11016 | 719.98 | 23.42% | 33.31% | 0.67s |
| **`berlin52`** | 7542 | Baseline_EqualTime (56.4k eval) | 10591 | 11303.60 | 11942 | 521.69 | 40.43% | 49.88% | 0.93s |
| **`berlin52`** | 7542 | **Lengkap (M1+M2+M3, `mr=0.30, k=3`)** | **7542** | **7542.00** | **7542** | **0.00** | **0.00%** | **0.00%** | **0.93s** |
| **`kroA100`** | 21282 | Baseline (Default `mr=0.05, k=20`) | 59295 | 61988.60 | 64677 | 2333.32 | 178.62% | 191.27% | 1.45s |
| **`kroA100`** | 21282 | Baseline (Tuned `mr=0.30, k=3`) | 56986 | 62759.60 | 67213 | 4683.94 | 167.77% | 194.90% | 1.08s |
| **`kroA100`** | 21282 | Baseline_EqualTime (90.0k eval) | 54738 | 57872.40 | 60506 | 2424.32 | 157.20% | 171.93% | 2.07s |
| **`kroA100`** | 21282 | **Lengkap (M1+M2+M3, `mr=0.30, k=3`)** | **21282** | **21364.00** | **21446** | **80.82** | **0.00%** | **0.39%** | **2.07s** |

---

## Cara Menjalankan Ulang (Reproducibility)

1. Pastikan berkas `eil51.tsp`, `berlin52.tsp`, `kroA100.tsp`, `berlin52.opt.tour`, dan `kroA100.opt.tour` berada pada direktori kerja yang sama dengan `AOM_GA_14423011.ipynb`.
2. Buka `AOM_GA_14423011.ipynb` di Jupyter Notebook / JupyterLab / VS Code dan pilih **Run All**.
3. Seluruh tabel statistik, grafik multi-instance, dan berkas `AOM_GA_Hasil_14423011.csv` akan otomatis dihasilkan ulang dengan hasil jarak yang identik karena menggunakan *fixed random seeds* (`[11, 22, 33]` untuk parameter tuning dan `[42, 101, 202, 303, 404]` untuk evaluasi utama).

---

## Rujukan Ilmiah

1. **Croes, G. A. (1958).** A method for solving traveling-salesman problems. *Operations Research*, 6(6), 791–812.
2. **Davis, L. (1985).** Applying adaptive algorithms to epistatic domains. *Proceedings of the 9th International Joint Conference on Artificial Intelligence (IJCAI'85)*, 162–164.
3. **Eshelman, L. J. (1991).** The CHC adaptive search algorithm: How to have safe search when engaging in nontraditional genetic recombination. *Foundations of Genetic Algorithms (FOGA)*, 1, 265–283.
4. **Martin, O., Otto, S. W., & Felten, E. W. (1991).** Large-step Markov chains for the traveling salesman problem. *Complex Systems*, 5(3), 299–326.
5. **Moscato, P. (1989).** On evolution, search, optimization, genetic algorithms and martial arts: Towards memetic algorithms. *Caltech Concurrent Computation Program, C3P Report*, 826.
6. **Talbi, E.-G. (2009).** *Metaheuristics: From Design to Implementation*. John Wiley & Sons.
