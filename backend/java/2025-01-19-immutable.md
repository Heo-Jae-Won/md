## <span style="color:#802548">_x-www-form-urlencoded일 경우 immutable DTO 만들기_</span>
- 불변성을 유지하는 것은 무엇보다 중요하다.
- Enttiy class들은 어쩔수없지만, DTO class들은 immutability를 유지하는 방향으로 가보자.
- 아래가 일반적으로 사용되는 DTO다.

```java
@Getter
@Setter
@ToString
@NoArgsConstrutor
public class CarResponseDTO {
    private  Long carSeq;
    private  String carType;
    private  boolean carStatus;

}
```

- immutability로 만들려면 아래와 같은 조건을 만족해야 한다.
    - final class
    - final field
    - only creation in Consturctor, no setter method
    - when changing, using new ()


- 따라서 아래와 같이 바꿔준다.
- class와 field를 모두 final로 바꿨다.

```java
@Getter
@Setter
@ToString
@NoArgsConstrutor
public final class CarResponseDTO {
    private final Long carSeq;
    private final String carType;
    private final boolean carStatus;

}
```

- 하지만 안타깝게도 Spring에서는 Data를 받으면　@NoArgsConstructor가 필요하다.
    - @RequestBody로 받을 때는 Jackson이 DTO mapping 시에 @NoArgsConsturctor를 필요로 한다.
        - 
    - @ModelAttribute로 받을 때는 Spring 차원에서 @NoArgsConstructor를 필요한다.
        - reflection으로 data를 binding하는데, 그 때 쓰이는 Class.newInstance()는 NoArgsConstrucotr를 필요로 한다.
- immutability를 활용하려면 결국 Reflection api의 기본 동작인 NoArgs를 활용하면 안 된다.

```
Exception ~~~~~~~
```


- Reflection api에는 다행히 NoArgs 말고 Parametetized constructor를 활용할 수 있다.
- 대신 Reflection api에 binding할 name을 전부 명시해주어야 한다.
- 그럼 Class.newInstance()가 아니라 Constructor.newInstance()가 발동한다.
    - Parametetized constructor를 활용하는 것이다.

```java
@Getter
@ToString
public final class CarResponseDTO {
    private final Long carSeq;
    private final String carType;
    private final boolean carStatus;

    @Builder
    @ConstructorProperties({"carSeq","carType", "carStatus"})
    public CarResponseDTO(Long carSeq, String carType, boolean carStatus) {
        this.carSeq = carSeq;
        this.carType = carType;
        this.carStatus = carStatus;
    }
}
```


## <span style="color:#802548">_x-www-form-urlencoded일 경우 response와 request DTO분리하기_</span>

- request용과 response용은 서로 쓰이는 field가 다르다.
    - 그런데 final class는 값이 생성자를 만들때까지는 반드시 설정되어야 한다.   
    - 따라서 null로 초기값을 설정해야 하는데, 그러면 해당 field를 response용에선 쓸 수 없다.
    - 서로를 분리해주자.

```java
@Getter
@ToString
public class CarInsertRequestDTO {
    private final Long carSeq;
    private final String carType;

    @Builder
    @ConstructorProperties({"carSeq","carType"})
    public CarInsertRequestDTO(Long carSeq, String carType) {
        this.carSeq = carSeq;
        this.carType = carType;
    }
}
```

- response에서는 data를 Spring이 binding하지 않는다. 
    - @ModelAttribute라던가 @RequestBody 등의 annotation이 없다.
- 따라서 @ConstructorProperties가 불필요하다. 
    - 혹시 response에서도 binding이 필요하다면 써야 한다.

```java
@Getter
@ToString
public class CarResponseDTO {
    private final Long carSeq;
    private final String carType;
    private final boolean carStatus;

    @Builder
    //@ConstructorProperties({"carSeq","carType", "carStatus"})
    public CarResponseDTO(Long carSeq, String carType, boolean carStatus) {
        this.carSeq = carSeq;
        this.carType = carType;
        this.carStatus = carStatus;
    }z
}
```

- 서로 분리해주었다면, 이제는 기본형은, 프론트에서 값을 무조건 주는 형태로 진행되게 될 것이다. 
- 따라서 객체형이 아닌 기본형으로 변경가능하다. 별건 아니지만, 성능이 좀 더 좋아진다.

```java
@Getter
@ToString
public class CarInsertRequestDTO {
    private final long carSeq;
    private final String carType;

    @Builder
    @ConstructorProperties({"carSeq","carType"})
    public CarInsertRequestDTO(long carSeq, String carType) {
        this.carSeq = carSeq;
        this.carType = carType;
    }
}
```


