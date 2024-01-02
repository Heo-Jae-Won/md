- Identity 방식(auto_increment)은 jpa batch insert를 활용할 수 없어서 jdbcTemplate을 활용해 write했다. 
- 그런데 나중에 보니 Train entity는 jdbcTemplate 방식으로 안 만들어도 되었다. 
- 하지만 bulk insert의 경우 jdbcBatchItemWriter가 성능 이득이 매우 크다고 하여 JpaItemWriter는 굳이 구현하지 않을 예정이다. 

- JpaItemWriter는 bulk insert를 하는 게 아니라 insert 쿼리를 한번에 모아서 날리는 방식이라고 한다. 
- 결국 bulk insert를 하기 위해서는 jdbc를 활용해야한다.

​

## <span style="color:#802548">_1. Job_</span>
- Job은 configuration안에서 bean으로 규정된다. 
- batch와 관련된 class를 활용하려면 @EnableBatchProcessing를 달아줘야 한다. 
- config class에 달아도 되고, main class에 달아도 된다.

```java
@Configuration
@RequiredArgsConstructor
@EnableBatchProcessing
@Slf4j
public class BatchJobConfig {
  @Bean
  public Job flowJob() throws Exception
  .
  .
}
```
- Job은 JobBuilderFactory class를 사용하여 생성하며, 복수의 step으로 구성되는 경우가 다수다.
```java
private final JobBuilderFactory jobBuilderFactory;
```
- get안에 담긴 문자열이 바로 Job의 이름이 된다. 1번 step을 start하고 성공한다면 계속 next로 넘어가게 된다.
```java
@Bean
public Job flowJob() throws Exception {
    Job flowJob = jobBuilderFactory.get("flow")
            .start(trainStep()) // trainStep을 시작시킨다.
            .next(scheduleStep())
            .next(standardSeatStep())
            .next(executiveSeatStep())
            .build();

    return flowJob;
}
```
- meta table 중 BATCH_JOB_INSTANCE를 참고하면 job의 name이 출력된다.
- meta table 중 BATCH_JOB_EXECUTION를 참고하면 job의 실행결과가 출력된다.


## <span style="color:#802548">_2. Step_</span>
​

- Step은 chunk-oriented tasklet(이하 chunk)와 simple tasklet(이하 tasklet)으로 구성되는데, 나는 chunk를 사용했다.
- Step은 StepBuilderFactory class를 사용해 생성한다. 

```java
@Bean
@JobScope 
    public Step trainStep(/* @Value("#{jobParameters[requestDate]}") String requestDate */) throws Exception {
.
.
.

        return trainStep;
    }
```

- @JobScope은  Job parameter를 매개변수로 사용하려면 반드시 있어야 하는 annotation이다. 
- job parameter에 의미있는 값을 넣으려면 @Value로 넣을 수 있으며 위와 같은 방식으로 사용된다. 
- 다만 jobParameter로 받을 수 있는 type은 정해져 있어 그 외의 것을 구현하려면 직접 custom으로 구현해야 한다. 

​

https://hodolman.com/17

 

- Job과 마찬가지로 get안의 문자열이 곧 step의 이름이다.  
- step 중 chunk는 read-processor-writer로 구성된다. 
- 참고로 chunk와 simple tasklet은 섞어서 쓸 수 없다. 
- generic의 첫번째는 reader가 return해야하는 type이며, 두번쨰는 writer가 return해야하는 type이다. 
- read나 write는 100개씩 단위로 몰아서 이뤄진다.

```java
  Step trainStep = stepBuilderFactory.get("trainStep")
                .<Train, Train>chunk(100)
                .reader(new TrainItemReader(jdbcTemplate))
                .writer(new TrainItemWriter(dataSource))
                .build();
```
- processor는 필수가 아니다. 
- read한 데이터를 조작할 필요가 있을 때만 사용하면 된다. 
- 그래서 주석처리 되어있다.

