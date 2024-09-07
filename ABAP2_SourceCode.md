## Part A
### Includes:
```java
 
CLASS lcl_products DEFINITION.
  PUBLIC SECTION.

    METHODS: set_attributes
      IMPORTING
        im_id             TYPE    z039_product-product_id
        im_name           TYPE    z039_product-productname
        im_model          TYPE    z039_product-model
        im_brand          TYPE   z039_product-brand
        im_engine         TYPE    z039_product-engine
        im_speed          TYPE     z039_product-topspeed
        im_fuelEfficiency TYPE   z039_product-fuel_efficiency
        im_price          TYPE  z039_product-price,

      get_attributes,
      constructor.
    CLASS-METHODS: get_no_of_products EXPORTING no_of_products TYPE   i.
  PRIVATE SECTION.

    DATA: gv_id             TYPE    z039_product-product_id,
          gv_name           TYPE    z039_product-productname,
          gv_model          TYPE    z039_product-model,
          gv_brand          TYPE    z039_product-brand,
          gv_engine         TYPE    z039_product-engine,
          gv_speed          TYPE    z039_product-topspeed,
          gv_fuelEfficiency TYPE    z039_product-fuel_efficiency,
          gv_price          TYPE    z039_product-price.

    CLASS-DATA gv_no_of_products    TYPE   i.

ENDCLASS.

CLASS lcl_products IMPLEMENTATION.
METHOD constructor.
  gv_no_of_products  = gv_no_of_products + 1.
 ENDMETHOD.

*Total number of products
METHOD get_no_of_products.
  no_of_products = gv_no_of_products.
ENDMETHOD.

 METHOD  set_attributes.
   gv_id = im_id.
   gv_name = im_name.
   gv_model = im_model.
   gv_brand = im_brand.
   gv_engine = im_engine.
   gv_speed = im_speed.
   gv_fuelefficiency = im_fuelefficiency.
   gv_price = im_price.
 ENDMETHOD.

 METHOD get_attributes.
 WRITE: /'Product ID:      ', gv_id,
        /'Product name:    ', gv_name,
        /'Product model:   ', gv_model,
        /'Product brand:   ', gv_brand,
        /'Product engine:  ',gv_engine,
        /'Product speed:   ', gv_speed,
        /'Fuel efficiency: ', gv_fuelefficiency,
        /'Product price: ', gv_price,
        /.
 ENDMETHOD.

 ENDCLASS.
```

### Reports
```java
*&---------------------------------------------------------------------*
*& Report z_039_ass2a
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*
REPORT z_039_ass2a.
INCLUDE z_039_class_include_ass2a.
 DATA: it_products TYPE TABLE OF z039_product,
       wa_products TYPE z039_product,
       lc_product TYPE REF TO lcl_products,
       it_product_list TYPE TABLE OF REF TO lcl_products.

 START-OF-SELECTION.

 SELECT *
 FROM z039_product INTO TABLE it_products.
 SORT it_products BY product_id DESCENDING.

 LOOP AT it_products INTO wa_products.
 CREATE OBJECT lc_product.
  lc_product->set_attributes(
    EXPORTING
      im_id             = wa_products-product_id
      im_name           = wa_products-productname
      im_model          = wa_products-model
      im_brand          = wa_products-brand
      im_engine         = wa_products-engine
      im_speed          = wa_products-topspeed
      im_fuelefficiency = wa_products-fuel_efficiency
      im_price          = wa_products-price
  ).

    APPEND lc_product TO it_product_list.
   ENDLOOP.

   DATA lv_no_of_products TYPE i.
   CALL METHOD lcl_products=>get_no_of_products
     IMPORTING
       no_of_products = lv_no_of_products
     .
     WRITE: /'Total number of products: ',lv_no_of_products.
     ULINE.

   LOOP AT it_product_list INTO lc_product.
   FORMAT COLOR 7.
   WRITE: /'Object: ',sy-tabix.
   FORMAT COLOR OFF.
   lc_product->get_attributes(  ).
   ENDLOOP.

```

