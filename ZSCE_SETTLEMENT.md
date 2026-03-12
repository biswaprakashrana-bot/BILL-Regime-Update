*&---------------------------------------------------------------------*
*&  Include           ZSCE_SETTLEMENT_RPT_CLS
*&---------------------------------------------------------------------*
*-----------------------------------------------------------------------
* CHANGE HISTORY
*-----------------------------------------------------------------------
*SrNo| Date  | User ID |  Description  |  Change Label   | report output
*-----------------------------------------------------------------------
* 1|26.12.2017| pwc_159| Route id - Issue in Road case.
**                    TR: RD2K9A1AY7, Functional: Bangarraju.
*-----------------------------------------------------------------------
* 2|23.03.2019| pwc_159| Remove hanging SES -CD:8032088
**                    Functional: Wilson Revoori.
*-----------------------------------------------------------------------


CLASS lcl_data_selection DEFINITION FINAL.
  PUBLIC SECTION.
    METHODS:
            auth_check,
            screen_modify,
            validt_shipment,
            validt_delivery,
* BOC by MSAT_050 on 02.01.2018 19:03:24
            validt_shipment2,
            validt_delivery2,
            validate_shipment,
            validate_shipment5,
            validate_scrren,
            validate_chain, "Added By Kalpesh/Akshay 20.11.2025 RD2K9A5E56
            vrm_values,     "Added By Kalpesh/Akshay 26.11.2025 RD2K9A5E56
            hang_ses_data, " service entry sheet
            get_shipment_data,
            change_shipment IMPORTING headerdata TYPE bapishipmentheader
                                      headerdataaction  TYPE bapishipmentheaderaction
                                      headerdeadline TYPE bapishipmentheaderdeadline OPTIONAL
                                      headerdeadlineaction  TYPE bapishipmentheaderdeadlineact
                                      OPTIONAL
                            EXPORTING et_return TYPE bapiret2_t,
* EOC by MSAT_050 on 02.01.2018 19:03:24
            get_road_data,
            get_vehicle_details,       "added by trilok 18.11.2022
            update_zscm_chainship,     "Added By Kalpesh/Akshay 17.11.2025 RD2K9A5E56
            get_rail_data,
            get_marine_air_data,
            fill_field_catalog,
            icon_exclude,
            alv_data_save,
            road_data_save,
            rail_data_save,
            marine_air_data_save,
            get_coal_data,

            "added by col_048 on 18-01-2019
            suppl_plnt_upd,
            valid_suppl_plnt,
            get_shrotage_data,
            shortage_field_catalog,
            shrtage_data_save,
            get_inv_data,
            create_obj,
            create_fcat,
            display_alv,
            user_command FOR EVENT user_command OF cl_gui_alv_grid
                       IMPORTING e_ucomm  sender,
            toolbar FOR EVENT toolbar OF cl_gui_alv_grid
                      IMPORTING e_object e_interactive,
            handle_hotspot_click FOR EVENT hotspot_click OF cl_gui_alv_grid
                               IMPORTING e_row_id e_column_id es_row_no,
            create_popup_obj,
            create_popup_fcat,
            display_popup_alv,
            user_command_popup FOR EVENT user_command OF cl_gui_alv_grid
                       IMPORTING e_ucomm  sender,
            toolbar_popup FOR EVENT toolbar OF cl_gui_alv_grid
                      IMPORTING e_object e_interactive.
    "end by col_048 on 18-01-2019
    METHODS : handle_data_changed FOR EVENT data_changed
                                           OF cl_gui_alv_grid
                                    IMPORTING er_data_changed.
* BOC by MSAT_050 on 03.01.2018 12:05:28
    METHODS: handle_uc FOR EVENT user_command
                              OF cl_gui_alv_grid
                       IMPORTING e_ucomm,
             handle_tb FOR EVENT toolbar
                              OF cl_gui_alv_grid
                       IMPORTING  e_object
                                  e_interactive,
             handle_tb_rd FOR EVENT toolbar
                              OF cl_gui_alv_grid
                       IMPORTING  e_object
                                  e_interactive.


* EOC by MSAT_050 on 03.01.2018 12:05:28

    METHODS:
              disp_alv
              IMPORTING i_cont  TYPE char10
              CHANGING it_table TYPE STANDARD TABLE.
    METHODS: vehicle_save.
ENDCLASS.                    "lcl_data_selection DEFINITION

*----------------------------------------------------------------------*
*       CLASS lcl_data_selection IMPLEMENTATION
*----------------------------------------------------------------------*
*
*----------------------------------------------------------------------*
CLASS lcl_data_selection IMPLEMENTATION.

  METHOD auth_check.

    AUTHORITY-CHECK OBJECT 'S_TCODE'
              ID 'TCD' FIELD gc_tcode.
    IF sy-subrc <> 0.
      MESSAGE e726(zlog) WITH sy-uname.
    ENDIF.

  ENDMETHOD.                    "auth_check

  METHOD screen_modify.
* BOC by MSAT_050 on 02.01.2018 18:53:47
*    IF p_sp IS INITIAL.
*    IF p_act EQ 'D'.
    IF p_act NE 'D'. " changed by omkar more on 02.12.2025 CD:8086356 TR:RD2K9A5ENP
* EOC by MSAT_050 on 02.01.2018 18:53:47
      LOOP AT SCREEN.
        IF screen-name = 'S_TKNUM-LOW' OR
           screen-name = 'S_VBELN-LOW'.
          screen-required = 2.  " 2- Oblicatory
          MODIFY SCREEN.
        ELSEIF screen-group1 = 'SPU'.
          screen-active = 0.
          MODIFY SCREEN.
        ENDIF.
        IF screen-group1 = 'SGT'.
          screen-input = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ENDIF.
      ENDLOOP.
* BOC by MSAT_050 on 02.01.2018 18:54:30
    ELSE.
      LOOP AT SCREEN.
        IF screen-group1 = 'MDF'.
          screen-active = 0.
          MODIFY SCREEN.
        ENDIF.
        IF screen-group1 = 'SGT'.
          screen-input = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ENDIF.
      ENDLOOP.
    ENDIF.
* EOC by MSAT_050 on 02.01.2018 18:54:30


    "@ Added by Sonu Kumar on 31.10.2018

*    IF p_uc EQ abap_true.
    IF p_act EQ 'E'.
      LOOP AT SCREEN.
        IF screen-group1 = 'MDF' OR screen-group1 = 'SPU'.
          screen-active = 0.
*          screen-input  = 1.
          MODIFY SCREEN.
        ENDIF.
        IF screen-group1 = 'SGT'.
          screen-input = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ENDIF.
      ENDLOOP.
    ELSE.
      LOOP AT SCREEN.
        IF screen-group1 = 'UC1'.
          screen-active = 0.
*          screen-input = 1.
          MODIFY SCREEN.
        ENDIF.
        IF screen-group1 = 'SGT'.
          screen-input = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ENDIF.
      ENDLOOP.
    ENDIF.
    "@ End of changes by Sonu Kumar

    "added by col_048 on 18-01-2019
    LOOP AT SCREEN.
*      IF p_sup IS NOT INITIAL.
      IF p_act EQ 'G'.
        IF  screen-group1  = 'UC1'
          OR screen-group1 = 'MDF'.

          screen-input = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ELSEIF screen-group1 = 'SPU'
          OR screen-group1 = 'SES'. " (+) by pwc_159 on 23.03.2019

          screen-input = 0.
          screen-required  = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ENDIF.
        IF screen-group1 = 'SGT'.
          screen-input = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ENDIF.

      ELSE.
        IF screen-group1 EQ 'SP1' OR
           screen-group1 EQ 'SES'.  " (+) by pwc_159 on 23.03.2019

          screen-input = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ENDIF.
        IF screen-group1 = 'SGT'.
          screen-input = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ENDIF.
      ENDIF.


      IF screen-name = 'P_STYP'.
        screen-input = 0.
        MODIFY SCREEN.
      ENDIF.

    ENDLOOP.

*    IF p_se EQ abap_true.
    IF p_act EQ 'H'.
      LOOP AT SCREEN.
        IF screen-group1  = 'SES'.
          screen-active = 1.  " to disply
          screen-input = 1.   " input enable
          screen-invisible = 0.

          IF screen-name = 'P_FKNUM' OR
             screen-name = 'S_LBLNI-LOW'.

            screen-required = 2.
          ENDIF.
          MODIFY SCREEN.
        ELSEIF screen-group1 = 'MDF'.
          screen-input = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ELSEIF screen-group1 = 'SGT'.
          screen-input = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ELSEIF screen-group1 =  'INV'.
          screen-input = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ENDIF.
      ENDLOOP.
    ENDIF.
*    IF p_st = abap_true.
    IF p_act EQ 'F'.
      LOOP AT SCREEN.
        IF screen-group1 = 'SGT'.
          screen-input = 1.
          screen-invisible = 0.
          MODIFY SCREEN.
        ENDIF.
        CASE screen-group1.
          WHEN 'MDF'.
            screen-input = 0.
            screen-invisible = 1.
            MODIFY SCREEN.
          WHEN 'SES'.
            screen-input = 0.
            screen-invisible = 1.
            MODIFY SCREEN.
          WHEN 'SP1'.
            screen-input = 0.
            screen-invisible = 1.
            MODIFY SCREEN.
          WHEN 'SPU'.
            screen-input = 0.
            screen-invisible = 1.
            MODIFY SCREEN.
          WHEN 'UC1'.
            screen-input = 0.
            screen-invisible = 1.
            MODIFY SCREEN.
          WHEN 'INV' .
            screen-input = 0.
            screen-invisible = 1.
            MODIFY SCREEN.
        ENDCASE.
      ENDLOOP.
    ENDIF.
*    IF p_inv = abap_true.
    IF p_act EQ 'I'.
      LOOP AT SCREEN.
        IF screen-group1 = 'INV'.
          screen-input = 1.
          screen-invisible = 0.
          MODIFY SCREEN.
        ENDIF.
        CASE screen-group1.
          WHEN 'SGT'.
            screen-input = 0.
            screen-invisible = 1.
            MODIFY SCREEN.
          WHEN 'MDF'.
            screen-input = 0.
            screen-invisible = 1.
            MODIFY SCREEN.
          WHEN 'SES'.
            screen-input = 0.
            screen-invisible = 1.
            MODIFY SCREEN.
          WHEN 'SP1'.
            screen-input = 0.
            screen-invisible = 1.
            MODIFY SCREEN.
          WHEN 'SPU'.
            screen-input = 0.
            screen-invisible = 1.
            MODIFY SCREEN.
          WHEN 'UC1' .
            screen-input = 0.
            screen-invisible = 1.
            MODIFY SCREEN.
        ENDCASE.
      ENDLOOP.
    ELSE.
      LOOP AT SCREEN.
        IF screen-group1 = 'INV'.
          screen-input = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ENDIF.
      ENDLOOP.
    ENDIF.
    "end on 18-01-2019

*****************BOC Added by Hiral on 21.06.2022***********
*    IF p_gt = abap_true.
    IF p_act EQ 'J'.
      LOOP AT SCREEN.
        IF screen-group1 = 'GTA'.
          screen-input = 1.
          screen-invisible = 0.
          MODIFY SCREEN.
        ELSEIF screen-group1 = 'MDF'
          OR screen-group1 = 'SPU'.
          screen-active = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ENDIF.
      ENDLOOP.
    ELSE.
      LOOP AT SCREEN.
        IF screen-group1 = 'GTA'.
          screen-active = 0.
          screen-invisible = 0.
          MODIFY SCREEN.
        ENDIF.
      ENDLOOP.
    ENDIF.
* ******************EOC Added by Hiral on 21.06.2022***********
*************************    BOC BY TRILOK 16.11.2022
*    IF p_vc = abap_true .
    IF p_act EQ 'K'.
      LOOP AT SCREEN.
        IF screen-group1 = 'M1'.
          screen-input = 1.
          screen-invisible = 0.
          MODIFY SCREEN.
        ELSEIF screen-group1 = 'MDF'
           OR screen-group1 = 'SPU'.
          screen-active = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ENDIF.
      ENDLOOP.
    ELSE.
      LOOP AT SCREEN.
        IF screen-group1 = 'M1'.
          screen-active = 0.
          screen-invisible = 0.
          MODIFY SCREEN.
        ENDIF.
      ENDLOOP.
    ENDIF.

*********************    EOC BY TRILOK 16.11.2022

    "Added By Kalpesh/Akshay 17.11.2025 RD2K9A5E56
*    IF p_mc EQ abap_true.
    IF p_act EQ 'L'.
      LOOP AT SCREEN.
        IF screen-group1 = 'MC'.
          screen-input = 1.
          screen-invisible = 0.
          MODIFY SCREEN.
        ELSEIF screen-group1 = 'MDF'
           OR screen-group1 = 'SPU'.
          screen-active = 0.
          screen-invisible = 1.
          MODIFY SCREEN.
        ENDIF.
      ENDLOOP.
    ELSE.
      LOOP AT SCREEN.
        IF screen-group1 = 'MC'.
          screen-active = 0.
          screen-invisible = 0.
          MODIFY SCREEN.
        ENDIF.
      ENDLOOP.
    ENDIF.
    "Added By Kalpesh/Akshay 17.11.2025 RD2K9A5E56

  ENDMETHOD.                    "screen_modify

  METHOD validt_shipment.

    IF sy-ucomm EQ 'ONLI' AND p_act NE 'D' AND p_act NE 'E' AND p_act NE 'J' "p_sp IS INITIAL AND p_uc EQ abap_false AND p_gt IS INITIAL
      AND p_act NE 'G' "AND p_sup IS INITIAL "added on 21-01-2019
      AND p_act NE 'H' "AND p_se IS INITIAL
      AND p_act NE 'F' "AND p_st IS INITIAL
      AND p_act NE 'I' "AND p_inv IS INITIAL
      AND p_act NE 'K' "AND p_vc IS INITIAL  "TRILOK 18.11.2022
      AND p_act NE 'L'. "AND p_mc IS INITIAL. "Added By Kalpesh/Akshay 17.11.2025 RD2K9A5E56
*      AND p_act NE 'B'.

      IF s_tknum[] IS INITIAL.
        MESSAGE 'Enter shipment number'(040) TYPE 'E'.
      ELSE.
        SELECT tknum
               FROM vttk CLIENT SPECIFIED
               INTO TABLE gt_tknum
               WHERE mandt = sy-mandt   " eswara
                 AND tknum IN s_tknum.
        IF sy-subrc NE 0.
          SELECT shnumber
                 FROM oigs CLIENT SPECIFIED
                 INTO TABLE gt_tknum
                WHERE client = sy-mandt  " eswara
                 AND shnumber IN s_tknum.
          IF sy-subrc NE 0.
            MESSAGE 'Given shipment number is not valid'(039) TYPE 'E'.
          ENDIF.
        ENDIF.
      ENDIF.
    ENDIF.
  ENDMETHOD.                    "validt_shipment

  METHOD validt_delivery.

    TYPES: BEGIN OF lty_vbeln,
            vbeln TYPE likp-vbeln,
           END OF lty_vbeln.

    DATA: lt_vbeln TYPE TABLE OF lty_vbeln.

    IF sy-ucomm EQ 'ONLI' AND p_act NE 'D' AND p_act NE 'E' AND p_act NE 'J' "p_sp IS INITIAL AND p_uc EQ abap_false AND p_gt IS INITIAL
      AND p_act NE 'G' "AND p_sup IS INITIAL "added on 21-01-2019
      AND p_act NE 'H' "AND p_se IS INITIAL
      AND p_act NE 'F' "AND p_st IS INITIAL
      AND p_act NE 'I' "AND p_inv IS INITIAL
      AND p_act NE 'K' "AND p_vc IS INITIAL  "TRILOK 18.11.2022
      AND p_act NE 'L'. "AND p_mc IS INITIAL. "Added By Kalpesh/Akshay 17.11.2025 RD2K9A5E56
*      AND p_act NE 'B'.


      IF s_vbeln[] IS INITIAL.
        MESSAGE 'Enter Delivery number'(037) TYPE 'E'.
      ELSE.
        SELECT vbeln
               FROM likp
               INTO TABLE lt_vbeln
               WHERE vbeln IN s_vbeln.
        IF sy-subrc NE 0.
          MESSAGE 'Given Delivery number is not valid'(038) TYPE 'E'.
        ENDIF.
        REFRESH: lt_vbeln.
      ENDIF.
    ENDIF.

  ENDMETHOD.                    "validt_delivery

  METHOD get_road_data.

    "added on 01-03-2019 by col_048
    TYPES:BEGIN OF lty_stlmntvr,
          name  TYPE rvari_vnam,
          numb  TYPE tvarv_numb,
          shtyp TYPE shtyp,
          lifnr TYPE lifnr,
          END OF lty_stlmntvr.
    DATA:lt_stlmnt TYPE TABLE OF lty_stlmntvr .
    "end on 01-03-2019

    TYPES: BEGIN OF lty_oigsv,
            shnumber TYPE oigsv-shnumber,
            route TYPE oigsv-route,
           END OF lty_oigsv,

           BEGIN OF lty_tvro,
             route TYPE tvro-route,
             routid TYPE tvro-routid,
           END OF lty_tvro.

    DATA:
          lt_oigsv  TYPE TABLE OF lty_oigsv,
          lt_tvro   TYPE TABLE OF lty_tvro,
          lw_oigsv  TYPE lty_oigsv,
          lw_tvro   TYPE lty_tvro,
          lw_shtyp  TYPE vttk-shtyp,
          lw_tdlnr  TYPE vttk-tdlnr,
          lt_prm    TYPE TABLE OF zlog_stlmntvr.

    REFRESH:
       gt_trstlmnt_rd, gt_trstlmnt_rd_old.

    SELECT tknum
           vbeln
           counter
           vsart
           bukrs
           pro_status
           mfrgr
           lr_no
           lr_dt
           lr_qty
           lr_uom
           trans_libty_amt
           trans_libty_cur    " Added by Santhosh Chowdary For liability amount
           grn_no
           grn_qty
           grn_uom
           route AS routid
           flt_ind
           digind
           bill_vendor
           saccode
           tr_bill_dt
           tr_bill_amt
           dep_point
           des_point
           frt_val
           headrun_amt   "+TRILOK 10.11.2022   RD2K9A417Y
           bill_regime
         FROM ztrstlmnt
         INTO TABLE gt_trstlmnt_rd
         WHERE tknum IN s_tknum
         AND   vbeln IN s_vbeln.
    IF sy-subrc NE 0.
      MESSAGE text-034 TYPE 'S'.
      LEAVE LIST-PROCESSING.
    ELSE.

      SELECT SINGLE shtyp tdlnr
               FROM vttk
               INTO (lw_shtyp , lw_tdlnr )
               WHERE tknum IN s_tknum.
      IF sy-subrc = 0.

        SELECT name
             numb
             shtyp
             lifnr
        FROM zlog_stlmntvr
        INTO TABLE lt_stlmnt
        WHERE name  = 'ZSCE_INVAMT_EDIT'
         AND shtyp  = lw_shtyp
         AND active = 'X'
         AND lifnr  = lw_tdlnr.

        "commented on 01-03-2019
*        SELECT * FROM zlog_stlmntvr INTO TABLE lt_prm
*                            WHERE name = 'ZSCE_GST_VENDOR_RD'
*                              AND shtyp = lw_shtyp
*                              AND lifnr = lw_tdlnr
*                              AND active = 'X'.
        "end on 01-03-2019
        IF sy-subrc = 0.
          gw_chk = 'X'.
        ENDIF.
      ENDIF.
    ENDIF.
**-----------------------------------------------------------------------
**  To get ROUTE value, Pass SHNUMBER to OIGSV table and get the ROUTE
**    and pass Route to TVRO and get the ROUTID.
**-----------------------------------------------------------------------
    IF s_tknum[] IS NOT INITIAL.
      SELECT shnumber
             route
             FROM oigsv INTO TABLE lt_oigsv
             WHERE shnumber IN s_tknum.
      IF sy-subrc EQ 0.
        SORT lt_oigsv BY route.
        DELETE lt_oigsv WHERE route IS INITIAL.
        DELETE ADJACENT DUPLICATES FROM lt_oigsv COMPARING route.
        IF lt_oigsv[] IS NOT INITIAL.
          SELECT route
                 routid
                 FROM tvro
                 INTO TABLE lt_tvro
                 FOR ALL ENTRIES IN lt_oigsv
                 WHERE route = lt_oigsv-route.
          IF sy-subrc EQ 0.
            SORT lt_tvro BY route.
          ENDIF.
        ENDIF.
      ENDIF.
    ENDIF.

    SORT lt_oigsv BY shnumber.
    LOOP AT gt_trstlmnt_rd ASSIGNING <gfs_trstlmnt_rd>.
      CLEAR <gfs_trstlmnt_rd>-routid.
      READ TABLE lt_oigsv INTO lw_oigsv WITH KEY
                                shnumber = <gfs_trstlmnt_rd>-tknum
                                BINARY SEARCH.
      IF sy-subrc EQ 0.
        READ TABLE lt_tvro INTO lw_tvro WITH KEY
                                route = lw_oigsv-route
                                BINARY SEARCH.
        IF sy-subrc EQ 0.
          <gfs_trstlmnt_rd>-routid = lw_tvro-routid .  " Routeid
        ENDIF.
      ENDIF.
      MOVE-CORRESPONDING <gfs_trstlmnt_rd> TO gw_trstlmnt_rd_chk.

      APPEND gw_trstlmnt_rd_chk TO gt_trstlmnt_rd_chk.

      CLEAR: lw_oigsv, lw_tvro, gw_trstlmnt_rd_chk.
    ENDLOOP.

    IF <gfs_trstlmnt_rd> IS ASSIGNED.
      UNASSIGN <gfs_trstlmnt_rd>.
    ENDIF.

    REFRESH: gt_tknum, lt_tvro, lt_oigsv.

    gt_trstlmnt_rd_old = gt_trstlmnt_rd.



  ENDMETHOD.                   "get_road_data