```java
@Bean
@JobScope
public Step scheduleStep() {
    Step ScheduleStep = stepBuilderFactory.get("scheduleStep")
            .<Schedule, Schedule>chunk(100)
            .reader(new ScheduleItemReader(jdbcTemplate))
            //.process(new ScheduleItemProcessor())  
            .writer(new ScheduleItemWriter(dataSource))
            .build();

    return ScheduleStep;
}
```
- @JobScope는 job parameter를 활용할 때 필요한 것이다. 
- 사실 지금은 Job parameter를 인자로 이용하거나 하지 않기 때문에 필요가 없다. 

```java
@Bean
@JobScope
public Step standardSeatStep() {
    Step seatStep = stepBuilderFactory.get("StandardSeatStep")
            .<Seat, Seat>chunk(100)
            .reader(new StandardSeatItemReader(jdbcTemplate))
            .writer(new SeatItemWriter(dataSource))
            .build();

    return seatStep;
}
```
- BATCH_STEP_EXECUTION에 step의 실행결과들이 기록된다.


## <span style="color:#802548">_3. chunk - Reader_</span>
- Reader는 데이터를 읽어온다. 
  - 내 경우에는 db에서 읽어오는 entity가 아니었다.
  - 내가 Java Object로 생성해서 db에 저장해 둘 entity였다. 
- class 단위로 따로 빼오게 되면 @Component로 bean 등록을 해준다. 
- reader라서 ItemReader를 implements해준다. 
- return type은 Train entity여야 하므로 generic으로 type을 한정한다. 
- 또한 step을 시작하기 전에 미리 해야하는 작업이 있을 때는 StepExecutionListener class를 implements한다.

```java
@Component
@StepScope
@RequiredArgsConstructor
@Slf4j
public class TrainItemReader implements ItemReader<Train>, StepExecutionListener {

}
```
- StepExecutionListener class를 implements하면 beforeStep() method를 override할 수 있다. 
- override한 beforeStep()을 활용해 db에서 값을 가져와 batch read를 시작하기 전에 특정 데이터를 할당할 수 있다. 
- 내 경우에는 train_no column 최대값을 가져와서 그 이후부터 늘리는 전략이었다. 
- 그래야 중복되지 않는 값이 batch로 만들어지기 때문이었다. 
```java
@BeforeStep
    public void beforeStep(StepExecution stepExecution) {
        this.maxCount = jdbcTemplate.queryForObject("SELECT ifnull(max(train_no),0) FROM train_info", Integer.class);
        log.info("Max count: {}", maxCount);
    }
```
- Train entity에서 i를 더하지 않고  i+1을 더하는 이유는 이전 배치 item의 최종값과 그 다음 배치 item의 초기값이 겹쳐서 만들어지지 않게 하기 위함이다. 
- i는 0부터 시작시키고, 특정 카운트 미만까지 반복하게 만들면 0< =i < 특정 count가 되기 때문에 딱 count만큼의 length를 만들 수 있다.  
```java
@Override
    public Train read() {
        if (i < TRAINCOUNT) {
            Train train = new Train(maxCount + (i + 1));
            i++;

            return train;
        } else {
            return null;
        }
    }
```
- 아래는 최종본이다.
```java
@Component
@StepScope
@RequiredArgsConstructor
@Slf4j
public class TrainItemReader implements ItemReader<Train>, StepExecutionListener {

    private final JdbcTemplate jdbcTemplate;
    public static final int TRAINCOUNT = 600;

    private int maxCount;

    private int i = 0;

    @BeforeStep
    public void beforeStep(StepExecution stepExecution) {
        this.maxCount = jdbcTemplate.queryForObject("SELECT ifnull(max(train_no),0) FROM train_info", Integer.class);
        log.info("Max count: {}", maxCount);
    }

    @Override
    public Train read() {
        if (i < TRAINCOUNT) {
            // i+1인 이유는 이전 배치 item의 최종값과 그 다음 배치 item의 초기값이 겹치지 않기 위함이다.
            Train train = new Train(maxCount + (i + 1));
            i++;
            return train;
        } else {
            return null;
        }
    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        return null;
    }

}
```
- 이번엔 좀 더 복잡한 reader의 예시를 보자. 
- 날짜를 가져오기 위해 아래와 같이 max값을 가져온다. 
- queryForObject는 row가 1개만 나오는 경우에 사용할 수 있다.
```java
LocalDateTime maxDateTime = jdbcTemplate.queryForObject("select ifnull(max(schedule_arrive_datetime),'2023-04-30 10:00:00') from schedule_info",LocalDateTime.class);
```
- schedule_arrive_datetime column은 date type column이다.
  - 따라서 Integer class로 required class를 지정할 수 없다.
  - 반드시 LocalDateTime class로 지정되어야 한다. 
  - 그렇지 않으면 TypeMismatchDataAccess Exception이 throw된다. 
    - 그렇다면 어떻게 int값으로 가져올 수 있는가? 
    - LocalDateTime class의 method --getYear(), getDayOfMonth(), getMonthValue()--를 활용하면 된다. 

