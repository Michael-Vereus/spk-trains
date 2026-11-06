# Catatan spk

`
  Jalanin spk.sql dulu buat tabel dan data database nya 
`

Arti tabel di schema spk.sql : 
1. roles -> buat nyimpen peran (admin, engineer, supervisor, test, technician)
2. subroles -> buat nyimpen catatan project / perawatan kereta
3. user -> literally nyimpen tabel user / pengguna
4. forms -> template cetak checksheet (dalam bahasa simple literally template form)
5. submissions -> hasil pengumpulan form 
6. user_notifications dan webauthn_credentials cuma tabel buat nyimpen notifikasi + krendesial -> engga relevan buat spk imo


Hmm harus e buat SPK bakal kebanyakan make data dari table forms dan submissions buat ngambil data terukur buat nentuin masalah yg bisa diangkat untuk sistem SPK

user, role dan subroles
MariaDB [spk]> select u.name, u.role, r.description, sr.name , sr.description from users u join roles r on u.role_id = r.id join subroles sr on JSON_CONTAINS(u.subrole_ids, CAST(sr.id AS CHAR));
+----------------+------------+-----------------------------------------------+-------------------+----------------------------------------------+
| name           | role       | description                                   | name              | description                                  |
+----------------+------------+-----------------------------------------------+-------------------+----------------------------------------------+
| Agus Pratama   | engineer   | Engineering role with project-specific access | LRT Jabodebek     | Proyek perawatan sarana LRT Jabodebek        |
| Siti Rahmawati | engineer   | Engineering role with project-specific access | KRL Commuter Line | Proyek perawatan rangkaian KRL Commuter Line |
| Siti Rahmawati | engineer   | Engineering role with project-specific access | KA Bandara        | Proyek perawatan kereta api bandara          |
| Hendra Wijaya  | technician | Technician role with project-specific access  | LRT Jabodebek     | Proyek perawatan sarana LRT Jabodebek        |
| Eko Prasetyo   | technician | Technician role with project-specific access  | LRT Jabodebek     | Proyek perawatan sarana LRT Jabodebek        |
| Eko Prasetyo   | technician | Technician role with project-specific access  | KRL Commuter Line | Proyek perawatan rangkaian KRL Commuter Line |
| Yusuf Maulana  | technician | Technician role with project-specific access  | KRL Commuter Line | Proyek perawatan rangkaian KRL Commuter Line |
| Dimas Aditya   | technician | Technician role with project-specific access  | KA Bandara        | Proyek perawatan kereta api bandara          |
+----------------+------------+-----------------------------------------------+-------------------+----------------------------------------------+


# Ide masalah yang diangkat buat SPK : 
## SPK untuk skor kelayakan Siap Operasi (SO) dan Tidak Siap Operasi (TSO)

Masalah : di tabel subrole kan bisa berisi banyak proyek perawatan kereta, nah spk e ini mau narget yg mana? 
MariaDB [spk]> select id, title from forms;
+----+-----------------------------------+
| id | title                             |
+----+-----------------------------------+
|  1 | PEMERIKSAAN BERKALA SARANA LRT    |
|  2 | PEMERIKSAAN BERKALA RANGKAIAN KRL |
|  3 | PENGUKURAN RODA DAN BOGIE KRL     |
|  4 | PEMERIKSAAN HARIAN KERETA BANDARA |
|  5 | INSPEKSI K3 DEPO DAN WORKSHOP     |
|  6 | FORM UJI COBA                     |
+----+-----------------------------------+
6 rows in set (0.001 sec)

- hmmm rasa e mending form ke lima dan enam engga usa

Okeh kalo gitu make yg form 1 aja (Pemeriksaan Berkala Sarana LRT)
Berarti judulnya nanti jadi : 
# Sistem Pendukung Keputusan Penentuan Kesiapan Operasi (SO/TSO) Rangkaian LRT Menggunakan Metode SMART

Sekarang detailin form pemeriksaan berkala sarana lrt
## A. Header (disimpan di `submissions.header_values`)

