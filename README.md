# SQL_Banking_Credit_Segment_Exploration
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