- 합치면 아래와 같이 쓸 수 있다.
```java
@Component
@StepScope
@RequiredArgsConstructor
public class ScheduleItemReader implements ItemReader<Schedule>, StepExecutionListener {

        private final JdbcTemplate jdbcTemplate;

        int i = 0;
        int maxMonth;
        int maxDay;
        int maxYear;
        String[] stationNameArray = { "수서", "동탄", "오송", "신경주", "부산", "공주", "목포" };
.
.
.
@Override
        public void beforeStep(StepExecution stepExecution) {

                // 5월부터 시작이면 4월 말일로 날짜를 지정. day에서 1을 더해줄 것이기 때문이다.
                LocalDateTime maxDateTime = jdbcTemplate.queryForObject(
                                "select ifnull(max(schedule_arrive_datetime),'2023-04-30 10:00:00') from schedule_info",
                                LocalDateTime.class);
                this.maxYear = maxDateTime.getYear();
                this.maxDay = maxDateTime.getDayOfMonth();
                this.maxMonth = maxDateTime.getMonthValue();
        }
}
```

- read에서는 내가 원하는 방식으로 batch를 이뤄내기 위해 chunk 단위인 100을 단위로 값을 변화시켰다. 
- 아래와 같은 방식으로 i/100을 int로 지정하면 정수 부분만 가져와지기 때문에 100 단위로 값을 바꾸는 게 가능하다. 
- 정확히는 배열이기 때문에 0<=i<=99(100개)면 배열의 0번째 index값을 가져오게 된다. 
- 100<=i<=199(그 다음 100개)면 배열의 1번째 index값을 가져오게 된다. 

```java
 int hundredNumberIndex = i / 100;
Station departStation = new Station(stationNameArray[hundredNumberIndex]);
Station arriveStation = new Station(stationNameArray[hundredNumberIndex + 1]);
ScheduleDto scheduleDto = ScheduleDto.builder()
                .scheduleArriveDateTime(scheduleTimeArray[hundredNumberIndex][0].plusDays(i % 100 + 1))
                .scheduleDepartDateTime(scheduleTimeArray[hundredNumberIndex][1].plusDays(i % 100 + 1))
```
- 배열을 사용할 때는 시작하는 i값은 0으로 고정시킨다고 생각하고, 다른 것들을 조정해준다. 
- 아래처럼 i에 +1을 주어 이전 batch item의 최종값과 다음 batch item의 최초값이 겹치지 않게 해주는 작업이 필요하다.

