SQL_Banking_Credit_Segment_Exploration
SQL_Banking_Credit_Segment_Exploration

## **1. INTRODUCTION**

This project focuses on compiling and analyzing credit data for VNC Bank in 2023. The data includes indicators related to credit scale, number of customers, collateral value (TSBD), and growth factors across different periods such as the end of 2022, the first half, and the second half of 2023. The goal of the project is to create a comprehensive report on the credit situation, evaluating changes in lending activities to help improve business strategy and credit risk management.

## **2. THE GOAL OF PROJECT**

**+ Compile and analyze credit data:** Collect and process data on credit contracts, customers, outstanding loans, and collateral values across different periods in 2023.

**+ Assess credit fluctuations:** Compare fluctuations in customer numbers and credit values, including increases and decreases, to evaluate the bank's financial health over specific time periods.

**+ Monitor collateral performance:** Review changes in the total collateral value (TSBD) and assess growth compared to the same period in the previous year to predict loan recovery potential and minimize credit risks.

**+ Provide strategic insights:** Offer useful information for management to adjust credit strategies and risk management, optimizing profits while minimizing losses.

=> This project plays a crucial role in supporting strategic decision-making and improving the bank's credit management activities for future operations.

## **3. EXPLORING DATASET**

**Query 1: Report on the Credit Situation for the First Half of the Year**

