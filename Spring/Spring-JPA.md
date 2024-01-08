## <span style="color:#802548">_0. h2 install_</span>
   <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
	</dependency>
	<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<scope>runtime</scope>
	</dependency>
    <dependency>
        <groupId>com.github.gavlyukovskiy</groupId>
        <artifactId>p6spy-spring-boot-starter</artifactId>
        <version>1.8.0</version>
</dependency>

- 그 다음으로 in-memory 방식이 있는데, 이것은 RAM에 저장되는 형식이라서 컴퓨터를 끄면 데이터가 사라진다. 
- 대신 통신이 통하지 않아도 상관없다. 컴퓨터의 메모리 영역에 저장되기 때문이다. 
```
spring.datasource.url=jdbc:h2:mem:test
```
url은 위와 같이 써주면 된다. tcp 방식과 in-memory는 서로 데이터가 공유되지 않는다. 따라서 tcp 접속으로 진행할 지 in-memory로 진행할 지 정해두고 하는 게 좋다. 


위의 옵션을 아무것도 주지 않으면 기본적으로 in-memory 방식이 default가 된다. 
- 로깅정책은 local의 경우, 큰 변화가 없으면 update를 많이 사용한다고 한다. 
- 변화를 주어야 한다면 create-drop을 사용한다.
- create나 create-drop은 Spring boot를 끄면 테이블도 날라가지만, update는 테이블이 날라가지 않는 차이가 있다. 
```
spring.jpa.hibernate.ddl-auto=update
spring.jpa.generate-ddl=true
spring.jpa.database=h2
```
- 보통의 JPA는 로그를 찍으면 parameter 값이 안나와서 특별한 처리가 필요하다. 
- 하지만 아래 dependency를 넣으면 그러한 처리 없이 알아서 잘 처리가 진행된다. 
- 참고로 아래 dependency를 쓰면 쿼리가 2번 출력되는데, 하나는 파라미터가 없는 것이고 파라미터가 있는 것이다. 비교를 위해 2번 출력되는 것이지 실제 쿼리가 2번 날아가는 것은 아니다. 

- dependency를 넣었다면 이제 아래 dependency를 이용한 loggin을 true로 바꾼다. 
- 운영에서는 false로 해두는데, 성능 문제 때문이라고 한다. 

decorator.datasource.p6spy.enable-logging= true

http://www.h2database.com/html/download.html

 

## <span style="color:#802548">_1. baseEntity_</span>
- 아래와 같이 class를 만든다. 
- 이름은 BaseEntity지만 @Entity가 붙지 않는다. 
- 대신 @MappedSuperClass가 붙어서 아래 class의 속성을 사용할 수 있다. 
```java
@Getter
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
public class BaseEntity {

    @CreatedDate
    @Column(updatable = false)
    protected LocalDateTime createDate; //생성날짜

    @LastModifiedDate
    @Column(insertable = false)
    protected LocalDateTime updateDate; //수정날짜

}
Entity class에 아래와 같이 BaseEntity를 상속해준다.

public class Member extends BaseEntity
main class에 @EnableJpaAuditing을 넣어준다.

@SpringBootApplication()
@EnableJpaAuditing
public class ReservationApplication {

	public static void main(String[] args) {
		SpringApplication.run(ReservationApplication.class, args);
	}
}
```
- 그럼 아래와 같은 형태로 들어가게 된다. entity에 column으로 만들지 않아도 자동 삽입된다.


## <span style="color:#802548">_2. Entity - mapping과 무관한 column_</span>
- 내 경우에는 아래 4개를 고정으로 쓰고 있다. 
- @NoArgsConstructor(access = AccessLevel.PROTECTED)는 인자를 받지 않는 생성자를 만들지 못하게 하는 annotation이다. 
- new로 만들 때는 반드시 인자를 받아야만 한다.
```java
@Getter
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table(name = "MEMBER_INFO")
@Id가 붙은 것은 PK고, 그 PK를 만드는 방식은 auto_increment다. 

@Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_no")
    private Long memberNo; // 회원 번호
PK를 UUID로 만드는 것도 가능하다.

 @Id
    @GeneratedValue(generator = "uuid2")
    @GenericGenerator(name = "uuid2", strategy = "uuid2")
    @Column(name = "payment_no", length = 100)
    private String paymentNo; // 결제 번호
제약조건의 경우 @Column에서 걸어줄 수 있다. length가 45면 45를 초과할 수 없다는 의미다. 45자만 받는다는 의미가 아니다.

  @Column(name = "member_id", length = 45, nullable = false, unique = true)
    private String memberId; // 회원 아이디
날짜의 경우 LocalDate이며 원하는 형태로 뽑아내려면 @JsonFormat을 활용한다.

@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd", timezone = "Asia/Seoul")
    @Column(name = "member_birth", nullable = false)
    private LocalDate memberBirth; // 회원 생년월일
날짜와 시간을 합쳐서 표기하는 것은 LocalDateTime이다.

@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss", timezone = "Asia/Seoul")
    @Column(name = "payment_date")
    @CreationTimestamp
    private LocalDateTime paymentDate; // 결제일시
enum은 아래와 같이 converter를 활용해서 db에 저장할 수 있다. 

@Column(name = "role_type", length = 2)
    @Convert(converter = RoleTypeConverter.class)
    private RoleType roleType; // 권한 유형
```
## <span style="color:#802548">_3. entity - mapping column_</span>
- table은 1:1, 1:다, 다:1, 1:1, 다:다 관계를 가지며 FK를 활용해 양방향으로 mapping이 가능하다. 
- 하지만 객체는 1:다의 관계를 구현하려면 단방향 관계를 2개를 만들어야 한다. 
- 그런데 아무 클래스에서나 값을 수정하게 하면 데이터의 무결성을 보장하기가 어렵다. 
- 따라서 연관관계의 주인을 정하고, 주인만이 수정, 삭제가 가능하게 만드는 것이다.

