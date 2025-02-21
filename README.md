# LearnAlgorithm
这是一个仅仅用于算法学习的仓库，您可以通过约定好的格式对您的算法和数据结构进行测试，以便于您进行算法的学习
## 使用语言
本仓库使用C/C++语言来进行算法的测试，并使用cmake来构建项目。但您无需关注cmake的具体细节，只需要按照我们约定好的格式来书写和提交代码. C++的版本请使用C++17及以上版本
## 目录结构
目录分为 include 和 src 来分别存放您所写的算法或者数据结构的头文件（.h）以及源文件（.c/.cpp）文件。src中将存放对应的算法或数据结构的文件，请妥善安置他们
在test文件中存放您的测试文件
最终的测试将在最外层的mian.cpp中进行
### 关于头文件和源文件
在头文件也就是.h文件中，以注释的形式注明函数签名，即参数的含义已经返回值的含义
对于.cpp文件 请将您想练习的算法写入对应的文件夹中，对于每一个算法或者数据结构都要以md为格式给出详细的文档，也即您对这个算法和数据结构的理解以及您的参考资料。作为学习资料请尽可能全面和准确

您可以写任意的算法和数据结构，包括各种刷题网站上的题目您都可以拿来操作并添加到这个仓库中，只是需要您补足文档。
文档内容包括但不仅限于（并不需要全都包括，只要可以表达清楚）：
* 问题的出处
* 问题的解决思路
* 算法的实现细节
* 算法的分析
* 参考资料

### 关于测试文件
测试文件为.cpp文件不需要在include中添加头文件，同时测试用例要尽可能全面并给出注释





With main As ( 
  SELECT
    CTM.main_id
    , CTM.task_no
    , CTM.main_task_flg
    , CTM.business_cd
    , MAX(task_sub_no) AS task_sub_no 
  FROM
    tb_wlt_create_task_mng CTM 
  WHERE
    CTM.main_id = '0000151573' 
  GROUP BY
    CTM.main_id
    , CTM.task_no
    , business_cd
    , CTM.main_task_flg
) 
, CTM As ( 
  Select
    CTM.task_no
    , CTM.task_sub_no
    , CTM.title
    , CTM.delivery_id
    , CTM.working_time
    , CTM.delivery_unit
    , CTM.business_cd
    , CTM.end_date
    , CTM.effectivedate
    , CTM.delivery_date
    , CTM.main_task_flg 
  From
    tb_wlt_create_task_mng CTM Join main 
      On main.task_no = CTM.task_no 
      And CTM.task_sub_no = main.task_sub_no
) 
, typeDiv As ( 
  SELECT
    CTT.task_no
    , string_agg(tcm.code_name, '/') typeDivCd 
  FROM
    tb_wlt_create_task_type CTT Join CTM 
      On CTM.task_no = CTT.task_no Join tb_wlt_table_code_mst tcm 
      On CTT.type = tcm.code 
      AND tcm.category_cd = '320' 
  GROUP BY
    CTT.task_no
) 
, Delivery As ( 
  SELECT
    DTD.delivery_id
    , Case CTM.delivery_unit 
      When '00' Then shopcd 
      When '01' Then mail_to 
      End delivery_to 
  FROM
    tb_wlt_delivery_task_def DTD Join CTM 
      On CTM.delivery_id = DTD.delivery_id 
      And DTD.mail_type = 'To' 
  Group by
    DTD.delivery_id
    , Case CTM.delivery_unit 
      When '00' Then shopcd 
      When '01' Then mail_to 
      End
) 
SELECT
  --  json_agg(
  --    json_build_array(
  task_no2
  , code
  , code_name
  , delivery_date
  , end_date
  , working_time
  , title
  , business_cd
  , send_to
  , delivery_unit
  , code_name2
  , typeDivCd
  , department_nm
  , end_date2
  , effectivedate2
  , delivery_date2
  , repeat_flg
  , management_flg
  , main_task_flg
  , task_no                                       --    )
  --  )