| Kunci | Pertanyaan | Tipe jawaban | Nilai | Catatan |
|---|---|---|---|---|
| `header_0` | Tgl. Pemeriksaan | date | `YYYY-MM-DD` | diisi teknisi |
| `header_1` | No. Rangkaian | teks | bebas, mis. `TS-03` | diisi teknisi |
| `header_2` | UPT. DIPO | statis | `Depo LRT Bekasi Timur` | tetap, tidak diisi |

## B. Pemeriksaan (disimpan di `submissions.answers`)

| ID | Bagian | Pertanyaan | Standar | Tipe jawaban | Nilai yang diizinkan | Diisi di | Bentuk penyimpanan |
|---|---|---|---|---|---|---|---|
| `_i02` | Sistem Pintu Penumpang | Fungsi buka-tutup pintu penumpang | Membuka dan menutup halus, tanpa bunyi abnormal | `oknok` | `OK`, `NOK` | TC1, M1, M2, TC2 | `{"_t01":"OK","_t02":"OK",...}` |
| `_i03` | Sistem Pintu Penumpang | Sensor penghalang pintu (door obstruction) | Pintu membuka kembali saat terhalang | `oknok_na` | `OK`, `NOK`, `NA` | TC1, M1, M2, TC2 | `{"_t01":"OK",...}` |
| `_i04` | Sistem Pintu Penumpang | Karet seal pintu (sub-item) | Tidak robek, tidak getas | `oknok` | `OK`, `NOK` | TC1, M1, M2, TC2 | `{"_t01":"OK",...}` |
| `_i07` | Sistem Pengereman (Rem Pneumatik) | Tekanan Main Reservoir (MR) | 8,0 - 10,0 bar | `input_num` | angka, satuan bar | M1, M2 | `{"_t02":"8.5","_t03":"8.3"}` |
| `_i08` | Sistem Pengereman (Rem Pneumatik) | Tekanan Brake Pipe (BP) | 4,8 - 5,2 bar | `input_num` | angka, satuan bar | TC1, M1, M2, TC2 | `{"_t01":"5.1",...}` |
| `_i09` | Sistem Pengereman (Rem Pneumatik) | Ketebalan kampas rem (brake pad) | Minimal 10 mm | `input_num_multi` | 2 angka per gerbong (Axle 1, Axle 2), satuan mm | TC1, M1, M2, TC2 | `{"_t01":[{"value":"14.5","label":"Axle 1"},{"value":"18.4","label":"Axle 2"}],...}` |
| `_i11` | Sistem Traksi dan Kelistrikan | Tegangan baterai (110 VDC) | Minimal 100 VDC | `input_num` | angka, satuan VDC | TC1, TC2 | `{"_t01":"107.7","_t04":"109.1"}` |
| `_i12` | Sistem Traksi dan Kelistrikan | Kondisi kabel jumper dan konektor antar kereta | Tidak ada kabel terkelupas, konektor terkunci | `oknok` | `OK`, `NOK` | TC1, M1, M2, TC2 | `{"_t01":"OK",...}` |
| `_i13` | Sistem Traksi dan Kelistrikan | Tahanan isolasi motor traksi | Lebih dari 10 MOhm | `input_num` | angka, satuan MOhm | M1, M2 | `{"_t02":"32.1","_t03":"30.1"}` |
| `_i15` | Kabin dan Interior | Fungsi lampu penerangan interior | Seluruh lampu menyala normal | `oknok` | `OK`, `NOK` | TC1, M1, M2, TC2 | `{"_t01":"OK",...}` |
| `_i16` | Kabin dan Interior | Fungsi AC (HVAC) kabin penumpang | Pendinginan normal, tidak ada bunyi abnormal | `oknok_na` | `OK`, `NOK`, `NA` | TC1, M1, M2, TC2 | `{"_t01":"OK","_t02":"NA",...}` |
| `_i17` | Kabin dan Interior | Kondisi kursi dan handgrip (sub-item) | Kokoh, tidak longgar | `oknok` | `OK`, `NOK` | TC1, M1, M2, TC2 | `{"_t01":"OK",...}` |