- 아래와 같이 mappedBy가 붙어있는 경우는 @OneToMany가 붙어있을 때이며, 이 경우 연관관계의 주인에 의해 mapping된 것이므로 연관관계의 주인이 아니다. 
- 주인이 아니므로 관리되는 참조이며 Member enttiy class에서는 아래의 Payment라는 entity를 조회만 가능하다. 
- 어쩌면 당연한 것이다. Member에서 Payment 값을 변경가능하게 하는 것은 db에서는 아예 불가능하다. 
- 불가능하게 하는 것을 Java에서도 구현한 것이라 볼 수 있다. 
```java
//Member entity class
@JsonManagedReference
@OneToMany(mappedBy = "member")
private List<Payment> payment = new ArrayList<>(); //조회용 entity
 ```
- 그럼 mappedBy에 대응되는 것은 무엇일까? 바로 Payment entity class의 field다. 
- 사실 아래는 잘못 설계된 것이긴 하다. 사실 referenceColumnName은 안 주는 게 좋다. 
- 안 주면 자동으로 PK로 참조 컬럼이 잡히게 되는데, 참조값들은 아무 의미없는 값을 넣어야 나중에 서비스에서 문제를 일으키지 않기 때문이다. 
- 나는 지금 PK가 아닌 의미있는 member_id column을 넣은 경우다. 다른 분들은 PK로 잡기를 바란다. 
```java
//Payment entity class
@JsonBackReference
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(referencedColumnName = "member_id", name = "member_id")
    private Member member; // Member entity의 FK
```
- @JsonBackReference나 @JsonManagedReference 같은 것은 mapping 관계에서 무한 참조가 일어나서 stackOverflow가 일어나는 현상을 방지하기 위한 annotation이다. 
- 연관관계의 주인을 정해주면 된다. BackReference가 주인이고, ManagedReference가 관리대상이다. 연관관계를 정해주면 더이상 무한 loop가 일어나지 않는다. 
- 즉 FK가 있는 column을 가진 entity class가 연관관계의 주인이며, 주인의 경우 @ManyToOne와 Lazy Fetch를 사용한다.
- Json을 반환하는데 무한 loop가 생긴다면 @JsonBackReference를 넣어준다. 
- join column의 이름은 @JoinColumn에서 결정한다. 그에 대응되는 관리대상은 @OneToMany와 mappedBy를 사용한다. 여기서는 @JsonManagedReference를 넣어준다. 
- 만약 여러개의 @JsonBackReference를 사용한다면 구분할 수 있게 value를 지정해줘야 한다.
```java
@JsonBackReference(value = "payment-order")
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "payment_no")
    private Payment payment;

    @JsonBackReference(value = "seat-order")
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "seat_code")
    private Seat seat; // Seat entity의 FK

    @JsonBackReference(value = "schedule-order")
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "schedule_no")
    private Schedule schedule; // Schedule entity의 FK
```
- 그리고 Managed도 어떤 @JsonBackReference와 연동되는 지를 알려주기 위해 value를 대응시켜준다.
```java
 @JsonManagedReference(value = "payment-order")
 ```