******************************************************  BOC BY TRILOK 18.11.2022
  METHOD  get_vehicle_details.

    TYPES : BEGIN OF lty_yttstm0001,
         truck_no TYPE ytruck_no,
         veh_text TYPE oig_vhtytx,
           END OF lty_yttstm0001.

    TYPES : BEGIN OF lty_oigv,
         vehicle TYPE oig_vhlnmr,
         veh_id TYPE oig_vehid,
           END OF lty_oigv.

    DATA : lt_yttstm0001 TYPE TABLE OF lty_yttstm0001,
           lw_yttstm0001 TYPE lty_yttstm0001,
           lt_oigv TYPE TABLE OF lty_oigv,
           lw_oigv TYPE lty_oigv,
           lw_vc_num LIKE LINE OF s_vc_num.

    IF s_vc_num IS NOT INITIAL.
      SELECT truck_no
             veh_text
             FROM yttstm0001 INTO TABLE lt_yttstm0001
             WHERE truck_no IN s_vc_num.
      IF lt_yttstm0001 IS NOT INITIAL .
        SORT : lt_yttstm0001 BY  truck_no .
        SELECT vehicle
               veh_id
               FROM oigv
              INTO TABLE lt_oigv
              WHERE veh_id  IN s_vc_num..
        IF lt_oigv IS NOT INITIAL.
          SORT lt_oigv BY vehicle.
          LOOP AT s_vc_num INTO lw_vc_num.
            READ TABLE lt_yttstm0001 INTO lw_yttstm0001 WITH KEY truck_no = lw_vc_num-low.
            IF sy-subrc = 0.
              gw_vcdet-truck_no = lw_yttstm0001-truck_no.
            ENDIF.

            READ TABLE lt_oigv INTO lw_oigv WITH KEY veh_id = lw_vc_num-low.
            IF sy-subrc = 0.
              gw_vcdet-vehicle = lw_oigv-vehicle .
              APPEND : gw_vcdet TO gt_vcdet.
            ENDIF.
          ENDLOOP.
*          LOOP AT lt_yttstm0001 INTO lw_yttstm0001  .
*            gw_vcdet-truck_no = lw_yttstm0001-truck_no.
*            APPEND : gw_vcdet TO gt_vcdet.
*            CLEAR : lw_yttstm0001 , gw_vcdet .
*          ENDLOOP.
*
*          LOOP AT lt_oigv INTO lw_oigv.
*            gw_vcdet-vehicle = lw_oigv-vehicle .
*            APPEND : gw_vcdet TO gt_vcdet.
*            CLEAR : lw_oigv , gw_vcdet .
*          ENDLOOP.
        ENDIF.
      ENDIF.
    ENDIF.

  ENDMETHOD.                       " GET VEHICLE DETAILS
******************************************************  EOC BY TRILOK 18.11.2022
  METHOD update_zscm_chainship.    ""Added By Kalpesh/Akshay 17.11.2025 RD2K9A5E56

    IF p_chain IS NOT INITIAL AND p_del IS NOT INITIAL AND
       p_shnu IS NOT INITIAL.
      CLEAR : gw_chain.
      SELECT mandt                                "Always single record against single chainID shipment and delivery
             chainid
             odpairid
             legid
             shnumber
             delivery
             del_ind
        FROM zscm_chainship CLIENT SPECIFIED
        INTO gw_chain UP TO 1 ROWS
        WHERE mandt = sy-mandt
        AND chainid = p_chain
        AND shnumber = p_shnu
        AND delivery = p_del.
      ENDSELECT.
      IF sy-subrc = 0.

        APPEND gw_chain TO gt_chain.
        UPDATE zscm_chainship SET del_ind = p_ind
                              WHERE chainid = gw_chain-chainid
                              AND  odpairid = gw_chain-odpairid
                              AND   legid   = gw_chain-legid
                              AND  shnumber = gw_chain-shnumber
                              AND  delivery = gw_chain-delivery.
        IF sy-subrc = 0.
          COMMIT WORK.
          gw_updchain = abap_true.
        ELSE.
          ROLLBACK WORK.
        ENDIF.
      ELSE.
        gw_flg = abap_true.
      ENDIF.
    ENDIF.
  ENDMETHOD.                    "update_zscm_chainship

  METHOD get_rail_data.

    REFRESH:
       gt_trstlmnt_rl, gt_trstlmnt_rl_old.

    SELECT tknum
           vbeln
           counter
           vsart
           pro_status      "trilok - 19.01.2023
           mfrgr
           lr_no
           lr_dt
           fknum          "trilok - 31.01.2023
           grn_no
           grn_dt
           grn_qty
           grn_uom
           rake_ldqty
           scenr
           bill_vendor
           saccode
           digind
         FROM ztrstlmnt_rail
         INTO TABLE gt_trstlmnt_rl
         WHERE tknum IN s_tknum
         AND   vbeln IN s_vbeln.
    IF sy-subrc NE 0.
      MESSAGE text-034 TYPE 'S'.
      LEAVE LIST-PROCESSING.
    ENDIF.

    gt_trstlmnt_rl_old = gt_trstlmnt_rl.

  ENDMETHOD.                    "gt_rail_data

  METHOD get_marine_air_data.

    REFRESH: gt_trstlmnt_ma, gt_trstlmnt_ma_old.

    SELECT
          tknum
          vbeln
          counter
          vsart
          digind
          bill_vendor
          saccode
          pro_status
          shtyp
          bukrs
          bl_no
          bl_dt
          hawb_no
          hawb_dt
          zchgbl_wgt
          chgbl_wt_uom
          mawbno
          mawbdt
          shipbillno
          shipbilldt
          FROM ztrstlmnt_m_a
          INTO TABLE gt_trstlmnt_ma
          WHERE tknum IN s_tknum
          AND   vbeln IN s_vbeln.
    IF sy-subrc NE 0.
      MESSAGE text-034 TYPE 'S'.
      LEAVE LIST-PROCESSING.
    ENDIF.

    CLEAR gt_trstlmnt_ma_temp[].
    gt_trstlmnt_ma_old = gt_trstlmnt_ma_temp = gt_trstlmnt_ma.

    " soc by omkar more on 12.03.2025 TR:RD2K9A54FE
    CLEAR gt_zlog_exec_var[].
    SELECT name numb shtyp active remarks
    FROM  zlog_exec_var CLIENT SPECIFIED
    INTO TABLE gt_zlog_exec_var
    WHERE mandt = sy-mandt
    AND   name IN (lc_zsce_ztrslmnt_imp,lc_zsce_ztrslmnt_shproc,lc_zsce_ztrstlmnt_exp)
    AND   active = abap_true.
    IF sy-subrc = 0.

      SORT gt_trstlmnt_ma_temp BY vbeln.
      DELETE ADJACENT DUPLICATES FROM gt_trstlmnt_ma_temp COMPARING vbeln.
      IF gt_trstlmnt_ma_temp IS NOT INITIAL.

        CLEAR gt_/obiz/zbn_dochdr[].
        SELECT doccat docnr docdate refdoccat refdocnr refdocdt bolnr boldt hawbno hawbdt mawbno mawbdt
        FROM /obiz/zbn_dochdr CLIENT SPECIFIED
        INTO TABLE gt_/obiz/zbn_dochdr
        FOR ALL ENTRIES IN gt_trstlmnt_ma_temp
        WHERE mandt    = sy-mandt
        AND   refdocnr = gt_trstlmnt_ma_temp-vbeln.
        IF sy-subrc = 0.
          SORT gt_/obiz/zbn_dochdr BY refdocnr.
        ENDIF.

        CLEAR gt_lips[].
        SELECT vbeln posnr vgbel
        FROM lips CLIENT SPECIFIED
        INTO TABLE gt_lips
        FOR ALL ENTRIES IN gt_trstlmnt_ma_temp
        WHERE mandt = sy-mandt
        AND   vbeln = gt_trstlmnt_ma_temp-vbeln.
        IF sy-subrc = 0.

          CLEAR gt_lips_temp[].
          gt_lips_temp = gt_lips.
          SORT gt_lips_temp BY vgbel.
          DELETE ADJACENT DUPLICATES FROM gt_lips_temp COMPARING vgbel.
          IF gt_lips_temp IS NOT INITIAL.

            CLEAR gt_zsosto[].
            SELECT  vbeln so_posnr ebeln posnr zusage eloek
            FROM zsosto CLIENT SPECIFIED
            INTO TABLE gt_zsosto
            FOR ALL ENTRIES IN gt_lips_temp
            WHERE mandt = sy-mandt
            AND   vbeln = gt_lips_temp-vgbel.
            IF sy-subrc = 0.

              DELETE gt_zsosto WHERE zusage NE 'ML'.
              DELETE gt_zsosto WHERE eloek  NE ''.

              CLEAR gt_zsosto_temp[].
              gt_zsosto_temp[] = gt_zsosto[].
              SORT gt_zsosto_temp BY ebeln.
              DELETE ADJACENT DUPLICATES FROM gt_zsosto_temp COMPARING ebeln.
              IF gt_zsosto_temp IS NOT INITIAL.

                CLEAR gt_lips_1[].
                SELECT vbeln posnr vgbel
                FROM lips CLIENT SPECIFIED
                INTO TABLE gt_lips_1
                FOR ALL ENTRIES IN gt_zsosto_temp
                WHERE mandt = sy-mandt
                AND   vgbel = gt_zsosto_temp-ebeln
                %_HINTS ORACLE 'INDEX("LIPS""LIPS~M05")'.
                IF sy-subrc = 0.

                  CLEAR gt_lips_1_temp[].
                  gt_lips_1_temp = gt_lips_1.
                  SORT gt_lips_1_temp BY vbeln.
                  DELETE ADJACENT DUPLICATES FROM gt_lips_1_temp COMPARING vbeln.
                  IF gt_lips_1_temp IS NOT INITIAL.

                    CLEAR gt_/obiz/zxp_item[].
                    SELECT doc_clas doc_nr posnr shp_bilno shp_bildt deliveryno
                    FROM /obiz/zxp_item CLIENT SPECIFIED
                    INTO TABLE gt_/obiz/zxp_item
                    FOR ALL ENTRIES IN  gt_lips_1_temp
                    WHERE mandt      = sy-mandt
                    AND   deliveryno =  gt_lips_1_temp-vbeln
                    %_HINTS ORACLE 'INDEX("/OBIZ/ZXP_ITEM""/OBIZ/ZXP_ITEM~DEL")'.
                    IF sy-subrc = 0.
                      SORT gt_/obiz/zxp_item BY deliveryno.
                      DELETE gt_/obiz/zxp_item WHERE doc_clas NE '0005'.
                    ENDIF.
                  ENDIF.
                ENDIF.
              ENDIF.
            ENDIF.
          ENDIF.
        ENDIF.
      ENDIF.

      SORT gt_zsosto BY vbeln.
      SORT gt_lips_1 BY vgbel.
      SORT gt_lips   BY vbeln.
      LOOP AT gt_trstlmnt_ma ASSIGNING <gfs_ztrstlmnt_m_a>.

        " IMPORT CASE SCENARIO
        READ TABLE gt_zlog_exec_var TRANSPORTING NO FIELDS WITH KEY name = lc_zsce_ztrslmnt_shproc
                                                                    remarks = <gfs_ztrstlmnt_m_a>-pro_status. " no binary search needed
        IF sy-subrc = 0.
          READ TABLE gt_zlog_exec_var TRANSPORTING NO FIELDS WITH KEY name  = lc_zsce_ztrslmnt_imp
                                                                     shtyp = <gfs_ztrstlmnt_m_a>-shtyp. " no binary search needed
          IF sy-subrc = 0.
            CLEAR gw_/obiz/zbn_dochdr.
            READ TABLE gt_/obiz/zbn_dochdr INTO gw_/obiz/zbn_dochdr WITH KEY refdocnr = <gfs_ztrstlmnt_m_a>-vbeln BINARY SEARCH.
            IF sy-subrc = 0.
              <gfs_ztrstlmnt_m_a>-bl_no   = gw_/obiz/zbn_dochdr-bolnr.
              <gfs_ztrstlmnt_m_a>-bl_dt   = gw_/obiz/zbn_dochdr-boldt.
              <gfs_ztrstlmnt_m_a>-hawb_no = gw_/obiz/zbn_dochdr-hawbno.
              <gfs_ztrstlmnt_m_a>-hawb_dt = gw_/obiz/zbn_dochdr-hawbdt.
              <gfs_ztrstlmnt_m_a>-mawbno  = gw_/obiz/zbn_dochdr-mawbno.
              <gfs_ztrstlmnt_m_a>-mawbdt  = gw_/obiz/zbn_dochdr-mawbdt.
            ENDIF.
          ENDIF.

          " EXPORT CASE SCENARIO
          READ TABLE gt_zlog_exec_var TRANSPORTING NO FIELDS WITH KEY name  = lc_zsce_ztrstlmnt_exp
                                                                      shtyp = <gfs_ztrstlmnt_m_a>-shtyp. " no binary search needed
          IF sy-subrc = 0.
            CLEAR gw_lips.
            READ TABLE gt_lips INTO gw_lips WITH KEY vbeln = <gfs_ztrstlmnt_m_a>-vbeln BINARY SEARCH.
            IF sy-subrc = 0.
              CLEAR gw_zsosto.
              READ TABLE gt_zsosto INTO gw_zsosto WITH KEY vbeln = gw_lips-vgbel BINARY SEARCH.
              IF sy-subrc = 0.
                CLEAR gw_lips.
                READ TABLE gt_lips_1 INTO gw_lips WITH KEY vgbel = gw_zsosto-ebeln BINARY SEARCH.
                IF sy-subrc = 0.
                  CLEAR gw_/obiz/zxp_item.
                  READ TABLE gt_/obiz/zxp_item INTO gw_/obiz/zxp_item WITH KEY deliveryno = gw_lips-vbeln BINARY SEARCH.
                  IF sy-subrc = 0.
                    <gfs_ztrstlmnt_m_a>-shipbillno = gw_/obiz/zxp_item-shp_bilno.
                    <gfs_ztrstlmnt_m_a>-shipbilldt = gw_/obiz/zxp_item-shp_bildt.
                  ENDIF.
                ENDIF.
              ENDIF.
            ENDIF.
          ENDIF.
        ENDIF.
      ENDLOOP.
    ENDIF.
    " eoc by omkar more on 12.03.2025 TR:RD2K9A54FE

  ENDMETHOD.                    "gt_marine_air_data

  METHOD fill_field_catalog.

    REFRESH gt_fcat.
*    ****************************************************************    BOC BY TRILOK 18.11.2022
    IF p_vc IS NOT INITIAL .
      PERFORM fill_fcat USING   text-041  'X' text-042.
      PERFORM fill_fcat USING   'TRUCK_NO' ' ' 'Vehicle Number'.
      PERFORM fill_fcat USING   'VEHICLE'  ' ' 'TD vehicle Number' .
    ELSE.
*********************************************************************    EOC BY TRILOK 18.11.2022
      IF p_rd EQ 'X' OR p_ma EQ 'X' .
        PERFORM fill_fcat USING   text-041  'X' text-042. " check box (+) by pwc_159 on 27.12.17
      ENDIF.
      PERFORM fill_fcat USING   text-003  'X' text-017.  " tknum
      PERFORM fill_fcat USING   text-004  'X' text-018.  " vbeln
      IF p_rd EQ 'X'.
        PERFORM fill_fcat USING   'BUKRS'   ' ' 'Company Code'(064).
      ENDIF.
      IF p_rd EQ 'X' OR p_rl EQ 'X'. " road (or) rail
        PERFORM fill_fcat USING   text-005  ' ' text-019.  " lr_no
        PERFORM fill_fcat USING   text-006  ' ' text-020.  " lr_dt
        PERFORM fill_fcat USING   text-007  ' ' text-021.  " grn_no
        PERFORM fill_fcat USING   text-008  ' ' text-023.  " grn_qty
        PERFORM fill_fcat USING   text-009  ' ' text-024.  " grn_uom
      ENDIF.

      IF p_rd EQ 'X'.  " Road
        PERFORM fill_fcat USING   text-035  ' ' text-025.  " routID
        PERFORM fill_fcat USING   text-011  ' ' text-026.  " flt_ind

      ELSEIF p_rl EQ 'X'. " Rail
        PERFORM fill_fcat USING   text-030  ' ' text-032.  " rake_ldqty
        PERFORM fill_fcat USING   text-031  ' ' text-033.  " scenr
        PERFORM fill_fcat USING   'PRO_STATUS' ' ' 'Processing Status'.     "trilok 19.01.2023
        PERFORM fill_fcat USING   'FKNUM' ' ' 'Shipment Cost Number'.     "trilok 31.01.2023
      ENDIF.

      PERFORM fill_fcat USING   text-012  ' ' text-029.  " digind
      PERFORM fill_fcat USING   text-013  ' ' text-027.  " bill_vendor
      PERFORM fill_fcat USING   text-014  ' ' text-028.  " saccode
* BOC by MSAT_050 on 03.01.2018 11:13:55
      IF p_sp IS NOT INITIAL.
        REFRESH gt_fcat.
        PERFORM fill_fcat USING   'TKNUM'   ' ' 'Shipment Number'(017).
        PERFORM fill_fcat USING   'VBELN'   ' ' 'Delivery'(015).
        PERFORM fill_fcat USING   'DATBG'   ' ' 'Act. Shipment Start Date'(016).
        PERFORM fill_fcat USING   'UATBG'   ' ' 'Act. Shipment Start Time'(043).
        PERFORM  fill_fcat USING  'REMARKS' ' ' 'Remarks'(044).
      ENDIF.
      IF p_rd = 'X'.
        PERFORM fill_fcat USING   'TR_BILL_DT'   ' ' 'TR BILL Date'.
        PERFORM fill_fcat USING   'TR_BILL_AMT'   ' ' 'TR BILL Amount'. "added on 01-03-2019
      ENDIF.
* EOC by MSAT_050 on 03.01.2018 11:13:55
*   BOC Santhosh Chowdary Liability amount to be display
      IF p_rd = 'X'.
        PERFORM fill_fcat USING   'TRANS_LIBTY_AMT'   ' ' 'Liability Amount'(065).
        PERFORM fill_fcat USING   'TRANS_LIBTY_CUR'   ' ' 'Liability Currency'(066).
        PERFORM fill_fcat USING   'LR_QTY'   ' ' 'LR Qty'.
        PERFORM fill_fcat USING   'LR_UOM'   ' ' 'LR UOM'.
        PERFORM fill_fcat USING   'DEP_POINT'   ' ' 'Departure Point'.
        PERFORM fill_fcat USING   'DES_POINT'   ' ' 'Destination Point'.
        PERFORM fill_fcat USING   'PRO_STATUS'   ' ' 'Processing Status'.
        PERFORM fill_fcat USING   'MFRGR'   ' ' 'Material Freight Group'.
        PERFORM fill_fcat USING   'FRT_VAL'   ' ' 'Freight value'.
        PERFORM fill_fcat USING   'HEADRUN_AMT' ' ' 'Headrun Amount' .  "+TRILOK 10.11.2022   RD2K9A417Y
        PERFORM fill_fcat USING   'BILL_REGIME' ' ' 'Bill Regime'.

      ENDIF.
      IF p_ma IS NOT INITIAL.
        PERFORM fill_fcat USING   'PRO_STATUS'   ' ' 'Processing Status'.
        PERFORM fill_fcat USING   'SHTYP'   ' ' 'Shipment type'.
        PERFORM fill_fcat USING   'BUKRS'   ' ' 'Company Code'.
        PERFORM fill_fcat USING   'SHIPBILLNO'   ' ' 'Shipping Bill Number'.
        PERFORM fill_fcat USING   'SHIPBILLDT'   ' ' 'Shipping Bill Date'.

        " soc by omkar more on 12.03.2025 TR:RD2K9A54FE
        PERFORM fill_fcat USING   'BL_NO'   ' ' 'Bill Of Lading No'.
        PERFORM fill_fcat USING   'BL_DT'   ' ' 'Bill of Ladd.Dt'.
        PERFORM fill_fcat USING   'HAWB_NO' ' ' 'House Air Way Bill Number'.
        PERFORM fill_fcat USING   'HAWB_DT' ' ' 'House Air Way Bill Date'.
        PERFORM fill_fcat USING   'MAWBNO'  ' ' 'Master Air Way Bill number'.
        PERFORM fill_fcat USING   'MAWBDT'  ' ' 'Master Air Way Bill Date'.
        PERFORM fill_fcat USING   'ZCHGBL_WGT'  ' ' 'Chargeable Weight'.
        PERFORM fill_fcat USING   'CHGBL_WT_UOM'  ' ' 'Bun'.
        " eoc by omkar more on 12.03.2025 TR:RD2K9A54FE

      ENDIF.
    ENDIF.
*   EOC Santhosh Chowdary Liability amount to be display

  ENDMETHOD.                    "fill_field_catalog

  METHOD icon_exclude.

    REFRESH gt_exclude.

    APPEND cl_gui_alv_grid=>mc_fc_loc_cut        TO  gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_loc_insert_row TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_loc_append_row TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_loc_copy       TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_loc_paste      TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_loc_copy_row   TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_loc_delete_row TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_loc_paste_new_row TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_loc_paste_new_row TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_mb_paste          TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_loc_undo       TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_view_excel     TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_graph          TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_help           TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_info           TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_check          TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_to_office      TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_call_xml_export TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_load_variant  TO gt_exclude .
    APPEND cl_gui_alv_grid=>mc_fc_refresh       TO gt_exclude .

  ENDMETHOD.                    "icon_exclude


  METHOD handle_data_changed.

    APPEND LINES OF er_data_changed->mt_good_cells TO gt_modify.

    SORT gt_modify BY row_id.
    DELETE ADJACENT DUPLICATES FROM gt_modify COMPARING row_id.

  ENDMETHOD.                    "handle_data_changed
