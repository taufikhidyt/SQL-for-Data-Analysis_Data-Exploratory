# SQL for Data Exploratory: TheLook Ecommerce

**Transforming Data into Stories**
Repository ini memuat eksplorasi data menggunakan Google Big Query.

Dataset: [theLook Ecommerce Dataset (BigQuery Public Data)](https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=thelook_ecommerce&page=dataset&project=my-gcp-data-projects&ws=!1m9!1m4!4m3!1sbigquery-public-data!2sthelook_ecommerce!3sorder_items!1m3!3m2!1sbigquery-public-data!2sthelook_ecommerce)

**Product and Brand Analysis**

1. Menghitung total penjualan dan pendapatan berdasarkan kategori produk
```sql
WITH product_sales AS(
  SELECT 
    p.category as product_category,
    ROUND(SUM(oi.sale_price * o.num_of_item), 2) AS total_revenue,
    SUM(o.num_of_item) AS total_sales
  FROM `bigquery-public-data.thelook_ecommerce.orders` AS o
  INNER JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS oi
  ON o.order_id = oi.order_id
  INNER JOIN `bigquery-public-data.thelook_ecommerce.products` AS p 
  ON oi.product_id = p.id
  WHERE oi.status NOT IN ('cancelled', 'Returned')
  GROUP BY 1
)
SELECT * FROM product_sales
ORDER BY 2 DESC
```

2. Menghitung total penjualan dan pendapatan berdasarkan brand
```sql
WITH brand_sales AS(
  SELECT 
    p.brand as product_brand,
    ROUND(SUM(oi.sale_price * o.num_of_item), 2) AS total_revenue,
    SUM(o.num_of_item) AS total_sales
  FROM `bigquery-public-data.thelook_ecommerce.orders` AS o
  INNER JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS oi
  ON o.order_id = oi.order_id
  INNER JOIN `bigquery-public-data.thelook_ecommerce.products` AS p 
  ON oi.product_id = p.id
  WHERE oi.status NOT IN ('cancelled', 'Returned')
  GROUP BY 1
)
SELECT * FROM brand_sales
ORDER BY 2 DESC
```
3. Brand cancellation and return
```sql
WITH brand_can_re AS (
  SELECT
    p.brand AS product_brand,
    SUM(CASE WHEN oi.status = 'Cancelled' THEN 1 ELSE null END) AS cancelled,
    SUM(CASE WHEN oi.status = 'Returned' THEN 1 ELSE null END) AS returned
  FROM bigquery-public-data.thelook_ecommerce.order_items oi
  INNER JOIN bigquery-public-data.thelook_ecommerce.products p
  ON oi.product_id = p.id
  GROUP BY 1
)
SELECT * FROM brand_can_re
ORDER BY 2 DESC
```

4. Product cancellation and return
```sql
WITH product_can_re AS (
  SELECT
    p.category AS product_category,
    SUM(CASE WHEN oi.status = 'Cancelled' THEN 1 ELSE null END) AS cancelled,
    SUM(CASE WHEN oi.status = 'Returned' THEN 1 ELSE null END) AS returned
  FROM bigquery-public-data.thelook_ecommerce.order_items oi
  INNER JOIN bigquery-public-data.thelook_ecommerce.products p
  ON oi.product_id = p.id
  GROUP BY 1
)
SELECT * FROM product_can_re
ORDER BY 2 DESC
```

5. Revenue
```sql
SELECT
  EXTRACT(MONTH FROM oi.Created_at) AS months,
  ROUND(SUM(oi.sale_price * o.num_of_item), 2) AS total_revenue,
  SUM(o.num_of_item) AS total_sales,
  COUNT(DISTINCT oi.order_id) AS order_count,
  COUNT(DISTINCT oi.user_id) AS customers_purchased
FROM `bigquery-public-data.thelook_ecommerce.order_items`AS oi
INNER JOIN `bigquery-public-data.thelook_ecommerce.orders` AS o 
ON oi.order_id = o.order_id
WHERE oi.status NOT IN ('Cancelled', 'Returned')
GROUP BY 1
ORDER BY 2 DESC
```

**Customer Analysis**

6. Menghitung keuntungan berdasarkan kelompok usia pelanggan
```sql
SELECT 
  CASE 
    WHEN u.age < 12 THEN 'Anak-anak'
    WHEN u.age BETWEEN 12 AND 17 THEN 'Remaja Awal'
    WHEN u.age BETWEEN 17 AND 25 THEN 'Remaja Akhir'
    WHEN u.age BETWEEN 25 AND 35 THEN 'Dewasa Awal'
    WHEN u.age BETWEEN 35 AND 55 THEN 'Dewasa Akhir'
    WHEN u.age > 55 THEN 'Lansia'
  END AS age_group,
  COUNT(DISTINCT oi.user_id) AS total_customer,
  ROUND(SUM(oi.sale_price * o.num_of_item), 2) AS total_revenue
FROM `bigquery-public-data.thelook_ecommerce.users` AS u
INNER JOIN `bigquery-public-data.thelook_ecommerce.orders` AS o 
ON u.id = o.user_id
INNER JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS oi
ON o.order_id = oi.order_id
WHERE oi.status NOT IN ('Cancelled', 'Returned') 
GROUP BY 1
ORDER BY 3 DESC
```

7. Menghitung keuntungan berdasarkan jenis kelamin pelanggan
```sql
SELECT 
  u.gender,
  ROUND(SUM(oi.sale_price * o.num_of_item), 2) AS total_revenue,
  SUM(o.num_of_item) AS total_sales
FROM `bigquery-public-data.thelook_ecommerce.users` AS u
INNER JOIN `bigquery-public-data.thelook_ecommerce.orders` AS o 
ON u.id = o.user_id
INNER JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS oi
ON o.order_id = oi.order_id
WHERE oi.status NOT IN ('Cancelled', 'Returned')
GROUP BY 1
ORDER BY 2 DESC
```

8. Mengidentifikasi negara dengan jumlah pelanggan terbesar
```sql
SELECT 
  country,
  COUNT(DISTINCT id) AS customer_count
FROM `bigquery-public-data.thelook_ecommerce.users` 
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```

9. Mengidentifikasi daftar 9 customer IDs dengan pembelian terbanyak. Pembeli tersebut akan diberi diskon 9.9
```sql
SELECT
  u.id AS customer_id,
  CONCAT(u.first_name, ' ', u.last_name) AS full_name,
  u.email AS  email,
  ROUND(SUM(oi.sale_price * o.num_of_item), 2) AS total_revenue
FROM bigquery-public-data.thelook_ecommerce.order_items AS oi
INNER JOIN bigquery-public-data.thelook_ecommerce.orders AS o
ON oi.order_id = o.order_id
INNER JOIN bigquery-public-data.thelook_ecommerce.users AS u
ON o.user_id = u.id
GROUP BY 1, 2, 3
ORDER BY 4 DESC
LIMIT 9
```


**Rekomendasi**