### b.	Part B
```java
Program Z_039_DYNPRO_ASS2_B

*&---------------------------------------------------------------------*
*& Module Pool      Z_039_DYNPRO_ASS2_B
*&---------------------------------------------------------------------*
*&
*&---------------------------------------------------------------------*

INCLUDE Z_039_DYNPRO_ASS2_TOP                   .    " Global Data

* INCLUDE Z_039_DYNPRO_ASS2_O01                   .  " PBO-Modules
* INCLUDE Z_039_DYNPRO_ASS2_I01                   .  " PAI-Modules
* INCLUDE Z_039_DYNPRO_ASS2_F01                   .  " FORM-Routines




INCLUDE z_039_dynpro_ass2_status_0100.

INCLUDE z_039_dynpro_ass2_b_user_0100.

INCLUDE z_039_dynpro_ass2_status_0200.

INCLUDE z_039_dynpro_ass2_user_0200.

INCLUDE z_039_dynpro_ass2_b_user_0200.

INCLUDE z_039_dynpro_ass2_input_valiid.


.

```



### Includes:
```java

*&---------------------------------------------------------------------*
*& Include Z_039_DYNPRO_ASS2_TOP                    - Module Pool      Z_039_DYNPRO_ASS2_B
*&---------------------------------------------------------------------*
PROGRAM Z_039_DYNPRO_ASS2_B.
TABLES z039_product.
DATA: ok_code LIKE sy-ucomm, wa_products TYPE z039_product.


*----------------------------------------------------------------------*
***INCLUDE Z_039_DYNPRO_ASS2_B_USER_0100.
*----------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_0100  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*


MODULE user_command_0100 INPUT.
  CASE ok_code.
    WHEN 'BACK'.
      LEAVE PROGRAM.
    WHEN 'SELECT'.
    CALL FUNCTION 'ENQUEUE_EZ_Z039_PRODUCT'
      EXPORTING
        mode_z039_product = 'E'
        client            = SY-MANDT
        product_id        = z039_product-product_id

      EXCEPTIONS
        foreign_lock      = 1
        system_failure    = 2
        others            = 3
      .

      SELECT SINGLE * FROM z039_product INTO wa_products WHERE product_id = z039_product-product_id.

  ENDCASE.
  CLEAR ok_code.



ENDMODULE.

*----------------------------------------------------------------------*
***INCLUDE Z_039_DYNPRO_ASS2_B_USER_0200.
*----------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_0200  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE user_command_0200 INPUT.

*Release shared lock

    CALL FUNCTION 'DEQUEUE_EZ_Z039_PRODUCT'
      EXPORTING
        mode_z039_product = 'S'
        client            = SY-MANDT
        product_id        = Z039_product-product_id
      .
  CASE ok_code.
    WHEN 'EXIT'.
      LEAVE PROGRAM.
    WHEN 'BACK'.
    WHEN'SAVE'.
        IF z039_product <> wa_products.
        
*       Create Exclusive lock

     CALL FUNCTION 'ENQUEUE_EZ_Z039_PRODUCT'
       EXPORTING
         mode_z039_product = 'E'
         client            = SY-MANDT
         product_id        = Z039_product-product_id

       EXCEPTIONS
         foreign_lock      = 1
         system_failure    = 2
         others            = 3
       .


          MODIFY z039_product FROM z039_product.

*Release Exclusive lock
      CALL FUNCTION 'DEQUEUE_EZ_Z039_PRODUCT'
        EXPORTING
          mode_z039_product = 'E'
          client            = SY-MANDT
          product_id        = Z039_product-product_id

        .
        ENDIF.
  ENDCASE.
  CLEAR ok_code.



ENDMODULE.



*----------------------------------------------------------------------*
***INCLUDE Z_039_DYNPRO_ASS2_INPUT_VALIID.
*----------------------------------------------------------------------*
*&---------------------------------------------------------------------*
*&      Module  Input_validation  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE Input_validation INPUT.

  SELECT SINGLE *
    FROM z039_product INTO wa_products
    WHERE product_id = z039_product-product_id.
  IF sy-subrc <> 0.
    MESSAGE e000(z_039_mecl) WITH ok_code.
  ENDIF.


ENDMODULE.

 

```