* BOC by MSAT_050 on 03.01.2018 12:08:13
  METHOD handle_uc.
    DATA: lt_rows TYPE lvc_t_row,
          lw_rows TYPE lvc_s_row,
          lw_headerdata TYPE bapishipmentheader,
          lw_headerdataaction  TYPE bapishipmentheaderaction,
          lw_headerdeadline TYPE bapishipmentheaderdeadline,
          lw_headerdeadlineaction  TYPE bapishipmentheaderdeadlineact,
          lt_return TYPE bapiret2_t,
          lw_return TYPE bapiret2,
          lv_timestamp TYPE tzonref-tstamps,
          lw_status TYPE numc2.
    FIELD-SYMBOLS: <fs_shipment> TYPE gty_shipment.

    DATA: lt_cdtxt              TYPE TABLE OF cdtxt,  " added by Arpit
      lt_yztrstlmnt_rd_old  TYPE TABLE OF yztrstlmnt,
      lt_yztrstlmnt_rd_new  TYPE TABLE OF yztrstlmnt,
      lw_yztrstlmnt_rd_old  TYPE yztrstlmnt,
      lw_yztrstlmnt_rd_new  TYPE yztrstlmnt,
      lw_route              TYPE ztrstlmnt-route,
      lt_ztrstlmnt_ma_old_m_a TYPE TABLE OF yztrstlmnt_m_a,
      lt_ztrstlmnt_ma_new_m_a TYPE TABLE OF yztrstlmnt_m_a,
      lw_ztrstlmnt_ma_old_m_a TYPE ztrstlmnt_m_a,
      lw_ztrstlmnt_ma_new_m_a TYPE ztrstlmnt_m_a.
    CASE e_ucomm.
      WHEN 'PROCESS'.
        CALL METHOD go_grid->get_selected_rows
          IMPORTING
            et_index_rows = lt_rows.
        IF lt_rows IS INITIAL.
          MESSAGE 'Select Record(s) To Be Posted'(047) TYPE 'S'.
        ELSE.
          CLEAR: gw_shipment , lw_rows.
          LOOP AT lt_rows INTO lw_rows.
            CLEAR gw_shipment.
            READ TABLE gt_shipment ASSIGNING <fs_shipment> INDEX lw_rows-index.
            IF sy-subrc = 0.
              IF <fs_shipment>-stten IS INITIAL .
                IF <fs_shipment>-sttbg  IS NOT INITIAL.
                  CLEAR:lw_headerdata ,lw_headerdataaction,lt_return.
                  " Remove the Shipment Start flag again
                  lw_headerdata-shipment_num = <fs_shipment>-tknum.
                  lw_headerdata-status_shpmnt_start = 'D'.
                  lw_headerdataaction-shipment_num = abap_true.
                  lw_headerdataaction-status_shpmnt_start = 'D'.
                  go_ref->change_shipment( EXPORTING headerdata = lw_headerdata
                                                     headerdataaction = lw_headerdataaction
                                           IMPORTING et_return = lt_return ).
                  CLEAR lw_return.
                  READ TABLE lt_return  INTO lw_return WITH KEY type = 'E'.
                  IF sy-subrc = 0.
                    <fs_shipment>-remarks = 'Error Ocurred While Deleting the Shipment Start Date'(048).
                    CALL FUNCTION 'BAPI_TRANSACTION_ROLLBACK'.
                  ELSE.
                    CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
                      EXPORTING
                        wait = abap_true.
                    CLEAR:lw_headerdata ,lw_headerdataaction,lw_headerdeadline,
                          lw_headerdeadlineaction, lt_return, lv_timestamp.
                    " Set the Shipment Start flag again
                    lw_headerdata-shipment_num = <fs_shipment>-tknum.
                    lw_headerdata-status_shpmnt_start = abap_true.
                    lw_headerdataaction-shipment_num = abap_true.
                    lw_headerdataaction-status_shpmnt_start = 'C'.
                    CALL FUNCTION 'CONVERT_INTO_TIMESTAMP'
                      EXPORTING
                        i_datlo     = <fs_shipment>-datbg
                        i_timlo     = <fs_shipment>-uatbg
*                       I_TZONE     = SY-ZONLO
                      IMPORTING
                        e_timestamp = lv_timestamp.
                    lw_headerdeadline-time_stamp_utc = lv_timestamp.
                    lw_headerdeadline-time_type = 'HDRSTSSADT'.
                    lw_headerdeadline-time_zone = 'INDIA'.
                    lw_headerdeadlineaction-time_stamp_utc = 'C'.
                    lw_headerdeadlineaction-time_type = 'C'.
                    lw_headerdeadlineaction-time_zone = 'C'.
                    go_ref->change_shipment( EXPORTING headerdata = lw_headerdata
                                                       headerdataaction = lw_headerdataaction
                                                       headerdeadline = lw_headerdeadline
                                                  headerdeadlineaction = lw_headerdeadlineaction
                                             IMPORTING et_return = lt_return ).
                    CLEAR lw_return.
                    READ TABLE lt_return  INTO lw_return WITH KEY type = 'E'.
                    IF sy-subrc = 0.
                      <fs_shipment>-remarks = 'Error Ocurred While Updating the the Shipment Start Date'(049).
                      CALL FUNCTION 'BAPI_TRANSACTION_ROLLBACK'.
                    ELSE.
                      <fs_shipment>-remarks = 'Shipment Start Date Updated Successfully'(050).
                      CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
                        EXPORTING
                          wait = abap_true.
                    ENDIF.
                  ENDIF.
                ELSE.
                  <fs_shipment>-remarks = 'Transport status is "Not Completed"'(052).
                ENDIF.
              ELSE.
                <fs_shipment>-remarks = 'Shipment is already Updated With End Date'(051).
              ENDIF.

            ENDIF.
          ENDLOOP.
        ENDIF.

      WHEN 'DELETE'.
        IF p_rd IS NOT INITIAL.

          READ TABLE gt_trstlmnt_rd_chk TRANSPORTING NO FIELDS WITH KEY
                                               check_box = 'X'.
          IF sy-subrc NE 0.
            MESSAGE 'No record selected'(010) TYPE 'S'.
            RETURN.
          ENDIF.
          SORT gt_trstlmnt_rd_old BY tknum vbeln counter vsart.
          LOOP AT gt_trstlmnt_rd_chk INTO gw_trstlmnt_rd_chk
                                            WHERE check_box = 'X'.

            MOVE-CORRESPONDING gw_trstlmnt_rd_chk TO gw_trstlmnt_rd.
            CLEAR : gw_objectid, gw_vendor.
**  "  by pwc_159 on 27.12.2017 -  ).

            gw_vendor = gw_trstlmnt_rd-bill_vendor.

            CALL FUNCTION 'CONVERSION_EXIT_ALPHA_INPUT'
              EXPORTING
                input  = gw_vendor
              IMPORTING
                output = gw_vendor.

            CLEAR: lw_yztrstlmnt_rd_old, lw_yztrstlmnt_rd_new, gw_trstlmnt_rd_old.
            REFRESH: lt_yztrstlmnt_rd_old, lt_yztrstlmnt_rd_new.

            READ TABLE gt_trstlmnt_rd_old INTO gw_trstlmnt_rd_old
                                        WITH KEY tknum = gw_trstlmnt_rd-tknum
                                                 vbeln = gw_trstlmnt_rd-vbeln
                                                 counter = gw_trstlmnt_rd-counter
                                                 vsart   = gw_trstlmnt_rd-vsart
                                                 BINARY SEARCH.
            IF sy-subrc EQ 0.

              MOVE-CORRESPONDING gw_trstlmnt_rd_old  TO lw_yztrstlmnt_rd_old.
              MOVE-CORRESPONDING gw_trstlmnt_rd      TO lw_yztrstlmnt_rd_new.

              APPEND lw_yztrstlmnt_rd_old TO lt_yztrstlmnt_rd_old.
              APPEND lw_yztrstlmnt_rd_new TO lt_yztrstlmnt_rd_new.

            ENDIF.

            CLEAR lw_route.
            CALL FUNCTION 'ENQUEUE_EZ_ZTRSTLMNT'
              EXPORTING
                mode_ztrstlmnt = 'E'
                mandt          = sy-mandt
                tknum          = gw_trstlmnt_rd-tknum
                vbeln          = gw_trstlmnt_rd-vbeln
                counter        = gw_trstlmnt_rd-counter
                vsart          = gw_trstlmnt_rd-vsart
              EXCEPTIONS
                foreign_lock   = 1
                system_failure = 2
                OTHERS         = 3.
            IF sy-subrc = 0.

              lw_route = gw_trstlmnt_rd-routid. "(+) pwc_159 on 26.12.2017
*               SHIFT gw_trstlmnt_rd-pro_status LEFT DELETING LEADING '0'.
              lw_status = gw_trstlmnt_rd-pro_status.
              IF lw_status LE '5'.
                DELETE FROM ztrstlmnt
                             WHERE    tknum     = gw_trstlmnt_rd-tknum
                              AND    vbeln     = gw_trstlmnt_rd-vbeln
                              AND    counter   = gw_trstlmnt_rd-counter
                              AND    vsart     = gw_trstlmnt_rd-vsart.
              ELSE.
                MESSAGE 'Selected line status is not below 5'(103) TYPE 'S' DISPLAY LIKE 'E'.
                LEAVE LIST-PROCESSING.
              ENDIF.
              IF sy-subrc EQ 0.

                CONCATENATE gw_trstlmnt_rd-tknum
                            gw_trstlmnt_rd-vbeln
                            gw_trstlmnt_rd-counter
                            gw_trstlmnt_rd-vsart
                            INTO gw_objectid.

** " Below Update Function module useful to create Change document number.
**        useful for tracking Change logs in Corresponding Table.

                CALL FUNCTION 'ZTRSTLMNT_WRITE_DOCUMENT'
                  EXPORTING
                    objectid                = gw_objectid
                    tcode                   = sy-tcode
                    utime                   = sy-uzeit
                    udate                   = sy-datum
                    username                = sy-uname
*                   PLANNED_CHANGE_NUMBER   = ' '
                    object_change_indicator = 'U'
                    planned_or_real_changes = 'R'
                    no_change_pointers      = ' '
*                   UPD_ICDTXT_ZTRSTLMNT    = ' '
                    upd_ztrstlmnt           = 'U'
                  TABLES
                    icdtxt_ztrstlmnt        = lt_cdtxt
                    xztrstlmnt              = lt_yztrstlmnt_rd_new
                    yztrstlmnt              = lt_yztrstlmnt_rd_old.

                COMMIT WORK AND WAIT.
                gw_flag_s = 'X'.
              ELSE.
                ROLLBACK WORK.
              ENDIF.

              CALL FUNCTION 'DEQUEUE_EZ_ZTRSTLMNT'
                EXPORTING
                  mode_ztrstlmnt = 'E'
                  mandt          = sy-mandt
                  tknum          = gw_trstlmnt_rd-tknum
                  vbeln          = gw_trstlmnt_rd-vbeln
                  counter        = gw_trstlmnt_rd-counter
                  vsart          = gw_trstlmnt_rd-vsart.

            ENDIF.
            CLEAR: gw_modify, gw_trstlmnt_rd, gw_trstlmnt_rd_chk.
*    ENDIF.
          ENDLOOP.

        ENDIF.
        IF p_ma IS NOT INITIAL.
          IF gt_modify IS INITIAL AND gt_trstlmnt_ma IS NOT INITIAL.
            MESSAGE s592(zlog).
            RETURN.
          ENDIF.

          SORT gt_trstlmnt_ma_old BY tknum vbeln counter vsart.
          LOOP AT gt_modify INTO gw_modify.

            CLEAR : gw_objectid, gw_vendor.
            READ TABLE gt_trstlmnt_ma INTO gw_trstlmnt_ma INDEX gw_modify-row_id.
            IF sy-subrc EQ 0.
              gw_vendor = gw_trstlmnt_ma-bill_vendor.

              CALL FUNCTION 'CONVERSION_EXIT_ALPHA_INPUT'
                EXPORTING
                  input  = gw_vendor
                IMPORTING
                  output = gw_vendor.

              CLEAR: lw_ztrstlmnt_ma_old_m_a, lw_ztrstlmnt_ma_new_m_a, gw_trstlmnt_ma_old.
              REFRESH: lt_ztrstlmnt_ma_old_m_a, lt_ztrstlmnt_ma_new_m_a.

              READ TABLE gt_trstlmnt_ma_old INTO gw_trstlmnt_ma_old
                                          WITH KEY tknum = gw_trstlmnt_ma-tknum
                                                   vbeln = gw_trstlmnt_ma-vbeln
                                                   counter = gw_trstlmnt_ma-counter
                                                   vsart   = gw_trstlmnt_ma-vsart
                                                   BINARY SEARCH.
              IF sy-subrc EQ 0.

                MOVE-CORRESPONDING gw_trstlmnt_ma_old TO lw_ztrstlmnt_ma_old_m_a.
                MOVE-CORRESPONDING gw_trstlmnt_ma     TO lw_ztrstlmnt_ma_new_m_a.

                APPEND lw_ztrstlmnt_ma_old_m_a TO lt_ztrstlmnt_ma_old_m_a.
                APPEND lw_ztrstlmnt_ma_new_m_a TO lt_ztrstlmnt_ma_new_m_a.

              ENDIF.

              CALL FUNCTION 'ENQUEUE_EZ_TRSTLMNT_MA'
                EXPORTING
                  mode_ztrstlmnt_m_a = 'E'
                  mandt              = sy-mandt
                  tknum              = gw_trstlmnt_ma-tknum
                  vbeln              = gw_trstlmnt_ma-vbeln
                  counter            = gw_trstlmnt_ma-counter
                  vsart              = gw_trstlmnt_ma-vsart
                EXCEPTIONS
                  foreign_lock       = 1
                  system_failure     = 2
                  OTHERS             = 3.
              IF sy-subrc = 0.
                " Update into the table.
*                SHIFT gw_trstlmnt_ma-pro_status LEFT DELETING LEADING '0'.
                lw_status = gw_trstlmnt_ma-pro_status.
                IF lw_status LE '5'.
                  DELETE FROM  ztrstlmnt_m_a
                                  WHERE tknum      = gw_trstlmnt_ma-tknum
                                  AND   vbeln      = gw_trstlmnt_ma-vbeln
                                  AND   counter    = gw_trstlmnt_ma-counter
                                  AND   vsart      = gw_trstlmnt_ma-vsart.
                ELSE.
                  MESSAGE 'Selected line status is not below 5'(103) TYPE 'S' DISPLAY LIKE 'E'.
                  LEAVE LIST-PROCESSING.
                ENDIF.
                IF sy-subrc = 0.

                  CONCATENATE gw_trstlmnt_ma-tknum
                              gw_trstlmnt_ma-vbeln
                              gw_trstlmnt_ma-counter
                              gw_trstlmnt_ma-vsart
                              INTO gw_objectid.

** " Below Update Function module useful to create Change document number.
**        useful for tracking Change logs in Corresponding Table.

                  CALL FUNCTION 'ZTRSTLMNT_M_A_WRITE_DOCUMENT'
                    EXPORTING
                      objectid                 = gw_objectid " 'ZTRSTLMNT_M_A'
                      tcode                    = sy-tcode
                      utime                    = sy-uzeit
                      udate                    = sy-datum
                      username                 = sy-uname
*                     PLANNED_CHANGE_NUMBER    = ' '
                      object_change_indicator  = 'U'
                      planned_or_real_changes  = 'R'
                      no_change_pointers       = ' '
*                     UPD_ICDTXT_ZTRSTLMNT_M_A = ' '
                      upd_ztrstlmnt_m_a        = 'U'
                    TABLES
                      icdtxt_ztrstlmnt_m_a     = lt_cdtxt
                      xztrstlmnt_m_a           = lt_ztrstlmnt_ma_new_m_a
                      yztrstlmnt_m_a           = lt_ztrstlmnt_ma_old_m_a.

                  COMMIT WORK AND WAIT.
                  gw_flag_s = 'X'.
                ELSE.
                  ROLLBACK WORK.
                ENDIF.

                CALL FUNCTION 'DEQUEUE_EZ_TRSTLMNT_MA'
                  EXPORTING
                    mode_ztrstlmnt_m_a = 'E'
                    mandt              = sy-mandt
                    tknum              = gw_trstlmnt_ma-tknum
                    vbeln              = gw_trstlmnt_ma-vbeln
                    counter            = gw_trstlmnt_ma-counter
                    vsart              = gw_trstlmnt_ma-vsart.

              ENDIF.

              CLEAR : gw_modify, gw_trstlmnt_ma.
            ENDIF.
          ENDLOOP.
        ENDIF.
    ENDCASE.

    IF go_grid IS BOUND.
      CALL METHOD go_grid->refresh_table_display.
    ENDIF.
  ENDMETHOD.                    "handle_uc

  METHOD handle_tb.
    DATA : lw_toolbar  TYPE stb_button.
    IF p_sp IS NOT INITIAL.
      lw_toolbar-function = 'PROCESS'.
      lw_toolbar-icon = icon_generate.
      lw_toolbar-text     = 'Process'(046).
      lw_toolbar-quickinfo = 'Process Records'(045).
      APPEND lw_toolbar TO e_object->mt_toolbar.
      MOVE abap_true TO e_interactive.
    ENDIF.
    IF p_rd IS NOT INITIAL OR p_ma IS NOT INITIAL .
      lw_toolbar-function = 'DELETE'.
      lw_toolbar-icon = icon_generate.
      lw_toolbar-text     = 'DELETE'.
      lw_toolbar-quickinfo = 'Delete Records'.
      APPEND lw_toolbar TO e_object->mt_toolbar.
      MOVE abap_true TO e_interactive.
    ENDIF.
  ENDMETHOD.                    "handle_tb
  METHOD handle_tb_rd.
    DATA : lw_toolbar  TYPE stb_button.

  ENDMETHOD.                    "handle_tb_RD
* EOC by MSAT_050 on 03.01.2018 12:08:13
  METHOD disp_alv.

    CLEAR gw_layo.

    IF go_cont IS INITIAL.
      CREATE OBJECT go_cont
        EXPORTING
          container_name              = i_cont
        EXCEPTIONS
          cntl_error                  = 1
          cntl_system_error           = 2
          create_error                = 3
          lifetime_error              = 4
          lifetime_dynpro_dynpro_link = 5
          OTHERS                      = 6.
      IF sy-subrc <> 0.
        MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
                   WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
      ENDIF.

      CREATE OBJECT go_grid
        EXPORTING
          i_parent          = go_cont
        EXCEPTIONS
          error_cntl_create = 1
          error_cntl_init   = 2
          error_cntl_link   = 3
          error_dp_create   = 4
          OTHERS            = 5.
      IF sy-subrc <> 0.
        MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
                   WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
      ENDIF.

      SET HANDLER go_ref->handle_data_changed FOR go_grid.
* BOC by MSAT_050 on 03.01.2018 12:11:51
      IF p_rd IS NOT INITIAL OR p_ma IS NOT INITIAL .
        SET HANDLER go_ref->handle_uc FOR go_grid.
        SET HANDLER go_ref->handle_tb FOR go_grid.
        gw_layo-sel_mode = 'D'.
      ENDIF.
      IF p_sp IS NOT INITIAL.
        SET HANDLER go_ref->handle_uc FOR go_grid.
        SET HANDLER go_ref->handle_tb FOR go_grid.
        gw_layo-sel_mode = 'D'.
      ENDIF.
* EOC by MSAT_050 on 03.01.2018 12:11:51
      gw_layo-cwidth_opt = 'X'.
      IF p_sp IS INITIAL  .
        gw_layo-no_rowmark   = 'X'. " To hide row's selection
      ENDIF.
      gw_layo-zebra   = 'X'.

      CALL METHOD go_grid->set_table_for_first_display
        EXPORTING
          is_layout                     = gw_layo
          it_toolbar_excluding          = gt_exclude
        CHANGING
          it_outtab                     = it_table
          it_fieldcatalog               = gt_fcat
        EXCEPTIONS
          invalid_parameter_combination = 1
          program_error                 = 2
          too_many_lines                = 3
          OTHERS                        = 4.
      IF sy-subrc <> 0.
        MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
                        WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
      ENDIF.

      CALL METHOD go_grid->register_edit_event
        EXPORTING
          i_event_id = cl_gui_alv_grid=>mc_evt_modified.

    ELSE.
      CALL METHOD go_grid->refresh_table_display.
    ENDIF.
  ENDMETHOD.                    "disp_alv

  METHOD alv_data_save.

    CLEAR gw_flag_s.

    IF p_rd EQ 'X'.
      go_ref->road_data_save( ).

    ELSEIF p_rl EQ 'X'.
      go_ref->rail_data_save( ).

    ELSEIF p_ma EQ 'X'.
      go_ref->marine_air_data_save( ).
    ELSEIF p_st EQ 'X'.
      go_ref->shrtage_data_save( ).
    ELSEIF p_vc EQ 'X'.
      go_ref->vehicle_save( ).
    ENDIF.

    IF gw_flag_s = 'X'.
      MESSAGE 'Data saved'(036) TYPE 'S'.
    ENDIF.

  ENDMETHOD.                    "alv_data_save

  METHOD road_data_save.

    DATA: lt_cdtxt              TYPE TABLE OF cdtxt,
          lt_yztrstlmnt_rd_old  TYPE TABLE OF yztrstlmnt,
          lt_yztrstlmnt_rd_new  TYPE TABLE OF yztrstlmnt,
          lw_yztrstlmnt_rd_old  TYPE yztrstlmnt,
          lw_yztrstlmnt_rd_new  TYPE yztrstlmnt,
          lw_route              TYPE ztrstlmnt-route.

**  " ( by pwc_159 on 26.12.2017 -
*    IF gt_modify IS INITIAL AND gt_trstlmnt_rd IS NOT INITIAL.
*      MESSAGE s592(zlog).
*      RETURN.
*    ENDIF.

*    LOOP AT gt_modify INTO gw_modify.


*      READ TABLE gt_trstlmnt_rd INTO gw_trstlmnt_rd INDEX gw_modify-row_id.
*          IF sy-subrc EQ 0.
**  "  by pwc_159 on 26.12.2017 -  ).

