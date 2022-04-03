---

title: Spring Batch와 Quartz 연동
date: 2022-04-03
categories: [Spring]  
tags: ['spring', 'batch', 'java', 'quartz']  

---

***

# Spring Batch와 Quartz연동

# application.yml

```yaml
spring:
  config:
    activate:
      on-profile: dev
  batch:
    job:
      enabled: false # cmd line runner 설정 해제 : 앱 구동시점에 특정 작업이 실행될 수 있도록 해주는 역할 - 스케쥴러 사용할 것이기 때문에 해제
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/batch
    username: batch
    password: batch1234
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      maximum-pool-size: 3
  jpa:
    hibernate:
      ddl-auto: validate
    database: mysql
    show-sql: false
    properties:
      hibernate:
        format_sql: true
        default_batch_fetch_size: 3
  flyway:
    locations: classpath:/db/migration
    sql-migration-suffixes: ddl
    baseline-on-migrate: true
    baseline-version: 1
  quartz:                      # Quartz 설정
    job-store-type: jdbc       # in-Memory방식이 아닌 jdbc를 사용하는 설정
    jdbc:
      initialize-schema: always          # 쿼츠 테이블을 자동으로 생성, jdbc를 사용하기 때문에 이 설정을 사용하지 않는다면 반드시 db에 테이블을 만들어 줘야한다.
    properties:
      org:
        quartz:
          scheduler:
            instanceName: JobOneScheduler             # 스케줄러 이름 설정, 반드시 할 필요는 없다.
            instanceId: AUTO
          jobStore:
            useProperties: true                              # jobStore를 커스텀 설정을 사용하는 설정으로 설정을 추가할 경우 반드시 필요
            dataSource: myDS                                 # 사용할 커스텀 데이터소스에 대한 이름으로 밑에서 추가로 선언
            # 사용할 수 있는 잡 스토어 클래스는 TX와 CMT, RAM 세 가지가 있으며, RAM은 in-Memory 방식에서 사용되고 나머지는 각각 사용용도가 다르다.
            class: org.quartz.impl.jdbcjobstore.JobStoreTX        # 독립 실행환경에서 사용하도록 되어있어 커밋과 롤백 모두 처리가 가능하다.
#            class: org.quartz.impl.jdbcjobstore.JobStoreCMT       # 어플리케이션 서버 환경내에서 사용할 경우 사용하며, 커밋/롤백 처리를 하지않는다.
            driverDelegateClass: org.quartz.impl.jdbcjobstore.StdJDBCDelegate 
            isClustered: true
          threadPool:                                          # 쿼츠의 쓰레드 풀 설정
            class: org.quartz.simpl.SimpleThreadPool
            threadCount: 10
            threadPriority: 5
            threadsInheritContextClassLoaderOfInitializingThread: true
          dataSource:
            myDS:
              provider: hikaricp           # 히카리 커넥션풀을 사용하는 설정으로 이 설정을 사용하지 않는 경우 gradle상에 implementation 'com.mchange:c3p0:0.9.5.5' 를 추가해야한다.
              driver: com.mysql.cj.jdbc.Driver
              URL: jdbc:mysql://127.0.0.1:3306/batch
              user: root
              password: root1234
```

# build.gradle

```groovy
configurations{
    all {
        resolutionStrategy.cacheChangingModulesFor 0, "seconds"
    }
}

dependencies {
    ...
    
    implementation 'org.springframework.boot:spring-boot-starter-batch'
    implementation 'org.springframework.boot:spring-boot-starter-quartz'
    implementation 'org.springframework:spring-context-support'            // context설정을 사용하기위해 반드시 추가로 필요한 설정

    ...
}

test {
    useJUnitPlatform()
}
```

# AutowiringSpringBeanJobFactory

```java
import org.quartz.spi.TriggerFiredBundle;
import org.springframework.beans.factory.config.AutowireCapableBeanFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.quartz.SpringBeanJobFactory;

@Configuration
@ComponentScan
public class AutowiringSpringBeanJobFactory extends SpringBeanJobFactory implements ApplicationContextAware {
    // Spring boot에서 자동으로 생성되는 빈 보다 늦게 생성되는 빈을 정상적으로 동작시키기 위해서 필요한 설정
    private AutowireCapableBeanFactory beanFactory;

    @Override
    public void setApplicationContext(final ApplicationContext context) {
        // 
        this.beanFactory = context.getAutowireCapableBeanFactory();
    }

    @Override
    protected Object createJobInstance(final TriggerFiredBundle bundle) throws Exception {
        final Object job = super.createJobInstance(bundle);
        beanFactory.autowireBean(job);
        return job;
    }
}
```