## <span style="color:#802548">_4. entity - 생성자와 method_</span>
- entity 생성자의 경우 반드시 Dto를 받아서 만들어지도록 설계했다. 
- entity는 db내용을 그대로 담고있으니 노출을 줄이는 게 좋다고 생각이 들어서이다.
```java
/**
 * @description: dto -> entity
 */
public Member(MemberDto memberDto) {
    this.memberName = memberDto.getMemberName();
    this.memberId = memberDto.getMemberId();
    this.memberPassword = memberDto.getMemberPassword();
    this.memberBirth = memberDto.getMemberBirth();
    this.memberPhone = memberDto.getMemberPhone();
    this.memberEmail = memberDto.getMemberEmail();
    this.memberAddress = memberDto.getMemberAddress();
    this.roleType = RoleType.MEMBER;
}
```
- 그럼 dto를 받아서 아래와 같이 entity를 새로 만들 수 있다. 
- entity를 new()로 만들어서 save하면 새로운 객체를 영속성 context가 인식하여 db에 값을 저장시킨다. 
- save()는 JpaRepository를 extends하여 활용한 것이다.
```java
public void save(MemberDto memberDto) {
        memberDto.encodePassword(passwordEncoder.encode(memberDto.getMemberPassword()));

        Member member = new Member(memberDto);
        try {
            memberRepository.save(member);
        } catch (DataIntegrityViolationException dataIntegrityViolationException) {
            throw new SignupException(ErrorEnum.DUPLICATED_EMAIL);
        }
    }
```
- method들의 경우 간단하게 값을 바꾸는 것만 entity에 들어있다. 
- domain 중심 설계로 만드는 것은 아직 잘 모르겠어서 값을 수정하는 method만 여기에 넣어두었다. 
```java
    /**
     * @description: 회원 정보 수정
     */
    public void update(String memberName, String memberId, LocalDate memberBirth,
            String memberPhone, String memberEmail, String memberAddress) {
        this.memberName = memberName;
        this.memberId = memberId;
        this.memberBirth = memberBirth;
        this.memberPhone = memberPhone;
        this.memberEmail = memberEmail;
        this.memberAddress = memberAddress;
    }

    /**
     * @description: 회원 비밀번호 변경.
     * @parameter: 암호문
     */
    public void updatePw(String cipherPassword) {
        this.memberPassword = cipherPassword;
    }
```
- 아래는 합본이다.
```java
@Getter
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table(name = "MEMBER_INFO")
public class Member extends BaseEntity implements Serializable{

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_no")
    private Long memberNo; // 회원 번호

    @Column(name = "member_id", length = 45, nullable = false, unique = true)
    private String memberId; // 회원 아이디

    @Column(name = "member_password", length = 100, nullable = false)
    private String memberPassword; // 회원 비밀번호

    @Column(name = "member_name", length = 50, nullable = false)
    private String memberName; // 회원 이름

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd", timezone = "Asia/Seoul")
    @Column(name = "member_birth", nullable = false)
    private LocalDate memberBirth; // 회원 생년월일

    @Column(name = "member_phone", length = 13, nullable = false)
    private String memberPhone; // 회원 전화번호

    @Column(name = "member_email", length = 45, nullable = false, unique = true)
    private String memberEmail; // 회원 이메일

    @Column(name = "member_address", length = 150, nullable = false)
    private String memberAddress; // 회원 주소

    @Column(name = "role_type", length = 2)
    @Convert(converter = RoleTypeConverter.class)
    private RoleType roleType; // 권한 유형

    @JsonManagedReference
    @OneToMany(mappedBy = "member")
    private List<Payment> payment = new ArrayList<>(); //조회용 entity

    /**
     * @description: dto -> entity
     */
    public Member(MemberDto memberDto) {
        this.memberName = memberDto.getMemberName();
        this.memberId = memberDto.getMemberId();
        this.memberPassword = memberDto.getMemberPassword();
        this.memberBirth = memberDto.getMemberBirth();
        this.memberPhone = memberDto.getMemberPhone();
        this.memberEmail = memberDto.getMemberEmail();
        this.memberAddress = memberDto.getMemberAddress();
        this.roleType = RoleType.MEMBER;
    }

    /**
     * @description: 회원 정보 수정
     */
    public void update(String memberName, String memberId, LocalDate memberBirth,
            String memberPhone, String memberEmail, String memberAddress) {
        this.memberName = memberName;
        this.memberId = memberId;
        this.memberBirth = memberBirth;
        this.memberPhone = memberPhone;
        this.memberEmail = memberEmail;
        this.memberAddress = memberAddress;
    }

    /**
     * @description: 회원 비밀번호 변경.
     * @parameter: 암호문
     */
    public void updatePw(String cipherPassword) {
        this.memberPassword = cipherPassword;
    }

}
```
## <span style="color:#802548">_5. n+1 query 문제 해결_</span>
- 그런데 연관관계를 조회할 때 n+1 문제가 날 때가 있다. 
  - 예를 들면 결제를 조회하면서 결제에 연관된 ticket을 가져올 때 결제 정보를 1개 조회하는데 결제 row의 수만큼 select ticket query가 나가는 경우이다. 
  - 그럴 때는 @BatchSize를 주어 해결한다. 