**  " ( by pwc_159 on 27.12.2017  - Route update - issue
    READ TABLE gt_trstlmnt_rd_chk TRANSPORTING NO FIELDS WITH KEY
                                         check_box = 'X'.
    IF sy-subrc NE 0.
      MESSAGE 'No record selected'(010) TYPE 'S'.
      RETURN.
    ENDIF.
    SORT gt_trstlmnt_rd_old BY tknum vbeln counter vsart.
    LOOP AT gt_trstlmnt_rd_chk INTO gw_trstlmnt_rd_chk
                                      WHERE check_box = 'X'.

      MOVE-CORRESPONDING gw_trstlmnt_rd_chk TO gw_trstlmnt_rd.
      CLEAR : gw_objectid, gw_vendor.
**  "  by pwc_159 on 27.12.2017 -  ).

      gw_vendor = gw_trstlmnt_rd-bill_vendor.

      CALL FUNCTION 'CONVERSION_EXIT_ALPHA_INPUT'
        EXPORTING
          input  = gw_vendor
        IMPORTING
          output = gw_vendor.

      CLEAR: lw_yztrstlmnt_rd_old, lw_yztrstlmnt_rd_new, gw_trstlmnt_rd_old.
      REFRESH: lt_yztrstlmnt_rd_old, lt_yztrstlmnt_rd_new.

      READ TABLE gt_trstlmnt_rd_old INTO gw_trstlmnt_rd_old
                                  WITH KEY tknum = gw_trstlmnt_rd-tknum
                                           vbeln = gw_trstlmnt_rd-vbeln
                                           counter = gw_trstlmnt_rd-counter
                                           vsart   = gw_trstlmnt_rd-vsart
                                           BINARY SEARCH.
      IF sy-subrc EQ 0.

        MOVE-CORRESPONDING gw_trstlmnt_rd_old  TO lw_yztrstlmnt_rd_old.
        MOVE-CORRESPONDING gw_trstlmnt_rd      TO lw_yztrstlmnt_rd_new.

        APPEND lw_yztrstlmnt_rd_old TO lt_yztrstlmnt_rd_old.
        APPEND lw_yztrstlmnt_rd_new TO lt_yztrstlmnt_rd_new.

      ENDIF.

      CLEAR lw_route.
      CALL FUNCTION 'ENQUEUE_EZ_ZTRSTLMNT'
        EXPORTING
          mode_ztrstlmnt = 'E'
          mandt          = sy-mandt
          tknum          = gw_trstlmnt_rd-tknum
          vbeln          = gw_trstlmnt_rd-vbeln
          counter        = gw_trstlmnt_rd-counter
          vsart          = gw_trstlmnt_rd-vsart
        EXCEPTIONS
          foreign_lock   = 1
          system_failure = 2
          OTHERS         = 3.
      IF sy-subrc = 0.

        lw_route = gw_trstlmnt_rd-routid. "(+) pwc_159 on 26.12.2017

        UPDATE ztrstlmnt SET
                         pro_status = gw_trstlmnt_rd-pro_status
                         mfrgr      = gw_trstlmnt_rd-mfrgr
                         lr_no      = gw_trstlmnt_rd-lr_no
                         lr_dt      = gw_trstlmnt_rd-lr_dt
                         lr_qty     = gw_trstlmnt_rd-lr_qty
                         lr_uom     = gw_trstlmnt_rd-lr_uom
                         trans_libty_amt = gw_trstlmnt_rd-trans_libty_amt
                         trans_libty_cur = gw_trstlmnt_rd-trans_libty_cur
                         grn_no     = gw_trstlmnt_rd-grn_no
                         grn_qty    = gw_trstlmnt_rd-grn_qty
                         grn_uom    = gw_trstlmnt_rd-grn_uom
**                           route      = gw_trstlmnt_rd-route
                         route      = lw_route            " (+) by pwc_159
                         flt_ind    = gw_trstlmnt_rd-flt_ind
                         digind     = gw_trstlmnt_rd-digind
                         bill_vendor = gw_vendor
                         saccode    = gw_trstlmnt_rd-saccode
                         tr_bill_dt  = gw_trstlmnt_rd-tr_bill_dt
                         tr_bill_amt  = gw_trstlmnt_rd-tr_bill_amt
                         bukrs = gw_trstlmnt_rd-bukrs
                         dep_point = gw_trstlmnt_rd-dep_point
                         des_point = gw_trstlmnt_rd-des_point
                         frt_val   = gw_trstlmnt_rd-frt_val  "added by arpit
                         headrun_amt = gw_trstlmnt_rd-headrun_amt  "+TRILOK 10.11.2022   RD2K9A417Y
                         bill_regime = gw_trstlmnt_rd-bill_regime
                    WHERE    tknum     = gw_trstlmnt_rd-tknum
                      AND    vbeln     = gw_trstlmnt_rd-vbeln
                      AND    counter   = gw_trstlmnt_rd-counter
                      AND    vsart     = gw_trstlmnt_rd-vsart.
        IF sy-subrc EQ 0.

          CONCATENATE gw_trstlmnt_rd-tknum
                      gw_trstlmnt_rd-vbeln
                      gw_trstlmnt_rd-counter
                      gw_trstlmnt_rd-vsart
                      INTO gw_objectid.

** " Below Update Function module useful to create Change document number.
**        useful for tracking Change logs in Corresponding Table.

          CALL FUNCTION 'ZTRSTLMNT_WRITE_DOCUMENT'
            EXPORTING
              objectid                = gw_objectid
              tcode                   = sy-tcode
              utime                   = sy-uzeit
              udate                   = sy-datum
              username                = sy-uname
*             PLANNED_CHANGE_NUMBER   = ' '
              object_change_indicator = 'U'
              planned_or_real_changes = 'R'
              no_change_pointers      = ' '
*             UPD_ICDTXT_ZTRSTLMNT    = ' '
              upd_ztrstlmnt           = 'U'
            TABLES
              icdtxt_ztrstlmnt        = lt_cdtxt
              xztrstlmnt              = lt_yztrstlmnt_rd_new
              yztrstlmnt              = lt_yztrstlmnt_rd_old.

          COMMIT WORK AND WAIT.
          gw_flag_s = 'X'.
        ELSE.
          ROLLBACK WORK.
        ENDIF.

        CALL FUNCTION 'DEQUEUE_EZ_ZTRSTLMNT'
          EXPORTING
            mode_ztrstlmnt = 'E'
            mandt          = sy-mandt
            tknum          = gw_trstlmnt_rd-tknum
            vbeln          = gw_trstlmnt_rd-vbeln
            counter        = gw_trstlmnt_rd-counter
            vsart          = gw_trstlmnt_rd-vsart.

      ENDIF.
      CLEAR: gw_modify, gw_trstlmnt_rd, gw_trstlmnt_rd_chk.
*    ENDIF.
    ENDLOOP.

  ENDMETHOD.                    "road_data_save

  METHOD rail_data_save.

    DATA:
          lt_cdtxt TYPE TABLE OF cdtxt,
          lt_yztrstlmnt_rl_old  TYPE TABLE OF yztrstlmnt_rail,
          lt_yztrstlmnt_rl_new  TYPE TABLE OF yztrstlmnt_rail,
          lw_yztrstlmnt_rl_old  TYPE yztrstlmnt_rail,
          lw_yztrstlmnt_rl_new  TYPE yztrstlmnt_rail.

    IF gt_modify IS INITIAL AND gt_trstlmnt_rl IS NOT INITIAL.
      MESSAGE s592(zlog).
      RETURN.
    ENDIF.

    SORT gt_trstlmnt_rl_old BY tknum vbeln counter vsart.

    LOOP AT gt_modify INTO gw_modify.

      CLEAR : gw_objectid, gw_vendor.
      READ TABLE gt_trstlmnt_rl INTO gw_trstlmnt_rl INDEX gw_modify-row_id.
      IF sy-subrc EQ 0.

        gw_vendor = gw_trstlmnt_rl-bill_vendor.

        CALL FUNCTION 'CONVERSION_EXIT_ALPHA_INPUT'
          EXPORTING
            input  = gw_vendor
          IMPORTING
            output = gw_vendor.

        CLEAR: lw_yztrstlmnt_rl_old, lw_yztrstlmnt_rl_new, gw_trstlmnt_rl_old.
        REFRESH: lt_yztrstlmnt_rl_old, lt_yztrstlmnt_rl_new.

        READ TABLE gt_trstlmnt_rl_old INTO gw_trstlmnt_rl_old
                                    WITH KEY tknum = gw_trstlmnt_rl-tknum
                                             vbeln = gw_trstlmnt_rl-vbeln
                                             counter = gw_trstlmnt_rl-counter
                                             vsart   = gw_trstlmnt_rl-vsart
                                             BINARY SEARCH.
        IF sy-subrc EQ 0.

          MOVE-CORRESPONDING gw_trstlmnt_rl_old  TO lw_yztrstlmnt_rl_old.
          MOVE-CORRESPONDING gw_trstlmnt_rl      TO lw_yztrstlmnt_rl_new.

          APPEND lw_yztrstlmnt_rl_old TO lt_yztrstlmnt_rl_old.
          APPEND lw_yztrstlmnt_rl_new TO lt_yztrstlmnt_rl_new.

        ENDIF.

        CALL FUNCTION 'ENQUEUE_EZ_TRSTLMNT_RAIL'
          EXPORTING
            mode_ztrstlmnt_rail = 'E'
            mandt               = sy-mandt
            tknum               = gw_trstlmnt_rl-tknum
            vbeln               = gw_trstlmnt_rl-vbeln
            counter             = gw_trstlmnt_rl-counter
            vsart               = gw_trstlmnt_rl-vsart
          EXCEPTIONS
            foreign_lock        = 1
            system_failure      = 2
            OTHERS              = 3.
        IF sy-subrc EQ 0.
          UPDATE ztrstlmnt_rail SET
                        pro_status = gw_trstlmnt_rl-pro_status    " trilok 19.01.2023
                        mfrgr      = gw_trstlmnt_rl-mfrgr
                        lr_no      = gw_trstlmnt_rl-lr_no
                        lr_dt      = gw_trstlmnt_rl-lr_dt
                        fknum      = gw_trstlmnt_rl-fknum  " trilok 31.01.2023
                        grn_no     = gw_trstlmnt_rl-grn_no
                        grn_dt     = gw_trstlmnt_rl-grn_dt
                        grn_qty    = gw_trstlmnt_rl-grn_qty
                        grn_uom    = gw_trstlmnt_rl-grn_uom
                        rake_ldqty  = gw_trstlmnt_rl-rake_ldqty
                        scenr      = gw_trstlmnt_rl-scenr
                        digind     = gw_trstlmnt_rl-digind
                        bill_vendor = gw_vendor
                        saccode    = gw_trstlmnt_rl-saccode
                    WHERE tknum = gw_trstlmnt_rl-tknum
                    AND   vbeln = gw_trstlmnt_rl-vbeln
                    AND   counter = gw_trstlmnt_rl-counter
                    AND   vsart   = gw_trstlmnt_rl-vsart.
          IF sy-subrc EQ 0.

            CONCATENATE gw_trstlmnt_rl-tknum
            gw_trstlmnt_rl-vbeln
            gw_trstlmnt_rl-counter
            gw_trstlmnt_rl-vsart
            INTO gw_objectid.

** " Below Update Function module useful to create Change document number.
**        useful for tracking Change logs in Corresponding Table.

            CALL FUNCTION 'ZTRSTLMNT_RAIL_WRITE_DOCUMENT'
              EXPORTING
                objectid                  = gw_objectid
                tcode                     = sy-tcode
                utime                     = sy-uzeit
                udate                     = sy-datum
                username                  = sy-uname
*               PLANNED_CHANGE_NUMBER     = ' '
                object_change_indicator   = 'U'
                planned_or_real_changes   = 'R'
                no_change_pointers        = ' '
*               UPD_ICDTXT_ZTRSTLMNT_RAIL = ' '
                upd_ztrstlmnt_rail        = 'U'
              TABLES
                icdtxt_ztrstlmnt_rail     = lt_cdtxt
                xztrstlmnt_rail           = lt_yztrstlmnt_rl_new
                yztrstlmnt_rail           = lt_yztrstlmnt_rl_old.

            COMMIT WORK AND WAIT.
          ELSE.
            ROLLBACK WORK.
          ENDIF.

        ENDIF.

        CLEAR: gw_modify, gw_trstlmnt_rl.

      ENDIF.
    ENDLOOP.

  ENDMETHOD.                    "rail_data_save

  METHOD marine_air_data_save.

    DATA:
          lt_cdtxt  TYPE TABLE OF cdtxt,
          lt_ztrstlmnt_ma_old TYPE TABLE OF yztrstlmnt_m_a,
          lt_ztrstlmnt_ma_new TYPE TABLE OF yztrstlmnt_m_a,
          lw_ztrstlmnt_ma_old TYPE ztrstlmnt_m_a,
          lw_ztrstlmnt_ma_new TYPE ztrstlmnt_m_a.

    IF gt_modify IS INITIAL AND gt_trstlmnt_ma IS NOT INITIAL.
      MESSAGE s592(zlog).
      RETURN.
    ENDIF.

    SORT gt_trstlmnt_ma_old BY tknum vbeln counter vsart.
    LOOP AT gt_modify INTO gw_modify.

      CLEAR : gw_objectid, gw_vendor.
      READ TABLE gt_trstlmnt_ma INTO gw_trstlmnt_ma INDEX gw_modify-row_id.
      IF sy-subrc EQ 0.
        gw_vendor = gw_trstlmnt_ma-bill_vendor.

        CALL FUNCTION 'CONVERSION_EXIT_ALPHA_INPUT'
          EXPORTING
            input  = gw_vendor
          IMPORTING
            output = gw_vendor.

        CLEAR: lw_ztrstlmnt_ma_old, lw_ztrstlmnt_ma_new, gw_trstlmnt_ma_old.
        REFRESH: lt_ztrstlmnt_ma_old, lt_ztrstlmnt_ma_new.

        READ TABLE gt_trstlmnt_ma_old INTO gw_trstlmnt_ma_old
                                    WITH KEY tknum = gw_trstlmnt_ma-tknum
                                             vbeln = gw_trstlmnt_ma-vbeln
                                             counter = gw_trstlmnt_ma-counter
                                             vsart   = gw_trstlmnt_ma-vsart
                                             BINARY SEARCH.
        IF sy-subrc EQ 0.

          MOVE-CORRESPONDING gw_trstlmnt_ma_old TO lw_ztrstlmnt_ma_old.
          MOVE-CORRESPONDING gw_trstlmnt_ma     TO lw_ztrstlmnt_ma_new.

          APPEND lw_ztrstlmnt_ma_old TO lt_ztrstlmnt_ma_old.
          APPEND lw_ztrstlmnt_ma_new TO lt_ztrstlmnt_ma_new.

        ENDIF.

        CALL FUNCTION 'ENQUEUE_EZ_TRSTLMNT_MA'
          EXPORTING
            mode_ztrstlmnt_m_a = 'E'
            mandt              = sy-mandt
            tknum              = gw_trstlmnt_ma-tknum
            vbeln              = gw_trstlmnt_ma-vbeln
            counter            = gw_trstlmnt_ma-counter
            vsart              = gw_trstlmnt_ma-vsart
          EXCEPTIONS
            foreign_lock       = 1
            system_failure     = 2
            OTHERS             = 3.
        IF sy-subrc = 0.
          " Update into the table.
          UPDATE ztrstlmnt_m_a
                          SET digind = gw_trstlmnt_ma-digind
                              bill_vendor = gw_vendor
                              saccode     = gw_trstlmnt_ma-saccode
                              pro_status  = gw_trstlmnt_ma-pro_status
                              shtyp       = gw_trstlmnt_ma-shtyp
                              bukrs       = gw_trstlmnt_ma-bukrs
                              bl_no        = gw_trstlmnt_ma-bl_no              " added by omkar more on 13.03.2025
                              bl_dt        = gw_trstlmnt_ma-bl_dt              " added by omkar more on 13.03.2025
                              hawb_no      = gw_trstlmnt_ma-hawb_no            " added by omkar more on 13.03.2025
                              hawb_dt      = gw_trstlmnt_ma-hawb_dt            " added by omkar more on 13.03.2025
                              zchgbl_wgt   = gw_trstlmnt_ma-zchgbl_wgt         " added by omkar more on 13.03.2025
                              chgbl_wt_uom = gw_trstlmnt_ma-chgbl_wt_uom       " added by omkar more on 13.03.2025
                              mawbno       = gw_trstlmnt_ma-mawbno             " added by omkar more on 13.03.2025
                              mawbdt       = gw_trstlmnt_ma-mawbdt
                              shipbillno  = gw_trstlmnt_ma-shipbillno
                              shipbilldt  = gw_trstlmnt_ma-shipbilldt
                          WHERE tknum      = gw_trstlmnt_ma-tknum
                          AND   vbeln      = gw_trstlmnt_ma-vbeln
                          AND   counter    = gw_trstlmnt_ma-counter
                          AND   vsart      = gw_trstlmnt_ma-vsart.
          IF sy-subrc = 0.

            CONCATENATE gw_trstlmnt_ma-tknum
                        gw_trstlmnt_ma-vbeln
                        gw_trstlmnt_ma-counter
                        gw_trstlmnt_ma-vsart
                        INTO gw_objectid.

** " Below Update Function module useful to create Change document number.
**        useful for tracking Change logs in Corresponding Table.

            CALL FUNCTION 'ZTRSTLMNT_M_A_WRITE_DOCUMENT'
              EXPORTING
                objectid                 = gw_objectid " 'ZTRSTLMNT_M_A'
                tcode                    = sy-tcode
                utime                    = sy-uzeit
                udate                    = sy-datum
                username                 = sy-uname
*               PLANNED_CHANGE_NUMBER    = ' '
                object_change_indicator  = 'U'
                planned_or_real_changes  = 'R'
                no_change_pointers       = ' '
