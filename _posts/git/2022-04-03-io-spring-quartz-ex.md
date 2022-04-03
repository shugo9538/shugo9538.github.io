---

title: Spring Quartz 예제
date: 2022-04-03
categories: [Spring]  
tags: ['spring', 'batch', 'java', 'quartz']  

---

***

# 설정

## Gradle

```groovy
implementation 'org.springframework.boot:spring-boot-starter-quartz'
```

## application.yml

```yaml
spring:
  quartz:
    job-store-type: jdbc **#using jdbc database
	jdbc:
	  initialize-schema: always # initialize Quartz Base Scehma**
```

## Config.java

```Java
import lombok.extern.slf4j.Slf4j;
import org.quartz.JobExecutionContext;
import org.springframework.scheduling.quartz.QuartzJobBean;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class QuartzJobWithoutBatch extends QuartzJobBean {
    //

    @Override
    protected void executeInternal(JobExecutionContext context) {
        //
        log.info("Quartz Scheduler Test");
    }
}
```

# 추가 설정
스프링에서 Quartz를 사용하는 방법은 여러가지 이지만 Spring Boot의 경우에는 properties파일 혹은 yml파일의 설정만으로 간략한 설정은 대부분 지정 가능하다.

- Quartz properties 예시

```Yaml
#Quartz
org.quartz.scheduler.instanceName = SampleJobScheduler
org.quartz.scheduler.instanceId = AUTO
org.quartz.scheduler.idleWaitTime = 10000
org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount = 4
org.quartz.threadPool.threadPriority = 5
org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.StdJDBCDelegate
org.quartz.jobStore.tablePrefix = QRTZ_
org.quartz.jobStore.misfireThreshold = 60000
org.quartz.jobStore.isClustered = false
org.quartz.jobStore.maxMisfiresToHandleAtATime = 10
org.quartz.jobStore.useProperties = true

#quartz mysql database connection
org.quartz.jobStore.dataSource = mySql
org.quartz.dataSource.mySql.driver = com.mysql.cj.jdbc.Driver
org.quartz.dataSource.mySql.URL = jdbc:mysql://localhost:3306/test_new_database
org.quartz.dataSource.mySql.user = root
org.quartz.dataSource.mySql.password = Testing123
org.quartz.dataSource.mySql.maxConnections = 10
org.quartz.dataSource.mySql.idleConnectionValidationSeconds = 50
org.quartz.dataSource.mySql.validationQuery=select 0 from dual
org.quartz.dataSource.mySql.maxIdleTime = 60
```

# 참고
- Quartz 공식 래퍼런스 : http://www.quartz-scheduler.org/documentation/2.4.0-SNAPSHOT/configuration.html
- https://kouzie.github.io/spring/Spring-Boot-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8-Quartz/#interruptjob
- https://github.com/sieunkr/spring-boot-quartz