```java
@BatchSize(size = 40)
    @JsonManagedReference(value = "seat-order")
    @OneToMany(mappedBy = "seat")
    private List<Ticket> ticket = new ArrayList<>(); 
```

## <span style="color:#802548">_6. Entity와 관련하여 request와 response를 담당하는 dto_</span>
- Dto에서도 Entity로 변형될 수 있는 Dto는 @Setter를 쓰지 않았다. 
- 대신 @Builder로 값을 만들어낼 수 있다.
```java
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
Buidler의 경우 entity를 dto로 변형하는데 사용한다. 

public static MemberDto toDTO(Member entity) {
        return MemberDto.builder()
                .memberId(entity.getMemberId())
                .memberPassword(entity.getMemberPassword())
                .memberName(entity.getMemberName())
                .memberAddress(entity.getMemberAddress())
                .memberBirth(entity.getMemberBirth())
                .memberEmail(entity.getMemberEmail())
                .memberPhone(entity.getMemberPhone())
                .build();
    }
```
- 그럼 아래와 같이 front로부터 memberId 값을 받은 뒤 그 memberId로 memberInfo를 조회하고, 얻어온 entity를 dto로 다시 변환해서 front에 전달할 수 있다. 
- 따라서 db의 내용이나 구조가 노출되지 않고 같은 내용을 전달할 수 있다. 이건 entity를 dto로 변환하여  response하는 용도다.
```java
@GetMapping("/api/member/{memberId}")
    public MemberDto getMemberInfo(@PathVariable String memberId) {
        Member memberInfo = memberService.getMemberInfo(memberId);
        MemberDto memberDto = MemberDto.toDTO(memberInfo);

        return memberDto;
    }
```
- 이와 반대로 아래는 request로 들어온 dto를 활용해 entity를 만들었다.
```java
public void save(MemberDto memberDto) {
        memberDto.encodePassword(passwordEncoder.encode(memberDto.getMemberPassword()));

        Member member = new Member(memberDto);
        try {
            memberRepository.save(member);
            //DataIntegrityViolationException로 잡아서 signupException을 던져야 함. signupException을 잡으면 안됨.
        } catch (DataIntegrityViolationException dataIntegrityViolationException) {
            throw new SignupException(ErrorEnum.DUPLICATED_EMAIL);
        }
    }
```

## <span style="color:#802548">_7. entity를 만들기 위한 request만 담당하는 dto_</span>
- 아래는 entity를 만들기 위한 request를 받는 dto다. 
- request만 담당하는 경우 Request를 class 이름에 붙였다.
```java
@Getter
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class PaymentRequest {
    private String paymentType; // 결제 유형
    private int paymentCharge; // 결제 요금
    private LocalDateTime paymentDate; // 결제 일시
    private String memberId; //회원아이디

}
```
- 받은 dto는 아래에서와 같이 entity를 만드는 데 활용된다.
```java
 public Payment(PaymentRequest paymentRequest, Member memberInfo) {
        this.paymentType = paymentRequest.getPaymentType();
        this.paymentCharge = paymentRequest.getPaymentCharge();
        this.member = memberInfo;
    }
```
- 실제 생성자는 아래와 같이 Service에서 쓰인다.
```java
public String createPayment(PaymentRequest paymentRequest) {
        String memberId = paymentRequest.getMemberId();
        Member member = entityManager.createQuery(
                "SELECT m FROM Member m WHERE m.memberId = :memberId", Member.class)
                .setParameter("memberId", memberId)
                .getSingleResult();

        Payment payment = new Payment(paymentRequest, member);
        entityManager.persist(payment);

        return payment.getPaymentNo();
    }
```