```sql

WITH SOLIEU AS 
(
	SELECT *
	FROM
	(
	SELECT	DAUKY.debid								AS 'DAUKY_KHOANNO',
			DAUKY.sodudauky							AS 'DAUKY_SODU',
			DAUKY.crcontract_date					AS 'DAUKY_NGAYBATDAU',
			DAUKY.crcontract_end_date				AS 'DAUKY_NGAYTATTOAN',
			PHATSINH.debid							AS 'PS_KHOANNO',
			PHATSINH.crlimit						AS 'PS_SOTIEN',
			PHATSINH.crcontract_date				AS 'PS_NGAYBATDAU',
			PHATSINH.crcontract_end_date			AS 'PS_NGAYTATTOAN'
	FROM 
		(
			select	A.debid, B.sodudauky,A.crcontract_date,A.crcontract_end_date
			from	credit_contract a 
					inner join CREDIT_PLAN b on a.debid = b.debid 
			where	b.ngaydauky <= '2022-12-31' and b.ngaycuoiky >= '2022-12-31'
					and a.crcontract_end_date > '2022-12-31'
		) DAUKY
		FULL OUTER JOIN
		(
			select	debid, crlimit,crcontract_date,crcontract_end_date
			from	credit_contract
			where	YEAR( crcontract_date ) = 2023
		) PHATSINH ON DAUKY.debid = PHATSINH.debid 
	) X 
	LEFT JOIN
	(
		select		A.DEBID							AS 'THUHOI_KHOANNO', 
					SUM(gocphaitra)					AS 'THUHOI_TONGTHU',     
					A.crcontract_date				AS 'THUHOI_NGAYBATDAU',
					A.crcontract_end_date			AS 'THUHOI_NGAYTATTOAN'
		from		credit_contract a 
					inner join CREDIT_PLAN b on a.debid = b.debid  
		where		YEAR(dateadd(day,1,ngaycuoiky)) = 2023
		group by	a.debid ,A.crcontract_date, A.crcontract_end_date
	) Y ON ISNULL(X.DAUKY_KHOANNO,'') +  ISNULL(X.PS_KHOANNO,'') = Y.THUHOI_KHOANNO
	LEFT JOIN
	(
		select		A.debid							AS 'CUOIKY_KHOANNO', 
					B.sodudauky						AS 'CUOIKY_SODU',
					A.crcontract_date				AS 'CUOIKY_NGAYBATDAU',
					A.crcontract_end_date			AS 'CUOIKY_NGAYTATTOAN'
		from		credit_contract a 
					inner join CREDIT_PLAN b on a.debid = b.debid 
		where		ngaydauky <= '2023-12-31' and ngaycuoiky >= '2023-12-31'
					and crcontract_end_date > '2023-12-31'
	) Z ON ISNULL(X.DAUKY_KHOANNO,'') +  ISNULL(X.PS_KHOANNO,'') = Z.CUOIKY_KHOANNO
	LEFT JOIN
	(
		select		A.debid							AS 'TATTOAN_KHOANNO', 
					A.crlimit						AS 'TATTOAN_SOTIEN',
					A.crcontract_date				AS 'TATTOAN_NGAYBATDAU',
					A.crcontract_end_date			AS 'TATTOAN_NGAYTATTOAN'
		from		credit_contract a 
		where		YEAR(crcontract_end_date) = '2023'
	) W ON ISNULL(X.DAUKY_KHOANNO,'') +  ISNULL(X.PS_KHOANNO,'') = W.TATTOAN_KHOANNO
)
--DROP TABLE SOLIEU_FIANL
SELECT * 
INTO SOLIEU_FIANL
FROM SOLIEU


--- 1. SỐ HỢP ĐỒNG TÍN DỤNG---

-- ĐẦU KỲ VÀ CUỐI KỲ
SELECT		COUNT(DAUKY_KHOANNO)	'Số khoản nợ đầu kỳ',
			COUNT(CUOIKY_KHOANNO)	'số cuối kỳ'
FROM		SOLIEU_FIANL

-- PHÁT SINH TĂNG 6T 
-- ĐẦU NĂM
SELECT		
			COUNT(PS_KHOANNO)	AS [PHÁT SINH TĂNG 6T ĐẦU NĂM]
FROM		SOLIEU_FIANL
WHERE PS_NGAYBATDAU BETWEEN '2023-01-01' AND '2023-06-30'
--CUỐI NĂM
SELECT		
			COUNT(PS_KHOANNO)	AS [PHÁT SINH TĂNG 6T CUỐI NĂM]
FROM		SOLIEU_FIANL
WHERE PS_NGAYBATDAU BETWEEN '2023-07-01' AND '2023-12-31'

-- PHÁT SINH GIẢM 6T 

-- ĐẦU NĂM

SELECT		
			COUNT(TATTOAN_KHOANNO)	AS [PHÁT SINH GIẢM 6T ĐẦU NĂM]
FROM		SOLIEU_FIANL
WHERE TATTOAN_NGAYTATTOAN BETWEEN '2023-01-01' AND '2023-06-30'

--CUỐI NĂM

SELECT		
			COUNT(TATTOAN_KHOANNO)	AS [PHÁT SINH GIẢM 6T CUỐI NĂM]
FROM		SOLIEU_FIANL
WHERE TATTOAN_NGAYTATTOAN BETWEEN '2023-07-01' AND '2023-12-31'


--- 2.DƯ NỢ TÍN DỤNG ---


-- ĐẦU KỲ VÀ CUỐI KỲ
SELECT		sum(DAUKY_SODU)		'dư nợ gốc đầu kỳ',
			sum(CUOIKY_SODU)	'số cuối kỳ'
FROM		SOLIEU_FIANL ;

-- PHÁT SINH TĂNG 6T 
-- ĐẦU NĂM

SELECT SUM(PS_SOTIEN)	AS [PHÁT SINH TĂNG 6T ĐẦU NĂM]
FROM SOLIEU_FIANL
WHERE PS_NGAYBATDAU BETWEEN '2023-01-01' AND '2023-06-30'

-- CUỐI NĂM

SELECT SUM(PS_SOTIEN)	AS [PHÁT SINH TĂNG 6T CUỐI NĂM]
FROM SOLIEU_FIANL
WHERE PS_NGAYBATDAU BETWEEN '2023-07-01' AND '2023-12-31'

-- PHÁT SINH GIẢM 6T 
-- ĐẦU NĂM

SELECT
SUM(THUHOI_TONGTHU)		AS [PHÁT SINH GIẢM 6T ĐẦU NĂM]
FROM 
(
				SELECT		DISTINCT
		THUHOI_KHOANNO
		,RANK () OVER (PARTITION BY THUHOI_KHOANNO ORDER BY ngaycuoiky desc) AS [RANK]
		,ngaycuoiky
		, THUHOI_TONGTHU
				--	sum(A.THUHOI_TONGTHU)		'Số phát sinh giảm'
		FROM		SOLIEU_FIANL A
		LEFT JOIN CREDIT_PLAN B
		ON A.THUHOI_KHOANNO = B.debid
		WHERE dateadd(day,1,B.ngaycuoiky) BETWEEN '2023-01-01' AND '2023-12-31') X
WHERE ngaycuoiky BETWEEN '2023-01-01' AND '2023-06-30'
AND RANK = 1

-- CUỐI NĂM

SELECT
	SUM(THUHOI_TONGTHU) AS [PHÁT SINH GIẢM 6T CUỐI NĂM]
FROM 
(
				SELECT		DISTINCT
		THUHOI_KHOANNO
		,RANK () OVER (PARTITION BY THUHOI_KHOANNO ORDER BY ngaycuoiky desc) AS [RANK]
		,ngaycuoiky
		, THUHOI_TONGTHU
				--	sum(A.THUHOI_TONGTHU)		'Số phát sinh giảm'
		FROM		SOLIEU_FIANL A
		LEFT JOIN CREDIT_PLAN B
		ON A.THUHOI_KHOANNO = B.debid
		WHERE dateadd(day,1,B.ngaycuoiky) BETWEEN '2023-01-01' AND '2023-12-31') X
WHERE ngaycuoiky BETWEEN '2023-07-01' AND '2023-12-31'
AND RANK = 1



-- 3. SỐ LƯỢNG KHÁCH HÀNG


WITH SOLIEUKHACHHANG AS 
(
	SELECT *
	FROM
	(
	SELECT		DAUKY.custid		AS 'DAUKY_KH',
			DAUKY.tongtien		AS 'DAUKY_ST',
			DAUKY.ngaybatdau	AS 'DAUKY_NGAYBATDAU',
			DAUKY.ngaytattoan	AS 'DAUKY_NGAYTATTOAN',
			PHATSINH.custid		AS 'PHATSINH_KH',
			PHATSINH.tongtien	AS 'PHATSINH_ST',
			PHATSINH.ngaybatdau	AS 'PHATSINH_NGAYBATDAU',
			PHATSINH.ngaytattoan	AS 'PHATSINH_NGAYTATTOAN'
	FROM 
		(
			select	A.custid, 
				SUM(B.crlimit) as 'tongtien',
				B.crcontract_date as 'ngaybatdau',
				B.crcontract_end_date 'ngaytattoan'
			from	CUSTOMER  a 
					inner join credit_contract  b on a.custid  = b.custid  
			where	b.crcontract_end_date > '2022-12-31'
					AND B.crcontract_date <= '2022-12-31'
			group by a.custid, B.crcontract_date, B.crcontract_end_date 
		) DAUKY
		FULL OUTER JOIN
		(
			select	A.custid, 
				SUM(B.crlimit) as 'tongtien',
				B.crcontract_date as 'ngaybatdau',
				B.crcontract_end_date AS 'ngaytattoan'
			from	CUSTOMER  a 
					inner join credit_contract  b on a.custid  = b.custid  
			where	year(crcontract_date) = 2023
			group by a.custid,B.crcontract_date, B.crcontract_end_date  
		) PHATSINH ON DAUKY.custid = PHATSINH.custid
	) X 
	LEFT JOIN
	(
			select	A.custid		AS 'TATTOAN_KH', 
				SUM(B.crlimit)		AS 'TATTOAN_ST',
				B.crcontract_date 	AS 'TATTOAN_NGAYBATDAU',
				B.crcontract_end_date 	AS 'TATTOAN_NGAYTATTOAN'
			from	CUSTOMER  a 
					inner join credit_contract  b on a.custid  = b.custid  
			where	YEAR(crcontract_end_date) = '2023'
			group by a.custid,B.crcontract_date, B.crcontract_end_date 
	) Y ON ISNULL(X.DAUKY_KH,'') = Y.TATTOAN_KH OR ISNULL(X.PHATSINH_KH,'') = Y.TATTOAN_KH 
	LEFT JOIN
	(
			select	A.custid		AS 'CUOIKY_KH', 
				SUM(B.crlimit)	AS 'CUOIKY_ST',
				B.crcontract_date 	AS 'CUOIKY_NGAYBATDAU',
				B.crcontract_end_date 	AS 'CUOIKY_NGAYTATTOAN'
			from	CUSTOMER  a 
					inner join credit_contract  b on a.custid  = b.custid  
			where	b.crcontract_end_date > '2023-12-31'
					AND B.crcontract_date <= '2023-12-31'
			group by a.custid ,B.crcontract_date, B.crcontract_end_date 
	) Z ON ISNULL(X.DAUKY_KH,'') = Z.CUOIKY_KH or ISNULL(X.PHATSINH_KH,'')  = Z.CUOIKY_KH
)

SELECT * 
INTO SOLIEUKHACHHANG_FINAL
FROM SOLIEUKHACHHANG
--- KHÁCH HÀNG---
SELECT * FROM SOLIEUKHACHHANG_FINAL

-- ĐẦU KỲ:

SELECT		COUNT(DISTINCT(DAUKY_KH))	 AS [DK_SOLUONGKH]
FROM		SOLIEUKHACHHANG_FINAL

-- PHÁT SINH MỚI:

-- 6 THÁNG ĐẦU NĂM

SELECT COUNT(DISTINCT(PHATSINH_KH))  AS [PSM6TDN_SOLUONGKH]
FROM
	(SELECT	
		PHATSINH_KH
	FROM SOLIEUKHACHHANG_FINAL
	WHERE PHATSINH_NGAYBATDAU BETWEEN '2023-01-01' AND '2023-06-30') PS_6TDN
	LEFT JOIN
	(SELECT	DAUKY_KH
	FROM SOLIEUKHACHHANG_FINAL) DK
	ON PS_6TDN.PHATSINH_KH = DK.DAUKY_KH
WHERE DK.DAUKY_KH IS NULL

-- 6 THÁNG CUỐI NĂM

SELECT COUNT(DISTINCT(PS_6TCN.PHATSINH_KH))  AS [PSM6TCN_SOLUONGKH]
FROM
	(SELECT	
		PHATSINH_KH
	FROM SOLIEUKHACHHANG_FINAL
	WHERE PHATSINH_NGAYBATDAU BETWEEN '2023-07-01' AND '2023-12-31') PS_6TCN
	LEFT JOIN
	(SELECT	
		DAUKY_KH
	FROM SOLIEUKHACHHANG_FINAL) DK
	ON PS_6TCN.PHATSINH_KH = DK.DAUKY_KH
	LEFT JOIN
	(SELECT	
		PHATSINH_KH
	FROM SOLIEUKHACHHANG_FINAL
	WHERE PHATSINH_NGAYBATDAU BETWEEN '2023-01-01' AND '2023-06-30') PS_6TDN
	ON PS_6TCN.PHATSINH_KH = PS_6TDN.PHATSINH_KH
WHERE DK.DAUKY_KH IS NULL
AND PS_6TDN.PHATSINH_KH IS NULL


--- TẤT TOÁN

-- 6 THÁNG ĐẦU NĂM

SELECT COUNT(DISTINCT(TATTOAN_KH))  AS [PSG6TDN_SOLUONGKH]
FROM
	(SELECT	
		TATTOAN_KH
	FROM SOLIEUKHACHHANG_FINAL
	WHERE TATTOAN_NGAYTATTOAN BETWEEN '2023-01-01' AND '2023-06-30') TT_6TDN
	LEFT JOIN
	(SELECT 
		CUOIKY_KH
	FROM SOLIEUKHACHHANG_FINAL) CK
	ON TT_6TDN.TATTOAN_KH = CK.CUOIKY_KH
WHERE CK.CUOIKY_KH IS NULL

-- 6 THÁNG CUỐI NĂM

SELECT COUNT(DISTINCT(TT_6TCN.TATTOAN_KH))  AS [PSG6TCN_SOLUONGKH]
FROM
	(SELECT	
		TATTOAN_KH
	FROM SOLIEUKHACHHANG_FINAL
	WHERE TATTOAN_NGAYTATTOAN BETWEEN '2023-07-01' AND '2023-12-31') TT_6TCN
	LEFT JOIN
	(SELECT	
		CUOIKY_KH
	FROM SOLIEUKHACHHANG_FINAL) CK
	ON TT_6TCN.TATTOAN_KH = CK.CUOIKY_KH
	LEFT JOIN
	(SELECT	
		TATTOAN_KH
	FROM SOLIEUKHACHHANG_FINAL
	WHERE TATTOAN_NGAYTATTOAN BETWEEN '2023-01-01' AND '2023-06-30') TT_6TDN
	ON TT_6TCN.TATTOAN_KH = TT_6TDN.TATTOAN_KH
WHERE CK.CUOIKY_KH IS NULL
AND TT_6TDN.TATTOAN_KH IS NULL

-- CUỐI KỲ:

SELECT COUNT(DISTINCT(CUOIKY_KH))	AS [CK_SOLUONGKH]
FROM SOLIEUKHACHHANG_FINAL


--- 4. SỐ TIỀN GIẢI NGÂN 

-- ĐẦU KỲ

SELECT 
	SUM(DAUKY_ST)	AS [DK_DUNO]
FROM
	(
	SELECT		DISTINCT
	DAUKY_KH
	,DAUKY_ST
	,DAUKY_NGAYBATDAU
	,DAUKY_NGAYTATTOAN-- CHECK LẠI XEM CÓ CẦN DÙNG DISTINCT HAY KO?
	FROM		SOLIEUKHACHHANG_FINAL
	) X

-- PHÁT SINH MỚI:

-- 6 THÁNG ĐẦU NĂM
SELECT
	SUM(PHATSINH_ST)	AS [PS6TDN_DUNO]
FROM(
	SELECT DISTINCT
	PHATSINH_KH
	,PHATSINH_ST
	,PHATSINH_NGAYBATDAU
	,PHATSINH_NGAYTATTOAN
	FROM SOLIEUKHACHHANG_FINAL
WHERE PHATSINH_NGAYBATDAU BETWEEN '2023-01-01' AND '2023-06-30') PS_6TDN


-- 6 THÁNG CUỐI NĂM

SELECT
	SUM(PHATSINH_ST)	AS [PS6TCN_DUNO]
FROM(
	SELECT DISTINCT
	PHATSINH_KH
	,PHATSINH_ST
	,PHATSINH_NGAYBATDAU
	,PHATSINH_NGAYTATTOAN
	FROM SOLIEUKHACHHANG_FINAL
WHERE PHATSINH_NGAYBATDAU BETWEEN '2023-07-01' AND '2023-12-31') PS_6TCN


--- TẤT TOÁN

-- 6 THÁNG ĐẦU NĂM

SELECT 
	SUM(TATTOAN_ST)			AS [TT6TDN_DUNO]
FROM(
	SELECT DISTINCT
	TATTOAN_KH
	,TATTOAN_ST
	,TATTOAN_NGAYBATDAU
	,TATTOAN_NGAYTATTOAN
	FROM SOLIEUKHACHHANG_FINAL
WHERE TATTOAN_NGAYTATTOAN BETWEEN '2023-01-01' AND '2023-06-30') TT_6TDN


-- 6 THÁNG CUỐI NĂM
SELECT 
	SUM(TATTOAN_ST)		 AS [TT6TCN_DUNO]
FROM(
	SELECT DISTINCT
	TATTOAN_KH
	,TATTOAN_ST
	,TATTOAN_NGAYBATDAU
	,TATTOAN_NGAYTATTOAN
	FROM SOLIEUKHACHHANG_FINAL
WHERE TATTOAN_NGAYTATTOAN BETWEEN '2023-07-01' AND '2023-12-31') TT_6TCN

-- CUỐI KỲ:
SELECT 
	SUM(CUOIKY_ST)  AS [CK_DUNO]
FROM
	(
	SELECT DISTINCT
		CUOIKY_KH
		,CUOIKY_ST
		,CUOIKY_NGAYBATDAU
		,CUOIKY_NGAYTATTOAN
	FROM		SOLIEUKHACHHANG_FINAL
	) X
```