*               UPD_ICDTXT_ZTRSTLMNT_M_A = ' '
                upd_ztrstlmnt_m_a        = 'U'
              TABLES
                icdtxt_ztrstlmnt_m_a     = lt_cdtxt
                xztrstlmnt_m_a           = lt_ztrstlmnt_ma_new
                yztrstlmnt_m_a           = lt_ztrstlmnt_ma_old.

            COMMIT WORK AND WAIT.
            gw_flag_s = 'X'.
          ELSE.
            ROLLBACK WORK.
          ENDIF.

          CALL FUNCTION 'DEQUEUE_EZ_TRSTLMNT_MA'
            EXPORTING
              mode_ztrstlmnt_m_a = 'E'
              mandt              = sy-mandt
              tknum              = gw_trstlmnt_ma-tknum
              vbeln              = gw_trstlmnt_ma-vbeln
              counter            = gw_trstlmnt_ma-counter
              vsart              = gw_trstlmnt_ma-vsart.

        ENDIF.

        CLEAR : gw_modify, gw_trstlmnt_ma.
      ENDIF.
    ENDLOOP.

  ENDMETHOD.                    "marine_air_data_save
  METHOD validt_shipment2.

    IF sy-ucomm EQ 'ONLI' AND p_act EQ 'D' AND p_act EQ 'E'. "p_sp IS NOT INITIAL AND p_uc EQ abap_false.
      IF s_tknum2[] IS INITIAL.
        MESSAGE 'Enter shipment number'(040) TYPE 'E'.
      ELSE.
        SELECT tknum
               FROM vttk
               INTO TABLE gt_tknum
               WHERE tknum IN s_tknum2.
        IF sy-subrc NE 0.
          SELECT shnumber
                 FROM oigs
                 INTO TABLE gt_tknum
                WHERE shnumber IN s_tknum2.
          IF sy-subrc NE 0.
            MESSAGE 'Given shipment number is not valid'(039) TYPE 'E'.
          ENDIF.
        ENDIF.
      ENDIF.
    ENDIF.

  ENDMETHOD.                    "validt_shipment
  METHOD validt_delivery2.

    TYPES: BEGIN OF lty_vbeln,
            vbeln TYPE likp-vbeln,
           END OF lty_vbeln.

    DATA: lt_vbeln TYPE TABLE OF lty_vbeln.

    IF sy-ucomm EQ 'ONLI' AND p_act EQ 'D' AND p_act EQ 'E'. "p_sp IS NOT INITIAL AND p_uc EQ abap_false.
      IF s_vbeln2[] IS NOT INITIAL.
        SELECT vbeln
               FROM likp
               INTO TABLE lt_vbeln
               WHERE vbeln IN s_vbeln2.
        IF sy-subrc NE 0.
          MESSAGE 'Given Delivery number is not valid'(038) TYPE 'E'.
        ENDIF.
        REFRESH: lt_vbeln.
      ENDIF.
    ENDIF.

  ENDMETHOD.                    "validt_delivery2
  METHOD get_shipment_data.
    DATA: lt_vttp TYPE TABLE OF gty_vttp.
    " Get Shiment Data
    SELECT tknum
           tpnum
           vbeln FROM vttp
                 INTO TABLE gt_vttp
                 WHERE tknum IN s_tknum2
                   AND vbeln IN s_vbeln2.
    IF sy-subrc = 0.
      SORT gt_vttp BY tknum vbeln.
      lt_vttp = gt_vttp.
      SORT lt_vttp BY tknum.
      DELETE ADJACENT DUPLICATES FROM lt_vttp COMPARING tknum.
      DELETE lt_vttp WHERE tknum IS INITIAL.
    ENDIF.
    IF lt_vttp IS NOT INITIAL.
      SELECT tknum
             vbtyp
             shtyp
             sttbg
             dptbg
             uptbg
             datbg
             uatbg
             stten FROM vttk
                   INTO TABLE gt_vttk
                   FOR ALL ENTRIES IN lt_vttp
                   WHERE tknum = lt_vttp-tknum.
      IF sy-subrc = 0.
        SORT gt_vttk BY tknum.
      ENDIF.
    ENDIF.
    CLEAR gw_vttp.
    LOOP AT gt_vttp INTO gw_vttp.
      gw_shipment-tknum = gw_vttp-tknum.
      gw_shipment-tpnum = gw_vttp-tpnum.
      gw_shipment-vbeln = gw_vttp-vbeln.
      CLEAR gw_vttk.
      READ TABLE gt_vttk INTO gw_vttk WITH KEY tknum = gw_vttp-tknum
                                               BINARY SEARCH.
      IF sy-subrc = 0.
        gw_shipment-vbtyp = gw_vttk-vbtyp.
        gw_shipment-shtyp = gw_vttk-shtyp.
        gw_shipment-sttbg = gw_vttk-sttbg.
        gw_shipment-dptbg = gw_vttk-dptbg.
        gw_shipment-uptbg = gw_vttk-uptbg.
        gw_shipment-datbg = gw_vttk-datbg.
        gw_shipment-uatbg = gw_vttk-uatbg.
        gw_shipment-stten = gw_vttk-stten.
      ENDIF.
      APPEND gw_shipment TO gt_shipment.
      CLEAR gw_shipment.
    ENDLOOP.
  ENDMETHOD.                    "get_shipment_data
  METHOD change_shipment.
    DATA : lt_headerdeadline TYPE TABLE OF bapishipmentheaderdeadline,
           lt_headerdeadlineaction  TYPE TABLE OF bapishipmentheaderdeadlineact.
    CLEAR: lt_headerdeadlineaction, lt_headerdeadline, et_return.
    IF headerdeadline-time_type IS NOT INITIAL.
      APPEND headerdeadline TO lt_headerdeadline.
      APPEND headerdeadlineaction TO lt_headerdeadlineaction.
    ENDIF.
    CALL FUNCTION 'BAPI_SHIPMENT_CHANGE'
      EXPORTING
        headerdata           = headerdata
        headerdataaction     = headerdataaction
      TABLES
        headerdeadline       = lt_headerdeadline
        headerdeadlineaction = lt_headerdeadlineaction
        return               = et_return.
  ENDMETHOD.                    "change_shipment


  METHOD validate_shipment.
    IF p_act EQ 'E' AND sy-ucomm EQ 'ONLI'.  "p_uc EQ abap_true AND sy-ucomm EQ 'ONLI'.
      IF s_tknum3 IS INITIAL.
        MESSAGE 'Shipment no. should not be initial' TYPE 'E'.
      ELSE.

        SELECT tknum
                     FROM vttk
                     INTO TABLE gt_tknum
                     WHERE tknum IN s_tknum3.
        IF sy-subrc NE 0.
          SELECT shnumber
                 FROM oigs
                 INTO TABLE gt_tknum
                WHERE shnumber = s_tknum3.
          IF sy-subrc NE 0.
            MESSAGE 'Given shipment number is not valid'(039) TYPE 'E'.
          ENDIF.
        ENDIF.
      ENDIF.
    ENDIF.
  ENDMETHOD.                    "validate_Shipment

  METHOD validate_shipment5.
    IF p_act EQ 'J' AND screen-group1 EQ 'GTA'. "p_gt EQ abap_true AND screen-group1 EQ 'GTA'.
      IF s_tknum5 IS INITIAL.
        MESSAGE 'Shipment no. should not be initial' TYPE 'E'.
      ENDIF.
    ENDIF.

  ENDMETHOD.                    "validate_shipment5

  METHOD validate_scrren.

    IF  sy-ucomm EQ 'ONLI' AND p_act EQ 'I'. "p_inv = abap_true.
      IF s_billdt IS INITIAL.
        MESSAGE 'Enter TR Bill Date'(084) TYPE 'E'.
      ENDIF.
    ENDIF.

  ENDMETHOD.                    "validate_scrren

  METHOD validate_chain.
    DATA : lw_chain TYPE zchainid.
    DATA : lw_del TYPE vbeln_vl.
    DATA : lw_tknum TYPE tknum.

    IF p_chain IS NOT INITIAL AND p_shnu IS NOT INITIAL AND p_del IS NOT INITIAL.
      CLEAR : lw_chain,lw_del,lw_tknum.
      SELECT chainid shnumber delivery
        FROM zscm_chainship CLIENT SPECIFIED
        INTO (lw_chain,lw_tknum,lw_del)
        WHERE mandt = sy-mandt
        AND chainid = p_chain
        AND shnumber = p_shnu
        AND delivery = p_del.
      ENDSELECT.
      IF sy-subrc <> 0.
        CLEAR : lw_chain.
        MESSAGE 'Given Input Is Not Valid'(108) TYPE 'E'.
      ENDIF.
    ENDIF.
  ENDMETHOD.                    "validate_chain
  METHOD vrm_values.

    REFRESH vrm_values.
    vrm_value-key = 'A'.
    vrm_value-text = text-109.
    APPEND vrm_value TO vrm_values.

    vrm_value-key = 'B'.
    vrm_value-text = text-110.
    APPEND vrm_value TO vrm_values.

    vrm_value-key = 'C'.
    vrm_value-text = text-111.
    APPEND vrm_value TO vrm_values.

    vrm_value-key = 'D'.
    vrm_value-text = text-112.
    APPEND vrm_value TO vrm_values.

    vrm_value-key = 'E'.
    vrm_value-text = text-113.
    APPEND vrm_value TO vrm_values.

    vrm_value-key = 'F'.
    vrm_value-text = text-114.
    APPEND vrm_value TO vrm_values.

    vrm_value-key = 'G'.
    vrm_value-text = text-120.
    APPEND vrm_value TO vrm_values.

    vrm_value-key = 'H'.
    vrm_value-text = text-115.
    APPEND vrm_value TO vrm_values.

    vrm_value-key = 'I'.
    vrm_value-text = text-116.
    APPEND vrm_value TO vrm_values.

    vrm_value-key = 'J'.
    vrm_value-text = text-117.
    APPEND vrm_value TO vrm_values.

    vrm_value-key = 'K'.
    vrm_value-text = text-118.
    APPEND vrm_value TO vrm_values.

    vrm_value-key = 'L'.
    vrm_value-text = text-119.
    APPEND vrm_value TO vrm_values.

    vrm_id = 'P_ACT'.

    CALL FUNCTION 'VRM_SET_VALUES'
      EXPORTING
        id              = vrm_id
        values          = vrm_values
      EXCEPTIONS
        id_illegavrm_id = 1
        OTHERS          = 2.
    IF sy-subrc <> 0.
      MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
      WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
    ENDIF.
  ENDMETHOD.                    "vrm_values

  METHOD hang_ses_data.
    " Logic to cancel Service entry sheet with dummy shipment cost document .

    IF s_lblni IS INITIAL.
      MESSAGE 'Input Service entry sheet is mandatory'(061) TYPE 'S'.
      LEAVE LIST-PROCESSING.
    ENDIF.
    IF p_fknum IS INITIAL.
      MESSAGE 'Input Dummy SCD is mandatory'(062) TYPE 'S'.
      LEAVE LIST-PROCESSING.
    ELSE.
      CLEAR gw_fknum.
      SELECT SINGLE fknum
             FROM vfkk INTO gw_fknum
             WHERE fknum EQ p_fknum.
      IF sy-subrc NE 0.
        MESSAGE 'Given Dummy SCD is invalid'(060) TYPE 'S'.
      ENDIF.
    ENDIF.

    CLEAR gt_essr.

    SELECT lblni " ses
           ebeln
           ebelp
           loekz  " del ind
           fknum  " scd
           fkpos
           FROM essr INTO TABLE gt_essr
           WHERE lblni IN s_lblni.
    IF sy-subrc EQ 0.

      SORT gt_essr BY loekz .
      DELETE gt_essr WHERE loekz IS NOT INITIAL.
      gt_essr_t = gt_essr.

      SORT gt_essr_t BY fknum.
      DELETE gt_essr_t WHERE fknum IS INITIAL.
      DELETE ADJACENT DUPLICATES FROM gt_essr_t COMPARING fknum.

      IF gt_essr_t IS NOT INITIAL.
        SELECT fknum
               FROM vfkk INTO TABLE gt_vfkk
               FOR ALL ENTRIES IN gt_essr
               WHERE fknum EQ gt_essr-fknum.
        IF sy-subrc EQ 0.
          SORT gt_vfkk BY fknum.
        ENDIF.
        CLEAR gt_essr_t.
      ENDIF.
    ENDIF.

    " 1. Make sure SES is fault.
    SORT gt_essr BY lblni.
    LOOP AT gt_essr INTO gw_essr.
      READ TABLE gt_vfkk TRANSPORTING NO FIELDS WITH KEY fknum = gw_essr-fknum BINARY SEARCH.
      IF sy-subrc NE 0.
        APPEND gw_essr TO gt_essr_t.
      ENDIF.
      CLEAR gw_essr.
    ENDLOOP.

    IF gt_essr_t IS INITIAL.
      MESSAGE 'No service entry sheets found to correct'(059) TYPE 'S'.
      LEAVE LIST-PROCESSING.
    ENDIF.


    " step2: Update Fault SES number  with dummy shipment cost document

    ""-Reading shipment cost document
    " Logic Copied from FM: Z_LSD_SCD_SES_DELETE (RD2)

    CLEAR : gw_scdd, gt_ref_obj.

    CALL FUNCTION 'SD_SCD_VIEW'
      EXPORTING
        i_fknum              = p_fknum
        i_t180               = gw_t180
        i_opt_document_flow  = gc_x
        i_opt_costs          = gc_x
        i_opt_costs_complete = gc_x
        i_opt_accounts       = gc_x
        i_opt_partners       = gc_x
        i_opt_refobj         = gc_x
        i_opt_refobj_lock    = gc_x
        i_opt_refobj_reduced = gc_x
      CHANGING
        c_scd                = gw_scdd
        c_refobj_tab         = gt_ref_obj
      EXCEPTIONS
        scd_not_found        = 1
        no_authority         = 2
        tvtf_type_not_valid  = 3
        refobj_lock          = 4
        refobj_not_found     = 5
        delivery_missing     = 6
        OTHERS               = 7.

    IF sy-subrc EQ 0.

      READ TABLE gt_ref_obj INTO gw_refobj INDEX 1.

      IF sy-subrc = 0.
        LOOP AT gt_essr_t INTO gw_essr.

          LOOP AT gw_scdd-x-item INTO gw_scd_itm.

            gw_scdd-y = gw_scdd-x.

            CLEAR gt_scd_itm.

            gw_scd_itm-konv_changed = gc_x.

            IF NOT ( gw_scd_itm-vfkp-updkz = gc_i )."updkz_new ).
              gw_scd_itm-vfkp-updkz = gc_u."updkz_update.
            ENDIF.

            gw_scd_itm-calc_complete = 'C'.
            gw_scd_itm-vfkp-slfrei   = 'X'.
            gw_scd_itm-vfkp-stdat    = sy-datum.

            gw_scd_itm-vfkp-lblni = gw_essr-lblni.
            gw_scd_itm-vfkp-ebeln = gw_essr-ebeln.
            gw_scd_itm-vfkp-ebelp = gw_essr-ebelp.

            gw_scd_itm-vfkp-stabr = 'C'.  " For Transfer

            gw_fkpos = gw_scd_itm-vfkp-fkpos. " scd item number

            APPEND gw_scd_itm TO gt_scd_itm.

            ""-Changing new values
            CALL FUNCTION 'SD_SCD_ITEM_CHANGE'            "#EC CI_SUBRC
              EXPORTING
                i_t180                     = gw_t180
                i_refobj                   = gw_refobj
              CHANGING
                c_scd_item_new             = gw_scd_itm
                c_scd_item_tab             = gt_scd_itm
              EXCEPTIONS                                    "#EC FB_RC
                release_flag_change_failed = 1
                OTHERS                     = 99.

            IF sy-subrc = 0.

              CLEAR : gw_scdd_tmp.
              gw_scdd_tmp = gw_scdd.

              gw_scdd_tmp-x-item = gt_scd_itm.

              APPEND gw_scdd_tmp TO gt_scdd_tab.
              ""-Saving changed price
              CALL FUNCTION 'SD_SCDS_SAVE'
                EXPORTING
                  i_t180            = gw_t180
                  i_opt_update_task = gc_x
                  i_refobj_tab      = gt_ref_obj
                CHANGING
                  c_scd_tab         = gt_scdd_tab
                EXCEPTIONS
                  no_change         = 1
                  no_save           = 2
                  OTHERS            = 3.
              IF sy-subrc NE 0.
                MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
                                    WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
              ELSE.

                " Below Standard table Update required. - here.
                " 2. Update dummy SCD and SCD item in SES
                UPDATE essr SET fknum = p_fknum
                                fkpos = gw_fkpos
                          WHERE lblni = gw_essr-lblni.
                IF sy-subrc EQ 0.

                  COMMIT WORK AND WAIT.
                  gw_flag_sv = abap_true.
                  SET PARAMETER ID 'FKK' FIELD p_fknum.

                  " Below FM used To cancel SES.
                  CALL FUNCTION 'Z_LSD_SCD_SES_DELETE'
                    EXPORTING
                      i_fknum = p_fknum.
                ENDIF.

              ENDIF.
            ENDIF.
            CLEAR : gw_scd_itm, gw_tabix, gt_scdd_tab.
          ENDLOOP.

        ENDLOOP.

      ENDIF.

    ENDIF.

    IF gw_flag_sv EQ abap_true.
      MESSAGE 'Data Saved'(063) TYPE 'S'.
      LEAVE LIST-PROCESSING.
    ENDIF.

  ENDMETHOD.                    "get_ses_data

  METHOD get_coal_data.
    DATA: lt_lrpo_t   TYPE TABLE OF lty_zptc_lrpo,
          lw_lrpo     TYPE          lty_zptc_lrpo,
          lw_tknum    TYPE          tknum,
          lw_flag     TYPE          flag,
          lt_trstlmnt TYPE TABLE OF ztrstlmnt,
          lw_trstlmnt TYPE          ztrstlmnt.

    FIELD-SYMBOLS: <lfs_trstlmnt> TYPE ztrstlmnt.

    IF s_tknum3 IS NOT INITIAL.
      SELECT lr_no
             lr_itemno
             quantity_l
             shnumber
             meins
             vbeln_d
             created_on
        FROM zptc_lrpo CLIENT SPECIFIED
        INTO TABLE gt_lrpo
        WHERE mandt = sy-mandt
        AND   shnumber IN s_tknum3." lw_tknum.
      IF sy-subrc NE  0.
        MESSAGE 'Shipment number not found for query'(053) TYPE 'E'.
      ENDIF.

      REFRESH lt_lrpo_t.
      lt_lrpo_t = gt_lrpo.
      SORT lt_lrpo_t BY shnumber vbeln_d.
      DELETE ADJACENT DUPLICATES FROM lt_lrpo_t COMPARING shnumber vbeln_d.

      IF lt_lrpo_t IS NOT INITIAL.
        SELECT *  "All field required
          FROM ztrstlmnt CLIENT SPECIFIED
          INTO TABLE lt_trstlmnt
          FOR ALL ENTRIES IN lt_lrpo_t
          WHERE mandt = sy-mandt
          AND   tknum = lt_lrpo_t-shnumber
          AND   vbeln = lt_lrpo_t-vbeln_d.
        IF sy-subrc EQ 0.
          SORT lt_trstlmnt BY tknum vbeln.
        ELSE.
          MESSAGE 'No data found to update'(055) TYPE 'E'.
        ENDIF.
      ENDIF.

      SORT gt_lrpo BY shnumber vbeln_d.

      LOOP AT lt_trstlmnt ASSIGNING <lfs_trstlmnt>.
        READ TABLE gt_lrpo INTO lw_lrpo WITH KEY shnumber = <lfs_trstlmnt>-tknum
                                                 vbeln_d  = <lfs_trstlmnt>-vbeln
                                                 BINARY SEARCH.
        IF sy-subrc EQ 0.
          <lfs_trstlmnt>-lr_no  = lw_lrpo-lr_no.
          <lfs_trstlmnt>-lr_dt  = lw_lrpo-created_on.
          <lfs_trstlmnt>-lr_qty = lw_lrpo-quantity_l.
          <lfs_trstlmnt>-lr_uom = lw_lrpo-meins.
          lw_flag = abap_true.
        ENDIF.
      ENDLOOP.

      IF lw_flag EQ abap_true.
        MODIFY ztrstlmnt FROM TABLE lt_trstlmnt.
        IF sy-subrc EQ 0.
          MESSAGE 'Data updated successfully'(054) TYPE 'S'.
          CLEAR: s_tknum3[].
          COMMIT  WORK.
        ELSE.
          ROLLBACK WORK.
        ENDIF.
      ENDIF.
    ENDIF.
    CLEAR gt_lrpo.
  ENDMETHOD.                    "get_coal_Data
  METHOD suppl_plnt_upd.
    TYPES: BEGIN OF lty_vttk,
           tknum  TYPE tknum,
           shtyp  TYPE shtyp,
           datbg TYPE  datbg,
           uatbg TYPE  uatbg,
           zzserv_plant TYPE zserv_plant,
           END OF lty_vttk,
           BEGIN OF lty_stlmnt,
           tknum   TYPE tknum,
           vbeln   TYPE vbeln_vl,
           counter TYPE  zcounter,
           vsart   TYPE vsarttr,
           bill_doc_no TYPE vbeln_vf,
           END OF lty_stlmnt,
           BEGIN OF lty_vbrp,
           vbeln TYPE  vbeln_vf,
           posnr TYPE  posnr_vf,
           werks TYPE werks_d,
           END OF lty_vbrp,
           BEGIN OF lty_serv_plant,
           bukrs      TYPE  bukrs,
           mat_plant  TYPE  zmat_plant,
           vsart      TYPE vsarttr,
           serv_plant TYPE zserv_plant,
           END OF lty_serv_plant.

    DATA: lt_vttk       TYPE TABLE OF lty_vttk,
          lt_stlmnt     TYPE TABLE OF lty_stlmnt,
          lt_stlmnt_t   TYPE TABLE OF lty_stlmnt,
          lt_vbrp       TYPE TABLE OF lty_vbrp,
          lt_vbrp_t     TYPE TABLE OF lty_vbrp,
          lt_serv_plant TYPE TABLE OF lty_serv_plant,
          lw_stlmnt     TYPE lty_stlmnt,
          lw_vbrp       TYPE lty_vbrp,
          lw_serv_plant TYPE lty_serv_plant.

    DATA :lw_headerdata       TYPE bapishipmentheader,
          lw_headerdataaction TYPE bapishipmentheaderaction,
          lw_headerdeadline   TYPE bapishipmentheaderdeadline,
          lv_timestamp        TYPE tzonref-tstamps,
          lw_headerdeadlineaction  TYPE bapishipmentheaderdeadlineact,
          lt_return TYPE STANDARD TABLE OF bapiret2,
          lt_return2 TYPE STANDARD TABLE OF bapiret2,
          lw_return TYPE bapiret2.
    FIELD-SYMBOLS <lfs_vttk> TYPE lty_vttk.

    go_ref->valid_suppl_plnt( ).

    "get shipments with service plant blank
    IF s_tknum4[] IS INITIAL.
      SELECT tknum shtyp datbg uatbg zzserv_plant
        FROM vttk  CLIENT SPECIFIED
        INTO TABLE lt_vttk
        WHERE mandt = sy-mandt
          AND shtyp = p_styp
          AND zzserv_plant = ' '.
      IF sy-subrc = 0.
        SORT lt_vttk BY tknum.
      ELSE.
        MESSAGE text-034 TYPE 'S' DISPLAY LIKE 'E'.
        LEAVE LIST-PROCESSING.
      ENDIF.
    ELSE.
      SELECT tknum shtyp datbg uatbg zzserv_plant
     FROM vttk  CLIENT SPECIFIED
     INTO TABLE lt_vttk
     WHERE mandt = sy-mandt
       AND tknum IN s_tknum4
       AND shtyp = p_styp
       AND zzserv_plant = ' '.
      IF sy-subrc = 0.
        SORT lt_vttk BY tknum.
      ELSE.
        MESSAGE text-034 TYPE 'S' DISPLAY LIKE 'E'.
        LEAVE LIST-PROCESSING.
      ENDIF.
    ENDIF.
    "get invoice number against shipments
    IF lt_vttk IS NOT INITIAL.
      SELECT tknum vbeln counter vsart bill_doc_no
        FROM ztrstlmnt CLIENT SPECIFIED
         INTO TABLE lt_stlmnt
          FOR ALL ENTRIES IN lt_vttk
           WHERE mandt = sy-mandt
            AND tknum = lt_vttk-tknum.
      IF sy-subrc = 0.
        CLEAR lt_stlmnt_t.
        lt_stlmnt_t = lt_stlmnt.
        SORT lt_stlmnt_t BY bill_doc_no.
        DELETE lt_stlmnt_t WHERE bill_doc_no IS INITIAL.
        DELETE ADJACENT DUPLICATES FROM lt_stlmnt_t COMPARING bill_doc_no.
      ENDIF.
    ENDIF.

    "get PO plant against invoice number
    IF lt_stlmnt_t IS NOT INITIAL.
      SELECT vbeln posnr werks
        FROM vbrp CLIENT SPECIFIED
         INTO TABLE lt_vbrp
         FOR ALL ENTRIES IN lt_stlmnt_t
           WHERE mandt = sy-mandt
            AND vbeln = lt_stlmnt_t-bill_doc_no.
      IF sy-subrc = 0.
        CLEAR lt_vbrp_t.
        lt_vbrp_t = lt_vbrp.
        SORT lt_vbrp_t BY werks.
        DELETE lt_vbrp_t WHERE werks IS INITIAL.
        DELETE ADJACENT DUPLICATES FROM lt_vbrp_t COMPARING werks.
      ENDIF.
    ENDIF.
    "get service plant against PO plant
    IF lt_vbrp_t IS NOT INITIAL.
      SELECT bukrs mat_plant vsart serv_plant
        FROM zlog_serv_plant CLIENT SPECIFIED
         INTO TABLE lt_serv_plant
         FOR ALL ENTRIES IN lt_vbrp_t
           WHERE mandt = sy-mandt
            AND mat_plant = lt_vbrp_t-werks
            AND vsart  = space.
      IF sy-subrc = 0.
        SORT lt_serv_plant BY mat_plant.
      ENDIF.
    ENDIF.
    SORT:lt_stlmnt BY tknum,
         lt_vbrp BY vbeln.

    UNASSIGN <lfs_vttk>.
    LOOP AT lt_vttk ASSIGNING <lfs_vttk>.
      READ TABLE lt_stlmnt INTO lw_stlmnt
         WITH KEY tknum = <lfs_vttk>-tknum BINARY SEARCH.
      IF sy-subrc = 0.
        READ TABLE lt_vbrp INTO lw_vbrp
           WITH KEY vbeln = lw_stlmnt-bill_doc_no BINARY SEARCH.
        IF sy-subrc = 0.
          READ TABLE lt_serv_plant INTO lw_serv_plant
             WITH KEY mat_plant = lw_vbrp-werks BINARY SEARCH.
          IF sy-subrc = 0.
            <lfs_vttk>-zzserv_plant = lw_serv_plant-serv_plant.
          ENDIF.
        ENDIF.
      ENDIF.
    ENDLOOP.

    DELETE lt_vttk WHERE zzserv_plant IS INITIAL.

    CLEAR gv_succ.
    LOOP AT lt_vttk ASSIGNING <lfs_vttk>.
      CLEAR: lt_return, lt_return2.
      CLEAR:lw_headerdata ,lw_headerdataaction.
      lw_headerdata-shipment_num = <lfs_vttk>-tknum.
      lw_headerdata-zzserv_plant = <lfs_vttk>-zzserv_plant.
      lw_headerdata-status_shpmnt_start = 'D'.
      lw_headerdataaction-status_shpmnt_start = 'D'.
      lw_headerdataaction-shipment_num = abap_true.

      CALL FUNCTION 'BAPI_SHIPMENT_CHANGE'
        EXPORTING
          headerdata       = lw_headerdata
          headerdataaction = lw_headerdataaction
        TABLES
          return           = lt_return.

      READ TABLE lt_return INTO lw_return WITH KEY type = 'E'."lc_e.
      IF sy-subrc IS NOT INITIAL.
        CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
          EXPORTING
            wait = 'X'.
        gv_succ = 'X'.

        CLEAR:lw_headerdata ,lw_headerdataaction,lw_headerdeadline.
        lw_headerdata-shipment_num = <lfs_vttk>-tknum.
        lw_headerdata-status_shpmnt_start = abap_true.
        lw_headerdataaction-shipment_num = abap_true.
        lw_headerdataaction-status_shpmnt_start = 'C'.

        CALL FUNCTION 'CONVERT_INTO_TIMESTAMP'
          EXPORTING
            i_datlo     = <lfs_vttk>-datbg
            i_timlo     = <lfs_vttk>-uatbg