## <span style="color:#802548">_8. entity를 전달하기 위한 response만 담당하는 dto_</span>
- Response만 담당하는 dto다. 
- Response의 경우 Dto class와 기본형으로 이뤄져있다. 
```java
@Getter
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class TicketResponse {
    private Long ticketNo; // 티켓 PK
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss", timezone = "Asia/Seoul")
    private LocalDateTime ticketDate; // 티켓 발행 일시
    private String paymentNo; // 결제PK
    private String trainNo; // 기차번호
    private SeatDto seatResponse; // 좌석정보
    private String arriveStationName; // 도착역
    private String departStationName; // 출발역
    private Long scheduleNo; // 일정PK
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss", timezone = "Asia/Seoul")
    private LocalDateTime departDateTime; // 출발 시간
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss", timezone = "Asia/Seoul")
    private LocalDateTime arriveDateTime; // 도착 시간

    public static TicketResponse toDto(Ticket entity) {
        return TicketResponse.builder()
                .ticketNo(entity.getTicketNo())
                .ticketDate(entity.getTicketDate())
                .paymentNo(entity.getPayment().getPaymentNo())
                .seatResponse(SeatDto.toDto(entity.getSeat()))
                .scheduleNo(entity.getSchedule().getScheduleNo())
                .departStationName(entity.getSchedule().getDepartStation().getStationName())
                .arriveStationName(entity.getSchedule().getArriveStation().getStationName())
                .trainNo(entity.getSchedule().getTrain().getTrainNo())
                .departDateTime(entity.getSchedule().getScheduleDepartDatetime())
                .arriveDateTime(entity.getSchedule().getScheduleArriveDatetime())
                .build();
    }
}
```
- 아래와 같이 entity를 받아서 Response로 바꿔서 return한다.
```java
@GetMapping("/api/ticket/{memberId}")
    public List<TicketResponse> getTicketList(@PathVariable String memberId) {
        List<Ticket> ticketEntity = ticketService.getTicketList(memberId);
        List<TicketResponse> ticketList = ticketEntity.stream()
                .map(ticket -> TicketResponse.toDto(ticket))
                .collect(Collectors.toList());

        return ticketList;
    }
```
- response로 바꾸는 이유는 Ticket entity는 다른 entity를 field로 갖고 있어 Ticket entity를 그대로 response로 전달하면 내용을 넘어 db구조가 노출될 수 있기 때문이다. 
```java
 @JsonBackReference(value = "payment-order")
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "payment_no")
    private Payment payment;

    @JsonBackReference(value = "seat-order")
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "seat_code")
    private Seat seat; // Seat entity의 FK

    @JsonBackReference(value = "schedule-order")
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "schedule_no")
    private Schedule schedule; // Schedule entity의 FK
```
## <span style="color:#802548">_9. entity를 조회하는 조건 dto_</span>
- 아래의 Dto는 그 자체로 entity 값으로 변환되지 않고, entity 조회 조건을 가져오기 위한 dto다.
```java
@Getter
@Setter //modelAttribute는 setter도 달아줘야함.
public class ScheduleFilteringConditionDto {
    private String scheduleDepartStation; //출발역
    private String scheduleArriveStation; //도착역
    private String scheduleDate; //일정날짜
    private String scheduleTime; //일정시간 //TODO: 현재 시간은 사용하지 않고 있다.
}
```
- 아래와 같이 조건 Dto를 가져와서 entity를 조회하는 데 활용한다.
```java
public List<Schedule> getScheduleList(ScheduleFilteringConditionDto scheduleFilteringConditionDto) {

        String dateString = scheduleFilteringConditionDto.getScheduleDate();

        // front에서 보내는 방식이 변경되면 parsing 방법도바뀔 수 있음
        LocalDateTime dateTime = LocalDateTime.parse(dateString.replace(" ", "T"));

        String sql = "select distinct sch from Schedule sch " +
                "where sch.departStation.stationName = :scheduleDepartStation " +
                "and sch.arriveStation.stationName = :scheduleArriveStation " +
                "and sch.scheduleArriveDatetime >= :scheduleDate";

        List<Schedule> resultList = entityManager
                .createQuery(sql, Schedule.class)
                .setParameter("scheduleDepartStation", scheduleFilteringConditionDto.getScheduleDepartStation())
                .setParameter("scheduleArriveStation", scheduleFilteringConditionDto.getScheduleArriveStation())
                .setParameter("scheduleDate", dateTime)
                .getResultList();

        return resultList;
    }
```

