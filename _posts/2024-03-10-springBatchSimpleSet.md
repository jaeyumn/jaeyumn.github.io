---
title: "[Spring] Spring Batch v5.0 이후 기본 설정"
categories:
  - develop
---

## 개발 환경

- Java 17
- SpringBoot 3.1.9
- Spring Batch 5.1.1

<br>

## v5.0 이전의 방식과 달라진 점

1. `JobBuilderFactory`, `StepBuilderFactory`가 deprecated 되어 `JobBuilder`, `StepBuilder`를 사용해야 합니다.
2. `@EnableBatchProcessing` 대신 Configuration 클래스에 `DefaultBatchConfiguration`을 상속받아 사용합니다. 이 방식은 `@EnableBatchProcessing`과 동일하게 동작합니다. (두 방법 중 하나를 사용하면 되지만 둘 다 적용하면 안된다고 합니다.)
3. `JobRepository`, `TransactionManager`를 명시적으로 사용하는 것을 권장합니다.

<br>

## Config 예시

```java
@Slf4j
@RequiredArgsConstructor
@Configuration
public class BatchConfig extends DefaultBatchConfiguration {

    @Bean
    public Job myJob(PlatformTransactionManager transactionManager, JobRepository jobRepository) {
        return new JobBuilder("myJob", jobRepository)
                .start(myStep(transactionManager, jobRepository))
                .build();
    }

    @Bean
    public Step myStep(PlatformTransactionManager transactionManager, JobRepository jobRepository) {
        return new StepBuilder("myStep", jobRepository)
                .tasklet(tasklet(), transactionManager)
                .build();
    }

    private Tasklet tasklet() {
        log.info("Execute MyStep");
        return (contribution, chunkContext) -> RepeatStatus.FINISHED;
    }
}
```

아래와 같이 정상적으로 로그가 출력되는 것을 확인할 수 있습니다.

```
2024-03-10T20:00:40.450+09:00  INFO 1888 --- [           main] com.example.BatchConfig          : Execute MyStep
```

---

참고

- [Spring Batch Documentation / Configuring and Running a Job / Java Configuration](https://docs.spring.io/spring-batch/reference/job/java-config.html)
