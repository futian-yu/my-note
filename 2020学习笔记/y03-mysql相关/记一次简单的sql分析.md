**记一次简单的sql分析**

-- 压测环境
-- 1；2063条
 SELECT
	h.BALANCE_HEADER_ID,
	h.RECEIPT_HEADER_ID,
	h.BLANCE_NUMBER,
	h.GENERATION_SOURCE,
	h.DOCUMENT_TYPE_CODE,
	h.BUSINESS_TYPE,
	h.COMPANY_CODE,
	h.SOURCE_SYSTEM,
	h.BALANCE_DATE,
	h.CUSTOMER_ID,
	h.CUSTOMER_CODE,
	h.CUSTOMER_TYPE,
	h.CURRENCY_ID,
	h.CURRENCY_CODE,
	h.EXRATE_TYPE_ID,
	h.EXRATE_DATE,
	h.EXCHANGE_RATE,
	h.SUM_AMOUNT,
	h.SUM_BASE_AMOUNT,
	h.ACCOUNTING_DATE,
	h.TRSACTION_NUMBER,
	h.EMPLOYEE_EMAIL,
	h.INITIAL_FLAG,
	h.ONLINE_FLAG,
	h.PAYMENT_CHANNEL,
	h.BALANCE_STATUS,
	h.ACCOUNTING_STATUS,
	h.DB_VALUE,
	l.BALANCE_LINE_ID,
	l.LINE_NUM,
	l.BALANCE_CATEGORY,
	l.AMOUNT,
	l.BASE_AMOUNT
FROM
	klk_ar_balance_headers h,
	klk_ar_balance_lines l
WHERE
	h.BALANCE_HEADER_ID = l.BALANCE_HEADER_ID
AND h.BUSINESS_TYPE IN ('2', '3', '1', '4')
AND h.BALANCE_STATUS = 'CONFIRM'
AND h.ACCOUNTING_STATUS = 'NEW'
AND h.INITIAL_FLAG = 'N';

-- 2；5
SELECT
 h.PAYMENT_COMPANY,
 p.ACCOUNTING_DATE,
 p.CURRENCY_ID,
 c.CURRENCY_CODE,
 p.PAYMENT_AMOUNT,
 p.CUSTOMER_ID,
 p.REFUND_CHANNEL,
 p.REFUND_WAY,
 p.ATTRIBUTE4,
 p.ATTRIBUTE1,
 h.EXRATE_DATE,
h.CUSTOMER_CODE,
p.PAYMENT_INF_ID,
p.TRANSACTION_NUM,
P.DB_VALUE
FROM
 hscs_pub_currency_details c,
 hscs_ar_act_payment_inf p,
 hscs_ar_payment_headers h
WHERE
 h.PAYMENT_HEADER_ID = p.PAYMENT_HEADER_ID
AND c.CURRENCY_ID = p.CURRENCY_ID
AND p.ACCOUNTING_STATUS='NEW';

-- 3；0
SELECT
a.REC_HEADER_ID,
a.CUST_REC_NUMBER,
a.GENERATION_METHOD,
a.BUSINESS_TYPE,
a.REC_STATUS,
a.MERCHANT_ID,
a.MERCHANT_NUMBER,
a.COMPANY_CODE,
a.CURRENCY_CODE,
a.PAYMENT_CURRENCY_CODE,
a.EXRATE_TYPE_ID,
a.EXRATE_DATE,
a.EXCHANGE_RATE,
a.TAX_ID,
a.TAX_RATE,
a.REC_AMOUNT,
a.ACCOUNTING_STATUS,
a.ACCOUNTING_DATE,
a.ACCOUNTING_REMARK,
a.VENDOR_SETTLE_NO,
a.DB_VALUE,
b.LINE_TYPE,
b.AMOUNT,
b.TAX_AMOUNT,
b.UNINCLUD_TAX_AMOUNT,
b.AP_OPPOSITE_ACCOUNT,
b.INPUT_TAX_ACCOUNT
FROM
 KLK_AP_CUST_REC_HEADERS a,
 KLK_AP_CUST_REC_ADJUSTS b
WHERE
 a.REC_HEADER_ID = b.REC_HEADER_ID
AND a.ACCOUNTING_STATUS ='NEW' 
AND a.GENERATION_METHOD='MANUAL'
AND a.REC_STATUS='APPROVED';


select * from klk_ar_balance_headers


-- 一、小表驱动大表;
-- 二、表关联字段添加索引;
-- 三、如果再慢，考虑添加状态字段的索引(不一定合适);
-- 压测环境
-- 1
select count(*) from klk_ar_balance_headers; -- 15830
select count(*) from klk_ar_balance_lines; -- 25557

-- 2
--  hscs_ar_act_payment_inf p,
-- hscs_ar_payment_headers h,
-- hscs_pub_currency_details c
select count(*) from hscs_ar_act_payment_inf; -- 4775 
select count(*) from hscs_ar_payment_headers; -- 17450
select count(*) from hscs_pub_currency_details; -- 155； 提前，小表驱动大表

-- 3
--  KLK_AP_CUST_REC_HEADERS a,
-- KLK_AP_CUST_REC_ADJUSTS b
select count(*) from KLK_AP_CUST_REC_HEADERS; -- 11226
select count(*) from KLK_AP_CUST_REC_ADJUSTS; -- 695344

-- show index from
-- 1
show index from klk_ar_balance_headers; -- BALANCE_HEADER_ID关联字段普通索引已加
show index from klk_ar_balance_lines;

-- 2
-- 关联：WHERE
-- hscs_pub_currency_details c,
-- hscs_ar_act_payment_inf p,
-- hscs_ar_payment_headers h
-- h.PAYMENT_HEADER_ID = p.PAYMENT_HEADER_ID
-- AND c.CURRENCY_ID = p.CURRENCY_ID
-- AND p.ACCOUNTING_STATUS='NEW';
show index from hscs_pub_currency_details; -- PAYMENT_HEADER_ID关联字段普通索引已加
show index from hscs_ar_act_payment_inf; -- p 可以加关联字段 currency_id 为普通索引 ================
show index from hscs_ar_payment_headers;

-- KLK_AP_CUST_REC_HEADERS a,
-- KLK_AP_CUST_REC_ADJUSTS b
-- WHERE
-- a.REC_HEADER_ID = b.REC_HEADER_ID
-- AND a.ACCOUNTING_STATUS ='NEW' 
-- AND a.GENERATION_METHOD='MANUAL'
-- AND a.REC_STATUS='APPROVED';
-- 3
show index from KLK_AP_CUST_REC_HEADERS; -- 关联字段 REC_HEADER_ID普通索引已加 ================
show index from KLK_AP_CUST_REC_ADJUSTS;