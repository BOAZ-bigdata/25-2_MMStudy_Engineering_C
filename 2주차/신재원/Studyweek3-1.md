구조화 데이터와 비구조화 데이터 

스키마 : 데이터베이스나 데이터 구조에서 데이터가 어떻게 구성되어 있는지, 어떤 종류의 데이터가 저장될 수 있는지, 그리고 데이터 요소들이 서로 어떻게 관련되어 있는지를 정의하는 논리적인 설계 

스키마가 있으면 구조화 데이터이며 SQL 사용 가능  , 스키마가 없으면 비구조화 데이터 사용 불가능 

CSV , JSON , XML 등의 데이터는 서식은 정해져 있지만, 칼럼 수나 데이터형은 명확하지 않아 스키마리스 데이터라고 불린다. 

데이터 파이프라인 -> 각각의 구조화 데이터 , 비구조화 데이터 , 스키마리스 데이터 를 분산 스토리지 에 보존 

비구조화 데이터  , 스키마리스 데이터는 구조화 데이터로의 ** 변환 필요 **

Hadoop

하둡의 동작흐름
 데이터가 들어오면, 데이터를 쪼갠다. 그리고 그 데이터를 분리해서 저장한다. 따라서 데이터를 쪼갠 후에 어느 데이터 노드에 저장이 되어 있는지를 기록해 놓는 부분(메타데이터)이 필요하다. 정리하면, 하둡에서 데이터를 저장하기 전에 네임노드에서 분산을 하고 저장위치를 분배한다. 그 후에 여러개 중에 지정된 데이터 노드에 저장을 한다고 간단히 이해하자.

분산 파일 시스템 Hadoop Distributed File System (HDFS), Amazon S3, Google Cloud Storage.

리소스 관리자 YARN, Apache Mesos, Kubernetes.

분산 데이터처리  Apache Spark, Apache Flink, Apache Hive, Apache Pig, Apache Hadoop MapReduce (초기 모델).


1. Hadoop에서 처리되는 데이터 대부분은 분산 파일 시스템 HDFS 에 저장된다. 이는 파일 서버와 유사, 다수의 컴퓨터에 파일 복사, 중복성 증가 


2. CPU, RAM 등 계산 리소스를 컨테이너 단위로 리소스 매니저인 YARN 에 의해 관리(할당), 

2-1. 분산 시스템은 많은 계산 리소스를 소비하지만, 호스트의 수에 따라 사용할 수 있는 리소스의 상한이 결정된다. 실제로는 리소스 자체도 한정되므로 애플리케이션 간 리소스 쟁탈이 발생하나, 리소스 관리자가 관리함으로써 모든 애플리케이션이 차질없이 실행되도록 제어한다. 

3. MapReduce : YARN 상에서 동작하는 분산 APP 중 하나, 분산 시스템에서 빅데이터 배치 처리 담당 , 비구조화 데이터 가공에 적합 

3-1. Map 과 Reduce 로 나뉜다 

4. Apache Hive : 구조화 데이터를 쿼리 언어에 의한 대규모 배치 처리 , 빅데이터 집계에 적합 . 쿼리를 자동으로  MapReduce 프로그램으로 변환하는 SW. MapReduce의 성질 계승. 

4-1. Hive 를 가속화하기 위해 개발된 Apache Tez 

4-2. Impala, Presto : Hive 와 달리 대화형 쿼리 엔진 사용  - 빠른 응답이 필요한 분석에 적합.

다만 쿼리 엔진 자체가 최종적인 실행 시간에 그다지 많은 영향을 끼치지 않는다 

PRESTO 의 구조  - SQL 실행 특화 , 쿼리 분석 후 바이트 코드 변환 - 기계 코드로 컴파일 

5. Spark : 대량의 메모리를 활용한 고속화. MapReduce 와 달리 디스크 읽고 쓰기 X  여유가 생길 때까지 기다리거나 오류로 실패, 배치 와 스트리밍 처리 모두 적합. 

배치 처리 측면에서의 강점:

MapReduce 대비 10~100배 빠른 성능 (특히 반복 계산이 많은 머신러닝/그래프 처리).

복잡한 ETL(추출, 변환, 적재) 파이프라인 구축 용이.

다양한 데이터 소스와 연동 가능.

스트리밍 처리 측면에서의 강점:

실시간에 가까운 데이터 처리 및 분석.

스트리밍 데이터에 대한 복잡한 연산(조인, 집계, 상태 저장) 가능.

배치 코드와 스트리밍 코드의 통합(Structured Streaming).

장애 내구성(Fault-tolerance): RDD의 내구성 특성으로 인해 노드 장애 시에도 데이터 손실 없이 복구 가능.


데이터 마트 구축의 파이프 라인 p101 

1. 분산 스토리지에 저장된 데이터를 구조화하고 열 지향 스토리지 형식으로 저장  - Hive 이용 

2. 저장한 구조화 데이터를 결합, 집계하고 비정규화 테이블로 데이터 마트에 써서 내보낸다. 

3. Hive 에서 만든 각 테이블의 정보 -> Hive 메타 스토어라고 불리는 특별한 데이터베이스에 저장. 

4. 열 지향 스토리지로 변환, 새로 저장 

5. 비정규화 테이블 생성 +  Hive (대규모) 와 Presto (실시간) 중 선택 

6. 쿼리 엔진용 SQL  최적화 


고속화를 방해하는 문제 - 데이터 편향 (Data Skew) p107 

분산 시스템에서 SELECT ~ 를 실행하는 것은 다른 처리보다 시간이 오래걸림. 분산 환경에서 중복이 없는 값을 세는 것이 복잡하기 때문 

전역 집계의 어려움

데이터 스큐(Data Skew) 문제: 특정 date에 해당하는 user_id가 매우 많거나, 특정 user_id가 여러 date에 걸쳐 많이 나타나면, 특정 노드에 데이터가 몰리면서 해당 노드의 처리 시간이 매우 길어져 전체 쿼리 속도를 저하시킬 수 있습니다.

해결법 

내부 서브쿼리 (SELECT DISTINCT date, user_id FROM access_log):

이 단계에서는 각 (date, user_id) 쌍의 중복을 제거합니다. 즉, 특정 날짜에 특정 사용자가 여러 번 접속했더라도, (date, user_id) 쌍은 단 한 번만 남게 됩니다.

이 DISTINCT 연산도 셔플과 정렬을 필요로 하지만, COUNT(DISTINCT ...)에 비해 효율적일 수 있습니다. 왜냐하면, 이는 개별 (date, user_id) 쌍을 고유하게 만드는 것이지, 모든 user_id를 모아서 전역적으로 고유성을 검사하는 것이 아니기 때문입니다.

또한, 이 DISTINCT는 GROUP BY date 이전에 수행되므로, 이후의 COUNT(*) 연산이 더 단순해지고 병렬화가 용이해집니다.

외부 쿼리 (SELECT date, count(*) users ... GROUP BY date):

서브쿼리의 결과 (t 테이블)는 이미 (date, user_id) 쌍에서 중복이 제거된 상태입니다.

이제 GROUP BY date와 COUNT(*)를 수행하는 것은 각 date별로 중복 없는 user_id의 개수를 세는 것이므로, COUNT(*)는 각 그룹의 행 수를 세는 단순한 집계가 되어 매우 효율적으로 분산 처리될 수 있습니다.



                                                 

















