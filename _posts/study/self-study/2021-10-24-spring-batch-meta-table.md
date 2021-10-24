---
title : "Spring Batch 메타 테이블"
categories : 
    - java
tag :
    - spring-batch
toc : true
---

3~4개월 정도 전 배치 프로그램을 스프링 배치를 활용해서 개발하려고 한 적이 있다.  
당시에 적은 개발기간으로 아쉽게 포기할 수 밖에 없었는데,  
인프런에 스프링 배치 강의가 업로드되어 이번 기회에 공부하기 위해 수강하게 되었다.  

[인프런 정수원님 - 스프링 배치](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B0%B0%EC%B9%98#curriculum)  

---

Spring Batch 프로젝트를 만들면 메타 테이블을 생성해야한다.  
의존성을 주입받고, org.springframework.batch.core 패키지를 보면  
각 DB 환경에 맞는 schema-*.sql 파일이 있다.  
스크립트를 수동으로 실행하거나, application.properties에  
**spring.batch.initialize-schema: always** 설정을 해줄 수 있다.  

![error](/assets/images/self-study/meta-data-erd.png)  

6개의 테이블에 대해 알아보도록 하자.  

---

#### **Job 관련 테이블**

---

##### **BATCH_JOB_INSTANCE**
Job이 실행될 때 JobInstance 정보가 저장된다.  
JOB_NAME, JOB_KEY가 Key, Value 형태로 저장되기 때문에 중복 저장할 수 없다.  

| 컬럼이름          | 내용                                       |
| :-------------- | :---------------------------------------- |
| JOB_INSTANCE_ID | 고유하게 식별할 수 있는 기본 키 (PK)             |
| VERSION         | 업데이트 될 때마다 1씩 증가                     |
| JOB_NAME        | Job을 구성할 때 부여하는 Job의 이름              |
| JOB_KEY         | JOB_NAME과 JobParameter를 합쳐 해싱한 값을 저장 |

---

##### **BATCH_JOB_EXECUTION**
Job의 실행정보가 저장된다.  
Job의 생성, 시작/종료 시간, 실행상태, 메시지 등을 관리한다.  

| 컬럼이름           | 내용                                       |
| :--------------- | :---------------------------------------- |
| JOB_EXECUTION_ID | Job의 실행정보를 고유하게 식별할 수 있는 기본 키     |
| VERSION          | 업데이트 될 때마다 1씩 증가                     |
| JOB_INSTANCE_ID  | JOB_INSTANCE의 키                          |
| CREATE_TIME      | Execution이 생성된 시점                      |
| START_TIME       | Execution이 시작된 시점                      |
| END_TIME         | Execution이 종료된 시점                      |
| STATUS           | 실행 상태 (COMPLETED, FAILED, STOPPED ...)  |
| EXIT_CODE        | 실행 종료 코드를 저장 (COMPLETED, FAILED ...)  |
| EXIT_MESSAGE     | Status가 실패인 경우 실패 원인을 저장            |
| LAST_UPDATED     | 마지막 실행 시점                              |

---

##### **BATCH_JOB_EXECUTION_PARAMS**
Job과 함께 실행되는 JobParameter 정보를 저장한다.  

| 컬럼이름           | 내용                                       |
| :--------------- | :---------------------------------------- |
| JOB_EXECUTION_ID | Job의 실행정보를 고유하게 식별할 수 있는 기본 키    |
| TYPE_CD          | String, Long, Date, Double 타입 정보        |
| KEY_NAME         | 파라미터 키 값                               |
| STRING_VAL       | 파라미터 문자 값                              |
| DATE_VAL         | 파라미터 날짜 값                              |
| LONG_VAL         | 파라미터 Long 값                             |
| DOUBLE_VAL       | 파라미터 Double 값                           |
| IDENTIFYING      | 식별여부 (True, False)                      |

---

##### **BATCH_JOB_EXECUTION_CONTEXT**
Job의 실행동안 여러 상태정보, 공유 데이터를 JSON 형식으로 직렬화 해서 저장한다.  
Step 간 서로 공유할 수 있다.  

| 컬럼이름             | 내용                                           |
| :----------------- | :-------------------------------------------- |
| JOB_EXECUTION_ID   | Job의 실행정보를 고유하게 식별할 수 있는 기본 키        |
| SHORT_CONTEXT      | JOB의 실행 상태정보, 공유 데이터 등의 정보를 문자열로 저장 |
| SERIALIZED_CONTEXT | 직렬화 된 전체 컨텍스트                             |

---

#### **Step 관련 테이블**

---

##### **BATCH_STEP_EXECUTION**
Step의 실행정보가 저장되며, 생성, 시작/종료 시간, 실행상태, 메시지 등을 관리한다.  

| 컬럼이름             | 내용                                         |
| :----------------- | :------------------------------------------ |
| STEP_EXECUTION_ID  | Step의 실행정보를 고유하게 식별할 수 있는 기본 키      |
| VERSION            | 업데이트 될 때마다 1씩 증가                       |
| STEP_NAME          | Step을 구성할 때 부여하는 Step 이름               |  
| JOB_EXECUTION_ID   | Job의 실행정보를 고유하게 식별할 수 있는 기본 키       |
| START_TIME         | Execution이 시작된 시점                         |
| END_TIME           | Execution이 종료된 시점                         |
| STATUS             | 실행상태를 저장 (COMPLETED, FAILED, STOPPED...) |
| COMMIT_COUNT       | 트랜잭션 당 커밋되는 수                           |
| READ_COUNT         | 실행 시점에 Read한 Item의 수                     |
| FILTER_COUNT       | 실행도중 필터링 된 Item의 수                      |
| WRITE_COUNT        | 실행도중 저장되고 커밋 된 Item의 수                |
| READ_SKIP_COUNT    | 실행도중 Read가 Skip된 Item의 수                |
| WRITE_SKIP_COUNT   | 실행도중 Write가 Skip된 Item의 수               |
| PROCESS_SKIP_COUNT | 실행도중 Process가 Skip된 Item의 수             |
| ROLLBACK_COUNT     | 실행도중 Rollback이 일어난 수                    |
| EXIT_CODE          | 실행 종료코드를 저장 (COMPLETED, FAILED ...)     |
| EXIT_MESSAGE       | Status가 실해인 경우 실패 원인을 저장              |
| LAST_UPDATED       | 마지막 실행 시점                                |

---

##### **BATCH_STEP_EXECUTION_CONTEXT**
Step의 실행동안 여러 상태정보, 공유 데이터를 JSON 형식으로 직렬화 해서 저장한다.  
Step 별로 저장되며, Step 간 서로 공유할 수 없다.  

| 컬럼이름             | 내용                                            |
| :----------------- | :--------------------------------------------- |
| STEP_EXECUTION_ID  | Step의 실행정보를 고유하게 식별할 수 있는 기본 키         |
| SHORT_CONTEXT      | Step의 실행 상태정보, 공유 데이터 등의 정보를 문자열로 저장 |
| SERIALIZED_CONTEXT | 직렬화 된 전체 컨텍스트                              |  

---