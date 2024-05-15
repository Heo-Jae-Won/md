## <span style="color:#802548">_1. Dto_</span>
- paging을 하게 되면 두 개의 정보를 보내야 한다.
  - lastPage
  - List
- 두 개를 동시에 보내야 하기 때문에 새로운 response용 class를 만들어준다.

```java
@Setter
@Getter
public class ScheduleResponse {
    List<ScheduleDto> scheduleList;
    int lastPage;
}
```
## <span style="color:#802548">_2. repository_</span>
- repository는 JPA로 db를 조회해 list를 가져온다.
  - 아래와 같이 pageable interface에서 값을 가져온다. 
    - Pageable은 front에서 보내주는 값을 받아 채워넣은 것이다. 
      - 여기서는 pageNumber는 1부터 lastPage까지 변동하는 값이고, pageSize는 10이다. 

```java
int pageNumber = pageable.getPageNumber();
int pageSize = pageable.getPageSize();
```

- 기존과 다르게 setFirstResult()와 setMaxResults() method가 더 추가되었다.

```java
 List<Schedule> resultList = entityManager
                                .createQuery(sql, Schedule.class)
                                .setParameter("scheduleDepartStation",
                                                scheduleFilteringConditionDto.getScheduleDepartStation())
                                .setParameter("scheduleArriveStation",
                                                scheduleFilteringConditionDto.getScheduleArriveStation())
                                .setParameter("scheduleDate", dateTime)
                                .setFirstResult(pageNumber * pageSize)
                                .setMaxResults(pageSize)
                                .getResultList();
public List<Schedule> getScheduleList(ScheduleFilteringConditionDto scheduleFilteringConditionDto,
                        Pageable pageable) {
    String dateString = scheduleFilteringConditionDto.getScheduleDate();
    int pageNumber = pageable.getPageNumber();
    int pageSize = pageable.getPageSize();

    Instant instant = Instant.parse(dateString);
    LocalDateTime dateTime = LocalDateTime.ofInstant(instant, ZoneId.of("Asia/Seoul"));

    String sql = "select distinct sch from Schedule sch " +
                    "where sch.departStation.stationName = :scheduleDepartStation " +
                    "and sch.arriveStation.stationName = :scheduleArriveStation " +
                    "and sch.scheduleArriveDatetime >= :scheduleDate";

    List<Schedule> resultList = entityManager
                    .createQuery(sql, Schedule.class)
                    .setParameter("scheduleDepartStation",
                                    scheduleFilteringConditionDto.getScheduleDepartStation())
                    .setParameter("scheduleArriveStation",
                                    scheduleFilteringConditionDto.getScheduleArriveStation())
                    .setParameter("scheduleDate", dateTime)
                    .setFirstResult(pageNumber * pageSize)
                    .setMaxResults(pageSize)
                    .getResultList();

    return resultList;
}
```
- 이를 통해 아래 sql과 동일한 효과를 낸다.
```sql
limit PageSize offset pageNumber * pageSize 
```
- pageNumber는 0부터 시작시켜줘야 하기 때문에 아래와 같이 1을 빼준다.
```java
 .setFirstResult((pageNumber-1) * pageSize)
 ```
 
- 또한 아래와 같이 totalCount를 가져온다. 
- totalScheduleCount는 int로 했더니 casting Exception이 나서 Long으로 진행했다.
- 아마 entityManager에서 강제되는 것으로 보인다. 
```java
public int getLastPage(ScheduleFilteringConditionDto scheduleFilteringConditionDto,
                        Pageable pageable) {
        String dateString = scheduleFilteringConditionDto.getScheduleDate();
        int pageSize = pageable.getPageSize();

        Instant instant = Instant.parse(dateString);
        LocalDateTime dateTime = LocalDateTime.ofInstant(instant, ZoneId.of("Asia/Seoul"));

        String countSql = "select count(distinct sch.id) from Schedule sch " +
                        "where sch.departStation.stationName = :scheduleDepartStation " +
                        "and sch.arriveStation.stationName = :scheduleArriveStation " +
                        "and sch.scheduleArriveDatetime >= :scheduleDate";

        Long totalScheduleCount = entityManager
                        .createQuery(countSql, Long.class)
                        .setParameter("scheduleDepartStation",
                                        scheduleFilteringConditionDto.getScheduleDepartStation())
                        .setParameter("scheduleArriveStation",
                                        scheduleFilteringConditionDto.getScheduleArriveStation())
                        .setParameter("scheduleDate", dateTime)
                        .getSingleResult();

        int lastPage = totalScheduleCount % pageSize == 0 ? (int) Math.ceil(totalScheduleCount / pageSize)
                        : (int) Math.ceil(totalScheduleCount / pageSize) + 1;

        return lastPage;
}
```
## <span style="color:#802548">_3. Service_</span>
- list의 type을 entity에서 dto로 바꾼다. 
- db 정보를 노출시키지 않기 위함이다. 
- lastPage는 그냥 그대로 보내도 된다. 보안 위험이 없다.
```java
 public List<ScheduleDto> getScheduleList(ScheduleFilteringConditionDto scheduleFilteringConditionDto,
            Pageable pageable) {
    List<Schedule> resultList = scheduleRepository.getScheduleList(scheduleFilteringConditionDto, pageable);
    List<ScheduleDto> ScheduleDtoList = resultList.stream()
            .map(schedule -> ScheduleDto.toDTO(schedule))
            .collect(Collectors.toList());

    return ScheduleDtoList;
}

public int getLastPage(ScheduleFilteringConditionDto scheduleFilteringConditionDto,
        Pageable pageable) {
    int lastPage = scheduleRepository.getLastPage(scheduleFilteringConditionDto, pageable);

    return lastPage;
}
```
## <span style="color:#802548">_4. Controller_</span>
- Controller에서는 보내고자하는 list와 lastPage를 담은 dto class를 만들어서 보낸다.
- controller method의 인수로 Pageable interface를 넣게 되면 특수한 인자들을 front에서 보내줘야 한다.
- 그건 아래에서 살펴보도록 하자. 
- 아래 @ModelAttiribute는 사실 빼도 된다.
```java
 @GetMapping("/api/schedule")
    public ScheduleResponse getScheduleListByScheduleFilteringConditionDto(
            @ModelAttribute ScheduleFilteringConditionDto scheduleFilteringConditionDto,
            Pageable Pageable) {
        ScheduleResponse scheduleResponse = new ScheduleResponse();
        scheduleResponse.setScheduleList(scheduleService.getScheduleList(scheduleFilteringConditionDto, Pageable));
        scheduleResponse.setLastPage(scheduleService.getLastPage(scheduleFilteringConditionDto, Pageable));

        return scheduleResponse;
    }
```
- front에서 보내는 정보에는 아래가 포함된다.
  - page
  - size
  - sort