**RESULT**

![image](https://github.com/user-attachments/assets/ad290612-5c61-4da0-ab63-2ca581f4d9af)

**Based on the data you provided, we can derive several important insights from the credit performance indicators of the bank from the end of 2022 to the end of 2023.**

**1. Number of Credit Contracts:**
+ Situation: The number of credit contracts decreased from 97 at the end of 2022 to 89 by the end of 2023.
+ Analysis: Although there was an increase of 7 new contracts in the first half of 2023, the overall number of contracts declined throughout the year due to the completion or termination of more contracts (a total of 21 contracts were closed during the year). This could indicate that the bank managed its loans well or that there were delays in signing new credit contracts.
  
**2. Number of Customers:**
+ Situation: The number of customers also showed a downward trend, from 70 customers at the end of 2022 to 60 customers by the end of 2023.
+ Analysis: The decline in customer numbers could suggest a more cautious credit strategy by the bank or a shift in customer demand for loans. This may impact the bank’s credit revenue if the trend of declining customer numbers continues.
  
**3. Total Disbursed Funds:**
+ Situation: The total amount of disbursed funds peaked by the end of 2023 at 3,377,306,475,047 VND, significantly higher than the initial amount at the beginning of 2023 (4,357,124,475,047 VND).
+ Analysis: Despite the drop in the number of contracts and customers, the disbursed value increased substantially, particularly in the second half of 2023, with 1,020,478,000,000 VND disbursed. This suggests that the bank may have shifted its strategy to focus on larger loan amounts or more significant clients, rather than targeting a higher volume of smaller customers.
  
**4. Outstanding Loan Balances (Remaining Principal):**
+ Situation: The outstanding loan balances decreased significantly from 2,602,895,025,966 VND at the end of 2022 to 1,650,433,320,888 VND by the end of 2023.
+ Analysis: The reduction in outstanding loans could reflect the bank’s efficient debt recovery, especially in the second half of 2023, where disbursements were substantial, yet the outstanding loan amounts decreased considerably. This is a positive sign, indicating that the bank is managing its cash flow well, recovering funds promptly, and minimizing risks associated with bad debts.
  
**Conclusions and Key Insights:**

**+ Cautious Credit Approach:** The decline in the number of credit contracts and customers suggests that the bank is taking a more cautious approach to lending, focusing on quality over quantity.

**+ Growth in Loan Value:** Despite the reduction in the number of contracts and customers, total disbursed funds and loan management remain efficient. This suggests that the bank has shifted its strategy towards disbursing larger loans, possibly targeting corporate clients or higher-quality loans.

**+ Effective Loan Management:** The significant reduction in outstanding loans indicates that the bank is effectively managing its debts, recovering funds in a timely manner, and reducing credit risks, which provides a solid foundation for future growth.

=> In summary, VNC Bank has achieved a balance between minimizing risks and maintaining credit growth by focusing on larger-value loans and managing outstanding debts effectively. However, the bank should also pay attention to attracting new customers to ensure long-term growth.

**Query 2: Report on the Collateral Situation for the First Half of the Year**

```
sql
WITH SOLIEU_TSĐB AS --- HOP DONG TIN DUNG -------
(	SELECT *
	FROM
	(
	SELECT	DAUKY.*, PHATSINH.*
	FROM 
		(
			select 	a.debid				'DK_KHOANNO'
				,a.sogiaodich			'DK_HDTC' 
				,a.crcontract_date 		'DK_NGAYBD'	
				,a.crcontract_end_date		'DK_NGAYKT'
			from credit_contract a 
			inner join CREDIT_PLAN b on a.debid = b.debid 
			where	b.ngaydauky <= '2022-12-31' and b.ngaycuoiky >= '2022-12-31'
					and a.crcontract_end_date > '2022-12-31'
		) DAUKY
		FULL OUTER JOIN
		(
			select	debid 				as [PS_KHOANNO]
				,sogiaodich 			AS [PS_HDTC]
				,crcontract_date 		AS [PS_NGAYBD]
				,crcontract_end_date 		AS [PS_NGAYKT]
			from	credit_contract
			where	YEAR( crcontract_date ) = 2023
		) PHATSINH ON DAUKY.DK_KHOANNO = PHATSINH.PS_KHOANNO
	) X 
	LEFT JOIN
	(
		select 
				a.debid 			[CK_KHOANNO]
				,HDTC_id 			[CK_HDTC]
				,crcontract_date 		[CK_NGAYBD]
				,crcontract_end_date 		[CK_NGAYKT]
		from CREDIT_PLAN a
		inner join credit_contract b on b.debid = a.debid
		left join Mortgage_Agreement c on b.sogiaodich = c.HDTC_id
		where 
			a.ngaydauky <= '2023-12-31'
			and a.ngaycuoiky >= '2023-12-31'
			and b.crcontract_end_date > '2023-12-31'
	) Z ON ISNULL(X.DK_KHOANNO,'') +  ISNULL(X.PS_KHOANNO,'') = Z.CK_KHOANNO
	LEFT JOIN
	(
		SELECT 
			crc.debid 				TT_KHOANNO
			,crc.sogiaodich 			TT_HDTC
			,crc.crcontract_date 			TT_NGAYBD
			,crc.crcontract_end_date 		TT_NGAYKT
		FROM
			credit_contract crc
			INNER JOIN
			mortgage_agreement ma ON crc.sogiaodich = ma.HDTC_id
		WHERE
		YEAR(crc.crcontract_end_date) = 2023
	) W ON ISNULL(X.DK_KHOANNO,'') +  ISNULL(X.PS_KHOANNO,'') = W.TT_KHOANNO
),

TSBD AS (
		SELECT		HDTC_id 'HDTC_ID', 
				COUNT(COL_ID) 'SO_TSBD'
				--SUM(latest_Value ) 'GIATRITSBD'
		FROM		Collateral A
		GROUP BY	HDTC_id 
),

DATA_RAW AS (
SELECT	A.*,
		B.HDTC_ID AS [TS_HCTC_ID]
		,B.SO_TSBD
		--,B.GIATRITSBD
FROM		SOLIEU_TSĐB A
INNER JOIN  TSBD B 
		ON A.DK_HDTC = B.HDTC_ID
		OR A.PS_HDTC = B.HDTC_ID
		OR A.CK_HDTC = B.HDTC_ID
		OR A.TT_HDTC = B.HDTC_ID)

-- JOIN với bảng Collateral để lấy thông tin về mã TSBĐ

--drop table QUYMO_TSBĐ
SELECT A.*, B.col_ID, B.latest_Value 
INTO QUYMO_TSBĐ
FROM DATA_RAW A
INNER JOIN Collateral B
		ON A.DK_HDTC = B.HDTC_ID
		OR A.PS_HDTC = B.HDTC_ID
		OR A.CK_HDTC = B.HDTC_ID
		OR A.TT_HDTC = B.HDTC_ID
		OR A.TS_HCTC_ID = B.HDTC_id
ORDER BY A.DK_KHOANNO

SELECT * FROM QUYMO_TSBĐ

-- SỐ LƯỢNG TSBĐ VÀ TỔNG GIÁ TRỊ TSBĐ

-- ĐẦU KỲ
SELECT COUNT(*)							AS [DAUKY_SO_LUONG_TSBĐ]
	,SUM(latest_Value)					AS [DAUKY_GIA_TRI_TSBĐ]
FROM(
	SELECT DISTINCT
		col_ID
		,latest_Value
	FROM QUYMO_TSBĐ
	WHERE DK_KHOANNO IS NOT NULL) DAU_KY

-- PHÁT SINH MỚI

-- 6 THÁNG ĐẦU NĂM

SELECT 
	COUNT(PHAT_SINH.col_ID)				AS [PST6TDN_SO_LUONG_TSBĐ]
	,SUM(PHAT_SINH.latest_Value)		AS [PST6TDN_GIA_TRI_TSBĐ]
	FROM 
		(SELECT DISTINCT
			col_ID
			,latest_Value
		FROM QUYMO_TSBĐ
		WHERE DK_KHOANNO IS NULL
		AND PS_NGAYBD BETWEEN '2023-01-01' AND '2023-06-30') PHAT_SINH
	LEFT JOIN
		(SELECT DISTINCT
			col_ID
			,latest_Value
		FROM QUYMO_TSBĐ
		WHERE DK_KHOANNO IS NOT NULL) DAU_KY
	ON PHAT_SINH.col_ID = DAU_KY.col_ID
	WHERE DAU_KY.col_ID IS NULL

-- 6 THÁNG CUỐI NĂM

SELECT 
	COUNT(PHAT_SINH_6TCN.col_ID)			AS [PST6TCN_SO_LUONG_TSBĐ]
	,SUM(PHAT_SINH_6TCN.latest_Value)		AS [PST6TCN_GIA_TRI_TSBĐ]
	FROM 
		(SELECT DISTINCT
			col_ID
			,latest_Value
		FROM QUYMO_TSBĐ
		WHERE DK_KHOANNO IS NULL
		AND PS_NGAYBD BETWEEN '2023-07-01' AND '2023-12-31') PHAT_SINH_6TCN
	LEFT JOIN
		(SELECT DISTINCT
			col_ID
			,latest_Value
		FROM QUYMO_TSBĐ
		WHERE DK_KHOANNO IS NOT NULL) DAU_KY
	ON PHAT_SINH_6TCN.col_ID = DAU_KY.col_ID
	LEFT JOIN
		(SELECT DISTINCT
			col_ID
			,latest_Value
		FROM QUYMO_TSBĐ
		WHERE DK_KHOANNO IS NULL
		AND PS_NGAYBD BETWEEN '2023-01-01' AND '2023-06-30') PHAT_SINH_6TDN
	ON PHAT_SINH_6TCN.col_ID = PHAT_SINH_6TDN.col_ID
	WHERE DAU_KY.col_ID IS NULL
	    AND PHAT_SINH_6TDN.col_ID IS NULL	


-- PHÁT SINH GIẢM

-- 6 THÁNG ĐẦU NĂM

SELECT 
	COUNT(*)							AS [PSG6TDN_SO_LUONG_TSBĐ]
	,SUM(TAT_TOAN.latest_Value)			AS [PSG6TDN_GIA_TRI_TSBĐ]
FROM(
	SELECT DISTINCT
		col_ID
		,latest_Value
	FROM QUYMO_TSBĐ
	WHERE CK_KHOANNO IS NULL
	AND TT_NGAYKT BETWEEN '2023-01-01' AND '2023-06-30') TAT_TOAN
LEFT JOIN 
	(SELECT DISTINCT
		col_ID
		,latest_Value
	FROM QUYMO_TSBĐ
	WHERE TT_KHOANNO IS NULL) CUOI_KY
ON TAT_TOAN.col_ID = CUOI_KY.col_ID
WHERE CUOI_KY.col_ID IS NULL

-- 6 THÁNG CUỐI NĂM

SELECT 
	COUNT(*)									AS [PSG6TCN_SO_LUONG_TSBĐ]
	,SUM(TAT_TOAN_6TCN.latest_Value)			AS [PSG6TCN_GIA_TRI_TSBĐ]
FROM(
	SELECT DISTINCT
		col_ID
		,latest_Value
	FROM QUYMO_TSBĐ
	WHERE CK_KHOANNO IS NULL
	AND TT_NGAYKT BETWEEN '2023-07-01' AND '2023-12-31') TAT_TOAN_6TCN
LEFT JOIN 
	(SELECT DISTINCT
		col_ID
		,latest_Value
	FROM QUYMO_TSBĐ
	WHERE TT_KHOANNO IS NULL) CUOI_KY
ON TAT_TOAN_6TCN.col_ID = CUOI_KY.col_ID
LEFT JOIN
	(SELECT DISTINCT
		col_ID
		,latest_Value
	FROM QUYMO_TSBĐ
	WHERE CK_KHOANNO IS NULL
	AND TT_NGAYKT BETWEEN '2023-01-01' AND '2023-06-30') TAT_TOAN_6TDN
ON TAT_TOAN_6TCN.col_ID = TAT_TOAN_6TDN.col_ID
WHERE CUOI_KY.col_ID IS NULL
AND TAT_TOAN_6TDN.col_ID IS NULL

-- CUỐI KỲ
SELECT 
	COUNT(*)							AS [CK_SO_LUONG_TSBĐ]
	,SUM(latest_Value)					AS [CK_GIA_TRI_TSBĐ]
FROM(
	SELECT DISTINCT
		col_ID
		,latest_Value
	FROM QUYMO_TS
	WHERE TT_KHOANNO IS NULL) CUOI_KY

```
**RESULT**

![image](https://github.com/user-attachments/assets/f3498f96-83ed-4ffd-be0f-eab8a3db1581)

**Based on the data regarding collateral (TSBD) for the first and second halves of 2023, we can draw several important insights as follows:**

**1. Number of Collateral (TSBD)**

+ Situation: The number of collateral assets increased from 145 at the end of 2022 to 154 by the end of 2023.
+ Analysis: In the first half of 2023, 25 new collateral assets were added, but 9 assets were reduced (likely through liquidation or resolving debts). In the second half of the year, the number of collateral increased by 19 assets, but 26 assets were reduced, leading to modest overall growth.
+ 
**Insight:** While the number of collateral assets did increase, the growth rate was slow, and the sharp reduction in collateral during the second half of the year suggests that the bank may have liquidated or decreased collateral to address non-performing loans.
  
**2. Total Collateral Value**

+ Situation: The total collateral value at the end of 2022 was 17,763,992,591,577 VND, and by the end of 2023, it slightly decreased to 17,583,123,160,056 VND.
+ Analysis:
In the first half of 2023, the total collateral value increased by 603,667,420,000 VND, but also saw a reduction of 429,394,000,000 VND, resulting in minimal net growth.
However, in the second half of the year, the total collateral value saw a significant increase of 1,309,214,248,973 VND, but also a large reduction of 1,664,357,100,494 VND. Ultimately, the total collateral value decreased slightly compared to the end of 2022.

**Insight:** Although there was growth in both the number and value of collateral in certain periods, the reduction in collateral value outpaced the growth, especially in the second half of the year. This could indicate that the bank had to liquidate a significant portion of collateral or experienced asset devaluation, leading to a decline in the total collateral value. The bank may need to review its collateral management process to preserve and enhance the value of these assets.

**Key Conclusions and Insights:**

**+ Volatility in Collateral Numbers:** The number of collateral assets fluctuated with slow growth and substantial reductions, particularly in the second half of the year. This could be a sign that the bank is addressing certain collateral assets or facing challenges in managing its debt portfolio.

**+ Slight Decrease in Total Collateral Value:** Although there was substantial growth in collateral value in some periods, the total collateral value at the end of 2023 slightly decreased compared to the end of 2022. This suggests that some collateral may have been liquidated or depreciated in value.

**+ Collateral Management:** The bank may need to improve its collateral management processes to avoid asset value depreciation and to optimize the use of collateral in mitigating credit risk.

In summary, while the number of collateral assets and their value did increase during certain periods of the year, the significant reductions in the second half require attention to ensure the bank can enhance its collateral value strategy in the future.

Query 3: 

```
sql

----------------------------
-- III. TĂNG TRƯỞNG QUA CÁC NĂM 
-----------------------------

--1. Tính tổng latest_Value cho từng năm

WITH GIATRIHDTD_CTT AS (
    SELECT 
        '2021' AS Nam,
        SUM(latest_Value) AS Tong_Gia_Tri
    FROM (
        SELECT DISTINCT
            COL.col_ID,
            col.latest_Value
        FROM COLLATERAL AS COL
        LEFT JOIN MORTGAGE_AGREEMENT AS MA ON COL.HDTC_id = MA.HDTC_id
        LEFT JOIN CREDIT_CONTRACT AS CC ON COL.HDTC_id = CC.sogiaodich
        LEFT JOIN CREDIT_PLAN AS CP ON CC.debid = CP.debid
        WHERE CP.ngaydauky <= '2021-12-31'
          AND CP.ngaycuoiky >= '2021-12-31'
          AND CC.crcontract_end_date > '2021-12-31'
    ) AS X

    UNION ALL

    SELECT 
        '2022' AS Nam,
        SUM(latest_Value) AS Tong_Gia_Tri
    FROM (
        SELECT DISTINCT
            COL.col_ID,
            col.latest_Value
        FROM COLLATERAL AS COL
        LEFT JOIN MORTGAGE_AGREEMENT AS MA ON COL.HDTC_id = MA.HDTC_id
        LEFT JOIN CREDIT_CONTRACT AS CC ON COL.HDTC_id = CC.sogiaodich
        LEFT JOIN CREDIT_PLAN AS CP ON CC.debid = CP.debid
        WHERE CP.ngaydauky <= '2022-12-31'
          AND CP.ngaycuoiky >= '2022-12-31'
          AND CC.crcontract_end_date > '2022-12-31'
    ) AS X

    UNION ALL

    SELECT 
        '2023' AS Nam,
        SUM(latest_Value) AS Tong_Gia_Tri
    FROM (
        SELECT DISTINCT
            COL.col_ID,
            col.latest_Value
        FROM COLLATERAL AS COL
        LEFT JOIN MORTGAGE_AGREEMENT AS MA ON COL.HDTC_id = MA.HDTC_id
        LEFT JOIN CREDIT_CONTRACT AS CC ON COL.HDTC_id = CC.sogiaodich
        LEFT JOIN CREDIT_PLAN AS CP ON CC.debid = CP.debid
        WHERE CP.ngaydauky <= '2023-12-31'
          AND CP.ngaycuoiky >= '2023-12-31'
          AND CC.crcontract_end_date > '2023-12-31'
    ) AS X
)

-- Tính tỷ lệ tăng trưởng so với năm gốc 2021

SELECT
    Nam,
    Tong_Gia_Tri,
    CASE 
        WHEN Nam = '2021' THEN 100
        WHEN Nam = '2022' THEN (Tong_Gia_Tri / (SELECT Tong_Gia_Tri FROM GIATRIHDTD_CTT WHERE Nam = '2021')) * 100
        WHEN Nam = '2023' THEN (Tong_Gia_Tri / (SELECT Tong_Gia_Tri FROM GIATRIHDTD_CTT WHERE Nam = '2022')) * 100
    END AS TyLe_TangTruong
FROM GIATRIHDTD_CTT

--2. Tính tổng dư nợ tín dụng cho từng năm

WITH GTTSBD_CTT AS (
    SELECT 
        YEAR(ngaydauky) AS Nam,
        SUM(B.sodudauky) AS Tong_Gia_Tri
    FROM credit_contract A
    INNER JOIN CREDIT_PLAN B ON A.debid = B.debid
    WHERE 
        (ngaydauky <= '2021-12-31' AND ngaycuoiky >= '2021-12-31' AND crcontract_end_date > '2021-12-31')
        OR (ngaydauky <= '2022-12-31' AND ngaycuoiky >= '2022-12-31' AND crcontract_end_date > '2022-12-31')
        OR (ngaydauky <= '2023-12-31' AND ngaycuoiky >= '2023-12-31' AND crcontract_end_date > '2023-12-31')
    GROUP BY YEAR(ngaydauky)
)

-- Tính tỷ lệ tăng trưởng so với năm gốc 2021

SELECT
    Nam,
    Tong_Gia_Tri,
    CASE 
        WHEN Nam = 2021 THEN 100
        WHEN Nam = 2022 THEN (Tong_Gia_Tri / (SELECT Tong_Gia_Tri FROM GTTSBD_CTT WHERE Nam = 2021)) * 100
        WHEN Nam = 2023 THEN (Tong_Gia_Tri / (SELECT Tong_Gia_Tri FROM GTTSBD_CTT WHERE Nam = 2022)) * 100
    END AS TyLe_TangTruong
FROM GTTSBD_CTT

```
**RESULT**

![image](https://github.com/user-attachments/assets/5edd8b36-2691-4817-a13d-5af4465cec6a)

**From the table provided, we can extract and analyze the key trends regarding both the outstanding loan contracts (HĐTD) and the collateral (TSBD) over the three years (2021, 2022, and 2023). Below is an analysis based on the data:**

**1. Outstanding Loan Contracts (HĐTD):**

+ 2021: The total outstanding loan value was 2,702,834,923,508 VND.
+ 2022: The value decreased slightly to 2,602,895,025,966 VND, showing a 96.30% growth rate compared to 2021.
+ 2023: There was a significant drop in outstanding loans to 1,650,433,320,888 VND, with a growth rate of 63.41% compared to 2022.
  
**Analysis:**

The decrease in loan values from 2021 to 2022 was modest, which suggests that the bank maintained a relatively stable loan portfolio during this period.

However, the sharp decline in 2023 indicates a major reduction in outstanding loans. This could reflect several possibilities:

+ The bank may have focused on recovering loans and reducing exposure to risk.
+ There could have been a decline in demand for loans or more stringent lending policies.
+ Loan repayments might have accelerated, or there could have been write-offs of bad debt, leading to lower outstanding amounts.
  
**2. Total Collateral Value (TSBD):**

+ 2021: The total collateral value was 17,989,974,014,144 VND.
+ 2022: The collateral value slightly decreased to 17,763,992,591,577 VND, indicating a 98.74% growth rate compared to 2021.
+ 2023: The collateral value further decreased to 17,583,123,160,056 VND, showing a 98.98% growth rate compared to 2022.

**Analysis:**

+ The slight decline in collateral value from 2021 to 2023 suggests a relatively stable collateral base with only minor fluctuations. The decrease is not as sharp as the reduction in outstanding loans, which might indicate that the bank has continued to maintain a healthy collateral structure even as the loan portfolio shrank.
+ The slight decline could also suggest that either some collateral was liquidated or devalued, but the bank was still able to preserve most of its collateral value over time.
  
**Key Insights:**

**1. Loan Portfolio Management:**

The significant reduction in outstanding loans in 2023 could point to a more cautious lending approach or successful efforts in loan recovery. This may have been part of a risk management strategy to safeguard against potential losses or external economic factors influencing lending practices.

**2. Collateral Stability:**

Despite the reduction in loans, the collateral value has remained relatively stable, with only minimal reductions over the three-year period. This stability in collateral indicates that the bank has not been overly aggressive in liquidating collateral and has retained a strong asset base.

**3. Growth Trends:**

The reduction in growth rates, particularly in 2023, suggests that the bank has shifted focus from expanding its loan portfolio to consolidating its current assets and possibly focusing on risk mitigation and loan recovery. This strategic shift could be a reaction to changing market conditions or an internal decision to prioritize financial health over growth.

**Recommendations:**

**+ Future Loan Strategy:** The bank might consider revisiting its lending policies to identify areas where it can safely expand its loan portfolio while managing risk effectively.

**+ Collateral Management:** Maintaining the collateral base will continue to be important for mitigating risks related to loan defaults. Ensuring that collateral is properly valued and managed will safeguard the bank’s assets.

**+ Focus on Loan Recovery:** Given the large drop in outstanding loans, the bank could focus on further improving loan recovery and managing non-performing loans (NPLs) to ensure financial stability moving forward.

Overall, while the reduction in outstanding loans is noticeable, the stability of the collateral values suggests that the bank is managing its credit risks effectively.
