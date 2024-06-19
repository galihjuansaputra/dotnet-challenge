## 1. Mengukur Algoritma berfikir
Menggunakan sistem FIFO (First In First Out) yang memastikan produk terlama didistribusikan terlebih dahulu dan membuat risk register yang bertujuan untuk mengumpulkan informasi tentang resiko-resiko yang mungkin terjadi dan langkah-langkah yang akan diambil untuk mengelolanya


## 2.Penguasaan Query DB

**Creating Table**

```sql
CREATE TABLE gudang (
  id_gudang INT PRIMARY KEY IDENTITY(1,1),
  kode_gudang VARCHAR(10) NOT NULL UNIQUE,
  nama_gudang VARCHAR(50) NOT NULL
);

CREATE TABLE barang (
  id_barang INT PRIMARY KEY IDENTITY(1,1),
  kode_barang VARCHAR(10) NOT NULL UNIQUE,
  nama_barang VARCHAR(50) NOT NULL,
  harga_barang DECIMAL(10,2) NOT NULL,
  jumlah_barang INT NOT NULL,
  expired_date DATE NOT NULL,
  id_gudang INT NOT NULL,
  FOREIGN KEY (id_gudang) REFERENCES gudang(id_gudang)
);
```

**Store Procedure, Dynamic Query, Paging**

```sql
CREATE PROCEDURE show_data(
  @start INT,  -- Starting position for paging (1-based)
  @rows INT   -- Number of rows per page
)
AS
BEGIN
  -- Calculate the offset based on start and page size
  DECLARE @offset INT;
  SET @offset = (@start - 1) * @rows;  -- Corrected calculation

  -- Query with OFFSET and FETCH NEXT
  SELECT g.kode_gudang, g.nama_gudang, b.kode_barang, b.nama_barang,
         b.harga_barang, b.jumlah_barang, b.expired_date
  FROM barang b
  INNER JOIN gudang g ON b.id_gudang = g.id_gudang
  ORDER BY b.id_barang
  OFFSET @offset ROWS FETCH NEXT @rows ROWS ONLY;
END
```

**Trigger ketika Input Barang di salah satu gudang muncul kan barang yang kadaluarsa**

```sql
CREATE TRIGGER show_expired_items ON barang
FOR INSERT
AS 
BEGIN
SELECT b.kode_barang, b.nama_barang, b.expired_date
    FROM barang b
    WHERE b.id_gudang = id_gudang
      AND b.expired_date < GETDATE();
END
```