## C. Isian tambahan per baris

| Kunci sel | Pertanyaan | Tipe jawaban | Nilai | Bentuk penyimpanan |
|---|---|---|---|---|
| `_t05` | Keterangan (tersedia di setiap baris pemeriksaan di atas) | teks | bebas, opsional | `"_t05":"Pad M1 axle 2 tipis"` (satu kunci bersama sel gerbong pada baris yang sama) |

## D. Bagian penutup

| ID | Pertanyaan | Tipe jawaban | Nilai | Bentuk penyimpanan |
|---|---|---|---|---|
| `_i18` | Tuliskan temuan di luar standar beserta tindak lanjutnya | catatan (teks) | bebas | `{"catatan":"Tidak ada temuan."}` |
| `_i19` | Berdasarkan hasil pemeriksaan, maka sarana tersebut diatas dinyatakan: | kesimpulan (pilihan) | `SO`, `TSO` | `{"text":"Berdasarkan hasil ...","selected":"SO"}` |

## E. Tanda tangan (disimpan di `submissions.signatures`)

| ID | Peran | Isian |
|---|---|---|
| `_g01` | Dibuat oleh, Teknisi Perawatan | nama, ID pegawai (`emp_id`), tanda tangan (`data_url`, gambar PNG base64) |
| `_g02` | Diperiksa oleh, Supervisor | nama, ID pegawai, tanda tangan |
| `_g03` | Disetujui oleh, Engineer | nama, ID pegawai, tanda tangan |


## Hmmm . . .


Scratch yg diatas kita pake SPK ini : 
    
# Sistem Pendukung Keputusan Penentuan Urgensi Perbaiki Sistem Pengereman di Rangkaian LRT Menggunakan Metode SMART

Data yang digunakan : 

| ID | Bagian | Pertanyaan | Standar | Tipe jawaban | Nilai yang diizinkan | Diisi di | Bentuk penyimpanan |
|--|--|--|--|--|--|--|--|
| `_i07` | Sistem Pengereman (Rem Pneumatik) | Tekanan Main Reservoir (MR) | 8,0 - 10,0 bar | `input_num` | angka, satuan bar | M1, M2 | `{"_t02":"8.5","_t03":"8.3"}` |
| `_i08` | Sistem Pengereman (Rem Pneumatik) | Tekanan Brake Pipe (BP) | 4,8 - 5,2 bar | `input_num` | angka, satuan bar | TC1, M1, M2, TC2 | `{"_t01":"5.1",...}` |
| `_i09` | Sistem Pengereman (Rem Pneumatik) | Ketebalan kampas rem (brake pad) | Minimal 10 mm | `input_num_multi` | 2 angka per gerbong (Axle 1, Axle 2), satuan mm | TC1, M1, M2, TC2 | `{"_t01":[{"value":"14.5","label":"Axle 1"},{"value":"18.4","label":"Axle 2"}],...}` |

Untuk gambaran kasar aplikasi ini penggunanya ada dua 
1. Petugas di lapangan 
2. Admin / Manager di kantor

Simpelnya 
Alur Petugas : 
- Petugas itu ngecek sistem rem udara di setiap rangkaian LRT, tiap kalo selesai ngecek 1 rangkaian langsung nyatet di aplikasi nya (mungking desainnya bakal kaya form gitu ya )
- Setiap kali user mengirimkna form / data maka secara otomatis sistem akan menghitung skor nya menggunakan metode SMART yang nanti hasil kalkulasi skor tersebut akan disimpan di database bersama data rangkaian LRT tsb.

Alur Admin : 
- Buka Aplikasi check rankingnya dan bisa ambil action untuk notifikasi Engineer reparasi.

Techstack : Laravel, MySQL

Harus normalisasikan data karena dalam skala dan satuan berbeda. 
Bobobt ? hmm bisa pake AHP untuk bobot atau asumsi sendiri kalo mau lebih simpel.

Bisa pake AHP buat bobot soale hasil dri AHP adalah matriks 1 dimensi yg isa dipake ke metode yg lain untuk perhitungan skor akhir.