## <span style="color:#802548">_10. entity와 관련된 다양한 서비스를 처리하기 위해 dto 묶음을 모은 dto_</span>
- Context라는 dto는 n개의 서비스가 같이 이뤄지는 경우에 활용한다.
```java
@Getter
public class PaymentContext {
    private PaymentRequest paymentRequest; // 결제 정보
    private List<SeatDto> seatDtoList; //복수 개의 좌석 정보
    private ScheduleDto scheduleDto; // 일정 정보
}
```
- 위의 Context dto는 아래와 같이 3개의 서비스를 실행한다.
```java
public void createPayment(PaymentContext paymentContext) {
        String paymentPK = paymentRepository.createPayment(paymentContext.getPaymentRequest());
        seatService.updateSeatSold(paymentContext.getSeatDtoList());
        ticketService.createTicket(paymentContext, paymentPK);
    }
```
## <span style="color:#802548">_11. entity와 무관한 dto_</span>
- 아래는 entity와는 무관하며, 오류 메시지를 전달하는 dto다. entity와 무관하여 @Setter를 활용했다.
```java
@Getter
@Setter
public class ExceptionResponse {
    private String message;
    private String requestURI;
}
```
- 실제로는 아래와 같이 ExceptionHandler에서 메시지를 만들고 front에 전달한다.
```java
@ExceptionHandler(PaymentException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ExceptionResponse paymentException(HttpServletRequest request, PaymentException paymentException) {
        ExceptionResponse exceptionResponse = new ExceptionResponse();
        exceptionResponse.setMessage(paymentException.getMessage());
        exceptionResponse.setRequestURI(request.getRequestURI());

        return exceptionResponse;
    }
```
- 아래는 또다른 entity와 무관한 dto다. 여기서는 @Builder를 활용했다. entity와 무관한 dto도 @Builder를 활용할 수 있다.
```java
@Builder
@Getter
@AllArgsConstructor
public class JwtResponse {
 
    private String grantType;
    private String accessToken;
    private String refreshToken;
}
```
- Controller에서 Service를 호출한다.
```java
@PostMapping("/api/signin")
    public JwtResponse login(@RequestBody SignInRequest signInRequestDto, HttpSession session)
            throws Exception {
        JwtResponse jwtResponse = signService.login(signInRequestDto.getMemberId(),
                signInRequestDto.getMemberPassword());

        redisService.setRefreshToken(signInRequestDto.getMemberId(), jwtResponse.getRefreshToken());

        return jwtResponse;
    }
```
- 그럼 아래와 같이 Jwt를 Builder로 생성하는 것을 관찰할 수 있다.
```java
 JwtResponse jwt = JwtResponse.builder()
                .grantType("Bearer")
                .accessToken(accessToken)
                .refreshToken(refreshToken)
                .build();
```
- 참고로 로그인 시에 id와 password를 받아 token을 만들고 authentication 객체를 만들어서 accessToken을 만드는 것이다. 
```java
public JwtResponse login(String memberId, String memberPassword) throws Exception {
        Member memberInfo = memberRepository.findByMemberId(memberId);
        if (memberInfo == null) {
            throw new SignException(ErrorEnum.NOT_AUTHENTICATED);
        }

        boolean isEqualPassword = passwordEncoder.matches(memberPassword, memberInfo.getMemberPassword());
        if (!isEqualPassword) {
            throw new SignException(ErrorEnum.NOT_AUTHENTICATED);
        }

        UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken(memberId,
                memberPassword);
        Authentication authentication = authenticationManagerBuilder.getObject().authenticate(token);

        String accessToken = jwtTokenProvider.generateAccessTokenBySignIn(authentication);
        String refreshToken = jwtTokenProvider.generateRefreshTokenBySignIn(memberId);

        JwtResponse jwt = JwtResponse.builder()
                .grantType("Bearer")
                .accessToken(accessToken)
                .refreshToken(refreshToken)
                .build();

        return jwt;
    }
```

## <span style="color:#802548">_12. JpaRepository interface_</span>
- 해당 interface를 상속하면 정해진 방식으로 JPQL이 호출된다. 
- 저장은 보통 void아닌가 할 수 있는데, JpaRepository를 상속하면 늘 entity class를 return한다. 
- 사실 JpaRepository를 상속받으면 아래 method 아무것도 쓰지 않아도 지정된 방식으로 사용할 수 있다. 
- 아래 사이트를 참조하자.
```java
public interface MemberRepository extends JpaRepository<Member, Long> {

}
​```

https://m.blog.naver.com/hj_kim97/222780110215



- 나는 내가 쓰고 싶은 것만 적어서 쓰고 싶어서 이렇게 만들었는데, 그럴거였다면 사실 JpaRepository를 상속하지 않고 보통의 class로 만드는 게 맞다. 
```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    public Member findByMemberId(String memberId);

    public Member findByMemberNameLike(String keyword);

    public Member save(MemberDto memberDto);

    public Long deleteByMemberId(String memeberId);
}
```
- 가령 예를 들면 위의 findByMemberId는 아래와 동일한 JPql이다. 
- 참고로 JPql은 테이블이 아니라 entity class를 대상으로 쿼리를 날린다. 
- 따라서 테이블이 아니라 entity class에서 가져오고, 조건절도 객체의 field를 대상으로 한다. 
- 또한 반드시 alias를 사용해서 필드를 지정해야 한다. Member entity class의 memeberId field는 m.memberId라고 적는 형식이다. 변수는 :를 붙여준다. 
```java
String JPql="select m from Member m where m.memberId=:memberId";
```
- Service class는 @Service를 달아주고, @Transactional을 달아 영속성 컨텍스트가 인지하게 만든다. 
```java
@Service
@Transactional
@RequiredArgsConstructor
```
- 서비스에서는 늘 entity를 조회해서 가져와 영속화시킨 뒤에 그 다음 행위를 이어가야 db에 transaction이 반영된다. 
- 따라서 아래와 같은 내용이 늘 사전작업으로 들어가게 된다. 
```java
Member memberInfo = memberRepository.findByMemberId(memberDto.getMemberId());
```
- 실제로 비밀번호를 변경할 때도 변경하려는 Member의 entity를 조회한 뒤에 비밀번호를 변경하는 모습을 볼 수 있다. 
```java
@Service
@Transactional
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;
    private final PasswordEncoder passwordEncoder;

    public void updatePw(MemberDto memberDto) {
        Member memberInfo = memberRepository.findByMemberId(memberDto.getMemberId());
        if (memberInfo == null) {
            throw new SignException(ErrorEnum.NO_VALID_USER);
        }

        memberDto.encodePassword(passwordEncoder.encode(memberDto.getMemberPassword()));
        memberInfo.updatePw(memberDto.getMemberPassword());
    }

}
```