Spring Boot에서의 Bean의 생성이 Quartz 스케줄러에 의해 생성되는 Batch Job의 설정에 의한 Bean 생성보다 먼저 일어난다. 이 설정을 통해 Job의 Bean을 등록해 `@Autowire`와 같은 생성된 빈을 사용할 수 있도록 하기위해 이 설정을 해야만한다.



# BatchJobConfiguration.java

```java
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import mz.batch.domain.vo.Authority;
import mz.batch.items.QuerydslPagingItemReader;
import mz.batch.jpa.jpo.UserJpo;
import mz.batch.jpa.repo.UserRepository;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepScope;
import org.springframework.batch.item.ItemProcessor;
import org.springframework.batch.item.ItemWriter;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.PlatformTransactionManager;

import javax.persistence.EntityManagerFactory;

import static mz.batch.jpa.jpo.QUserJpo.userJpo;

/**
 * Batch 기반의 일반적인 Job 생성
 */
@Slf4j
@Configuration
@RequiredArgsConstructor
public class BatchJobWithQuartzConfig {
    // Batch 상의 Job 생성
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final PlatformTransactionManager transactionManager; // Quartz와 사용할 경우 사용
    private final UserRepository userRepository;
    private final EntityManagerFactory entityManagerFactory; // QueryDsl을 사용하는 경우 사용

    @Bean
    @Qualifier("userGuestToAdminJob")
    public Job userGuestToAdminJob() {
        //
        return jobBuilderFactory.get("userGuestToAdminJob")
                .preventRestart()
                .start(userGustToAdminStep())  // 파라미터에서 주입받은 Job 실행 설정
                .build();
    }

    /**
     * 트랜잭션 매니저를 추가해야하는 이유는 Batch만 단일로 사용하는 경우 스프링에서 자동으로 이 트랜잭션 매니저를 연결해준다.
     * 하지만 쿼츠와 함께 사용하는 경우 쿼츠와 배치를 함께 사용하기위해 스프링의 자동설정을 건드리게되며 이 과정에서 트랜잭션 매니저와 연결이 자동으로 되지 않게된다.
     * 이는 쿼츠 스케줄러 상에 배치를 포함시키기 때문에 발생하는 문제이며, 쿼츠와 동일한 트랜잭션 매니저를 불러와 연결시켜주면 해결된다.
    */
    @Bean
    public Step userGustToAdminStep() {
        //
        return stepBuilderFactory.get("userGustToAdminStep")
                .<UserJpo, UserJpo>chunk(10)
                .reader(userGuestItemReader())
                .processor(userGuestToAdminProcessor())
                .writer(userGuestToAdminWriter())
                .transactionManager(transactionManager) // 여기서 트랜잭션 매니저를 추가해야한다.
                .build();
    }

    @Bean
    @StepScope // Step의 주기에 따라 새로운 빈을 생성 하기때문에 파라미터 설정때문에 할당이 늦어진다.
    public QuerydslPagingItemReader<UserJpo> userGuestItemReader() {
        //
        log.info("item reader");
        int chunkSize = 10;

        // ItemReader를 상속받아 Querdydsl을 기반으로 read를 수정할 수 있다. 대신 대량의 데이터를 읽는 경우에는 성능상의 이슈가 있기때문에 이를 처리하기 위한 작업처리가 더 필요하다.
        return new QuerydslPagingItemReader<>(
                entityManagerFactory,
                chunkSize,
                queryFactory -> {
                    log.info("queryFactory is created? : {}", queryFactory);

                    return queryFactory
                            .selectFrom(userJpo)
                            .where(userJpo.birthdayYear.eq(2022));
                });
    }

    public ItemProcessor<UserJpo, UserJpo> userGuestToAdminProcessor() {
        //
        log.info("item processor");
        return userJpo -> {
            userJpo.setAuthority(Authority.GUEST);
            return userJpo;
        };
    }

    public ItemWriter<UserJpo> userGuestToAdminWriter() {
        //
        log.info("Item writer in log");
        return (userRepository::saveAllAndFlush);
    }
}
```

# QuartzConfiguration.java

