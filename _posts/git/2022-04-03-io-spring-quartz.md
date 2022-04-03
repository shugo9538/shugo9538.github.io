---

title: Spring Quartz란 
date: 2022-04-03
categories: [Spring]  
tags: ['spring', 'batch', 'java', 'quartz']  

---

***

# Quartz란?
Quartz는 다양한 Java 애플리케이션에 통합 될 수있는 작업 스케줄링 오픈소스 라이브러리입니다.
단순 반복작업이 아닌 경우 활용됩니다. 단순 반복작업인 경우에는 Quartz보다 `@Scheduled` 어노테이션을 활용하는 것이 훨씬 유용합니다.

# Quartz의 사용
Quartz의 사용은 단순 반복작업에 비해 훨씬 사용하기 까다롭습니다. 대신 쓰레드의 관리, 배치작업의 성공 여부에 따라 동작을 제어할 수 있기 때문에 이러한 기능을 사용하기 위해 사용됩니다.

# 구조
- Job :

스케줄링할 실제 작업을 구현한 객체 Quartz API에서 단 하나의 메서드 execute(JobExecutionContext context) 를 가진 Job 인터페이스를 제공. Quartz를 사용하는 개발자는 수행해야 하는 실제 작업을 이 메서드에서 구현하면 된다.

매개변수인 JobExecutionContext는 Scheduler, Trigger, JobDetail 등을 포함하여 Job 인스턴스에 대한 정보를 제공하는 객체

- JobDetail :

Job을 실행시키기 위한 정보를 담고 있는 객체 Job의 이름, 그룹, JobDataMap 속성 등을 지정할 수 있음.

Trigger가 Job을 수행할 때 이 정보를 기반으로 스케줄링을 한다

- JobDataMap :

JobDataMap은 Job 인스턴스가 execute 함수를 실행할 때 사용할 수 있게 원하는 정보를 담을 수 있는 객체
JobDetail을 생성할 때 JobDataMap도 같이 세팅해주면 된다

JobFactory 실제로 Job 을 인스턴스화 시키는 클래스, 스프링에선 SpringBeanJobFactory 클래스로 구현되며 Scheduler 구현시에 스프링 구성설정에 따라 구현되어 내부 인스턴스에 저장된다.
알아서 Job 에 SpringBean 을 의존성 주입 시키는등의 작업을 수행한다.

- JobStore :


Scheduler 에서 등록된 Job, Trigger, 그리고 실행이력이 저장되는 공간이다. 기본적으로 메모리공간에 저장되어 JVM 에서 관리되지만, 원한다면 다른 RDB 에서 관리할 수 있다.

- Trigger :

Trigger는 Job을 실행시킬 스케줄링 조건 (ex. 반복 횟수, 시작시간) 등을 담고 있고
Scheduler는 이 정보를 기반으로 Job을 수행시킨다.
N Trigger = 1 Job
한개이상의 Trigger는 반드시 하나의 Job을 지정할 수 있다

SimpleTrigger - 특정 시간에 Job을 수행할 때 사용되며 반복 횟수와 실행 간격등을 지정할 수 있다 CronTrigger - CronTrigger는 cron 표현식으로 Trigger를 정의하는 방식이다

- Scheduler :

JobDetail 과 Trigger 을 시스템에 등록하고 스케쥴에 맞춰 Job 을 실행시키는 객체, 일반적으로 StdScheduler 로 구현된다.

- SchedulerFactory :

Scheduler 인스턴스를 생성하는 역할, 스프링 부트에선 application.properties 를 사용해 다양한 설정을 통해 자동으로 구현가능하다.

# 실행 프로세스
- 그림 1. Quartz의 실행 프로세스

![image](uploads/bc65b706b3f1d7544b5393e6945571b9/image.png)

- 그림 2. Quartz의 클래스 관계도

![image](uploads/4a399d6564aeb6cd586ab3a346e29a4b/image.png)

# Quartz의 테이블 구성도

- 그림 3. Quartz의 메타 테이블

![image](uploads/e55ce8285297a1dd681272927f6d8b47/image.png)




# 참고
- https://github.com/sieunkr/spring-boot-quartz
- https://kouzie.github.io/spring/Spring-Boot-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8-Quartz/#crone-expression