```java
Train train = new Train(i + 1);
```
- plusDays에 1을 더해주는 방식을 취하는 이유는 위의 beforeStep에서 maxDay에 1을 더하는 경우, 4월 30일을 가져왔을 때 maxDay가 30이되는데, 여기서 1을 더하면 31일이 된다. 
- 그런데 4월 31일은 존재하지 않으므로 Date관련 exception이 throw된다. plusDays에 1을 더하면 알아서 5월 1일로 넘어가게 처리된다. 
- 따라서 LocalDateTime의 method에 일을 위임한다고 생각하면 된다. 
```java
scheduleDepartDateTime(scheduleTimeArray[hundredNumberIndex][1].plusDays(i % 100 + 1))
```
- 내가 원하는 수만큼만 읽으려면 특정 수 이상에서는 read를 멈춰야한다. 
- 0<=i<특정count 구간만큼 반복되기 때문에 특정 count만큼 반복되는 효과를 발생시킨다. 
```java
if (i >= TrainItemReader.TRAINCOUNT) {
    return null;
}
 @Override
public Schedule read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {

    // read문 밖에 있으면 maxYear, maxMonth, maxDay가 계속 0으로 초기화되므로 read()안에 넣는다. 긴 if문 대신 2차원배열로 간소화한다.
    LocalDateTime[][] scheduleTimeArray = {
                    { LocalDateTime.of(maxYear, maxMonth, maxDay, 12, 00, 00),
                                    LocalDateTime.of(maxYear, maxMonth, maxDay, 14, 00, 00) },
                    { LocalDateTime.of(maxYear, maxMonth, maxDay, 14, 00, 05),
                                    LocalDateTime.of(maxYear, maxMonth, maxDay, 16, 00, 05) },
                    { LocalDateTime.of(maxYear, maxMonth, maxDay, 16, 00, 15),
                                    LocalDateTime.of(maxYear, maxMonth, maxDay, 20, 00, 15) },
                    { LocalDateTime.of(maxYear, maxMonth, maxDay, 20, 00, 25),
                                    LocalDateTime.of(maxYear, maxMonth, maxDay, 22, 00, 25) },
                    { LocalDateTime.of(maxYear, maxMonth, maxDay, 16, 00, 35),
                                    LocalDateTime.of(maxYear, maxMonth, maxDay, 18, 00, 35) },
                    { LocalDateTime.of(maxYear, maxMonth, maxDay, 18, 00, 55),
                                    LocalDateTime.of(maxYear, maxMonth, maxDay, 20, 00, 55) }
    };

    // 일정은 만들어지는 기차번호 갯수 이상 존재할 수 없다.
    if (i >= TrainItemReader.TRAINCOUNT) {
            return null;
    }

    //100개를 단위로 저장해야 할 값이 변화한다.
    int hundredNumberIndex = i / 100;
    Train train = new Train(i + 1);
    Station departStation = new Station(stationNameArray[hundredNumberIndex]);
    Station arriveStation = new Station(stationNameArray[hundredNumberIndex + 1]);
    ScheduleDto scheduleDto = ScheduleDto.builder()
                    .scheduleArriveDateTime(scheduleTimeArray[hundredNumberIndex][0].plusDays(i % 100 + 1))
                    .scheduleDepartDateTime(scheduleTimeArray[hundredNumberIndex][1].plusDays(i % 100 + 1))
                    .build();
    Schedule schedule = new Schedule(scheduleDto, train, departStation, arriveStation);
    i++;

    return schedule;
}
```
- 아래는 scheduleReader의 최종본이다.
```java
@Component
@StepScope
@RequiredArgsConstructor
public class ScheduleItemReader implements ItemReader<Schedule>, StepExecutionListener {

        private final JdbcTemplate jdbcTemplate;

        int i = 0;
        int maxMonth;
        int maxDay;
        int maxYear;
        String[] stationNameArray = { "수서", "동탄", "오송", "신경주", "부산", "공주", "목포" };

        @Override
        public Schedule read()
                        throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {

                // read문 밖에 있으면 maxYear, maxMonth, maxDay가 계속 0으로 초기화되므로 read()안에 넣는다. 긴 if문 대신 2차원배열로 간소화한다.
                LocalDateTime[][] scheduleTimeArray = {
                                { LocalDateTime.of(maxYear, maxMonth, maxDay, 12, 00, 00),
                                                LocalDateTime.of(maxYear, maxMonth, maxDay, 14, 00, 00) },
                                { LocalDateTime.of(maxYear, maxMonth, maxDay, 14, 00, 05),
                                                LocalDateTime.of(maxYear, maxMonth, maxDay, 16, 00, 05) },
                                { LocalDateTime.of(maxYear, maxMonth, maxDay, 16, 00, 15),
                                                LocalDateTime.of(maxYear, maxMonth, maxDay, 20, 00, 15) },
                                { LocalDateTime.of(maxYear, maxMonth, maxDay, 20, 00, 25),
                                                LocalDateTime.of(maxYear, maxMonth, maxDay, 22, 00, 25) },
                                { LocalDateTime.of(maxYear, maxMonth, maxDay, 16, 00, 35),
                                                LocalDateTime.of(maxYear, maxMonth, maxDay, 18, 00, 35) },
                                { LocalDateTime.of(maxYear, maxMonth, maxDay, 18, 00, 55),
                                                LocalDateTime.of(maxYear, maxMonth, maxDay, 20, 00, 55) }
                };

                // 일정은 만들어지는 기차번호 갯수 이상 존재할 수 없다.
                if (i >= TrainItemReader.TRAINCOUNT) {
                        return null;
                }

                //100개를 단위로 저장해야 할 값이 변화한다.
                int hundredNumberIndex = i / 100;
                Train train = new Train(i + 1);
                Station departStation = new Station(stationNameArray[hundredNumberIndex]);
                Station arriveStation = new Station(stationNameArray[hundredNumberIndex + 1]);
                ScheduleDto scheduleDto = ScheduleDto.builder()
                                .scheduleArriveDateTime(scheduleTimeArray[hundredNumberIndex][0].plusDays(i % 100 + 1))
                                .scheduleDepartDateTime(scheduleTimeArray[hundredNumberIndex][1].plusDays(i % 100 + 1))
                                .build();
                Schedule schedule = new Schedule(scheduleDto, train, departStation, arriveStation);
                i++;

                return schedule;
        }

        @Override
        public void beforeStep(StepExecution stepExecution) {

                // 5월부터 시작이면 4월 말일로 날짜를 지정. day에서 1을 더해줄 것이기 때문이다.
                LocalDateTime maxDateTime = jdbcTemplate.queryForObject(
                                "select ifnull(max(schedule_arrive_datetime),'2023-04-30 10:00:00') from schedule_info",
                                LocalDateTime.class);
                this.maxYear = maxDateTime.getYear();
                this.maxDay = maxDateTime.getDayOfMonth();
                this.maxMonth = maxDateTime.getMonthValue();
        }

        @Override
        public ExitStatus afterStep(StepExecution stepExecution) {
                return null;
        }

}
```
## <span style="color:#802548">_4. chunk - Writer_</span>
- db에 저장하거나 혹은 파일로 저장할 수도 있다. 나는 db에 저장하였다. 
- 다량을 효율적으로 저장하기 위해 bulk insert를 사용했다. 
- jpa로는 identity 전략에 따른 bulk insert를 수행할  없어 JdbcBatchItemWriter를 활용했다. 

