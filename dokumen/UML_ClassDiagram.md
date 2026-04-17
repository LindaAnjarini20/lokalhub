# Diagram Kelas UML untuk Sistem LokalHub

## Entitas dan Hubungannya

### Pengguna
- Pengguna adalah entitas utama dalam sistem, yang dapat berupa Konsumen, Penjual, Produsen, atau Administrator.

### Konsumen
- Seorang Pengguna yang melakukan pembelian produk.

### Penjual
- Seorang Pengguna yang menjual produk.

### Produsen
- Seorang Pengguna yang memproduksi barang untuk dijual.

### Administrator
- Seorang Pengguna yang mengelola sistem dan melakukan pemeliharaan berbagai entitas.

### Produk
- Barang yang dijual oleh Penjual dan diproduksi oleh Produsen.
- Hubungan: 
  - Penjual menjual Produk.
  - Produsen memproduksi Produk.

### Pesanan
- Proses pembelian yang dilakukan oleh Konsumen. Terkait dengan Produk.
- Hubungan: 
  - Konsumen melakukan Pesanan.
  - Pesanan terdiri dari satu atau lebih Produk.

### Review
- Ulasan yang diberikan oleh Konsumen terhadap Produk.
- Hubungan:
  - Konsumen memberikan Review untuk Produk.

### Pesan
- Komunikasi antara Konsumen dan Penjual. 
- Hubungan:
  - Konsumen mengirim Pesan kepada Penjual.

---

### Hubungan Antar Entitas
- **Konsumen** ----> **Pesanan**
- **Pesanan** ----> **Produk**
- **Konsumen** ----> **Review**
- **Konsumen** ----> **Pesan** ----> **Penjual**
- **Penjual** ----> **Produk**
- **Produsen** ----> **Produk**
- **Administrator** mengelola semua entitas di atas.