*           I_TZONE     = SY-ZONLO
          IMPORTING
            e_timestamp = lv_timestamp.

        lw_headerdeadline-time_stamp_utc = lv_timestamp.
        lw_headerdeadline-time_type = 'HDRSTSSADT'.
        lw_headerdeadline-time_zone = 'INDIA'.
        lw_headerdeadlineaction-time_stamp_utc = 'C'.
        lw_headerdeadlineaction-time_type = 'C'.
        lw_headerdeadlineaction-time_zone = 'C'.

        CALL FUNCTION 'BAPI_SHIPMENT_CHANGE'
          EXPORTING
            headerdata       = lw_headerdata
            headerdataaction = lw_headerdataaction
          TABLES
            return           = lt_return2.

        READ TABLE lt_return2 INTO lw_return WITH KEY type = 'E'."lc_e.
        IF sy-subrc IS NOT INITIAL.
          CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
            EXPORTING
              wait = 'X'.
        ELSE.
          CALL FUNCTION 'BAPI_TRANSACTION_ROLLBACK'.
        ENDIF.

      ELSE.
        CALL FUNCTION 'BAPI_TRANSACTION_ROLLBACK'.
      ENDIF.

    ENDLOOP.

  ENDMETHOD.                    "suppl_plnt_upd
  METHOD valid_suppl_plnt.
    TYPES: BEGIN OF lty_tknum,
           tknum TYPE tknum,
           END OF lty_tknum.
    DATA:lt_tknum TYPE TABLE OF lty_tknum.

    IF s_tknum4[] IS NOT INITIAL.
      SELECT  tknum
        FROM vttk
         INTO TABLE lt_tknum
        WHERE tknum IN s_tknum4.
      IF sy-subrc NE 0.
        MESSAGE 'Enter Valid Shipment'(058) TYPE 'S' DISPLAY LIKE 'E'.
        LEAVE LIST-PROCESSING.
      ENDIF.
    ENDIF.
  ENDMETHOD.                    "valid_suppl_plnt
  METHOD get_shrotage_data.
    DATA: lw_dest       TYPE char70,
          lt_dcpi       TYPE efg_tab_ranges,
          lw_dcpi       LIKE LINE OF lt_dcpi,
          lt_date       TYPE efg_tab_ranges,
          lw_date       LIKE LINE OF lt_date,
          lw_ip_dcpi    LIKE LINE OF s_dcpi,
          lw_ip_date    LIKE LINE OF s_date.

    IF s_dcpi IS NOT INITIAL OR s_date IS NOT INITIAL.
      LOOP AT s_dcpi INTO lw_ip_dcpi.
        lw_dcpi-sign = lw_ip_dcpi-sign.
        lw_dcpi-option = lw_ip_dcpi-option.
        lw_dcpi-low = lw_ip_dcpi-low.
        lw_dcpi-high = lw_ip_dcpi-high.
        APPEND lw_dcpi TO lt_dcpi.
        CLEAR: lw_dcpi,lw_ip_dcpi.
      ENDLOOP.

      LOOP AT s_date INTO lw_ip_date.
        lw_date-sign = lw_ip_date-sign.
        lw_date-option = lw_ip_date-option.
        lw_date-low = lw_ip_date-low.
        lw_date-high = lw_ip_date-high.
        APPEND lw_date TO lt_date.
        CLEAR: lw_date,lw_ip_date.
      ENDLOOP.

      CALL METHOD zcl_log_fcpl=>fcpl_rfc_dest
        IMPORTING
          ex_rfc_dest = lw_dest.

      CALL FUNCTION 'RFC_PING' DESTINATION lw_dest
        EXCEPTIONS
          system_failure        = 1
          communication_failure = 2.

      IF sy-subrc EQ 0.
        CALL FUNCTION 'Z_SCE_SHORTAGE_GETDATA' DESTINATION lw_dest
          EXPORTING
            i_dcpino    = lt_dcpi
            i_dcpidt    = lt_date
          TABLES
            lt_shortage = gt_shortage.
        LOOP AT gt_shortage INTO gw_shortage.
          gw_shortage_final-tknum               = gw_shortage-tknum.
          gw_shortage_final-lifnr               = gw_shortage-lifnr.
          gw_shortage_final-vbeln               = gw_shortage-vbeln.
          gw_shortage_final-bill_doc_no         = gw_shortage-bill_doc_no.
          gw_shortage_final-chrg_code           = gw_shortage-chrg_code.
          gw_shortage_final-shortage_qty        = gw_shortage-shortage_qty.
          gw_shortage_final-shortage_uom        = gw_shortage-shortage_uom.
          gw_shortage_final-recovery_val        = gw_shortage-recovery_val.
          gw_shortage_final-recovery_curr       = gw_shortage-recovery_curr.
          gw_shortage_final-zzserv_plant        = gw_shortage-zzserv_plant.
          gw_shortage_final-erdat               = gw_shortage-erdat.
          gw_shortage_final-erzet               = gw_shortage-erzet.
          gw_shortage_final-ernam               = gw_shortage-ernam.
          gw_shortage_final-invoice1            = gw_shortage-invoice1.
          gw_shortage_final-invoice2            = gw_shortage-invoice2.
          gw_shortage_final-invoice3            = gw_shortage-invoice3.
          gw_shortage_final-remarks             = gw_shortage-remarks.
          gw_shortage_final-fi_doc1             = gw_shortage-fi_doc1.
          gw_shortage_final-fcpl_by_ril         = gw_shortage-fcpl_by_ril.
          gw_shortage_final-fi_doc2             = gw_shortage-fi_doc2.
          gw_shortage_final-cust_credit_amt     = gw_shortage-cust_credit_amt.
          gw_shortage_final-unique_ref_no       = gw_shortage-unique_ref_no.
          gw_shortage_final-unique_ref_no2      = gw_shortage-unique_ref_no2.
          gw_shortage_final-ro_outdate          = gw_shortage-ro_outdate.
          gw_shortage_final-gst_inv_no          = gw_shortage-gst_inv_no.
          gw_shortage_final-invoice_dt          = gw_shortage-invoice_dt.
          gw_shortage_final-invoice_amt         = gw_shortage-invoice_amt.
          gw_shortage_final-ship_to_party       = gw_shortage-ship_to_party.
          gw_shortage_final-dms_no              = gw_shortage-dms_no.
          gw_shortage_final-app_cust_amt        = gw_shortage-app_cust_amt.
          gw_shortage_final-scrtype             = gw_shortage-scrtype.
          gw_shortage_final-mfrgr               = gw_shortage-mfrgr.
          gw_shortage_final-delay_dy            = gw_shortage-delay_dy.
          gw_shortage_final-business            = gw_shortage-business.
          gw_shortage_final-sub_business        = gw_shortage-sub_business.
          gw_shortage_final-sub_format          = gw_shortage-sub_format.
          gw_shortage_final-cn_date             = gw_shortage-cn_date.
          gw_shortage_final-cn_no               = gw_shortage-cn_no.
          gw_shortage_final-rate                = gw_shortage-rate.
          APPEND gw_shortage_final TO gt_shortage_final.
          CLEAR: gw_shortage_final,gw_shortage.
        ENDLOOP.
      ENDIF.
    ENDIF.
  ENDMETHOD.                    "GET_SHROTAGE_DATA
  METHOD get_inv_data.

    SELECT  lifnr
            tr_billno
            tr_bill_dt
            counter
            bl_no
            shtyp
            pro_status
            tr_bill_amt
            tr_bill_curr
            blk_rsn
            del_ind
            FROM zsce_rec_invsl
            INTO TABLE gt_rec_invsl
            WHERE lifnr IN s_lifnr
            AND   tr_billno IN s_billno
            AND   tr_bill_dt IN s_billdt.
    IF sy-subrc = 0.
      SORT gt_rec_invsl BY lifnr tr_billno tr_bill_dt counter.
      gt_rec_invsl_tmp = gt_rec_invsl.
      DELETE  gt_rec_invsl_tmp WHERE lifnr IS INITIAL
                               AND tr_billno IS INITIAL
                               AND tr_bill_dt IS INITIAL.
      DELETE ADJACENT DUPLICATES FROM gt_rec_invsl_tmp COMPARING lifnr tr_billno tr_bill_dt.
      IF gt_rec_invsl_tmp IS NOT INITIAL.

        SELECT  scrnum
                bukrs
                apcode
                syear
                xblnr
                bldat
                lifnr
                currproc
                FROM yinvhead CLIENT SPECIFIED
                INTO TABLE gt_yinvhead
                FOR ALL ENTRIES IN gt_rec_invsl_tmp
                WHERE mandt = sy-mandt
                AND xblnr =   gt_rec_invsl_tmp-tr_billno
                AND bldat =   gt_rec_invsl_tmp-tr_bill_dt.
        IF sy-subrc = 0.
          SORT  gt_yinvhead BY xblnr bldat lifnr.
        ENDIF.
      ENDIF.
    ENDIF.

    CLEAR:gt_rec_invsl_tmp,gt_rec_invchrg_tmp.
    gt_rec_invsl_tmp = gt_rec_invsl.
    SORT:gt_rec_invsl_tmp BY lifnr tr_billno tr_bill_dt counter.

  ENDMETHOD.                    "get_inv_data
  METHOD create_obj.
    IF go_ref_alv IS BOUND.
      go_ref_alv->free( ).
    ENDIF.

    IF go_ref_dock IS BOUND.
      go_ref_dock->free( ).
    ENDIF.

    CREATE OBJECT go_ref_dock
      EXPORTING
        side                        = cl_gui_docking_container=>dock_at_left
        extension                   = 1530
      EXCEPTIONS
        cntl_error                  = 1
        cntl_system_error           = 2
        create_error                = 3
        lifetime_error              = 4
        lifetime_dynpro_dynpro_link = 5
        OTHERS                      = 6.
    IF sy-subrc <> 0.
      MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
                WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
    ENDIF.

    CREATE OBJECT go_ref_doc.

    CREATE OBJECT go_ref_alv
      EXPORTING
        i_parent          = go_ref_dock
      EXCEPTIONS
        error_cntl_create = 1
        error_cntl_init   = 2
        error_cntl_link   = 3
        error_dp_create   = 4
        OTHERS            = 5.
    IF sy-subrc <> 0.
      MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
                 WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
    ENDIF.


  ENDMETHOD.                    "create_obj
  METHOD  create_fcat.

    CLEAR:gt_fcatlog.

    CLEAR:gw_fcatlog.
    gw_fcatlog-fieldname = 'LIFNR'.
    gw_fcatlog-col_pos =   '1'.
    gw_fcatlog-coltext =   'Vendor Code'(067).
    APPEND gw_fcatlog TO gt_fcatlog.

    CLEAR:gw_fcatlog.
    gw_fcatlog-fieldname = 'TR_BILLNO'.
    gw_fcatlog-col_pos =   '2'.
    gw_fcatlog-coltext =   'TR Bill No'(068).
    gw_fcatlog-hotspot =   'X'.
    APPEND gw_fcatlog TO gt_fcatlog.

    CLEAR:gw_fcatlog.
    gw_fcatlog-fieldname = 'TR_BILL_DT'.
    gw_fcatlog-col_pos =   '3'.
    gw_fcatlog-coltext =   'TR Bill Date'(069).
    APPEND gw_fcatlog TO gt_fcatlog.

    CLEAR:gw_fcatlog.
    gw_fcatlog-fieldname = 'BL_NO'.
    gw_fcatlog-col_pos =   '4'.
    gw_fcatlog-coltext =   'BL No'(070).
    APPEND gw_fcatlog TO gt_fcatlog.

    CLEAR:gw_fcatlog.
    gw_fcatlog-fieldname = 'SHTYP'.
    gw_fcatlog-col_pos =   '5'.
    gw_fcatlog-coltext =   'Shipment Type'(071).
    APPEND gw_fcatlog TO gt_fcatlog.

    CLEAR:gw_fcatlog.
    gw_fcatlog-fieldname = 'PRO_STATUS'.
    gw_fcatlog-col_pos =   '6'.
    gw_fcatlog-coltext =   'Staus'(072).
    gw_fcatlog-edit    =   'X'.
    gw_fcatlog-ref_table = 'ZSCE_REC_INVSL'.
    gw_fcatlog-tabname  =  'GT_REC_INVSL'.
    APPEND gw_fcatlog TO gt_fcatlog.

    CLEAR:gw_fcatlog.
    gw_fcatlog-fieldname = 'TR_BILL_AMT'.
    gw_fcatlog-col_pos =   '7'.
    gw_fcatlog-coltext =   'Bill Amount'(073).
    APPEND gw_fcatlog TO gt_fcatlog.

    CLEAR:gw_fcatlog.
    gw_fcatlog-fieldname = 'TR_BILL_CURR'.
    gw_fcatlog-col_pos =   '8'.
    gw_fcatlog-coltext =   'TR Bill Currency'(074).
    APPEND gw_fcatlog TO gt_fcatlog.

    CLEAR:gw_fcatlog.
    gw_fcatlog-fieldname = 'BLK_RSN'.
    gw_fcatlog-col_pos =   '9'.
    gw_fcatlog-coltext =   'Reason'(075).
    gw_fcatlog-edit    =   'X'.
    gw_fcatlog-ref_table = 'ZSCE_REC_INVSL'.
    APPEND gw_fcatlog TO gt_fcatlog.

    CLEAR:gw_fcatlog.
    gw_fcatlog-fieldname = 'DEL_IND'.
    gw_fcatlog-col_pos =   '10'.
    gw_fcatlog-coltext =   'Deletion Indicator'(076).
    gw_fcatlog-edit    =   'X'.
    gw_fcatlog-ref_table = 'ZSCE_REC_INVSL'.
    APPEND gw_fcatlog TO gt_fcatlog.


  ENDMETHOD.                    "create_fcat
  METHOD display_alv.

    DATA : lw_lvc_lay TYPE lvc_s_layo.
    DATA : lt_sort TYPE STANDARD TABLE OF lvc_s_sort,
           lw_sort TYPE lvc_s_sort,
           lt_grp TYPE TABLE OF lvc_s_sgrp,
           lw_grp TYPE lvc_s_sgrp,
           lw_layout   TYPE lvc_s_layo,
           lw_variant  TYPE disvariant.

    DATA : lt_functions TYPE ui_functions.
    CLEAR: lw_sort.

    DATA : lw_excludebutton TYPE ui_func,
           lt_excludebutton TYPE ui_functions.


    "excluding graph anf info button of alv toolbar
    lw_excludebutton = cl_gui_alv_grid=>mc_fc_graph.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_info.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_loc_copy_row.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_loc_cut.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_loc_delete_row.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_loc_insert_row.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.
    lw_excludebutton = cl_gui_alv_grid=>mc_fc_loc_delete_row.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_print.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_save_variant.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_sort_asc.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_sort_dsc.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_loc_append_row.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_loc_undo.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_maintain_variant.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_subtot.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    SET HANDLER  go_ref->toolbar FOR go_ref_alv.
    SET HANDLER  go_ref->user_command FOR go_ref_alv.
    "set handler for your alv
    SET HANDLER go_ref->handle_hotspot_click FOR go_ref_alv.

    lw_layout-cwidth_opt    = abap_true.
    lw_layout-col_opt       = abap_true.


    CALL METHOD go_ref_alv->set_table_for_first_display
      EXPORTING
        is_variant                    = lw_variant
        is_layout                     = lw_layout
        it_toolbar_excluding          = lt_excludebutton "lt_functions
      CHANGING
        it_outtab                     = gt_rec_invsl
        it_fieldcatalog               = gt_fcatlog
        it_sort                       = lt_sort
      EXCEPTIONS
        invalid_parameter_combination = 1
        program_error                 = 2
        too_many_lines                = 3
        OTHERS                        = 4.
    IF sy-subrc <> 0.
      MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
                 WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
    ENDIF.
    CALL METHOD go_ref_alv->register_edit_event
      EXPORTING
        i_event_id = cl_gui_alv_grid=>mc_evt_enter.


  ENDMETHOD.                    "display_alv
  METHOD toolbar.

    DATA: ls_toolbar  TYPE stb_button.

    ls_toolbar-function  = 'SAVE'.                          "#EC NOTEXT
    ls_toolbar-icon      =  icon_system_save.
    ls_toolbar-text      = 'SAVE'.
    ls_toolbar-butn_type = '0'.
    APPEND ls_toolbar TO e_object->mt_toolbar.
    CLEAR : ls_toolbar.

  ENDMETHOD.                    "toolbar
  METHOD toolbar_popup.

    DATA: ls_toolbar  TYPE stb_button.

    ls_toolbar-function  = 'SAVE_POPUP'.                    "#EC NOTEXT
    ls_toolbar-icon      =  icon_system_save.
    ls_toolbar-text      = 'SAVE'.
    ls_toolbar-butn_type = '0'.
    APPEND ls_toolbar TO e_object->mt_toolbar.
    CLEAR : ls_toolbar.

  ENDMETHOD.                    "toolbar_popup
  METHOD user_command.

    DATA:lt_selected_row    TYPE TABLE OF lvc_s_row,
         lw_selected        TYPE          lvc_s_row,
         lw_success         TYPE char1.

    DATA:   lw_obj    TYPE balobj_d,
            lw_billno TYPE string,
            lw_subobj  TYPE balsubobj,
            lt_msg    TYPE STANDARD TABLE OF lmess,
            lw_msg    LIKE LINE OF lt_msg,
            lw_text TYPE char50.

    FIELD-SYMBOLS: <lfs_rec_invsl> TYPE   gty_rec_invsl.

    CLEAR:lt_selected_row.

    CALL METHOD go_ref_alv->get_selected_rows
      IMPORTING
        et_index_rows = lt_selected_row.

    IF lt_selected_row IS INITIAL.
      MESSAGE 'Select atleast one row to process'(080)      "#EC NOTEXT
                 TYPE 'S' DISPLAY LIKE 'E'.
      RETURN.
    ENDIF.

    CASE e_ucomm.
      WHEN 'SAVE'.

        LOOP AT lt_selected_row INTO lw_selected.
          READ TABLE gt_rec_invsl ASSIGNING <lfs_rec_invsl>  INDEX lw_selected-index.
          IF sy-subrc = 0.
            READ TABLE gt_rec_invsl_tmp INTO gw_rec_invsl_tmp WITH  KEY lifnr = <lfs_rec_invsl>-lifnr
                                                                        tr_billno = <lfs_rec_invsl>-tr_billno
                                                                        tr_bill_dt = <lfs_rec_invsl>-tr_bill_dt
                                                                        counter  = <lfs_rec_invsl>-counter
                                                                        BINARY SEARCH.
            IF sy-subrc = 0 AND ( gw_rec_invsl_tmp-pro_status <> <lfs_rec_invsl>-pro_status
                             OR gw_rec_invsl_tmp-blk_rsn <> <lfs_rec_invsl>-blk_rsn
                             OR gw_rec_invsl_tmp-del_ind <> <lfs_rec_invsl>-del_ind )
                             AND <lfs_rec_invsl>-lifnr IS NOT INITIAL
                             AND <lfs_rec_invsl>-tr_billno IS NOT INITIAL
                             AND <lfs_rec_invsl>-tr_bill_dt IS NOT INITIAL
                             AND <lfs_rec_invsl>-counter IS NOT INITIAL.

              READ TABLE gt_yinvhead INTO gw_yinvhead WITH  KEY  xblnr = <lfs_rec_invsl>-tr_billno
                                                                 bldat = <lfs_rec_invsl>-tr_bill_dt
                                                                 lifnr = <lfs_rec_invsl>-lifnr
                                                                 BINARY SEARCH.
              IF sy-subrc = 0 AND  gw_rec_invsl_tmp-pro_status <> <lfs_rec_invsl>-pro_status.
                IF ( gw_yinvhead-currproc = '10' OR gw_yinvhead-currproc = '11' ).
                ELSE.
                  MESSAGE 'Scroll is not cancelled'(083) TYPE 'E'.
                  CLEAR:lw_success.
                ENDIF.
              ENDIF.

              UPDATE zsce_rec_invsl   SET pro_status = <lfs_rec_invsl>-pro_status
                                           blk_rsn = <lfs_rec_invsl>-blk_rsn
                                           del_ind = <lfs_rec_invsl>-del_ind
                                           changed_by = sy-uname
                                           changed_on = sy-datum
                                           changed_tm = sy-uzeit
                                       WHERE lifnr      = <lfs_rec_invsl>-lifnr
                                       AND tr_billno  =   <lfs_rec_invsl>-tr_billno
                                       AND tr_bill_dt =   <lfs_rec_invsl>-tr_bill_dt
                                       AND counter    = <lfs_rec_invsl>-counter.
              IF sy-subrc = 0.
                lw_success  = abap_true.
                CLEAR:gt_rec_invsl_tmp.
                gt_rec_invsl_tmp = gt_rec_invsl.


                lw_billno = <lfs_rec_invsl>-tr_billno.
                lw_msg-msgid = 'ZLOG'.
                lw_msg-msgno = '040'.
                lw_msg-msgty = 'S'.

                CONCATENATE  'Reason:'(087)  <lfs_rec_invsl>-blk_rsn INTO lw_msg-msgv1
                                                 SEPARATED BY space.
                CONCATENATE 'Staus'(072) <lfs_rec_invsl>-pro_status INTO  lw_msg-msgv2
                                       SEPARATED BY space.

                CONCATENATE  'Del Ind'(086)  <lfs_rec_invsl>-del_ind INTO  lw_msg-msgv3
                                                   SEPARATED BY space.
                APPEND: lw_msg TO  lt_msg.
                CLEAR:lw_msg,lw_text.

                IF lw_billno IS NOT INITIAL AND lt_msg IS NOT INITIAL.

                  lw_obj    =   'ZLOGRECINV'.
                  lw_subobj =   'ZLOGRECINV_S'.

                  CALL FUNCTION 'Z_CREATE_APPL_LOG'
                    EXPORTING
                      i_log_object            = lw_obj
                      i_extnumber             = lw_billno
                      i_subobject             = lw_subobj
                    TABLES
                      t_log_message           = lt_msg
                    EXCEPTIONS
                      log_header_inconsistent = 1
                      logging_error           = 2
                      OTHERS                  = 3.

                ENDIF.
                CLEAR:lw_billno,lt_msg,lw_obj,lw_subobj.
              ELSE.
                CLEAR:lw_success.
              ENDIF.


            ENDIF.
            CLEAR:gw_rec_invsl_tmp.
          ENDIF.
        ENDLOOP.

        IF  lw_success = abap_true..
          MESSAGE 'Data updated'(081) TYPE 'S'.
        ELSE.
          MESSAGE 'Data not updated'(082) TYPE 'S' DISPLAY LIKE 'E'.
        ENDIF.

      WHEN OTHERS.
    ENDCASE.

    IF  lw_success  = abap_true.
      COMMIT WORK AND WAIT.
    ELSE.
      ROLLBACK WORK.
    ENDIF.

    IF go_ref_alv IS BOUND.
      CALL METHOD go_ref_alv->refresh_table_display( ).
    ENDIF.

  ENDMETHOD.                    "user_command
  METHOD user_command_popup.

    DATA:lt_selected_row    TYPE TABLE OF lvc_s_row,
          lw_selected        TYPE          lvc_s_row,
          lw_success         TYPE char1.

    DATA: lw_obj    TYPE balobj_d,
          lw_billno TYPE string,
          lw_subobj  TYPE balsubobj,
          lt_msg    TYPE STANDARD TABLE OF lmess,
          lw_msg    LIKE LINE OF lt_msg,
          lw_text TYPE char50.

    FIELD-SYMBOLS: <lfs_chrg_alv> TYPE gty_rec_invchrg.
    CLEAR:lt_selected_row.

    CALL METHOD go_popup_alv->get_selected_rows
      IMPORTING
        et_index_rows = lt_selected_row.

    IF lt_selected_row IS INITIAL.
      MESSAGE 'Select atleast one row to process'(080)      "#EC NOTEXT
                 TYPE 'S' DISPLAY LIKE 'E'.
      RETURN.
    ENDIF.

    CASE e_ucomm.
      WHEN 'SAVE_POPUP'.
        LOOP AT lt_selected_row INTO lw_selected.
          READ TABLE gt_chrg_alv ASSIGNING <lfs_chrg_alv>  INDEX lw_selected-index.
          IF sy-subrc = 0.
            READ TABLE gt_chrg_alv_tmp INTO gw_chrg_alv_tmp WITH  KEY lifnr = <lfs_chrg_alv>-lifnr
                                                                      tr_billno = <lfs_chrg_alv>-tr_billno
                                                                      tr_bill_dt = <lfs_chrg_alv>-tr_bill_dt
                                                                      counter = <lfs_chrg_alv>-counter
                                                                      fkpty  = <lfs_chrg_alv>-fkpty
                                                                      kschl = <lfs_chrg_alv>-kschl
                                                                      BINARY SEARCH.
            IF sy-subrc = 0 AND gw_chrg_alv_tmp-del_ind <> <lfs_chrg_alv>-del_ind
                            AND  <lfs_chrg_alv>-tr_billno IS NOT INITIAL
                            AND  <lfs_chrg_alv>-tr_bill_dt IS NOT INITIAL
                            AND  <lfs_chrg_alv>-counter IS NOT INITIAL
                            AND  <lfs_chrg_alv>-fkpty IS NOT INITIAL
                            AND  <lfs_chrg_alv>-kschl IS NOT INITIAL.

              UPDATE zsce_rec_invchrg    SET del_ind =  <lfs_chrg_alv>-del_ind
                                             changed_by = sy-uname
                                             changed_on = sy-datum
                                             changed_tm = sy-uzeit
                                       WHERE lifnr      = <lfs_chrg_alv>-lifnr
                                       AND tr_billno  =  <lfs_chrg_alv>-tr_billno
                                       AND tr_bill_dt =  <lfs_chrg_alv>-tr_bill_dt
                                       AND counter = <lfs_chrg_alv>-counter
                                       AND fkpty = <lfs_chrg_alv>-fkpty
                                       AND kschl = <lfs_chrg_alv>-kschl.
              IF sy-subrc = 0.
                lw_success  = abap_true.
                CLEAR: gt_chrg_alv_tmp.
                gt_chrg_alv_tmp = gt_chrg_alv.

                lw_billno = <lfs_chrg_alv>-tr_billno.

                lw_msg-msgid = 'ZLOG'.
                lw_msg-msgno = '040'.
                lw_msg-msgty = 'S'.

                CONCATENATE  'Invchrg'(085) 'Del Ind'(086)  <lfs_chrg_alv>-del_ind INTO  lw_msg-msgv3
                                                   SEPARATED BY space.
                APPEND: lw_msg TO  lt_msg.
                CLEAR:lw_msg,lw_text.

              ELSE.
                CLEAR:lw_success.
              ENDIF.

            ENDIF.

          ENDIF.
        ENDLOOP.

        IF  lw_success = abap_true.

          IF lw_billno IS NOT INITIAL AND lt_msg IS NOT INITIAL.

            lw_obj    =   'ZLOGRECINV'.
            lw_subobj =   'ZLOGRECINV_S'.

            CALL FUNCTION 'Z_CREATE_APPL_LOG'
              EXPORTING
                i_log_object            = lw_obj
                i_extnumber             = lw_billno
                i_subobject             = lw_subobj
              TABLES
                t_log_message           = lt_msg
              EXCEPTIONS
                log_header_inconsistent = 1
                logging_error           = 2
                OTHERS                  = 3.

          ENDIF.
          CLEAR:lw_billno,lt_msg,lw_obj,lw_subobj.

          MESSAGE 'Data updated'(081) TYPE 'S'.
        ELSE.
          MESSAGE 'Data not updated'(082) TYPE 'S' DISPLAY LIKE 'E'.
        ENDIF.

      WHEN OTHERS.
    ENDCASE.

    IF  lw_success  = abap_true.
      COMMIT WORK AND WAIT.
    ELSE.
      ROLLBACK WORK.
    ENDIF.

    IF go_popup_alv IS BOUND.
      CALL METHOD go_popup_alv->refresh_table_display( ).
    ENDIF.
    IF go_ref_alv IS BOUND.
      CALL METHOD go_ref_alv->refresh_table_display( ).
    ENDIF.

  ENDMETHOD.                    "user_command_popup
  METHOD  handle_hotspot_click.

    DATA:lw_tab TYPE sy-tabix.

    CLEAR:gt_chrg_alv,gt_chrg_alv_tmp,gt_rec_invchrg.

    READ TABLE gt_rec_invsl INTO gw_rec_invsl  INDEX es_row_no-row_id.
    IF sy-subrc = 0.

      SELECT  lifnr
              tr_billno
              tr_bill_dt
              counter
              fkpty
              kschl
              shtyp
              del_ind
              FROM zsce_rec_invchrg
              INTO TABLE gt_rec_invchrg
              WHERE  lifnr = gw_rec_invsl-lifnr
              AND tr_billno = gw_rec_invsl-tr_billno
              AND tr_bill_dt = gw_rec_invsl-tr_bill_dt.
      IF gt_rec_invchrg IS NOT INITIAL.
        SORT gt_rec_invchrg BY lifnr tr_billno tr_bill_dt counter fkpty kschl.
      ENDIF.

    ENDIF.
    gt_chrg_alv = gt_rec_invchrg.
    SORT gt_chrg_alv BY lifnr tr_billno tr_bill_dt counter fkpty kschl.
    gt_chrg_alv_tmp = gt_chrg_alv.
    SORT gt_chrg_alv_tmp BY lifnr tr_billno tr_bill_dt counter fkpty kschl.


    IF gt_chrg_alv IS NOT INITIAL.
      CALL SCREEN 9002 STARTING AT 5 10
           ENDING AT 110 20.
    ENDIF.



  ENDMETHOD.                    "handle_hotspot_click
  METHOD  create_popup_obj.

    IF go_popup_alv IS BOUND.
      go_popup_alv->free( ).
    ENDIF.

    IF go_popup_dock IS BOUND.
      go_popup_dock->free( ).
    ENDIF.

    CREATE OBJECT go_popup_dock
      EXPORTING
        side                        = cl_gui_docking_container=>dock_at_left
        extension                   = 1530
      EXCEPTIONS
        cntl_error                  = 1
        cntl_system_error           = 2
        create_error                = 3
        lifetime_error              = 4
        lifetime_dynpro_dynpro_link = 5
        OTHERS                      = 6.
    IF sy-subrc <> 0.
      MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
                WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
    ENDIF.

    CREATE OBJECT go_popup_doc.

    CREATE OBJECT go_popup_alv
      EXPORTING
        i_parent          = go_popup_dock
      EXCEPTIONS
        error_cntl_create = 1
        error_cntl_init   = 2
        error_cntl_link   = 3
        error_dp_create   = 4
        OTHERS            = 5.
    IF sy-subrc <> 0.
      MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
                 WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
    ENDIF.

  ENDMETHOD.                    "create_popup_obj
  METHOD create_popup_fcat.

    CLEAR:gt_fcatpopup.

    CLEAR:gw_fcatlog.
    gw_fcatpopup-fieldname = 'TR_BILLNO'.
    gw_fcatpopup-col_pos =   '1'.
    gw_fcatpopup-coltext =   'TR Bill No'(068).
    APPEND gw_fcatpopup TO gt_fcatpopup.

    CLEAR:gw_fcatlog.
    gw_fcatpopup-fieldname = 'COUNTER'.
    gw_fcatpopup-col_pos =   '2'.
    gw_fcatpopup-coltext =   'Counter'(077).
    APPEND gw_fcatpopup TO gt_fcatpopup.

    CLEAR:gw_fcatlog.
    gw_fcatpopup-fieldname = 'FKPTY'.
    gw_fcatpopup-col_pos =   '3'.
    gw_fcatpopup-coltext =   'Item Category'(078).
    APPEND gw_fcatpopup TO gt_fcatpopup.

    CLEAR:gw_fcatlog.
    gw_fcatpopup-fieldname = 'KSCHL'.
    gw_fcatpopup-col_pos =   '4'.
    gw_fcatpopup-coltext =   'Charge Code'(079).
    APPEND gw_fcatpopup TO gt_fcatpopup.

    CLEAR:gw_fcatlog.
    gw_fcatpopup-fieldname = 'DEL_IND'.
    gw_fcatpopup-col_pos =   '5'.
    gw_fcatpopup-coltext =   'Deletion Indicator'(076).
    gw_fcatpopup-edit    =   'X'.
    APPEND gw_fcatpopup TO gt_fcatpopup.

  ENDMETHOD.                    "create_popup_fcat
  METHOD display_popup_alv.

    DATA : lw_lvc_lay TYPE lvc_s_layo.
    DATA : lt_sort TYPE STANDARD TABLE OF lvc_s_sort,
           lw_sort TYPE lvc_s_sort,
           lt_grp TYPE TABLE OF lvc_s_sgrp,
           lw_grp TYPE lvc_s_sgrp,
           lw_layout   TYPE lvc_s_layo,
           lw_variant  TYPE disvariant.

    DATA : lt_functions TYPE ui_functions.
    CLEAR: lw_sort.

    DATA : lw_excludebutton TYPE ui_func,
           lt_excludebutton TYPE ui_functions.


    CLEAR: lt_excludebutton.
    "excluding graph anf info button of alv toolbar
    lw_excludebutton = cl_gui_alv_grid=>mc_fc_graph.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_info.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