```java
JdbcBatchItemWriter<Train> writer = new JdbcBatchItemWriter<>();
```
- sql을 써야하는데, insert문은 sql이 일반적인 형태가 아님. values 구문 이전까지는 sql이고, values 구문은 JPQL이 섞여있다. 

```java
writer.setSql("INSERT INTO train_info (train_no) VALUES (:trainNo)");
```
- JPQL은 field와 column을 mapping시키는 작업이 필요한데, field와 column을 mapping시키는 것이 바로 setItemSqlParameterSourceProvider class다. 
```java
writer.setItemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider<>());
```

- 소스코드에 직접 넣는 sql이 늘 그렇듯, 띄어쓰기에 유념해주어야 한다. 여기선 짧아서 그렇지 않았다.
```java
@Component
@StepScope
@RequiredArgsConstructor
public class TrainItemWriter implements ItemWriter<Train> {

    private final DataSource dataSource;

    /**
     * @description: values 이전은 sql같이, values 안은 jpql같이 쓴다. field와 column을
     *               mapping시키는 것이 바로 setItemSqlParameterSourceProvider class다.
     */
    @Override
    public void write(List<? extends Train> items) throws Exception {
        JdbcBatchItemWriter<Train> writer = new JdbcBatchItemWriter<>();
        writer.setDataSource(dataSource);
        writer.setSql("INSERT INTO train_info (train_no) VALUES (:trainNo)");
        writer.setItemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider<>());
        writer.afterPropertiesSet();
        writer.write(items);
    }
}
```
- 하지만 문장이 길어져 두 개로 나뉘게 되면 한 문장이 끝날 때 띄어쓰기를 한번하고 다음 문장을 시작해주어야 한다.
```java
writer.setSql(
                "INSERT INTO seat_info (seat_carriage, seat_class, seat_no, seat_sold, train_no) "
                        +
                        "VALUES ( :seatCarriage, :seatClass, :seatNo, :seatSold, :train.trainNo)");
```
## <span style="color:#802548">_5. flow_</span>
​

