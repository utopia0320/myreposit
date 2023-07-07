LMS 발송 문서
================

## 프로젝트 접속 정보

 1. 위버 접속단말기(가이온과 인포빕 접근을 위한 원격접속)

 
 > 서버 IP : 210.16.195.67  (mstsc)  
 > ID : uber  
 > pw : Uber0815!  


2. 가이온 : 발신내용과 수신타겟정보 파일을 지정된 경로 디렉토리안에 넣어줍니다.(원격서버를 통해 접근)


 > 원격 서버 IP : 183.110.213.2  (mstsc / admin)   
 > ID : administrator    
 > PW : Uber51##    


3. 인포빕 : 현재 대용량 LMS발송을 담당합니다.(DBMS,SSH,SFTP이용하여 접근)


 > DB : MySQL 5.7.40 ver.     
 >> 서버 IP : 183.110.213.3    
 >> PORT : 3306  
 >> ID  : uber-asp  
 >> PW  : ehowlRjqeorl#ekfrEhdwlq2@aodnstnseorhqckd5
> >  
 > sftp : filezila 3.63.2.1 ver. 
 >> 호스트 : 183.110.213.3  
 >> 포트 : 1022   
 >> ID : dev001
 >> PW : uber0815!
> >
 >ssh
 >>ID: dev001
 >>PW : uber0815!

 ```
   su - root
   암호 : Uber0815!
 ```


4. 휴머스온 : 대용량 LMS 이외의 발송을 담당하며, LMS 발송 통계 데이터를 보내줍니다. 따라서 LMS발송 완료 후 이 DB에 발송내역과 타깃정보 데이터를 동일하게 넣어줍니다. 


 > DB : MySQL 5.7.40 ver.    
 > 서버 IP : 183.110.213.2  
 > ID : uber-asp  
 > PW : ehowlRjqeorl#ekfrEhdwlq2@aodnstnseorhqckd5  
 > 포트 : 3306 


--------


## 발송 FLOW


평균 월 2~3회 대용량 LMS 발송건이 있습니다. 
1. 월 차종별 구매독려(15~16일)
2. 월 초 판매조건 (1~3일)

 ### - 발송순서(실발송기준)
### 1. 기획팀에서 EXCEL 파일로 LMS내용을 발송해 줍니다.
 ![image](https://github.com/utopia0320/myreposit/assets/121841502/e8c5c62d-7b7c-4935-9d7c-111ce237d13d)


### 2. 이 EXCEL 파일을 YYMMDD_01.TXT 형태로 각각 저장하여,SFTP를 통해 인포빕 서버(183.110.213.3) /var/www/ems_sender/template/camp 경로 안에 넣어줍니다.

  ![image](https://github.com/utopia0320/myreposit/assets/121841502/6e65df03-72ba-4c1d-a7dd-ffd9bbb341bc).

### 3. 가이온서버(183.110.213.2)에 들어와 있는 발송 및 타겟 데이터 파일을 SFTP로 인포빕 서버(183.110.213.3)에 옮긴 후 타깃파일을 적재시켜줍니다.
  
  ![image](https://github.com/utopia0320/myreposit/assets/121841502/de991232-e991-43af-9104-74a529b728c7)


  만약 시간이 지나서 대상 파일이 없으면 D:\uber\sftp\file\bakup 에 해당 파일이 있습니다.   
  또한 권한문제로 SFTP로 파일 전송이 안될 시 SFTP로  /var/www/temp/chevy/LMS에 파일을 올려놓은 후 SSH 를 이용하여 파일을 옮깁니다.    
  아래와 같이 명령어를 순차 실행합니다.  


   ```

     dev001  / uber0815!
     su - root / Uber0815!

    #cp -r /var/www/temp/chevy/LMS/* /var/www/file/load
    #cd /var/www/file/load
    #chown -R dev001:dev001 ./*
    #chmod  -R 755 ./*

   ```
    
   
### 4. 인포빕DB(183.110.213.3)에 발송 및 타겟 데이터를 insert 해줍니다.

  발송용 쿼리예시       
  이 때 POST_ID는 값으로 날짜+순차 증가 규칙으로 등록합니다. '20%' 형식이 아니면 TMS오류가 발생합니다.


```
    INSERT INTO tms.tms_camp_chn_info (
    CHANNEL_TYPE,POST_ID,P_POST_ID,MSG_ID,APP_GRP_ID,CHANNEL_MSG_NAME,TEMPLATE_WIDTH,CONTENT_HTML,CONTENT_TEXT,CONTENT_TYPE,
    JOB_STATUS,SUBJECT,PUSH_TYPE,MSG_TYPE,PUSH_TITLE,PUSH_MSG,PUSH_KEY,PUSH_VALUE,PUSH_IMG,PUSH_TTL,
    FROM_NAME,FROM_EMAIL,FROM_NUMBER,RETURN_PATH,OPEN_CHECK,CLICK_CHECK,QUE_CLOSE_DATE,TRACKING_CLOSE,NLS_LANG,SMS_TYPE,
    SMART_HTML,FILTER_USE_YN,REDENY_FLAG,MAIL_KIND,AUTO_USE,SAFEMAIL_YN,SMS_FLAG,DIVIDE_SEND_USE_YN,DIVIDE_CNT,DIVIDE_MINUTE,
    REG_ID,DEL_YN,REG_DATE,UPT_DATE )
    values
    ('SM','202306090001',NULL,'202306090001',0,'20230602_6월 판매조건 캠페인_LMS',NULL,'/var/www/ems_sender/template/camp/230602_01.txt',NULL,NULL,'--',
    '(광고)[쉐보레] TRAX CROSSOVER 출시! 쉐보레 6월 스페셜 프로모션',NULL,NULL,NULL,NULL,NULL,NULL,NULL,1800,'한국지엠',NULL,'0803000500',
    'gmaster-return@theuber.co.kr','A','Y',NULL,NULL,'UTF-8','LMS',NULL,'N',NULL,NULL,NULL,'N',NULL,'N',-1,-1,'tms','N',sysdate(),NULL);  
```

   
   발송내용 및 발송타겟 매핑 쿼리 예시    
     CAMP_ID 는 tms_camp_chn_info 의 POST_ID와 동일하게 넣어줍니다.    
     기획팀에서 발송타겟 정보를 메일로 보내주는 데 그 안에 CAMP_DESC는 COMMUNICATION_ID를, CAMP_NAME은 타겟캠페인명을 적어주면 됩니다 ! (이때 차량 모델명과 잘 비교하여 값을 넣어 줍니다 )

   
```
   INSERT into tms_camp_info(CAMP_ID,SITE_ID,CAMP_NAME,CAMP_DESC,START_DATE,END_DATE,CHANNEL_FLAG,REG_ID,DEPT_ID,CAMP_TYPE,REG_DATE,UPT_DATE,DEL_YN)
   VALUES('202306090001',6,'20230602_6월 판매조건 캠페인_LMS','9c73bb092c35','2023-06-02 00:00:00','2023-06-02 17:30:00','T','Uber','001','S',sysdate(),NULL,'N');
```


       