*      lw_excludebutton = cl_gui_alv_grid=>mc_fc_loc_copy_row.
*      APPEND lw_excludebutton TO lt_excludebutton.
*      CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_loc_cut.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_loc_delete_row.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_loc_insert_row.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.
    lw_excludebutton = cl_gui_alv_grid=>mc_fc_loc_delete_row.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_print.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_save_variant.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_sort_asc.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_sort_dsc.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_loc_append_row.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_loc_undo.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_maintain_variant.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.

    lw_excludebutton = cl_gui_alv_grid=>mc_fc_subtot.
    APPEND lw_excludebutton TO lt_excludebutton.
    CLEAR lw_excludebutton.
*
    SET HANDLER  go_ref->toolbar_popup FOR go_popup_alv.
    SET HANDLER  go_ref->user_command_popup FOR go_popup_alv.
*      "set handler for your alv
*      SET HANDLER go_ref->handle_hotspot_click FOR go_popup_alv.

    lw_layout-cwidth_opt    = abap_true.
    lw_layout-col_opt       = abap_true.


    CALL METHOD go_popup_alv->set_table_for_first_display
      EXPORTING
        is_variant                    = lw_variant
        is_layout                     = lw_layout
        it_toolbar_excluding          = lt_excludebutton "lt_functions
      CHANGING
        it_outtab                     = gt_chrg_alv
        it_fieldcatalog               = gt_fcatpopup
        it_sort                       = lt_sort
      EXCEPTIONS
        invalid_parameter_combination = 1
        program_error                 = 2
        too_many_lines                = 3
        OTHERS                        = 4.
    IF sy-subrc <> 0.
      MESSAGE ID sy-msgid TYPE sy-msgty NUMBER sy-msgno
                 WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4.
    ENDIF.
    CALL METHOD go_popup_alv->register_edit_event
      EXPORTING
        i_event_id = cl_gui_alv_grid=>mc_evt_enter.

  ENDMETHOD.                    "display_popup_alv
  METHOD shortage_field_catalog.
    CLEAR: gw_fcat,gt_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'CHECK_BOX'.
    gw_fcat-coltext = 'Check'.
    gw_fcat-checkbox = 'X'.
    gw_fcat-edit = 'X'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'LIFNR'.
    gw_fcat-coltext = 'Vendor'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'TKNUM'.
    gw_fcat-coltext = 'Shipment'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'VBELN'.
    gw_fcat-coltext = 'Delivery'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'BILL_DOC_NO'.
    gw_fcat-coltext = 'Billing Document'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'CHRG_CODE'.
    gw_fcat-coltext = 'Charge Code'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'RECOVERY_VAL'.
    gw_fcat-coltext = 'Recovery Value'.
    gw_fcat-ref_table = 'ZSCE_SHORTAGE_GT'.
    gw_fcat-ref_field = 'RECOVERY_VAL'.
    gw_fcat-valexi    = 'X'.
    gw_fcat-edit = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'CUST_CREDIT_AMT'.
    gw_fcat-coltext = 'Customer Credit Amount'.
    gw_fcat-ref_table = 'ZSCE_SHORTAGE_GT'.
    gw_fcat-ref_field = 'CUST_CREDIT_AMT'.
    gw_fcat-edit = 'X'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'SHORTAGE_QTY'.
    gw_fcat-coltext = 'Shortage Qty'.
    gw_fcat-ref_table = 'ZSCE_SHORTAGE_GT'.
    gw_fcat-ref_field = 'SHORTAGE_QTY'.
    gw_fcat-edit = 'X'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'ZZSERV_PLANT'.
    gw_fcat-coltext = 'Service Plant'.
    gw_fcat-edit = 'X'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'INVOICE1'.
    gw_fcat-coltext = 'Invoice1'.
    gw_fcat-valexi    = 'X'.
    gw_fcat-edit = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'INVOICE2'.
    gw_fcat-coltext = 'Invoice2'.
    gw_fcat-valexi    = 'X'.
    gw_fcat-edit = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'INVOICE3'.
    gw_fcat-coltext = 'Invoice3'.
    gw_fcat-edit = 'X'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'FI_DOC1'.
    gw_fcat-coltext = 'FI Document1'.
    gw_fcat-edit = 'X'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'FI_DOC2'.
    gw_fcat-coltext = 'FI Document2'.
    gw_fcat-edit = 'X'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'FCPL_BY_RIL'.
    gw_fcat-coltext = 'FCPL by Rail'.
    gw_fcat-edit = 'X'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'UNIQUE_REF_NO'.
    gw_fcat-coltext = 'Unique Ref No'.
    gw_fcat-edit = 'X'.
    gw_fcat-valexi    = 'X'.
    gw_fcat-outputlen = '35'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'UNIQUE_REF_NO2'.
    gw_fcat-coltext = 'Unique Ref No2'.
    gw_fcat-edit = 'X'.
    gw_fcat-valexi    = 'X'.
    gw_fcat-outputlen = '35'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'DELAY_DY'.
    gw_fcat-coltext = 'Delay Days'.
    gw_fcat-edit = 'X'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'MFRGR'.
    gw_fcat-coltext = 'Material Freight Group'.
    gw_fcat-edit = 'X'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'BUSINESS'.
    gw_fcat-coltext = 'Business'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'SUB_BUSINESS'.
    gw_fcat-coltext = 'Sub Business'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.


    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'SUB_FORMAT'.
    gw_fcat-coltext = 'Sub Format'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'SCRTYPE'.
    gw_fcat-coltext = 'Scroll Type'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'CN_DATE'.
    gw_fcat-coltext = 'CN Date'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.
    gw_fcat-fieldname = 'CN_NO'.
    gw_fcat-coltext = 'CN Number'.
    gw_fcat-valexi    = 'X'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

    gw_fcat-tabname = 'GT_SHORTAGE_FINAL'.  " added by arpit RD2K9A3L6S
    gw_fcat-fieldname = 'RO_OUTDATE'.
    gw_fcat-coltext = 'RO_OUTDATE'.
    gw_fcat-edit = 'X'.
    gw_fcat-valexi    = 'X'.
    gw_fcat-f4availabl  = 'X'.
    gw_fcat-ref_table  = 'VBAK'.
    gw_fcat-ref_field  = 'ERDAT'.
    APPEND gw_fcat TO gt_fcat.
    CLEAR gw_fcat.

  ENDMETHOD.                    "shortage_field_catalog
  METHOD shrtage_data_save.
    DATA: lw_dest TYPE char70,
          lw_msg TYPE string,
          lt_shortage TYPE STANDARD TABLE OF gty_shortage_final,
          lw_shortage TYPE gty_shortage_final.

    IF gt_shortage_final IS NOT INITIAL.
      DELETE gt_shortage_final WHERE check_box IS INITIAL.
      REFRESH: lt_shortage.
      LOOP AT gt_shortage_final INTO gw_shortage_final.
        lw_shortage-lifnr               = gw_shortage_final-lifnr.
        lw_shortage-tknum               = gw_shortage_final-tknum.
        lw_shortage-vbeln               = gw_shortage_final-vbeln.
        lw_shortage-bill_doc_no         = gw_shortage_final-bill_doc_no.
        lw_shortage-chrg_code           = gw_shortage_final-chrg_code.
        lw_shortage-shortage_qty        = gw_shortage_final-shortage_qty.
        lw_shortage-shortage_uom        = gw_shortage_final-shortage_uom.
        lw_shortage-recovery_val        = gw_shortage_final-recovery_val.
        lw_shortage-recovery_curr       = gw_shortage_final-recovery_curr.
        lw_shortage-zzserv_plant        = gw_shortage_final-zzserv_plant.
        lw_shortage-cn_date             = gw_shortage_final-cn_date.
        lw_shortage-erdat               = gw_shortage_final-erdat.
        lw_shortage-erzet               = gw_shortage_final-erzet.
        lw_shortage-ernam               = gw_shortage_final-ernam.
        lw_shortage-invoice1            = gw_shortage_final-invoice1.
        lw_shortage-invoice2            = gw_shortage_final-invoice2.
        lw_shortage-invoice3            = gw_shortage_final-invoice3.
        lw_shortage-remarks             = gw_shortage_final-remarks.
        lw_shortage-fi_doc1             = gw_shortage_final-fi_doc1.
        lw_shortage-fcpl_by_ril         = gw_shortage_final-fcpl_by_ril.
        lw_shortage-fi_doc2             = gw_shortage_final-fi_doc2.
        lw_shortage-cust_credit_amt     = gw_shortage_final-cust_credit_amt.
        lw_shortage-unique_ref_no       = gw_shortage_final-unique_ref_no.
        lw_shortage-unique_ref_no2      = gw_shortage_final-unique_ref_no2.
        lw_shortage-ro_outdate          = gw_shortage_final-ro_outdate.
        lw_shortage-gst_inv_no          = gw_shortage_final-gst_inv_no.
        lw_shortage-invoice_dt          = gw_shortage_final-invoice_dt.
        lw_shortage-invoice_amt         = gw_shortage_final-invoice_amt.
        lw_shortage-ship_to_party       = gw_shortage_final-ship_to_party.
        lw_shortage-dms_no              = gw_shortage_final-dms_no.
        lw_shortage-app_cust_amt        = gw_shortage_final-app_cust_amt.
        lw_shortage-scrtype             = gw_shortage_final-scrtype.
        lw_shortage-mfrgr               = gw_shortage_final-mfrgr.
        lw_shortage-delay_dy            = gw_shortage_final-delay_dy.
        lw_shortage-business            = gw_shortage_final-business.
        lw_shortage-sub_business        = gw_shortage_final-sub_business.
        lw_shortage-sub_format          = gw_shortage_final-sub_format.
        lw_shortage-cn_no               = gw_shortage_final-cn_no.
        lw_shortage-cn_date             = gw_shortage_final-cn_date.
        lw_shortage-rate                = gw_shortage_final-rate.
        APPEND lw_shortage TO lt_shortage.
        CLEAR: gw_shortage_final,lw_shortage.
      ENDLOOP.

      IF gt_shortage_final IS NOT INITIAL.
        "Class & method to fetch RFC details for RP5 related server and clients
        CALL METHOD zcl_log_fcpl=>fcpl_rfc_dest
          IMPORTING
            ex_rfc_dest = lw_dest.

        IF lw_dest IS NOT INITIAL.

          CALL FUNCTION 'Z_SCE_SHORTAGE_UPD' DESTINATION lw_dest
            EXPORTING
              it_shortage = lt_shortage
            IMPORTING
              e_msg       = lw_msg.
        ENDIF.
      ENDIF.
      CLEAR:lt_shortage.
    ENDIF.
  ENDMETHOD.                    "SHRTAGE_DATA_SAVE
  METHOD vehicle_save.
    LOOP AT gt_vcdet INTO gw_vcdet WHERE check_box = 'X'.
      UPDATE yttstm0001 SET tdvehicleno = gw_vcdet-vehicle
                        WHERE truck_no = gw_vcdet-truck_no.
      IF sy-subrc = 0.
        COMMIT WORK.
        gw_flag_s = abap_true.
      ENDIF.
    ENDLOOP.

  ENDMETHOD.                    "VEHICLE_SAVE