- 복수 개의 step으로 구성되는 아래의 형태를 flow라고 한다. 
```java
@Bean
public Job flowJob() throws Exception {
    Job flowJob = jobBuilderFactory.get("flow")
            .start(trainStep()) // trainStep을 시작시킨다.
            .next(scheduleStep())
            .next(standardSeatStep())
            .next(executiveSeatStep())
            .build();

    return flowJob;
}
```
- 특정 step을 건너띄는 기능도 있다. 중요한 것은 tasklet과 chunk를 섞어쓰지 않는 것이다. 
- 둘은 서로 호환되지 않는다.
```java
 @Bean
public Job stepNextConditionalJob() {
    return jobBuilderFactory.get("stepNextConditionalJob")
            .start(trainStep())
                .on("FAILED") // FAILED 일 경우
                .to(failedStep()) // failedStep으로 이동한다.
                .on("*") // failedStep의 결과 관계 없이 
                .end() // failedStep으로 이동하면 Flow가 종료한다.
            .from(trainStep()) // trainStep로부터
                .on("*") // FAILED 외에 모든 경우
                .to(scheduleStep()) // scheduleStep으로 이동한다.
                .next(standardSeatStep()) // scheduleStep가 정상 종료되면 standardSeatStep으로 이동한다.
                .on("*") // standardSeatStep의 결과 관계 없이 
                .end() // standardSeatStep으로 이동하면 Flow가 종료한다.
            .end() // Job 종료
            .build();
}
```
## <span style="color:#802548">_6. JobLauncher_</span>
- job parameter를 가지고 job을 시작시킨다. 
- commanLine에 직접 적지 않고 소스코드로 적는다. 
- 위에 적었듯 Job parameter으로 LocalDateTime은 불가능하다. 
- int도 마찬가지다. 따라서 현재날짜를 Long type(System.currentTimeMillis())으로 받고 String으로 변환한다. 
```java
JobParameters jobParameters = new JobParametersBuilder()
                .addString("currentTime", String.valueOf(System.currentTimeMillis()))
                .toJobParameters();
```
- commanLine에 적는 것들도 있다. 아래 args 옵션을 쓰는 법은 vscode에 한정된다. 
- eclipse의 경우에는 또 다르고, intelliJ는 또 다를 것이다.
```java
"args": ["--spring.profiles.active=local","--job.name=flow"]
```
- 위와 같이 args로 job.name을 적어두고 properties 파일에 아래 옵션을 주면 programm args로 주지 않은 job은 활성화되지 않는다. 
- 복수 개의 job을 관리하는데 그 중 한개만 필요한 경우라면 유용한 옵션이다.
```
//properties 파일
spring.batch.job.names=${job.name:NONE}
BATCH_JOB_EXECUTION_PARAMS를 보면 Job을 실행시킨 parameter의 key - value를 볼 수 있다.
```
 