## <span style="color:#802548">_13.EntityManager_</span>
- 위에서는 interface 방식을 활용했지만, 아래에서는 entityManager를 활용한다. 
- 밑의 createQuery의 경우에는 주로 PK가 아닌 것들은 조건절에 넣어 검색을 할 때 사용된다. 
```java
 List<Schedule> resultList = entityManager
                .createQuery(sql, Schedule.class)
                .setParameter("scheduleDepartStation", scheduleFilteringConditionDto.getScheduleDepartStation())
                .setParameter("scheduleArriveStation", scheduleFilteringConditionDto.getScheduleArriveStation())
                .setParameter("scheduleDate", dateTime)
                .getResultList();
```
- 만약 PK를 알고 있다면 find() method를 사용할 수 있다.
```java
Payment payment = entityManager.find(Payment.class, PaymentNo);
```
- 저장은 아래와 같이 persist()라는 method를 사용한다.
```java
entityManager.persist(payment);
```
- 삭제는 아래와 같이 remove()라는 method를 사용한다.
```java
 entityManager.remove(schedule);
 ```
- update는 위 두개 방식 모두 공통으로 쓰는 dirty checking을 활용한다. entity를 조회해 영속성 컨텍스트에서 인지하게 만든 뒤 값을 바꾸는 방식이다. 그렇기에 별다른 method가 없다.
```java
public void updateMemberInfo(MemberDto memberDto) {
        Member memberInfo = memberRepository.findByMemberId(memberDto.getMemberId());
        if (memberInfo == null) {
            throw new SignException(ErrorEnum.NO_VALID_USER);
        }

        memberInfo.update(memberDto.getMemberName(), memberDto.getMemberId(), memberDto.getMemberBirth(),
                memberDto.getMemberPhone(), memberDto.getMemberEmail(), memberDto.getMemberAddress());
    }
```
- 아래와 같이 for문을 활용해 여러개의 update query를 날릴 수도 있다.
```java
 public void updateSeatSold(List<SeatDto> seatDtoList) {
        List<Seat> seatList = seatDtoList.stream().map(seatDto -> new Seat(seatDto)).collect(Collectors.toList());
        if (seatList.isEmpty()) {
            throw new SeatException(ErrorEnum.NO_VALID_SEAT);
        }

        for (Seat seat : seatList) {
            Seat seatInfo = seatRepository.getSeatBySeatCode(seat.getSeatCode());
            seatInfo.updateSeatSold();
        }

    }
```

