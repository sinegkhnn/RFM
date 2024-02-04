# RFM Analysis with Flo Dataset 🛒📊🔎

`PowerBI Dashboard for FLO`
![rfm1](https://github.com/sinegkhnn/RFM/assets/101671498/f9309d6e-0655-411c-a160-3e853771b571)
![wwww](https://github.com/sinegkhnn/RFM/assets/101671498/16d2ac71-d4a0-4639-9976-a3495e6e161a)
![pb2](https://github.com/sinegkhnn/RFM/assets/101671498/6c8d582c-0ead-4f78-b4f3-8dc8be20b344)


#### Flo datasına genel bir bakışla başlayalım. Data 20k satır ve 12 değişkene sahip.
```
SELECT COLUMN_NAME AS DEGISKEN_ISIMLERI
FROM flodb.INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'FLO'
```
![github](https://github.com/sinegkhnn/RFM/assets/101671498/6b873fa4-3567-4cdb-9e88-e1093828182c)

#### Datada yer alan customer_value_total_ever_offline, customer_value_total_ever_online değerleri ve order_num_total_ever_online, order_num_total_ever_offline değerlerini daha doğru kullanabilmek adına toplam olarak yazalım.(Total Value)
```
ALTER TABLE flo ADD customer_value_total AS (customer_value_total_ever_offline + customer_value_total_ever_online);
ALTER TABLE flo ADD order_num_total AS (order_num_total_ever_online + order_num_total_ever_offline);
SELECT * FROM flo
ALTER TABLE FLO
DROP COLUMN order_total;
ALTER TABLE FLO
DROP COLUMN customer_total;
```
#### RFM metriklerini belirleyelim. Analiz için kullandığım date : "2021-06-01", customer_id, recency, frequnecy ve monetary değerlerinin yer aldığı yeni bir rfm adında tablosu oluşturalım.
```
SELECT master_id AS CUSTOMER_ID,
       DATEDIFF(DAY, last_order_date, '20210601') AS Receny,
       DATEDIFF(MONTH,first_order_date,'20210601') AS Tenure,
       order_num_total AS Frequency,
       customer_value_total AS Monetary,
       NULL RECENCY_SCORE,
     NULL FREQUENCY_SCORE,
     NULL MONETARY_SCORE
INTO RFM_test1
FROM flo;
```
#### Recency score gibi frequency score ve monetary score oluşturalım.
```
UPDATE RFM_test1 SET RECENCY_SCORE = 
(SELECT SCORE FROM
(SELECT A.*,
        NTILE(5) OVER(ORDER BY Recency DESC) SCORE
FROM RFM_test1 AS A
) T 
WHERE T.Customer_ID = RFM_test1.Customer_ID
);
```
#### Son olarak RFM tablomuza RFM scorelarını ekleyelim.
```
ALTER TABLE RFM_test1 ADD RFM AS (CONVERT(VARCHAR,RECENCY_SCORE) + CONVERT(VARCHAR,FREQUENCY_SCORE) + CONVERT(VARCHAR, MONETARY_SCORE));
```