- 모두 Pageable interface에 필요하다.
- 그 외는 ScheduleFilteringConditionDto class에 들어갈 requestParam이다.


## <span style="color:#802548">_5. Axios Request_</span>
- axios request는 아래와 같이 page, size, sort를 담아서 보낸다. 
- 저 3개가 Pageable에서 필요로 하는 parameter다. 
- scheduleDate는 내 비즈니스 로직에 필요한 데이터다.
```javascript
const schedulePaginationRequest = ref<SchedulePaginationRequest>({
    ...scheduleRequest.value,
    scheduleDate: new Date(scheduleRequest.value.scheduleDate).toISOString(),
    page: page.value,
    size: 10,
    sort: 'desc',
});
```
- ISOstring으로 온다면 LocalDateTime으로 바꿀 떄 아래와 같이 해주면 된다.
```java
LocalDateTime dateTime = LocalDateTime.parse(dateString.replace(" ", "T"));
```
- 그냥 string으로 받아서 처리하려면 아래와 같이 변환하지 않고 보내면 된다.
- 대신 처리법이 instant class를 활용하지 못하게 되어 다른 방식으로 처리해야 한다.
```javascript
const schedulePaginationRequest = ref<SchedulePaginationRequest>({
      ...scheduleRequest.value,
      page: page.value,
      size: 10,
      sort: 'desc',
    });
```


- page.value가 변동했을 때, 이를 인지시켜 값을 변경하려면 아래와 같이 watchEffect()에 넣어주면 된다.
- 그냥 setup()에서만 위와 같이 쓰고 놔두면 axios로 보낼때 page값이 변경되지 않는다.
- 그리고 sync 옵션을 달아줘야만 원하는대로 paging이 작동한다. 
- 아니면 처음에 다음페이지를 누를 때 값이 증가하지 않는 것을 볼 수 있다.
```javascript
   watchEffect(
      () => {
        schedulePaginationRequest.value = {
          ...schedulePaginationRequest.value,
          page: page.value,
        };
      },
      {
        flush: 'sync',
      }
    );
```
## <span style="color:#802548">_6. emit_</span>
- paging 기능을 담당하는 component와 paging 표시를 담당하는 component가 분리된 경우, emit을 활용할 수 있다.  
​

- 우선 아래와 같이 page값 증감에 따른 fetch를 실행하는 함수를 만든다.
```javascript
 const nextPage = () => {
      ++page.value;
      fetchSchedule();
};

const prevPage = () => {
    --page.value;
    fetchSchedule();
};
```
그 함수들을 아래와 같이 child component에 emit으로 보낸다. page와 last 값도 표시해줘야 하기 때문에 props로 보낸다.
```javascript
 <schedule-result :scheduleList="scheduleList" :headcountInfo="headcountInfo" @prevPage="prevPage" @nextPage="nextPage" :page="page" :last="last" />
 ```
그럼 아래와 같이 child component에서 data를 props로 꺼내 쓸 수 있다. 첫 번째 button의 경우, 클릭 시에 goPrevPage라는 함수가 발동되는데, 그럼 parent가 갖고 있는 nextPage 함수가 실행된다. 
```javascript
 <v-btn-group class="btn-cover">
      <v-btn :disabled="page <= 1" @click="goPrevPage" class="page-btn">이전</v-btn>
      <span class="page-count">{{ page }} / {{ last }} 페이지</span>
      <v-btn :disabled="page >= last" @click="goNextPage" class="page-btn">다음</v-btn>
    </v-btn-group>
//아래는 script 
export default defineComponent({
  name: 'ScheduleResult',
  props: {
    scheduleList: {
      type: Array as PropType<Array<ScheduleResponse>>,
      required: true,
    },
    headcountInfo: {
      type: Object as PropType<HeadCount>,
      required: true,
    },
    page: {
      type: Number,
      required: true,
    },
    last: {
      type: Number,
      required: true,
    },
  },
 setup(props) {
    const { proxy } = getCurrentInstance() as ComponentInternalInstance;
   
   function goNextPage() {
      proxy?.$emit('nextPage');
    }

    function goPrevPage() {
      proxy?.$emit('prevPage');
    }
```
- 그럼 paging이 완성된다. 