## <span style="color:#802548">_7. meta data table_</span>
- batch의 결과, 과정, 쓰인 요소 등이 저장되는 table이다. params, step, job이 중심이 된다. 
- 그게 바로 아래 4개의 테이블 --BATCH_JOB_EXECUTION, BATCH_JOB_EXECUTION_PARAMS, BATCH_JOB_INSTANCE, BATCH_STEP_EXECUTION--이다. 
- 그 외에도 여러 테이블이 있다. 이 meta data table이 없으면 batch가 실행되지 않는다. 
- 각 rdb마다 meta data table이 조금씩 생김새가 다르다고 한다. 
- 아래는 mysql이다. 따라서 mariadb까지 가능하다. 
```sql
CREATE TABLE BATCH_JOB_INSTANCE  (
    JOB_INSTANCE_ID BIGINT  NOT NULL PRIMARY KEY ,
    VERSION BIGINT ,
    JOB_NAME VARCHAR(100) NOT NULL,
    JOB_KEY VARCHAR(32) NOT NULL,
    constraint JOB_INST_UN unique (JOB_NAME, JOB_KEY)
) ENGINE=InnoDB;

CREATE TABLE BATCH_JOB_EXECUTION  (
    JOB_EXECUTION_ID BIGINT  NOT NULL PRIMARY KEY ,
    VERSION BIGINT  ,
    JOB_INSTANCE_ID BIGINT NOT NULL,
    CREATE_TIME DATETIME NOT NULL,
    START_TIME DATETIME DEFAULT NULL ,
    END_TIME DATETIME DEFAULT NULL ,
    STATUS VARCHAR(10) ,
    EXIT_CODE VARCHAR(2500) ,
    EXIT_MESSAGE VARCHAR(2500) ,
    LAST_UPDATED DATETIME,
    JOB_CONFIGURATION_LOCATION VARCHAR(2500) NULL,
    constraint JOB_INST_EXEC_FK foreign key (JOB_INSTANCE_ID)
    references BATCH_JOB_INSTANCE(JOB_INSTANCE_ID)
) ENGINE=InnoDB;

CREATE TABLE BATCH_JOB_EXECUTION_PARAMS  (
    JOB_EXECUTION_ID BIGINT NOT NULL ,
    TYPE_CD VARCHAR(6) NOT NULL ,
    KEY_NAME VARCHAR(100) NOT NULL ,
    STRING_VAL VARCHAR(250) ,
    DATE_VAL DATETIME DEFAULT NULL ,
    LONG_VAL BIGINT ,
    DOUBLE_VAL DOUBLE PRECISION ,
    IDENTIFYING CHAR(1) NOT NULL ,
    constraint JOB_EXEC_PARAMS_FK foreign key (JOB_EXECUTION_ID)
    references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
) ENGINE=InnoDB;

CREATE TABLE BATCH_STEP_EXECUTION  (
    STEP_EXECUTION_ID BIGINT  NOT NULL PRIMARY KEY ,
    VERSION BIGINT NOT NULL,
    STEP_NAME VARCHAR(100) NOT NULL,
    JOB_EXECUTION_ID BIGINT NOT NULL,
    START_TIME DATETIME NOT NULL ,
    END_TIME DATETIME DEFAULT NULL ,
    STATUS VARCHAR(10) ,
    COMMIT_COUNT BIGINT ,
    READ_COUNT BIGINT ,
    FILTER_COUNT BIGINT ,
    WRITE_COUNT BIGINT ,
    READ_SKIP_COUNT BIGINT ,
    WRITE_SKIP_COUNT BIGINT ,
    PROCESS_SKIP_COUNT BIGINT ,
    ROLLBACK_COUNT BIGINT ,
    EXIT_CODE VARCHAR(2500) ,
    EXIT_MESSAGE VARCHAR(2500) ,
    LAST_UPDATED DATETIME,
    constraint JOB_EXEC_STEP_FK foreign key (JOB_EXECUTION_ID)
    references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
) ENGINE=InnoDB;

CREATE TABLE BATCH_STEP_EXECUTION_CONTEXT  (
    STEP_EXECUTION_ID BIGINT NOT NULL PRIMARY KEY,
    SHORT_CONTEXT VARCHAR(2500) NOT NULL,
    SERIALIZED_CONTEXT TEXT ,
    constraint STEP_EXEC_CTX_FK foreign key (STEP_EXECUTION_ID)
    references BATCH_STEP_EXECUTION(STEP_EXECUTION_ID)
) ENGINE=InnoDB;

CREATE TABLE BATCH_JOB_EXECUTION_CONTEXT  (
    JOB_EXECUTION_ID BIGINT NOT NULL PRIMARY KEY,
    SHORT_CONTEXT VARCHAR(2500) NOT NULL,
    SERIALIZED_CONTEXT TEXT ,
    constraint JOB_EXEC_CTX_FK foreign key (JOB_EXECUTION_ID)
    references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
) ENGINE=InnoDB;

CREATE TABLE BATCH_STEP_EXECUTION_SEQ (
    ID BIGINT NOT NULL,
    UNIQUE_KEY CHAR(1) NOT NULL,
    constraint UNIQUE_KEY_UN unique (UNIQUE_KEY)
) ENGINE=InnoDB;

INSERT INTO BATCH_STEP_EXECUTION_SEQ (ID, UNIQUE_KEY) select * from (select 0 as ID, '0' as UNIQUE_KEY) as tmp where not exists(select * from BATCH_STEP_EXECUTION_SEQ);

CREATE TABLE BATCH_JOB_EXECUTION_SEQ (
    ID BIGINT NOT NULL,
    UNIQUE_KEY CHAR(1) NOT NULL,
    constraint UNIQUE_KEY_UN unique (UNIQUE_KEY)
) ENGINE=InnoDB;

INSERT INTO BATCH_JOB_EXECUTION_SEQ (ID, UNIQUE_KEY) select * from (select 0 as ID, '0' as UNIQUE_KEY) as tmp where not exists(select * from BATCH_JOB_EXECUTION_SEQ);

CREATE TABLE BATCH_JOB_SEQ (
    ID BIGINT NOT NULL,
    UNIQUE_KEY CHAR(1) NOT NULL,
    constraint UNIQUE_KEY_UN unique (UNIQUE_KEY)
) ENGINE=InnoDB;

INSERT INTO BATCH_JOB_SEQ (ID, UNIQUE_KEY) select * from (select 0 as ID, '0' as UNIQUE_KEY) as tmp where not exists(select * from BATCH_JOB_SEQ)
```