```java
import lombok.extern.slf4j.Slf4j;
import mz.batch.job.QuartzJob;
import org.quartz.JobDataMap;
import org.quartz.JobDetail;
import org.quartz.Trigger;
import org.springframework.batch.core.configuration.JobLocator;
import org.springframework.batch.core.configuration.JobRegistry;
import org.springframework.batch.core.configuration.support.JobRegistryBeanPostProcessor;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.boot.autoconfigure.quartz.QuartzProperties;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.scheduling.quartz.CronTriggerFactoryBean;
import org.springframework.scheduling.quartz.JobDetailFactoryBean;
import org.springframework.scheduling.quartz.SchedulerFactoryBean;
import org.springframework.scheduling.quartz.SpringBeanJobFactory;
import org.springframework.transaction.PlatformTransactionManager;

import javax.annotation.PostConstruct;
import javax.persistence.EntityManagerFactory;
import javax.sql.DataSource;
import java.text.ParseException;
import java.util.Arrays;
import java.util.Properties;

@Slf4j
@Configuration
public class QuartzConfig {
    //
    private final JobLauncher jobLauncher;
    private final JobLocator jobLocator;
    private final QuartzProperties quartzProperties;
    private final ApplicationContext applicationContext;
    private final EntityManagerFactory emf;
    private final DataSource dataSource;

    public QuartzConfig(JobLauncher jobLauncher, JobLocator jobLocator, QuartzProperties quartzProperties, ApplicationContext applicationContext, EntityManagerFactory emf, DataSource dataSource) {
        this.jobLauncher = jobLauncher;
        this.jobLocator = jobLocator;
        this.quartzProperties = quartzProperties;
        this.applicationContext = applicationContext;
        this.emf = emf;
        this.dataSource = dataSource;
    }

    @PostConstruct
    public void init() {
        //
        QuartzConfig.log.info("QuartzScheduler initialized");
    }

    @Bean
    public SpringBeanJobFactory springBeanJobFactory() {
        AutowiringSpringBeanJobFactory jobFactory = new AutowiringSpringBeanJobFactory();
        QuartzConfig.log.debug("Configuring Job factory");

        jobFactory.setApplicationContext(this.applicationContext);
        return jobFactory;
    }

    /**
     * Bean을 구성하면 JobRegistryBeanPostProcessorBean을 등록 하여 (JobLocator) 를 통해 Bean Job을
     * JobRegistry가져올 QuartzJobBean수 있습니다.
     * 2. 확장 되어 Spring 배치와 Quartz 사이의 다리 역할을 합니다.JobJobRegister
     *
     * @param jobRegistry
     * @return
     */
    @Bean
    public JobRegistryBeanPostProcessor jobRegistryBeanPostProcessor(JobRegistry jobRegistry) {
        JobRegistryBeanPostProcessor jobRegistryBeanPostProcessor = new JobRegistryBeanPostProcessor();
        jobRegistryBeanPostProcessor.setJobRegistry(jobRegistry);
        return jobRegistryBeanPostProcessor;
    }

    /**
     * 스케줄러상에서 사용할 Job의 상세 내용들을 추가하는 구간
     * name 작업 이름 지정
     * group 작업 그룹 지정
     * durability 한번만 사용할 것인지, 계속 사용할 것인지 지정
     * jobclass 구현될 Job의 클래스 지정
     * 
     */
    @Bean
    public JobDetail jobOneDetail() {
        //Set Job data map
        JobDetailFactoryBean jobDetailFactoryBean = new JobDetailFactoryBean();
        jobDetailFactoryBean.setName("userGuestToAdminJob");
        jobDetailFactoryBean.setGroup("JobTwoScheduler");
        jobDetailFactoryBean.setJobClass(QuartzJob.class);
        jobDetailFactoryBean.setDurability(true);
        jobDetailFactoryBean.afterPropertiesSet(); // 그 외 기본설정을 따르기 위해 사용

        return jobDetailFactoryBean.getObject();
    }

    public JobDataMap jobDataMap() {
        // 실제 사용될 클래스 구현체의 데이터를 넘겨주기 위한 Map
        JobDataMap jobDataMap = new JobDataMap();
        jobDataMap.put("userGuestToAdminJob", "userGuestToAdminJob");
        log.info("jobLauncher : {}", jobLauncher.toString());
        jobDataMap.put("jobLauncher", jobLauncher);
        jobDataMap.put("jobLocator", jobLocator);

        return jobDataMap;
    }

    @Bean
    public Trigger jobOneTrigger() {
        CronTriggerFactoryBean cronTriggerFactoryBean = new CronTriggerFactoryBean();
        cronTriggerFactoryBean.setCronExpression("0/30 * * * * ?");
        cronTriggerFactoryBean.setJobDetail(jobOneDetail());  // 사용할 JobDetail 지정, 여러 건 등록 가능
        cronTriggerFactoryBean.setName("jobOneTrigger");
        cronTriggerFactoryBean.setGroup("JobTwoScheduler");

        try {
            cronTriggerFactoryBean.afterPropertiesSet();
        } catch (ParseException e) {
            e.printStackTrace();
        }

        return cronTriggerFactoryBean.getObject();
    }

    /**
     * 트랜잭션 매니저를 불러와 Batch상에서도 사용할 수 있게하기 위해 연결이 필요하기 때문에 필요한 설정
     */
    @Bean
    public PlatformTransactionManager transactionManager() {
        JpaTransactionManager transactionManager = new JpaTransactionManager();
        transactionManager.setDataSource(dataSource);
        transactionManager.setEntityManagerFactory(emf);
        return transactionManager;
    }
 
    /**
     * 쿼츠 스케줄러의 최종적인 구성
     */
    @Bean
    public SchedulerFactoryBean schedulerFactoryBean() {
        SchedulerFactoryBean scheduler = new SchedulerFactoryBean();
        scheduler.setSchedulerName("JobTwoScheduler");
        scheduler.setJobFactory(springBeanJobFactory());
        scheduler.setSchedulerContextAsMap(jobDataMap());
        scheduler.setTransactionManager(transactionManager());
        scheduler.setTriggers(jobOneTrigger());
        scheduler.setJobDetails(jobOneDetail());
        scheduler.setQuartzProperties(quartzProperties());
        return scheduler;
    }

    public Properties quartzProperties() {
        Properties properties = new Properties();
        properties.putAll(quartzProperties.getProperties());
        log.info("properties : {}", Arrays.toString(properties.keySet().toArray()));
        return properties;
    }
}
```