## <span style="color:#802548">_14. n개의 entity로 만드는 entity_</span>
​- ticket을 만들 때 3개의 entity를 가져와 만드는 경우다. 우선 front에서 받은 값으로 entity를 조회해 가져온다.
```java
Schedule schedule = entityManager.createQuery(
                                "SELECT sch FROM Schedule sch WHERE sch.scheduleNo = :scheduleNo", Schedule.class)
                                .setParameter("scheduleNo", paymentContext.getScheduleDto().getScheduleNo())
                                .getSingleResult();
```
- 그리고 front에서 받은 dto를 entity로 바꾼다. 
- map()에서 쓴 new 생성자로 만든 Seat entity는 cascade를 걸지 않아도 된다. map() 안의 new는 또 다른 영역에서 이뤄지기 때문으로 생각된다. 
- 정확한 이유는 모르겠다.
```java
List<Seat> SeatList = paymentContext.getSeatDtoList().stream().map(seatDto -> new Seat(seatDto))
                                .collect(Collectors.toList());
```
- 결제 entity를 조회해 가져온 뒤 Ticket을 new 생성자를 활용해 만든다. 
- @Transactional에서 new를 쓰게 되면 새로운 객체가 생성되는 것으로 인식하며 persists()를 실행하면 영속성 컨텍스트가 db에 값을 저장한다.
```java
for (Seat seatInfo : SeatList) {
                        Payment payment = entityManager
                                        .createQuery("SELECT pay FROM Payment pay WHERE pay.paymentNo = :paymentNo",
                                                        Payment.class)
                                        .setParameter("paymentNo", paymentPK)
                                        .getSingleResult();

                        Ticket ticketInfo = new Ticket(payment, seatInfo, schedule);
                        entityManager.persist(ticketInfo);
                }
```
- SeatList에서 볼 수 있듯 entity를 반드시 조회해야 할 필요는 없다. 다만 대다수의 경우 entity를 만들기 위해 가장 편한게 entity 조회다.
```java
@Repository
public class TicketRepository {

        @PersistenceContext
        private EntityManager entityManager;

        /**
         * @parameter: front에서 건낸 일정 관련 정보로 일정, 좌석, 결제 entity를 조회
         */
        public void createTicket(PaymentContext paymentContext, String paymentPK) {

                Schedule schedule = entityManager.createQuery(
                                "SELECT sch FROM Schedule sch WHERE sch.scheduleNo = :scheduleNo", Schedule.class)
                                .setParameter("scheduleNo", paymentContext.getScheduleDto().getScheduleNo())
                                .getSingleResult();

                List<Seat> SeatList = paymentContext.getSeatDtoList().stream().map(seatDto -> new Seat(seatDto))
                                .collect(Collectors.toList());

                for (Seat seatInfo : SeatList) {
                        Payment payment = entityManager
                                        .createQuery("SELECT pay FROM Payment pay WHERE pay.paymentNo = :paymentNo",
                                                        Payment.class)
                                        .setParameter("paymentNo", paymentPK)
                                        .getSingleResult();

                        Ticket ticketInfo = new Ticket(payment, seatInfo, schedule);
                        entityManager.persist(ticketInfo);
                }

        }

}
```
## <span style="color:#802548">_15. JPQL_</span>
- 일반 sql과 달리 JPQL은 entity class를 대상으로 하기에 entity class 이름을 from절에 써준다. 
```java
select distinct * from Schedule
```
- 전부 가져올 때도 *가 아니라 sch 같은 특정한 값을 적는다.
```java
select distinct sch from Schedule
```
- 그 값은 entity class의 alias와 동일하게 맞춰준다.
```java
select distinct sch from Schedule sch
```
- 그리고 객체의 field를 가져올 때도 해당 entity의 field임을 나타내줘야 한다. 따라서 alias.으로 붙여준다. 
```java
select distinct sch from Schedule sch where sch.scheduleArriveDatetime
```
- Schedule entity에서 조회하는 Station entity의 stationName이라는 column이 필요하다면 아래와 같이 .을 두 번이어 써준다.
```
sch.departStation.stationName
```
- 변수의 경우 :를 붙여준다. 
```java
select distinct sch from Schedule sch where sch.departStation.stationName = :scheduleDepartStation "
```
- 쿼리가 너무 길어져서 보기 어려워 질 수 있기 때문에 보통 절을 나누는 편이다. 
- 띄어쓰기에 민감하니 끝 부분은 늘 띄어쓰기를 해주자. 
```java
String sql = "select distinct sch from Schedule sch " +
                "where sch.departStation.stationName = :scheduleDepartStation " +
```
- 아래에서 parameter를 설정하여 변수값을 대입한다.
```java
 .setParameter("scheduleDepartStation", scheduleFilteringConditionDto.getScheduleDepartStation())
 ```
- 날짜의 경우 특정날짜 이후의 날짜를 가져오게 하려면 부등호를 사용하면 된다. 
```java
  "and sch.scheduleArriveDatetime >= :scheduleDate";
```
- 날짜는 iso 8601을 만족해야 한다. 2023-05-08 20:42:00를 보냈다면, 빈칸 대신 T를 넣어준다.
```java
LocalDateTime dateTime = LocalDateTime.parse(dateString.replace(" ", "T"));
```
- row가 여러개라면 ResultList로 받아야 한다.
```java
 .getResultList();
 ```
 ```java
String sql = "select distinct sch from Schedule sch " +
                "where sch.departStation.stationName = :scheduleDepartStation " +
                "and sch.arriveStation.stationName = :scheduleArriveStation " +
                "and sch.scheduleArriveDatetime >= :scheduleDate";

        List<Schedule> resultList = entityManager
                .createQuery(sql, Schedule.class)
                .setParameter("scheduleDepartStation", scheduleFilteringConditionDto.getScheduleDepartStation())
                .setParameter("scheduleArriveStation", scheduleFilteringConditionDto.getScheduleArriveStation())
                .setParameter("scheduleDate", dateTime)
                .getResultList();
```
반면에 row가 한 개라면 SingleResult로 받아야 한다.
```java
.getSingleResult();
```
- JPQL이 길지 않다면 아래와 같이 바로 직접 써줄 수도 있다. 
```java
Member member = entityManager.createQuery(
                "SELECT m FROM Member m WHERE m.memberId = :memberId", Member.class)
                .setParameter("memberId", memberId)
                .getSingleResult();
```