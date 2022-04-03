---

title: Spring Batch란  
date: 2022-04-03
categories: [Spring]  
tags: ['spring', 'batch', 'java']  

---

***

# Batch란?
일괄 처리(batch processing)란 최종 사용자의 개입 없이 또는 (자원이 허가한다면) 실행을 스케줄링할 수 있는 작업(job)의 실행을 의미합니다.(출처: wikipedia)
Batch는 일반적으로 다음과 같은 용도로 사용됩니다.
- 대용량 데이터베이스, 파일, 큐를 읽는 것
- 일정하고 주기적인 데이터 처리 과정
- 수정된 형식으로 데이터를 다시 작성
  Batch는 유사한 트랜잭션을 하나로 묶어 처리해줌으로써(특히 사용자 인터랙션이 없는 오프라인 환경에서) 이런 반복적인 작업을 자동화 합니다.

# Spring Batch 소개
비즈니스 운영에 필수적인 작업을 종종 벌크 프로세싱으로 개발하게 되는데 이러한 배치 작업을 지원하는 경량 배치 프레임워크입니다. 구분해야하는 것은 스케줄링 프레임워크가 아닌 배치 프레임워크라는 것 입니다. 따라서 배치는 스케줄러와 함께 동작하도록 설계 되어 있습니다.

스프링 배치는 다음과 같은 기능을 제공합니다.
- 로깅/추적
- 트랜잭션 관리
- 작업 처리 통계
- 작업 재시작
- 건너뛰기
- 리소스 관리 등
- 대용량 레코드 처리에 필수적인 재사용 가능한 기능 제공
- 최적화 및 파티셔닝 기술을 통해 대용량 및 고성능 일괄 작업을 가능하게 하는 고급 기술 서비스 및 기능 제공

# 아키텍쳐
- 그림 1. Spring Batch의 Architecture

![img](uploads/b369ee95c5385fba19ae85db9a68e187/image.png)

Application : 스프링 배치를 사용하는 개발자가 만드는 모든 배치 job과 커스텀 코드를 말합니다.

Batch Core : Job을 실행하고 제어하는 데 필요한 핵심 런타임 클래스를 포함합니다. 여기에는 JobLauncher, Job, Step 구현체도 포합됩니다.

Batch InfraStructure : Application과 Core 모두 공통 Infra위에서 빌드합니다. 공통 reader와 writer, service(RetryTemplate 등)을 포함합니다.

- 그림 2. Spring Batch에서의 배치 프로세스

![img](uploads/bd5e448e817b18ba21074a19f3671023/image.png)

## Job
Job은 전체 배치 프로세스를 캡슐화한 엔티티입니다. Job은 인터페이스 이며, 단순히 Step 인스턴스의 컨테이너 개념입니다. 여러 Step을 결합하고, 재 시작 같은 속성을 전역으로 구성할 수 있습니다. 구성은 다음과 같습니다.

- Job의 name
- Step인스턴스의 정의와 순서
- Job의 재시작 가능 여부


```java
// 공식으로 제공되는 예제 메소드
@Bean
public Job footballJob() {
    return this.jobBuilderFactory.get("footballJob")
                     .start(playerLoad())
                     .next(gameLoad())
                     .next(playerSummarization())
                     .end()
                     .build();
}
```

## JobInstance

Parameters가 포함된 논리적인 Job의 구현체입니다. JobInstance는 실행결과를 여럿 가질 수 있습니다.

## JobParametes

JobParameters는 JobInstance를 구분할 수 있는 인자이다.

배치 Job을 시작할 때 사용하는 파라미터 셋을 가지고 있는 객체며 실행 중 job을 식별하거나 참조 데이터로 사용할 수 있습니다.

## JobExecution

JobExecution 은 job을 한번 실행하려 했다는 동작입니다. 성공, 실패와 관계없이 Execution은 동작하며 실행에 상응하는 JobInstance가 성공적으로 종료되기 전까지는 완료되지 않은 것으로 간주됩니다.

JobExecution은 실제 실행 중에 필요한 기본 스토리지 메커니즘을 제공하며, 많은 프로퍼티를 관리하고 유지해야 합니다.

# Step

Step은 배치 Job의 독립적이고 순차적인 단계를 캡슐화한 도메인 객체입니다. Job은 하나 이상의 Step으로 구성되며 Step은 실제 배치 동작에 대한 처리를 정의, 컨트롤하는데 필요한 모든 정보를 가지게 됩니다.

- 그림 3. Spring Batch에서 Job과 Step의 동작 프로세스

![image](uploads/4ca5cbe0d0d65020cfb903fcdd4f2f71/image.png)

## StepExecution

