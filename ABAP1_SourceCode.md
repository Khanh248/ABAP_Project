```java
REPORT z_039_ass1_24.

"Data definition specific for all button"
DATA: it_invoice TYPE TABLE OF z039_invoices.
DATA: wa_invoice TYPE z039_invoices.
DATA: it_products TYPE TABLE OF z039_product.
DATA: wa_products TYPE z039_product.

"Begin of selection screen block with frame and title"
SELECTION-SCREEN BEGIN OF BLOCK grp WITH FRAME TITLE TEXT-016. 
   
  SELECTION-SCREEN BEGIN OF LINE.                    "The first button"
    SELECTION-SCREEN POSITION 10.
    PARAMETERS rbutton1 RADIOBUTTON GROUP rb1.
    SELECTION-SCREEN COMMENT 25(20) TEXT-001 FOR FIELD Rbutton1.
  SELECTION-SCREEN END OF LINE.

  SELECTION-SCREEN BEGIN OF LINE.                 " The second button"
    SELECTION-SCREEN POSITION 10.
    PARAMETERS rbutton2 RADIOBUTTON GROUP rb1.
    SELECTION-SCREEN COMMENT 18(32) TEXT-002 FOR FIELD Rbutton2.
    PARAMETERS input1 TYPE Z039_product-product_id.
  SELECTION-SCREEN END OF LINE.


  SELECTION-SCREEN BEGIN OF LINE.                  "The third button"
    SELECTION-SCREEN POSITION 10.
    PARAMETERS rbutton3 RADIOBUTTON GROUP rb1.
    SELECTION-SCREEN COMMENT 30(20) TEXT-003 FOR FIELD Rbutton3.
    PARAMETERS input2 TYPE p DECIMALS 2.
  SELECTION-SCREEN END OF LINE.


  SELECTION-SCREEN BEGIN OF LINE.                "The fourth button"
    SELECTION-SCREEN POSITION 10.      
    PARAMETERS rbutton4 RADIOBUTTON GROUP rb1.
    SELECTION-SCREEN COMMENT 20(32) TEXT-004 FOR FIELD Rbutton4.
  SELECTION-SCREEN END OF LINE.
SELECTION-SCREEN END OF BLOCK grp.


*General report information
WRITE:/'General Report Information:' COLOR 7 INTENSIFIED OFF.
ULINE AT /(40).
WRITE:/ 'Current Date: ', sy-datum DD/MM/YYYY.
WRITE:/ 'Current Time:', sy-uzeit USING EDIT MASK '__:__:____'.
WRITE: /'User name: ',sy-uname.
ULINE AT /(40).


**The first button
IF rbutton1 ='X'.
  INCLUDE Z_039_ASS1_RD1.      
  PERFORM Results_generated.
ENDIF.

** The second button
IF rbutton2 = 'X'.
  INCLUDE Z_039_ASS1_RD2.
  ULINE.
  PERFORM Results_generated.
ENDIF.

***The third button

IF rbutton3 = 'X'.
INCLUDE z_039_ASS1_RD3.
ULINE.
PERFORM Results_generated.
ENDIF.

****The fourth button
IF rbutton4 ='X'.
INCLUDE Z_039_ASS1_RD4.

ENDIF.

*Subroutine: The number of results
FORM Results_generated.
  IF sy-dbcnt = 0.    "sy-dbcnt: count number of records from a table"
    WRITE: / 'There are no results generated.'.
  ELSE.
    WRITE:/ 'Result generated: ', sy-dbcnt.

  ENDIF.
ENDFORM.


*&---------------------------------------------------------------------*
*& Include z_039_ass1_rd1
*&---------------------------------------------------------------------*

WRITE:/'Display all cars: '.
ULINE.

** Selecting rows from table product, copy rows to internal table.
SELECT *
  FROM z039_product
  INTO TABLE it_products.

*Write heading
FORMAT COLOR 7.
WRITE:/ Text-019, Text-013, Text-014, Text-012, Text-022, Text-023, Text-024, Text-015.
ULINE.
FORMAT COLOR OFF.

LOOP AT it_products INTO wa_products.
  WRITE:/ wa_products-product_id UNDER Text-019,
         wa_products-productname UNDER Text-013,
         wa_products-model UNDER Text-014,
         wa_products-brand UNDER Text-012 ,
         wa_products-engine UNDER Text-022,
         wa_products-topspeed UNDER Text-023,
         wa_products-fuel_efficiency UNDER Text-024 CENTERED ,
         wa_products-price UNDER Text-015 LEFT-JUSTIFIED.


ENDLOOP.

ULINE.


*&---------------------------------------------------------------------*
*& Include z_039_ass1_rd2
*&---------------------------------------------------------------------*
 DATA:BEGIN OF wa_invpr,
         invoice_num   TYPE  z039_invoices-invoice_num,
         invoice_date  TYPE  z039_invoices-invoice_date,
         customer_name TYPE  z039_invoices-customer_name,
         postal_code   TYPE  z039_invoices-postal_code,
         city          TYPE  z039_invoices-city,
         quantity      TYPE  z039_invoices-quantity,
         product_id    TYPE z039_invoices-product_id,
         amount        TYPE  z039_invoices-amount,
         brand         TYPE z039_product-brand,
         productname   TYPE z039_product-productname,
         model         TYPE z039_product-model,
         price         TYPE z039_product-price,
       END OF wa_invpr.
  DATA it_invpr LIKE STANDARD TABLE OF wa_invpr INITIAL SIZE 0.

  SELECT i~invoice_num, i~invoice_date,
i~customer_name,
i~postal_code,
i~city,
i~quantity,
i~amount,
p~brand,
p~productname,
p~product_id,
p~model,
p~price
FROM z039_product AS p
INNER JOIN z039_invoices AS i
ON i~product_id = p~product_id WHERE p~product_id = @input1
INTO CORRESPONDING FIELDS OF TABLE @it_invpr.

  IF sy-subrc = 0 AND sy-dbcnt > 0.
*Write heading
    FORMAT COLOR 7.
    WRITE:/ TEXT-005, TEXT-006, TEXT-007, TEXT-008, TEXT-009, TEXT-010,TEXT-011,TEXT-012,TEXT-013,TEXT-014, TEXT-015.
    ULINE.
    FORMAT COLOR OFF.

    LOOP AT it_invpr INTO wa_invpr.
      WRITE: / wa_invpr-invoice_num UNDER TEXT-005,
                wa_invpr-invoice_date UNDER TEXT-006,
                wa_invpr-customer_name UNDER TEXT-007,
                wa_invpr-postal_code UNDER TEXT-008 CENTERED,
                wa_invpr-city UNDER TEXT-009,
                wa_invpr-quantity UNDER TEXT-010 CENTERED,
               wa_invpr-amount UNDER TEXT-011 LEFT-JUSTIFIED,
                wa_invpr-brand UNDER TEXT-012,
                wa_invpr-productname UNDER TEXT-013,
                wa_invpr-model UNDER TEXT-014,
                wa_invpr-price UNDER TEXT-015 CENTERED.

    ENDLOOP.
  ENDIF.


*&---------------------------------------------------------------------*
*& Include z_039_ass1_rd3
*&---------------------------------------------------------------------*
 SELECT * FROM z039_invoices INTO TABLE it_invoice.
  SELECT * FROM z039_product INTO TABLE it_products.
**Updating the amount in internal table.
  LOOP AT it_invoice INTO wa_invoice.
    READ TABLE it_products INTO wa_products WITH KEY product_id = wa_invoice-product_id.
    IF sy-subrc = 0.
      wa_invoice-amount = wa_invoice-quantity * wa_products-price * input2.
      MODIFY TABLE it_invoice FROM wa_invoice.
    ENDIF.
  ENDLOOP.
** Updating invoice table
  MODIFY z039_invoices FROM TABLE it_invoice.

  WRITE:/'The amount is updated with following multiplication factor: ', input2.
  ULINE.
  SELECT * FROM z039_invoices INTO TABLE it_invoice.
  FORMAT COLOR 7.
  WRITE:/ TEXT-005, TEXT-006, TEXT-007, TEXT-008, TEXT-009, TEXT-010, TEXT-011.
  ULINE.
  FORMAT COLOR OFF.
  LOOP AT it_invoice INTO wa_invoice.

    WRITE:/ wa_invoice-invoice_num UNDER TEXT-005,
            wa_invoice-invoice_date UNDER TEXT-006,
            wa_invoice-customer_name UNDER TEXT-007,
            wa_invoice-postal_code UNDER TEXT-008,
            wa_invoice-city UNDER TEXT-009,
            wa_invoice-quantity UNDER TEXT-010,
            wa_invoice-amount UNDER TEXT-011.

  ENDLOOP.

*&---------------------------------------------------------------------*
*& Include z_039_ass1_rd4
*&---------------------------------------------------------------------*
  SELECT * FROM z039_invoices INTO TABLE it_invoice.
  DATA: id_count TYPE i.
  DATA: counter TYPE i.
  DATA: BEGIN OF wa_suminv,
          product_id    TYPE z039_product-product_id,
          total_amount  TYPE p DECIMALS 2,
          invoice_count TYPE i,
        END OF wa_suminv.
  DATA it_suminv LIKE STANDARD TABLE OF wa_suminv INITIAL SIZE 0.

  SELECT COUNT( DISTINCT product_id ) INTO @id_count
  FROM z039_product.

**count number of invoices and sum of invoice amount
  DO id_count TIMES.
    counter = sy-index.
    LOOP AT it_invoice INTO wa_invoice.

      IF wa_invoice-product_id = 'C0000' && counter OR wa_invoice-product_id = 'C000' && counter OR wa_invoice-product_id = 'C00' && counter.
        wa_suminv-invoice_count += 1.
        wa_suminv-total_amount += wa_invoice-amount.
      ENDIF.
    ENDLOOP.
**if a product exists in invoice table, add product_id to wa_suminv
    IF wa_suminv-invoice_count <> 0.
      wa_suminv-product_id = 'C0000' && counter.
      IF counter > 9.
        wa_suminv-product_id = 'C000' && counter.
      ELSEIF counter > 99.
        wa_suminv-product_id = 'C00' && counter.

      ENDIF.
      APPEND wa_suminv TO it_suminv.
      CLEAR wa_suminv.
    ENDIF.

  ENDDO.
  FORMAT COLOR 7.
  WRITE:/ TEXT-019, TEXT-018, TEXT-017.
  ULINE.
  LOOP AT it_suminv INTO wa_suminv.
    FORMAT COLOR OFF.
    WRITE: / wa_suminv-product_id UNDER TEXT-019,
             wa_suminv-invoice_count UNDER TEXT-018,
             wa_suminv-total_amount UNDER TEXT-017.

  ENDLOOP.
ULINE.
**determine the records existing in an internal table.
   IF sy-tfill = 0.
    WRITE: / 'There are no results generated.'.
  ELSE.
    WRITE:/ 'Results generated: ', sy-tfill.
  ENDIF.
```