FROM
  ( 
    SELECT DISTINCT
      replace (CTM.task_no, 'WORK', '0000') task_no2
      , statusTbl.code
      , statusTbl.code_name
      , TO_CHAR(CTM.delivery_date, 'yyyy/mm/dd hh24:mi') delivery_date
      , TO_CHAR(CTM.end_date, 'yyyy/mm/dd hh24:mi') end_date
      , CTM.working_time
      , replace (CTM.title, '<', '&lt;') title
      , CTM.business_cd
      , CASE CTM.delivery_unit 
        WHEN '01' THEN ( 
          Select
            staff_nm 
          From
            tb_wlt_mst_staff_alldata 
          Where
            Delivery.delivery_to = crew_cd 
          Limit
            1
        ) 
        WHEN '00' THEN ( 
          Select
            shop_nm 
          From
            tb_wlt_mst_shop 
          Where
            Delivery.delivery_to = shop_cd 
          Limit
            1
        ) 
        ELSE '' 
        END send_to
      , CTM.delivery_unit
      , duCdMst.code_name code_name2
      , typeDiv.typeDivCd
      , CASE 
      WHEN STAFF1.DEPARTMENT_NM IS NOT NULL AND STAFF1.DEPARTMENT_NM != '' 
      THEN STAFF1.DEPARTMENT_NM 
      WHEN STAFF2.DEPARTMENT_NM IS NOT NULL AND STAFF2.DEPARTMENT_NM != '' 
      THEN STAFF2.DEPARTMENT_NM 
        END AS department_nm
      , TO_CHAR(CTM.end_date, 'yyyy/mm/dd hh24:mi:ss') end_date2
      , TO_CHAR(CTM.effectivedate, 'yyyy/mm/dd hh24:mi:ss') effectivedate2
      , TO_CHAR(CTM.delivery_date, 'yyyy/mm/dd hh24:mi:ss') delivery_date2
      , twtlc.repeat_flg
      , CTA.management_flg
      , CTM.main_task_flg
      , CTM.task_no 
    FROM
      CTM Join tb_wlt_task_loop_condition twtlc ON CTM.task_no = twtlc.task_no 
		Join tb_wlt_create_task_admit CTA ON CTM.task_no = CTA.task_no And CTA.admit_status = '020' 
      LEFT JOIN TB_WLT_MST_STAFF_ALLDATA AS STAFF1 
        ON CTA.apply_user = STAFF1.CREW_CD 
        AND STAFF1.SHOP_TYPE = 'm' 
        AND CASE 
          WHEN CTA.management_flg = '1' 
          THEN True 
          ELSE STAFF1.auth IN ('020') 
          END 
      LEFT JOIN TB_WLT_MST_STAFF_ALLDATA AS STAFF2 
        ON CTA.apply_user = STAFF2.CREW_CD 
        AND ( 
          STAFF2.SHOP_TYPE IS NULL 
          OR STAFF2.SHOP_TYPE != 'm'
        ) 
        AND CASE 
          WHEN CTA.management_flg = '1' 
          THEN True 
          ELSE STAFF2.auth IN ('020') 
          END 
      Left Join typeDiv 
        ON typeDiv.task_no = CTM.task_no 
      Left Join Delivery 
        On CTM.delivery_id = Delivery.delivery_id Join tb_wlt_table_code_mst statusTbl 
        On statusTbl.category_cd = '130' 
        And CTA.admit_status = statusTbl.code 
      Left Join tb_wlt_table_code_mst duCdMst 
        On duCdMst.category_cd = '140' 
        And CTM.delivery_unit = duCdMst.code 
      Left Join tb_wlt_delivery_task_def DTD 
        ON Delivery.delivery_id = DTD.delivery_id 
        And DTD.mail_type = 'To' 
      Left Join tb_wlt_mst_admit_mng MAM 
        On MAM.admit_cd = '3019990002' 
        And CTA.apply_user = MAM.crew_cd 
      Left Join tb_wlt_mst_auth twma 
        On twma.conf_type = '03' 
        And twma.scr_cont1 = '1' 
    WHERE
      Exists ( 
        SELECT
          1 
        FROM
          tb_wlt_mst_staff_alldata 
        WHERE
          CTA.apply_user = crew_cd
      ) 
      AND Case 
        When CTA.management_flg = '1' 
        Then Exists ( 
          SELECT
            1 
          FROM
            tb_wlt_mst_staff_alldata 
          WHERE
            '3019990002' in ( 
              management_crew_cd_1
              , management_crew_cd_2
              , management_crew_cd_3
              , management_crew_cd_4
              , management_crew_cd_5
            )                                     
		And employment_type in ('4', '5')
            And DTD.mail_to = crew_cd
        ) 
        Else Exists ( 
          SELECT
            1 
          FROM
            tb_wlt_mst_staff_alldata STAFF 
          WHERE
            STAFF.crew_cd = MAM.admit_cd 
            AND STAFF.auth = twma.auth_cd
        ) 
        End
  ) RT