StepExecution은 한 번의 Step 실행 시도를 의미합니다. StepExecution은 JobExecution과 유사하게 Step을 실행할 때마다 생성됩니다. 하지만 이전 단계 Step이 실패해서 실행하지 않았다면 execution을 저장하지 않습니다. Step이 실제로 시작됬을 때만 StepExecution을 생성합니다.

execution은 실행/종료 시각이나 커밋/롤백 횟수를 포함한 해당 step, JobExecution, 트랜잭션 관련 데이터를 가지고 있습니다.

## ExecutionContext

ExecutionContext 프레임워크에서 유지/관리하는 키/값 쌍의 컬렉션으로, StepExecution 객체 또는 JobExecution 객채에 속하는 상태를 저장합니다.

## JobRepositlory

JobRepository는 위에서 언급된 모든 저장 메커니즘을 담당합니다.

자바 기반 설정은 @EnableBatchProcessing 어노테이션만 달아주면 JobRepository를 자동으로 컴포넌트로 설정합니다.

## JobLauncher

JobLauncher는 JobParameters로 Job을 실행하는 인터페이스 입니다.

```java
public interface JobLauncher {

public JobExecution run(Job job, JobParameters jobParameters)
            throws JobExecutionAlreadyRunningException, JobRestartException,
                   JobInstanceAlreadyCompleteException, JobParametersInvalidException;
}
```

## ItemReader & Processor & Writer

Step에서 실행시킬 동작단위입니다. 각각 읽기, 처리, 쓰기를 담당하며 각각의 인터페이스를 기반으로 QueryDsl, JPA등의 ORM형태로 수정하는 것이 가능합니다.

# 참고 사항
## [배치 솔루션의 핵심원칙, 가이드라인, 그 외 일반적으로 고려해야할 사항](https://godekdls.github.io/Spring%20Batch/introduction/#14-general-batch-piplelines-and-guidlines)
- 배치 아키텍처는 보통 온라인 아키텍처에 영향을 주고, 반대도 마찬가지임을 기억하라.
  가능하면 공통 구성 요소를 활용하고 아키텍처와 환경을 모두 고려해 설계해라.
- 단일 배치 어플리케이션은 가능한 단순화하고 복잡한 로직은 피하라.
- 데이터 처리와 저장은 물리적으로 가까운 곳에서 수행해라 (다시 말해 데이터가 처리되는 곳에 데이터를 저장해라).
- 시스템 리소스 사용, 특히 I/O를 최소화해라. 내부 메모리에서 가능한 많은 연산을 실행해라.
- 어플리케이션 I/O를 점검해서 불필요한 물리적 I/O를 줄여라 (SQL 구문을 분석하라).
  특히 아래 네 가지 결함에 주의해라:
    - 한 번 읽고 나서 캐시하거나 작업 스토리지에 저장해도 되는 데이터를 매 트랜잭션마다 읽는 경우.
    - 같은 트랜잭션 내에서 이미 읽은 데이터를 다시 읽는 경우.
    - 불필요한 테이블 스캔이나 인덱스 스캔을 유발하는 경우.
    - SQL 구문에서 WHERE 절에 키를 지정하지 않는 경우.
- 배치 실행에서 같은 작업을 두 번 하지 마라.
  예를 들어 리포팅 목적으로 데이터를 요약하는 어플리케이션이라면
  데이터를 처음 처리할 때 저장돼 있는 토탈값을 증가시켜서(가능하다면)
  같은 데이터를 다시 처리하지 않게 해라.
- 실행 중에 재할당에 시간을 쏟지 않게 배치 어플리케이션 시작 시 충분한 메모리를 할당해라.
- 데이터 무결성은 최악의 상황도 고려해라.
  데이터 무결성을 유지하고 싶으면 적절한 유효성 검증 로직을 추가해라.
- 가능한 곳에 내부 검증을 위한 체크섬을 구현해라.
  예를 들어 플랫(flat) 파일은 파일의 총 레코드 수와
  주요 필드의 집계 결과를 알 수 있는 트레일러 레코드가 필요하다.
- 실제 프로덕션 환경과 그에 맞는 데이터 볼륨을 가지고 가능한 빨리 부하 테스트를 계획하고 실행해라.
- 배치 규모가 크면, 특히 그 시스템이 온라인과 동시에 무중단으로 실행된다면
  백업이 매우 어려울 수도 있다. 하지만 온라인 설계에서 진행하는 데이터베이스 백업만큼 파일 백업도 중요하다.
  시스템이 플랫(flat) 파일을 사용한다면 파일 백업 절차를 수립하고 문서화하고 정기적으로 테스트해야 한다.

# Reference
- 스프링 배치 래퍼런스 문서 번역 : [https://godekdls.github.io/Spring Batch](https://godekdls.github.io/Spring%20Batch/introduction/)
- Spring Batch 공식 Reference : https://docs.spring.io/spring-batch/docs/current/reference/html