ENDCLASS.                    "lcl_data_selection IMPLEMENTATION
*&---------------------------------------------------------------------*
*&      Module  STATUS_9001  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE status_9001 OUTPUT.

  SET PF-STATUS 'PF_9001'.

  go_ref->icon_exclude( ).
  IF p_st IS NOT INITIAL.
    go_ref->shortage_field_catalog( ).
  ELSE.
    go_ref->fill_field_catalog( ).
  ENDIF.

  IF p_rd EQ 'X'.
    SET TITLEBAR 'T1_ROAD'.

**    go_ref->disp_alv(
**                     EXPORTING i_cont = gc_cont
**                     CHANGING it_table = gt_trstlmnt_rd ).

    go_ref->disp_alv(
                 EXPORTING i_cont = gc_cont
                 CHANGING it_table = gt_trstlmnt_rd_chk ).

  ELSEIF p_rl EQ 'X'.
    SET TITLEBAR 'T1_RAIL'.
    go_ref->disp_alv(
                    EXPORTING i_cont = gc_cont
                    CHANGING it_table = gt_trstlmnt_rl ).
  ELSEIF p_ma EQ 'X'.
    SET TITLEBAR 'T1_MARINE_AIR'.
    go_ref->disp_alv(
                    EXPORTING i_cont = gc_cont
                    CHANGING it_table = gt_trstlmnt_ma ).
* BOC by MSAT_050 on 03.01.2018 12:01:28
  ELSEIF p_sp EQ 'X'.
    SET TITLEBAR 'T1_SERVICE_PLANT_UPD'.
    go_ref->disp_alv(
                    EXPORTING i_cont = gc_cont
                    CHANGING it_table = gt_shipment ).
* EOC by MSAT_050 on 03.01.2018 12:01:28
  ELSEIF p_st EQ 'X'.
    go_ref->disp_alv(
                     EXPORTING i_cont = gc_cont
                     CHANGING it_table = gt_shortage_final ).
*********************************************   BOC BY TRILOK 18.11.2022
  ELSEIF p_vc EQ 'X' .
    SET TITLEBAR 'VEHICLE DETAILS'.
    go_ref->disp_alv(
                    EXPORTING i_cont = gc_cont
                    CHANGING it_table = gt_vcdet ).
***********************************************   EOC BY TRILOK 18.11.2022
  ENDIF.

ENDMODULE.                 " STATUS_9001  OUTPUT
*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_9001  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE user_command_9001 INPUT.

  CASE sy-ucomm.

    WHEN 'BACK' OR 'CANCEL'.
      IF p_mc IS INITIAL. "Added By kalpesh/Akshay 17.11.2025 RD2K9A5E56
        CALL METHOD go_grid->refresh_table_display.

        REFRESH: gt_trstlmnt_ma, gt_trstlmnt_rd, gt_trstlmnt_rl.
      ENDIF. "Added By kalpesh/Akshay 17.11.2025 RD2K9A5E56

      LEAVE TO SCREEN 0.
    WHEN 'EXIT'.
      LEAVE PROGRAM.
    WHEN 'SAVE'.
      go_ref->alv_data_save( ).
    WHEN 'DELETE'.

    WHEN OTHERS.
      LEAVE PROGRAM.

  ENDCASE.

ENDMODULE.                 " USER_COMMAND_9001  INPUT
*&---------------------------------------------------------------------*
*&      Form  FILL_FCAT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
*      -->P_TEXT_003  text
*      -->P_0283   text
*      -->P_TEXT_017  text
*----------------------------------------------------------------------*
FORM fill_fcat  USING    lp1 TYPE any
                         lp2 TYPE any
                         lp3 TYPE any.

  CLEAR: gw_fcat.

  gw_fcat-fieldname = lp1.
  gw_fcat-key       = lp2.

  IF p_rd = 'X'.
    gw_fcat-tabname   =  gc_gt_trstlmnt_rd. " 'GT_TRSTLMNT_RD'.
  ELSEIF p_rl = 'X'.
    gw_fcat-tabname   =  gc_gt_trstlmnt_rl. " 'GT_TRSTLMNT_RL'.
  ELSEIF p_ma = 'X'.
    gw_fcat-tabname   =  gc_gt_trstlmnt_ma. " 'GT_TRSTLMNT_MA'.
  ENDIF.

  CASE gw_fcat-fieldname.
    WHEN 'CHECK_BOX'.
      gw_fcat-edit = 'X'.
      gw_fcat-checkbox = 'X'.
    WHEN 'TKNUM' OR 'VBELN' .
      gw_fcat-edit  = space.
      gw_fcat-no_zero = 'X'.
    WHEN 'TR_BILL_DT'.
      IF gw_chk = 'X'.
        gw_fcat-edit = 'X'.
      ENDIF.
      gw_fcat-ref_field = 'TR_BILL_DT'.
      gw_fcat-f4availabl = 'X'.
      gw_fcat-ref_table = gc_ztrstlmnt.
    WHEN 'TR_BILL_AMT'.
      IF gw_chk = 'X'.
        gw_fcat-edit = 'X'.
      ENDIF.
      gw_fcat-ref_field = 'TR_BILL_AMT'.  " For Quanity decimal
      gw_fcat-ref_table = gc_ztrstlmnt.
      CLEAR gw_chk.
*      gw_fcat-rollname =  'ZLR_DT_STL'.
    WHEN 'LR_DT'.
      gw_fcat-ref_table = gc_ztrstlmnt. " 'ZTRSTLMNT'.
      gw_fcat-ref_field = 'LR_DT'.
      gw_fcat-rollname =  'ZLR_DT_STL'.
      gw_fcat-f4availabl = 'X'.
      gw_fcat-edit  = 'X'.
    WHEN  'GRN_QTY'.
      gw_fcat-ref_table = gc_ztrstlmnt. " 'ZTRSTLMNT'.
      gw_fcat-ref_field = 'GRN_QTY'.  " For Quanity decimal
      gw_fcat-edit  = 'X'.
    WHEN 'ROUTID'.
      gw_fcat-edit  = space.
    WHEN 'FLT_IND' OR 'DIGIND'.
      gw_fcat-checkbox = 'X'.
      gw_fcat-edit  = 'X'.
    WHEN 'GRN_NO' OR 'BILL_VENDOR'.
      gw_fcat-edit  = 'X'.
      gw_fcat-no_zero = 'X'.
    WHEN 'TRANS_LIBTY_AMT'.
      gw_fcat-ref_table = gc_ztrstlmnt. " 'ZTRSTLMNT'.
      gw_fcat-ref_field = 'TRANS_LIBTY_AMT'.
      gw_fcat-edit  = 'X'.
    WHEN 'TRANS_LIBTY_CUR'.
      gw_fcat-edit  = space.
    WHEN 'BUKRS'.
      gw_fcat-edit  = space.
      gw_fcat-no_zero = 'X'.
      gw_fcat-edit = 'X'.
    WHEN 'LR_QTY'.
      gw_fcat-ref_table = gc_ztrstlmnt. " 'ZTRSTLMNT'.
      gw_fcat-ref_field = ''.
      gw_fcat-edit  = 'X'.
    WHEN 'LR_UOM'.
      gw_fcat-ref_table = gc_ztrstlmnt. " 'ZTRSTLMNT'.
      gw_fcat-ref_field = ''.
      gw_fcat-edit  = 'X'.
    WHEN 'DEP_POINT'.
      gw_fcat-edit  = 'X'.
    WHEN 'DES_POINT'.
      gw_fcat-edit  = 'X'.
    WHEN 'PRO_STATUS'.
      gw_fcat-edit  = 'X'.      "trilok 19.01.2023
    WHEN 'FKNUM'.
      gw_fcat-edit  = 'X'.      "trilok 31.01.2023
    WHEN 'MFRGR'.
      gw_fcat-edit  = 'X'.
    WHEN 'HEADRUN_AMT' .          "+TRILOK 10.11.2022   RD2K9A417Y
      gw_fcat-ref_table = gc_ztrstlmnt.
      gw_fcat-ref_field = 'HEADRUN_AMT'.
      gw_fcat-edit  = 'X'.
    WHEN OTHERS.
      gw_fcat-edit  = 'X'.
  ENDCASE.

  IF p_sp IS NOT INITIAL.
    gw_fcat-edit  = ''.
  ENDIF.
  gw_fcat-coltext   = lp3.  " Column heading
  gw_fcat-valexi    = 'X'.  " Existence of fixed values
  APPEND gw_fcat TO gt_fcat.

ENDFORM.                    " FILL_FCAT
*&---------------------------------------------------------------------*
*&      Module  STATUS_9000  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE status_9000 OUTPUT.
  SET PF-STATUS '9000_PF'.
  SET TITLEBAR '9000_TITLE'.

ENDMODULE.                 " STATUS_9000  OUTPUT
*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_9000  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE user_command_9000 INPUT.

  CASE sy-ucomm.
    WHEN 'BACK'.
      LEAVE TO SCREEN 0.
    WHEN 'EXIT' OR 'CANCEL'.
      LEAVE PROGRAM.
  ENDCASE.
ENDMODULE.                 " USER_COMMAND_9000  INPUT
*&---------------------------------------------------------------------*
*&      Module  DATA_DISPLAY  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE data_display OUTPUT.

  CALL METHOD go_ref->create_obj.
  CALL METHOD go_ref->create_fcat.
  CALL METHOD go_ref->display_alv.

ENDMODULE.                 " DATA_DISPLAY  OUTPUT
*&---------------------------------------------------------------------*
*&      Module  STATUS_9002  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE status_9002 OUTPUT.

  SET PF-STATUS '9002_PF'.
  SET TITLEBAR '9002_TITLE'.

ENDMODULE.                 " STATUS_9002  OUTPUT
*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_9002  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE user_command_9002 INPUT.

  CASE sy-ucomm.
    WHEN 'BACK'.
      LEAVE TO SCREEN 0.
    WHEN 'EXIT' OR 'CANCEL'.
*      LEAVE PROGRAM.
      LEAVE TO SCREEN  0.
  ENDCASE.

ENDMODULE.                 " USER_COMMAND_9002  INPUT
*&---------------------------------------------------------------------*
*&      Module  POPUP_DISPLAY  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE popup_display OUTPUT.

  CALL METHOD go_ref->create_popup_obj.
  CALL METHOD go_ref->create_popup_fcat.
  CALL METHOD go_ref->display_popup_alv.

ENDMODULE.                 " POPUP_DISPLAY  OUTPUT

************BOc Added by Hiral on 21.06.2022*****
CLASS lcl_gta DEFINITION .

  PUBLIC SECTION .

    METHODS : get_data,
              field_cat,
              display .

    METHODS : check_changed_data FOR EVENT data_changed OF cl_gui_alv_grid.

*              IMPORTING  e_ucomm
*                         er_data_changed.
    .
*    METHODS : get_selected_rows FOR EVENT DATA_CHANGED OF cl_gui_alv_grid.
    METHODS :  save_data FOR EVENT data_changed OF cl_gui_alv_grid

    IMPORTING er_data_changed
              e_ucomm.



ENDCLASS.                    "lcl_gta DEFINITION

*----------------------------------------------------------------------*
*       CLASS lcl_gta IMPLEMENTATION
*----------------------------------------------------------------------*
*
*----------------------------------------------------------------------*
CLASS lcl_gta IMPLEMENTATION .

  METHOD get_data.

    DATA: lw_tknum5 LIKE LINE OF s_tknum5.


    IF s_tknum5 IS NOT INITIAL.

      LOOP AT s_tknum5 INTO lw_tknum5.
        CLEAR: lw_tknum.
        lw_tknum-tknum = lw_tknum5-low.
        APPEND lw_tknum TO lt_tknum.
        CLEAR: lw_tknum,lw_tknum5.
      ENDLOOP.

    ENDIF.


    CALL FUNCTION 'Z_SCE_SETTLEMENT_GTA_DET' "DESTINATION lw_dest
      EXPORTING
        it_shipmentno = lt_tknum
      IMPORTING
        et_gta_det    = lt_zgstgtainb
        et_return     = lt_return.

    IF lt_return IS NOT INITIAL.
      CLEAR: lw_return.
      LOOP AT lt_return INTO lw_return WHERE type = 'E'.
        MESSAGE lw_return-message TYPE 'W'.
        CLEAR: lw_return.
      ENDLOOP.
    ENDIF.


    IF lt_zgstgtainb IS NOT INITIAL.
      REFRESH: lt_final.
      CLEAR: lw_zgstgtainb.
      LOOP AT  lt_zgstgtainb INTO lw_zgstgtainb.
        MOVE-CORRESPONDING  lw_zgstgtainb TO lw_final.
        APPEND lw_final TO lt_final.
        CLEAR:lw_final.
        CLEAR: lw_zgstgtainb.
      ENDLOOP.
      go_object->field_cat( ).
    ELSE.
      MESSAGE text-034 TYPE 'E' DISPLAY LIKE 'I'.
    ENDIF.

  ENDMETHOD.                    "get_data

  METHOD field_cat.

    REFRESH: lt_fieldcat.

    lt_layout-zebra = 'X'.
    lt_layout-cwidth_opt = 'X'.
    REFRESH: lt_fieldcat.

    DATA : n TYPE i.
    n = n + 1.
    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n.
    lw_fieldcat-fieldname     = 'CHECK' .
    lw_fieldcat-tabname       = 'LT_FINAL'.
    lw_fieldcat-no_zero     = 'X'.
    lw_fieldcat-checkbox     = 'X'.
    IF lv_flag = abap_true.
      lw_fieldcat-edit     = ''.
    ELSE.
      lw_fieldcat-edit     = 'X'.
    ENDIF.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'TKNUM' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-017.
    lw_fieldcat-no_zero     = 'X'.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'FREIGHT_TYPE' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-088.
    lw_fieldcat-no_zero     = 'X'.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'VBELN' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-015.
    lw_fieldcat-no_zero     = 'X'.
    lw_fieldcat-edit        = 'X'.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'EBELN' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-089.
*    lw_fieldcat-outputlen     = '12'.
    lw_fieldcat-no_zero     = 'X'.
    lw_fieldcat-edit        = 'X'.
    APPEND lw_fieldcat TO lt_fieldcat.


    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'LR_NO' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-019.
    lw_fieldcat-no_zero     = 'X'.

    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'LR_DT' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-020.
    lw_fieldcat-no_zero     = 'X'.
    APPEND lw_fieldcat TO lt_fieldcat.


    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'MFRGR' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-090.
    lw_fieldcat-no_zero     = 'X'.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'SHCD' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-091.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'BUKRS' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-092.
    IF lv_flag = abap_true.
      lw_fieldcat-edit     = ''.
    ELSE.
      lw_fieldcat-edit     = 'X'.
    ENDIF.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'WERKS_S' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-093.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'WERKS_R' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-094.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'WERKS_GTA' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-095.
    lw_fieldcat-edit          = 'X'.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'GTA_TYP' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-edit          = 'X'.          " Sagar
    lw_fieldcat-coltext       = text-101.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'LIFNR' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-097.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'FRT_VAL' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-096.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'CREATED_BY' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-098.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'CREATED_ON' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-099.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'SERVER' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-100.
    APPEND lw_fieldcat TO lt_fieldcat.

    CLEAR lw_fieldcat.
    lw_fieldcat-col_pos       = n .
    lw_fieldcat-fieldname     = 'SEGMENT' .
    lw_fieldcat-tabname       = 'lt_final'.
    lw_fieldcat-coltext       = text-102.
    APPEND lw_fieldcat TO lt_fieldcat.

    go_object->display( ).
  ENDMETHOD.                    "field_cat

  METHOD display.
    CREATE OBJECT go_cust
      EXPORTING
        container_name = 'CONT'.


    CREATE OBJECT go_gta
      EXPORTING
        i_parent = go_cust.



    CALL METHOD go_gta->set_table_for_first_display
      EXPORTING
        i_save                        = 'A'
        i_default                     = 'X'
        is_layout                     = lt_layout
      CHANGING
        it_outtab                     = lt_final[]
        it_fieldcatalog               = lt_fieldcat
      EXCEPTIONS
        invalid_parameter_combination = 1
        program_error                 = 2
        too_many_lines                = 3
        OTHERS                        = 4.

    IF go_cust IS NOT INITIAL.

      CALL METHOD go_gta->register_edit_event
        EXPORTING
          i_event_id = cl_gui_alv_grid=>mc_evt_modified
        EXCEPTIONS
          error      = 1
          OTHERS     = 2.

      lw_stable-row = abap_true.
      lw_stable-col = abap_true.



      CALL METHOD go_gta->refresh_table_display
        EXPORTING
          is_stable = lw_stable
        EXCEPTIONS
          finished  = 1
          OTHERS    = 2.
      IF sy-subrc <> 0.
*             Implement suitable error handling here
      ENDIF.
    ENDIF.






  ENDMETHOD.                    "display

  METHOD : check_changed_data.
    CLEAR: lv_flag.
    REFRESH: lt_zgstgtainb1.
    CLEAR: lw_final,lw_zgstgtainb.
    LOOP AT lt_final INTO lw_final WHERE check = abap_true.
      MOVE-CORRESPONDING lw_final TO lw_zgstgtainb.
      APPEND lw_zgstgtainb TO lt_zgstgtainb1.
      CLEAR: lw_final,lw_zgstgtainb.
    ENDLOOP.


    IF lt_zgstgtainb1 IS NOT INITIAL.

      REFRESH: lt_return1,lt_message.
      CALL FUNCTION 'Z_SCE_SETTLEMENT_GTA_UPDATE' "DESTINATION lw_dest
        EXPORTING
          it_gta_det = lt_zgstgtainb1
        IMPORTING
          et_return  = lt_return1.

      IF lt_return1 IS NOT INITIAL.

        CLEAR: lw_return,lw_message.
        LOOP AT lt_return1 INTO lw_return.

          IF lw_return-type = 'E'.
            lw_message-icon = icon_message_error.
          ELSEIF lw_return-type = 'S'.
            lw_message-icon = icon_okay.
          ENDIF.

          lw_message-message = lw_return-message.
          APPEND lw_message TO lt_message.

          CLEAR: lw_return,lw_message.

        ENDLOOP.

        IF lt_message IS NOT INITIAL.
          lv_flag = abap_true.
          cl_salv_table=>factory( IMPORTING r_salv_table = lt_display CHANGING t_table = lt_message ).
          lt_display->display( ).
        ENDIF.
      ENDIF.

    ENDIF.



  ENDMETHOD.                    ":
  METHOD : save_data.

  ENDMETHOD.                    ":

ENDCLASS.                    "lcl_gta IMPLEMENTATION
*&---------------------------------------------------------------------*
*&      Module  STATUS_9003  OUTPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE status_9003 OUTPUT.

  SET PF-STATUS '9003'.
  SET TITLEBAR '9003'.
  CREATE OBJECT go_object .
  go_object->get_data( ).

ENDMODULE.                 " STATUS_9003  OUTPUT

*&---------------------------------------------------------------------*
*&      Module  USER_COMMAND_9003  INPUT
*&---------------------------------------------------------------------*
*       text
*----------------------------------------------------------------------*
MODULE user_command_9003 INPUT.
  CASE sy-ucomm.
    WHEN 'BACK' .
      LEAVE TO SCREEN 0.

    WHEN 'EXIT' OR 'CANCEL'.
      LEAVE PROGRAM.

    WHEN 'SAVE'.
      go_object->check_changed_data( ).




  ENDCASE.
ENDMODULE.                 " USER_COMMAND_9003  INPUT

************EOc Added by Hiral on 21.06.2022*****