#
QuartzJob

```java
package mz.batch.job;

import lombok.Getter;
import lombok.Setter;
import lombok.extern.slf4j.Slf4j;
import org.quartz.JobExecutionContext;
import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.batch.core.configuration.JobLocator;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.quartz.QuartzJobBean;
import org.springframework.stereotype.Component;
import org.springframework.transaction.PlatformTransactionManager;

import javax.transaction.Transactional;
import java.util.Date;
import java.util.Map;

@Slf4j
@Component
@Getter
@Setter
//@DisallowConcurrentExecution //synchronized 설정, 작업 순서 보장
public class QuartzJob extends QuartzJobBean {
    //
    private static final String JOB_NAME = "userGuestToAdminJob";
    private JobLauncher jobLauncher;
    private JobLocator jobLocator;

    @Autowired
    private PlatformTransactionManager transactionManager;

    @Transactional
    @Override
    protected void executeInternal(JobExecutionContext context) {
        //
        Map<String, Object> jobDataMap = context.getMergedJobDataMap();
        log.info("Quartz trigger firing with Spring Batch jobName=" + JOB_NAME);
        JobParametersBuilder jobParametersBuilder = new JobParametersBuilder();

        JobParameters jobParameters = jobParametersBuilder
                .addJobParameters(getJobParametersFromJobMap(jobDataMap))
                .addDate("date", new Date())
                .toJobParameters();

        try {
            jobLauncher.run(jobLocator.getJob(JOB_NAME), jobParameters);
        } catch (Exception e) {
            log.error("exception : {}", JOB_NAME);
            e.printStackTrace();
        }
    }

    private JobParameters getJobParametersFromJobMap(Map<String, Object> jobDataMap) {
        //
        JobParametersBuilder builder = new JobParametersBuilder();

        for (Map.Entry<String, Object> entry : jobDataMap.entrySet()) {
            String key = entry.getKey();
            Object value = entry.getValue();
            log.info("dataMap is getting value : {}", value);
            if (value instanceof String && !key.equals(JOB_NAME)) {
                builder.addString(key, (String) value);
            } else if (value instanceof Float || value instanceof Double) {
                builder.addDouble(key, ((Number) value).doubleValue());
            } else if (value instanceof Integer || value instanceof Long) {
                builder.addLong(key, ((Number) value).longValue());
            } else if (value instanceof Date) {
                builder.addDate(key, (Date) value);
            } else {
                log.debug("JobDataMap contains values which are not job parameters (ignoring).");
            }
        }

        return builder.toJobParameters();

    }
}
```
