# Catatan spk

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