## <span style="color:#802548">_x-www-form-urlencoded일 경우  domain 별로 모은 static inner class화_</span>

- domain 별로 명확하게 구분짓기 위해서 static inner class를 활용하는 것도 좋은 선택이다.
- top level class는 1개로 설정 하는 대신, 아무런 method와 field를 가지지 않는다.

```java
public class CarDTO {
    
    @Getter
    @ToString
    public static final class CarRequestDTO {
        private final Long carSeq;
        private final String carType;

        @Builder
        @ConstructorProperties({"carSeq","carType"})
        public CarRequestDTO(Long carSeq, String carType) {
            this.carSeq = carSeq;
            this.carType = carType;
        }
    }

    @Getter
    @ToString
    public static final class CarResponseDTO {
        private final Long carSeq;
        private final String carType;
        private final boolean carStatus;

        @Builder
        public CarResponseDTO(Long carSeq, String carType, boolean carStatus) {
            this.carSeq = carSeq;
            this.carType = carType;
            this.carStatus = carStatus;
        }
    }
}
```

## <span style="color:#802548">_Json type일 경우 immutable DTO class_</span>
- JSON일 경우, Jackson library로 parsing을 진행하기 때문에, 다른 annotation을 써야 한다.
    - JSON serialization/deserialization와 java beans의 serialization/deserialization의 구조가 다르기 때문이다.
    - JavaBeans는 DTO class같은 Java의 객체들이다. 주로 www-x-form-urlencoded로 content를 front와 주고받을 때 사용한다.
    - Json은 주로 application/json으로 content를 front와 주고 받을 때 사용한다. 

```java
@Getter
@ToString
public final class CarResponseDTO {
    private final Long carSeq;
    private final String carType;
    private final boolean carStatus;

    @Builder
    @JsonCreator
    public CarResponseDTO(@JsonProperty("carSeq")       Long carSeq, 
                          @JsonProperty("carType")      String carType, 
                          @JsonProperty("carStatus")    boolean carStatus) {
        this.carSeq = carSeq;
        this.carType = carType;
        this.carStatus = carStatus;
    }
}
```

## <span style="color:#802548">_더 간단하게, record class_</span>
- record를 쓰면 여러가지 고충을 덜을 수 있다.
    - final class로 선언할 필요X
    - final field 선언할 필요 X
    - getter, setter, NoArgs, ReArgs, ConstructorProperties 선언할 필요 X
    - content type 변경에 따른 class field parsing annotation 교체 X
        - front framework를 사용하게 되면 www-x-form-urlencoded에서 application/json으로 변경된다.

```java
public record CarRequestDTO(Long carSeq, String carType){

}
```



## <span style="color:#802548">_domain 별로 모은 static record화_</span>
- record의 경우 top level이면 당연히 static이다.
- 그러나 inner에서는 static을 붙여줘야 한다.

```java
public class CarDTO {
    
    @Builder
    public static record CarRequestDTO(Long carSeq, String carType) {
        public static CarRequestDTO TODTO(CarEntity carEntity) {
            return CarRequestDTO.builder().carSeq(carEntity.getCarSeq())
                                            .carType(carEntity.getCarType())
                                            .build();
        }
    }

    @Builder
    public static record CarResponseDTO(Long carSeq, String carType, boolean carStatus) {
        public static CarResponseDTO TODTO(CarEntity carEntity) {
            return CarResponseDTO.builder()
                                    .carSeq(carEntity.getCarSeq())
                                    .carType(carEntity.getCarType())
                                    .carStatus(carEntity.getYnEnum().getBooleanValue()) 
                                    .build();
                            
        }
    }
}
```


## <span style="color:#802548">_요약_</span>
- form 방식은 아래와 같다.

```
front content-type: www-x-form-urlencoded
controller annotation: @ModelAttribute
serialization/deserialization mechanism: form binding (like JavaBeans serialization/deserialization)
Dto annotation for immutability: @ConstructorProperties
```

- 변환은 Spring이 알아서 해준다.

```java
public class MyDto {
    private String name;
    private int age;

    // Getters and Setters
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

//name=John&age=25
```

- Json 방식은 아래와 같다.
```
front content-type: application/json
controller annotation: @RequestBody
serialization/deserialization mechanism: Jackson library (JSON serialization/deserialization)
DTO annotation for immutability: @JsonCreator, @JsonProperties
```

- 변환은 Spring이 아니라 Jackson이 알아서 해준다.

```java
public class MyDto {
    private String name;
    private int age;

    // Getters and Setters
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

/*
    {
        "orderSeq": 123,
        "carSeq": 456
    }
/*
```