- BATCH_JOB_EXECUTION에서 JOB_EXECUTION_ID column은 실행시킨 job이다. 
- 실행시킨 job이 실패할 경우 JOB_EXECUTION_ID 값은 auto_increment로 계속 늘어나지만, JOB_INSTANCE_ID colunmn의 값은 똑같다. 
- 실패한 Job Parameter를 지닌 Job instance를 다시 시도하는 것이기 때문이다. 실제로 STATUS column의 값이 FAILED인 것을 볼 수 있다. 

​

 

- BATCH_JOB_EXECUTION_PARAMS는 Job_instance_id column이 없다. 실행된 job(JOB_EXECUTION_ID)의 job params를 기록하기 때문이다. 


- BATCH_JOB_INSTANCE은 job의 이름과 job parameter를 지닌 job_instance를 기록한다. 즉 job이름을 지닌 최상위 job이 있고, 그를 구현하기 위해 job parameter를 지닌 job_instance가 있으며, 그 job_instance의 실행결과(실패, 성공, 실행중)인 job execution이 있다. 


- BATCH_STEP_EXECUTION은 job_instance를 구성하는 하위 단계들이다. job 1개에 여러 step이 엮여있다. 


## <span style="color:#802548">_8. Spring Scheduler_</span>
- 소스코드를 고치고 저장하면 tomcat이 재시작하면서 batch도 실행돼서 짜증날 것이다. 
- scheduler를 사용하면 원하는 때에 batch를 실행시킬 수 있다. 
- 시작시키기 위해서는 위에서 만들었던 JobLaucher를 가져와서 run시킨다. 

​

- cron은 아래와 같이 읽으면 된다.

- 초-분-시-일-월-년. 시는 24시간 단위. 

- 모든 요일 6:30 AM, 9:30PM에 실행: 0 30 6,21 * * *

- 모든 요일 21PM,22PM,23PM, 0AM에 실행: 0 0 21-00 * * *

- 매년 8월 25일 9AM에 실행: 0 0 9 25 8 *
```java
@Scheduled(cron = "15 40 10 * * *")
    public void runBatchJob() {
        JobParameters jobParameters = new JobParametersBuilder()
                .addString("currentTime", String.valueOf(System.currentTimeMillis()))
                .toJobParameters();

        // 해당 job parameter를 지닌 job을 실행시킨다.
        try {
            JobExecution execution = jobLauncher.run(flowJob(), jobParameters);
            log.info("Exit Status: " + execution.getStatus());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```
- scheduler를 쓴다는 것은 처음 application을 running할 때 batch를 돌리고 싶지 않다는 의미일 것이다. 
- 그렇다면 아래 옵션을 properties 파일에 부여한다.
```
spring.batch.job.enabled=false
```