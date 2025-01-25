# Data-Analysis-for-E-Commerce-Challenge
This is a project from DQLab to analyze data using SQL

### Dataset yang digunakan
Dataset yang digunakan merupakan data dari DQLab Store yang memuat 4 tabel, berupa:
1. _users_ (detail data pengguna):
   * _user_id_, _nama_user_, _kodepos_, _email_
2. _products_ (detail produk yang dijual):
   * _product_id_, _desc_product_, _category_, _base_price_
3. _orders_ (transaksi pembelian):
   * _order_id_, _seller_id_, _buyer_id_, _kodepos_, _subtotal_, _discount_, _total_, _created_at_, _paid_at_, _delivery_at_
4. _order_details_ (detail barang saat transaksi):
   * _order_detail_id_, _order_id_, _product_id_, _price_, _quantity_

Adapun pertanyaan yang akan dijawab pada _project_ ini adalah:
1. Mencari 10 transaksi terbesar dari salah seorang _user_
2. Menampilan _summary_ transaksi per bulan di tahun 2020
3. Menampilkan _user_ dengan rata rata transaksi terbesar di Januari 2020
4. Menampilkan transaksi besar di Desember 2019
5. Menampilkan kategori produk terlaris di 2020
6. Mencari _user_ yang high value
7. Mencari _user_ yang menjadi _dropshipper_
8. Mencari _reseller offline_
9. Mencari _user_ yang menjadi pembeli sekaligus penjual
10. Mengetahui _trend_ lama transaksi dibayar sejak dibuat

### 10 Transaksi terbesar _user_ 12476
Berikut adalah _query_ untuk menampilkan 10 transaksi dari pembelian _user_ dengan _user_id_ 12476 dengan urutan transaksi paling besar.
```
SELECT
  seller_id, buyer_id, total AS nilai_transaksi, created_at AS tanggal_transaksi
FROM orders
WHERE buyer_id = 12476
ORDER BY 3 DESC
LIMIT 10
```
Output:

![image](https://github.com/user-attachments/assets/1b4825a8-9889-43d6-8b3f-6e53417796e7)

### _Summary_ Transaksi per bulan di tahun 2020
_Summary_ transaksi dapat dicari dengan menghitung berapa banyak jumlah transaksi yang terjadi dan berapa total keseluruhan nilai transaksinya. Berikut adalah _query_ untuk menampilkan _summary_ transaksi sepanjang tahun 2020
```
SELECT
  EXTRACT(YEAR_MONTH FROM created_at) AS tahun_bulan,
  COUNT(1) AS jumlah_transaksi,
  SUM(total) AS total_nilai_transaksi
FROM orders
WHERE created_at>='2020-01-01'
GROUP BY 1
ORDER BY 1
```
Output:

![image](https://github.com/user-attachments/assets/d096861d-3d9d-48de-9c55-7ba1202dd551)

### Pengguna dengan Rata-rata Transaksi Terbesar di Januari 2020
_user_ yang termasuk dalam rata-rata transaksi terbesar merupakan _user_ dengan jumlah transaksi minimal 2 kali di Januari 2020. Berikut adalah _query_ untuk menampilkan _top 10 user_ di Januari 2020
```
SELECT
  buyer_id, COUNT(1) AS jumlah_transaksi, AVG(total) AS avg_nilai_transaksi
FROM orders
WHERE created_at>='2020-01-01' AND created_at<'2020-02-01'
GROUP BY 1
HAVING COUNT(1)>= 2
ORDER BY 3 DESC
LIMIT 10
```
Output:

![image](https://github.com/user-attachments/assets/1a8cba46-bb79-4a20-9841-26a1a68f7391)

### Transaksi besar di Desember 2019
Yang termasuk dalam transaksi besar adalah nilai transaksi minimal 20,000,000 di bulan Desember 2019. Berikut adalah _query_ untuk menampilkan transaksi besar sepanjang Desember 2019 yang memuat nama pembeli, nilai transaksi dan tanggal transaksi
```
SELECT
  nama_user AS nama_pembeli,
  total AS nilai_transaksi,
  created_at AS tanggal_transaksi
FROM orders
INNER JOIN users ON buyer_id = user_id
WHERE created_at>='2019-12-01' AND created_at<'2020-01-01' AND total >= 20000000
ORDER BY 1
```
Output:

![image](https://github.com/user-attachments/assets/ed2bba4a-64a9-453a-8ad3-22b9deeb71b4)

### Kategori Produk Terlaris di 2020
Kategori produk terlaris dapat diketahui dengan mencari total _quantity_ terbanyak di tahun 2020. Berikut adalah _query_ untuk menampilkan 5 kategori produk terlaris yang memuat nama kategori, total _quantity_ dan total _price_
```
SELECT
  category, SUM(quantity) AS total_quantity, SUM(price) AS total_price
FROM orders
INNER JOIN order_details using(order_id)
INNER JOIN products using(product_id)
WHERE created_at>='2020-01-01'
AND delivery_at IS NOT NULL
GROUP BY 1
ORDER BY 2 DESC
limit 5
```
Output:

![image](https://github.com/user-attachments/assets/bc5b3a91-ab10-4ebc-b863-12842ad6cce1)

### Mencari Pembeli _High Value_
Yang termasuk dalam kategori _high value_ adalah _user_ yang sudah bertransaksi lebih dari 5 kali dan setiap transaksi lebih dari 2,000,000. Berikut adalah _query_ untuk menampilkan _user high value_ yang memuat nama pembeli, jumlah transaksi, total nilai transaski dan minimal nilai transaksi
```
SELECT
  nama_user AS nama_pembeli,
  COUNT(1) AS jumlah_transaksi,
  SUM(total) AS total_nilai_transaksi,
  MIN(total) AS min_nilai_transaksi
FROM orders
INNER JOIN users ON buyer_id = user_id
GROUP BY user_id, nama_user
HAVING COUNT(1) > 5 AND MIN(total) > 2000000
ORDER BY 3 DESC
```
Output:

![image](https://github.com/user-attachments/assets/3de3f207-f069-4108-bca1-b3834e4c6f59)

### Mencari _Dropshipper_
_Dropshipper_ yang dimaksud adalah pembeli yang membeli barang tetapi dikirim ke orang lain, ciri-cirinya berupa transaksi banyak dengan alamat yang berbeda beda. Berikut adalah _query_ untuk mencari _dropshipper_ dengan menampilkan pembeli dengan minimal 10 transaksi yang memuat nama pembeli, jumlah transaksi, distinct kodepos, total nilai transaksi dan rata-rata nilai transaksi
```
SELECT
  nama_user AS nama_pembeli,
  COUNT(1) AS jumlah_transaksi,
  COUNT(DISTINCT orders.kodepos) AS distinct_kodepos,
  SUM(total) AS total_nilai_transaksi,
  AVG(total) AS avg_nilai_transaksi
FROM orders
INNER JOIN users ON buyer_id = user_id
GROUP BY user_id, nama_user
HAVING COUNT(1) >= 10 AND COUNT(1) = COUNT(DISTINCT orders.kodepos)
ORDER BY 2 DESC
```
Output:

![image](https://github.com/user-attachments/assets/7afe86c5-6d15-4079-b498-b88baea71e16)

### Mencari _Reseller Offline_
_Reseller Offline_ yang dimaksud adalah pembeli yang dering membeli barang dan seringnya dikirimkan ke alamat yang sama disertai dengan pembelian dalam jumlah banyak, sehingga kemungkinan barang akan dijual lagi. Berikut adalah _query_ untuk mencari _reseller offline_ dengan minimal 8 transaksi dan rata-rata total _quantity_ per transaksi lebih dari 10 yang memuat nama pembeli, jumlah transaksi, total nilai transaksi, rata-rata nilai transaski, rata-rata quantity
```
SELECT
   nama_user AS nama_pembeli,
   COUNT(1) AS jumlah_transaksi, 
   SUM(total) AS total_nilai_transaksi, 
   AVG(total) AS avg_nilai_transaksi,
   AVG(total_quantity) AS avg_quantity_per_transaksi
FROM
   orders INNER JOIN users ON buyer_id = user_id 
   INNER JOIN (SELECT
                order_id, SUM(quantity) AS total_quantity
              FROM order_details
              GROUP BY 1) AS summary_order USING(order_id)
WHERE orders.kodepos=users.kodepos
GROUP BY user_id, nama_user
HAVING COUNT(1)>= 8 AND AVG(total_quantity)>10
ORDER BY 3 DESC
```
Output:

![image](https://github.com/user-attachments/assets/31964c32-c728-445c-9234-d90ab7488cf9)

### Pembeli Sekaligus Penjual
Adapun yang tergolong pembeli sekaligus penjual adalah _user_ yang pernah bertransaksi minimal 7 kali. Berikut adalah _query_ untuk mencari pembeli sekaligus penjual yang memuat nama pengguna, jumlah transaksi beli, dan jumlah transaksi jual
```
SELECT
	nama_user AS nama_pengguna,
	jumlah_transaksi_beli,
	jumlah_transaksi_jual
FROM users
INNER JOIN (SELECT
              buyer_id, COUNT(1) AS jumlah_transaksi_beli
            FROM orders
            GROUP BY 1) AS buyer ON buyer_id = user_id
INNER JOIN ( SELECT
              seller_id, COUNT(1) AS jumlah_transaksi_jual
            FROM orders
            GROUP BY 1) AS seller ON seller_id = user_id
WHERE jumlah_transaksi_beli >= 7
ORDER BY 1
```
Output:

![image](https://github.com/user-attachments/assets/7813a124-729f-4116-960a-0a37e39b53ac)

### Lama Transaksi Dibayar
Berikut adalah _query_ untuk menghitung rata-rata lama waktu dari transaksi dibuat sampai dibayar yang dikelompokkan per bulan.
```
SELECT
  EXTRACT(YEAR_MONTH FROM created_at) AS tahun_bulan,
  COUNT(1) AS jumlah_transaksi,
  AVG(DATEDIFF(paid_at,created_at)) AS avg_lama_dibayar,
  MIN(DATEDIFF(paid_at,created_at)) AS min_lama_dibayar,
  MAX((DATEDIFF(paid_at,created_at)) AS max_lama_dibayar
FROM orders 
WHERE paid_at IS NOT NULL 
GROUP BY 1 
ORDER BY 1
```
Output:

![image](https://github.com/user-attachments/assets/408d71aa-3dc4-4299-9432-9998024e6e86)
