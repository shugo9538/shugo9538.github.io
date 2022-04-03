---

title: Spring Batch 예제
date: 2022-04-03
categories: [Spring]  
tags: ['spring', 'batch', 'java']  

---

***
# Batch 설정

## 기본 설정

기본적으로 Spring Batch는 In-memory(H2 db)를 활용한다. 따라서 MySql과 같은 실제 환경에서는 추가적인 설정이 필요하다.

### Gradle 설정

```groovy
    implementation 'org.springframework.boot:spring-boot-starter-batch'
    testImplementation 'org.springframework.batch:spring-batch-test'
```

배치 설정을 위한 Spring Batch와 Spring Quartz Gradle 추가

### BootApplication 설정

```java
@EnableScheduling       // 스케줄러 기능 활성화
@EnableBatchProcessing  // 배치 기능 활성화
@SpringBootApplication(scanBasePackages = {
        ...
})
public class MzBatchApplication {
    public static void main(String[] args) {
        SpringApplication.run(MzBatchApplication.class, args);
    }
}
```

BootApplication에 `@EnableBatchProcessing`추가

### application.yml 설정(MySql을 사용할 경우)

```yaml
spring:
  batch:
    jdbc:
      initialize-schema: always
```

Meta Data를 자동으로 생성해주는 설정

- Spring Batch에서 제공되는 Meta Data Schema의 이미지

![schema_serach](uploads/99314a4895a3bc56d927e90270909427/schema_serach.png)

또한 IDE에서 Files 검색에서 schem-로 검색하면 Spring Batch에서 생성해놓은 스키마를 찾을 수 있다.


# 예제코드
- BatchJobConfig.java
```java
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import mz.batch.QueueItemReader;
import mz.batch.domain.vo.Authority;
import mz.batch.jpa.jpo.UserJpo;
import mz.batch.jpa.repo.UserRepository;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepScope;
import org.springframework.batch.item.ItemProcessor;
import org.springframework.batch.item.ItemReader;
import org.springframework.batch.item.ItemWriter;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.List;

/**
 * Batch 기반의 일반적인 Job 생성
 */
@Slf4j
@Configuration
@RequiredArgsConstructor
public class BatchJobConfig {
    //
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final UserRepository userRepository;

    @Bean
    @Qualifier("userGuestToAdminJob")
    public Job userGuestToAdminJob() {
        //
        return jobBuilderFactory.get("userGuestToAdminJob")
                .preventRestart()
                .start(userGustToAdminStep())  // 파라미터에서 주입받은 Job 실행 설정
                .build();
    }

    @Bean
    public Step userGustToAdminStep() {
        //
        return stepBuilderFactory.get("userGustToAdminStep")
                .<UserJpo, UserJpo>chunk(10)
                .reader(userGuestItemReader())
                .processor(userGuestToAdminProcessor())
                .writer(userGuestToAdminWriter())
                .build();
    }

    /**
     * 이번 예제에서는 추가 설정이 없어서 없이도 동작을 한다.
     * 하지만 Listener에 대한 설정과 같이 `Item`에 대한 Bean 생성이 필요한 부분이 존재한다.
     * 따라서 `@Bean`은 가능한한 붙여주는 것이 좋다. 
     * 단순한 설정인 경우 `@Bean` 없이도 메서드 호출이 되어 처리되기 때문에 상관없다.
     */
    @Bean      // 원칙상으로는 전부 bean을 생성해야한다
    @StepScope // Step의 주기에 따라 새로운 빈을 생성 하기때문에 파라미터 설정때문에 할당이 늦어진다.
    public ItemReader<UserJpo> userGuestItemReader() {
        //
        List<UserJpo> updateUser = userRepository.findUserJposByBirthdayYear(2022);

        return new QueueItemReader<UserJpo>(updateUser);
    }

    @Bean
    public ItemProcessor<UserJpo, UserJpo> userGuestToAdminProcessor() {
        //
        log.info("item processor");
        return userJpo -> {
            userJpo.setAuthority(Authority.GUEST);
            return userJpo;
        };
    }

    @Bean
    public ItemWriter<UserJpo> userGuestToAdminWriter() {
        //
        log.info("Item writer in log");
        return (userRepository::saveAllAndFlush);
    }
}
```

- Scheduler.java
```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobExecutionException;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

@Slf4j
@Component
public class UserScheduler {
    //
    private JobLauncher jobLauncher;
    private Job job;

    @Autowired
    public UserScheduler(JobLauncher jobLauncher, @Qualifier("userGuestToAdminJob") Job job) {
        this.jobLauncher = jobLauncher;
        this.job = job;
    }

    @Scheduled(fixedDelay = 5 * 1000L)
    public void executeJob() {
        try {
            jobLauncher.run(
                    job,
                    new JobParametersBuilder()
                            .addString("datetime", LocalDateTime.now().toString())
                            .toJobParameters()
            );
        } catch (JobExecutionException e) {
            log.error(e.getMessage());
            e.printStackTrace();
        }
    }
}
```

# Reference
- Spring Batch 예제 : [https://cheese10yun.github.io/spring-batch-basic/](https://cheese10yun.github.io/spring-batch-basic/)
- Spring Batch Test Code : [https://cheese10yun.github.io/spring-batch-test-2/](https://cheese10yun.github.io/spring-batch-test-2/)
- 참고 블로그 : https://devfunny.tistory.com/category/NOTES/Spring Batch?page=1
- Batch 성능 관련 글 : https://cheese10yun.github.io/spring-batch-reader-performance/
