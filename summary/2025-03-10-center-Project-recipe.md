## <span style="color:#802548">_chatGPT에 request 보내기- webclient config_</span>
- chatGPT에 request를 보내기 위해선 api key를 넣어서 request를 보내야 한다.
- 그를 위해 webflux의 webclient를 활용하자.
- 기본 dependency에는 없으므로 추가해줘야한다.

```java
implementation 'org.springframework.boot:spring-boot-starter-webflux'
```

- 추가했다면 webflux를 쓰기 편하게 config로 하여 bean을 만들어준다.
- webcline는 이제 bean으로 만들어졌다. 

```java
@Configuration
public class RecipeRecommendConfig {

    @Value("${openai.api.key}") // application.properties에 저장된 openai.api.key를 불러온다.
    private String openAiKey;

    @Bean
    public WebClient webClient() {
        return WebClient.builder()
                .baseUrl("https://api.openai.com")
                .defaultHeader("Authorization", "Bearer " + openAiKey)
                .build();
    }
    
}
```

## <span style="color:#802548">_chatGPT에 request 보내기- requestDTO_</span>

- 이제는 DTO를 만들어줄 차례다.
- json 구조는 아래와 같다.

```json
{
  "model": "gpt-4-turbo",
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful assistant."
    },
    {
      "role": "user",
      "content": "What is the capital of France?"
    }
  ],
  "temperature": 0.7,
  "max_tokens": 100,
  "top_p": 1,
  "frequency_penalty": 0,
  "presence_penalty": 0
}
```

- 위의 json 구조를 우선 Java Object로 만들어준다.
- 여기서 아래 max_tokens부터 presence_penalty는 optional field라서 그냥 놔둔다.

```java
public class ChatGPTRequestDTO  {
    private String model;
    private List<MessageDTO> messages;
    private Double temperature;
}
```

- 어떤 dto의 구조는 아래와 같다.

```java
public class MessageDTO {
    private String role;
    private String content;
}
```

- 외부로 보내야 하기 때문에 class 위에 다음과 같은 annotation이 필요하다.
- 외부로는 JSON구조로 통신하기 때문에 Serializable을 implements할 필요는 없다.

```java
@ToString
@Builder
@AllArgsConstructor
@Setter
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class ChatGPTRequestDTO {
    private String model;
    private List<MessageDTO> messages;
    private Double temperature;
}
```

- 덕지덕지 붙는 게 싫은 사람들은 Java17에서 사용가능한 record class를 사용하면 더 좋다.

```java
@Builder
public record ChatGPTRequestDTO(
                                    String model,
                                    List<MessageDTO> messages,
                                    Double temperature) {

                                    }
```

## <span style="color:#802548">_chatGPT에서 response 받기- responseDTO_</span>

- response DTO의 구조는 request와는 다르다.
- ChatGPT의 response json 구조가 다르기 때문이다.

```json
{
    "id": "chatcmpl-abc123",
    "object": "chat.completion",
    "created": 1700000000,
    "model": "gpt-4-turbo",
    "choices": [
        {
            "index": 0,
            "message": {
                "role": "assistant",
                "content": "The capital of France is Paris."
            },
            "finish_reason": "stop"
        }
    ],
    "usage": {
        "prompt_tokens": 20,
        "completion_tokens": 10,
        "total_tokens": 30
    }
}
```


- choices는 request와 response 어디에서도 사용되지 않는다.
- 따라서 그냥 inner class로 만들어 버린다.
- json 속성에서 finish_reason이란 이름으로 들어오기 때문에 java의 field와 이를 맞춰준다.
    - 그 역할을 @JsonProperty가 해준다.

```java
@ToString
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Builder
@AllArgsConstructor
@Setter
@Getter
public class RecipeChatGPTResponseDTO {
    private String id;
    private String object;
    private LocalDate created;
    private String model;
    private List<Choice> choices;

    @Getter
    @Setter
    public static class Choice {
        private int index;
        private MessageChatGPTDTO message;

        @JsonProperty("finish_reason")
        private String finishReason;
    }
}
```


- MessageChatGPT는 role과 content로 이뤄져있다.
- role은 system, user를 사용하게 되므로 이를 맞춰 값을 만들어 준다.

```java
@ToString
@Builder
@AllArgsConstructor
@Setter
@Getter
@Slf4j
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class MessageChatGPTDTO {

    private String role;
    private String content;

    public static MessageChatGPTDTO TOSYSTEMMESSAGE() {
        return MessageChatGPTDTO.builder()
                .role("system")
                .content("너는 이제 음식 레시피를 추천해주는거야")
                .build();

    }
}
```

- 재료, 음식용도, 메뉴, 맛, 난이도의 변수를 받아올 수 있는 DTO를 만든다.
- 아래와 같이 간편한 방식(""")은 Java 15이상에서만 가능하다. 

```java
.
.
.
public class MessageChatGPTDTO {
    public static MessageChatGPTDTO TOUSERMESSAGE(RecipeUserRequestDTO recipeUserRequestDTO) {

        String content = String.format("""
                재료: %s 
                음식용도: %s
                메뉴: %s
                맛: %s
                난이도: %s
                100자 이내로 작성해줘. 출력은 다음과 같이 양식을 맞춰서 출력해줘
                조리방법은 5개 step으로만 구성해줘.
                재료는 자취생 1인분용으로 몇개가 필요한 지까지 알려줘.
                한자는 쓰지말아줘.
                아래와 같이 구성해줘.
                아래 예시의 양식을 지켜서 반드시 지켜서 만들어줘.
                
                예시)
                    타이틀: 요리이름
                    재료: 재료1, 재료2, 재료3, 재료4, 재료5 .....
                    조리방법: ~~합니다.
                            ~~합니다.
                """, recipeUserRequestDTO.getIngredients(), 
                    recipeUserRequestDTO.getRecipeConditionDTO().getUsage(),
                    recipeUserRequestDTO.getRecipeConditionDTO().getMenu(),
                    recipeUserRequestDTO.getRecipeConditionDTO().getTaste(), 
                    recipeUserRequestDTO.getRecipeConditionDTO().getLevel()
        );

        return MessageChatGPTDTO.builder()
                                    .role("user")
                                    .content(content)
                                    .build();

    }

}
```

- ChatGPT server에 request할 body는 user의 request에서 받아온다.
- ingredients는 HTML단에서 string[]이 아닌 ,로 이어진 string으로 들어온다.
- 여기선 굳이 ,를 구분자로 배열로 파싱하여 가져올 필요가 없다. 
- prompt가 모두 string 값이기 때문이다.

```java
@ToString
@NoArgsConstructor
@Builder
@AllArgsConstructor
@Setter
@Getter
public class RecipeUserRequestDTO  {
    private String ingredients;
    private RecipeConditionDTO recipeConditionDTO;
}
```

- recipeConditionDTO는 추가될 수 있다.
- 유지보수하기 쉽게 따로 class를 만들어 관리한다.

```java
@ToString
@Builder
@AllArgsConstructor
@Setter
@Getter
@NoArgsConstructor
public class RecipeConditionDTO {
    private String usage;
    private String menu;
    private String taste;
    private String level;

    public static RecipeConditionDTO TODTO(String usage, String menu, String taste, String level) {
        return RecipeConditionDTO.builder().usage(usage)
                                            .menu(menu)
                                            .taste(taste)
                                            .level(level)
                                            .build();
    }
}
```

- user에게 보내야하는 response는 아래와 같다.
    - recipe의 제목(title)
    - recipe의 재료(ingredients)
    - recipe의 조리법(cookingmethods)
    - recipe가 골라진 조건(recipeConditionDTO)
- 따라서 아래와 같이 response를 만든다.
    - ingredients와 cookingmethods는 배열로 담는다. 

```java
@ToString
@NoArgsConstructor
@Builder
@AllArgsConstructor
@Setter
@Getter
public class RecipeUserResponseDTO  {
    private String title;
    private String[] ingredients;
    private String[] cookingMethods;
    private RecipeConditionDTO recipeConditionDTO;

    public static RecipeUserResponseDTO TODTO(String title, String[] ingredients, String[] cookingMethods, RecipeConditionDTO recipeConditionDTO) {
        return RecipeUserResponseDTO.builder()
                                    .title(title)
                                    .ingredients(ingredients)
                                    .cookingMethods(cookingMethods)
                                    .recipeConditionDTO(recipeConditionDTO)
                                    .build();
    }
}
```


## <span style="color:#802548">_chatGPT에서 request 보내고 response 받기- service_</span>

- 아래 server to server request는 @Transactional을 붙여도 무의미하다.
- 자기 DB에 binding된 request가 아니라 rollback할 수 없다.
- chatGPT가 참고할 prompt message를 만들고 이를 requestDTO로 만든다.

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class RecipeService {

    private final WebClient webClient;
    private final ObjectMapper objectMapper;

    @Value("${openai.api.key}")
    private String apiKey;

    @Value("${openai.api.model}")
    private String model;

    @Value("${openai.api.temperature}")
    private float temperature;

    // model:
    public RecipeUserResponseDTO sendRequestToChatGPT(RecipeUserRequestDTO recipeUserRequestDTO) {

        List<MessageChatGPTDTO> messages = List.of(MessageChatGPTDTO.TOUSERMESSAGE(recipeUserRequestDTO),
                MessageChatGPTDTO.TOSYSTEMMESSAGE(recipeUserRequestDTO));
        RecipeChatGPTRequestDTO recipeChatGPTRequestDTO = RecipeChatGPTRequestDTO.TODTO(model, messages, temperature);

    }
}
```

- chatGPT server에서 값을 받아온다.
- 값을 받아왔으면 json의 parsing이 필요하다.
- parsing은 jsonNode로 해준다.
    - string으로 parsing할 땐 asText
    - 타이틀, 재료, 조리방법의 이름을 확인하고 그만큼을 length를 더해서 잘라온다
    - cooking method는 숫자로 구분되어 온다. 1. ~~~ 2.라면, ~~~를 가져와 배열로 순서대로 저장하게 한다.
    

```java
public RecipeUserResponseDTO sendRequestToChatGPT(RecipeUserRequestDTO recipeUserRequestDTO) {
    List<MessageChatGPTDTO> messages = List.of(MessageChatGPTDTO.TOUSERMESSAGE(recipeUserRequestDTO),
                    MessageChatGPTDTO.TOSYSTEMMESSAGE(recipeUserRequestDTO));
    RecipeChatGPTRequestDTO recipeChatGPTRequestDTO = RecipeChatGPTRequestDTO.TODTO(model, messages, temperature);

    try {
        String jsonString = webClient.post()
                .uri("https://api.openai.com/v1/chat/completions")
                .header("Authorization", "Bearer " + apiKey)
                .bodyValue(recipeChatGPTRequestDTO)
                .retrieve()
                .bodyToMono(String.class)
                .block();
        JsonNode jsonNode = objectMapper.readTree(jsonString);

        JsonNode choicesNode = jsonNode.get("choices");
        if (choicesNode.isArray()) {
            String content = jsonNode.get("choices").get(0).get("message").get("content").asText();
            String firstElement = "타이틀: ";
            String secondElement = "재료: ";
            String thridElement = "조리방법: ";
            String title = content.substring(content.indexOf(firstElement) + firstElement.length(), content.indexOf(secondElement));
            String[] ingredients = content.substring(content.lastIndexOf(secondElement) + secondElement.length(), content.indexOf(thridElement)).split(",");
            String cookingMethod = content.substring(content.lastIndexOf(thridElement) + thridElement.length());

            Pattern pattern = Pattern.compile("\\d+\\.\\s*(.*?)(?=\\s*\\d+\\.|$)");
            Matcher matcher = pattern.matcher(cookingMethod);
            List<String> steps = new ArrayList<>();
            while (matcher.find()) {
                steps.add(matcher.group(1).trim());
            }
            String[] cookingMethods = steps.toArray(new String[0]);
            return RecipeUserResponseDTO.TODTO(title, ingredients, cookingMethods, 
                                                        recipeUserRequestDTO.getRecipeConditionDTO()
                                                );
        } else {
            throw new RuntimeException("choiesNode가 array형태가 아닙니다.");
        }

    } catch (Exception exception) {
        exception.printStackTrace();
        throw new RuntimeException("알 수 없는 에러가 발생하였습니다");
    }
}
```

- 위를 보다 명확하게 의미를 드러내주기 위해선 private method로 빼주자.
- 과정에 의미를 부여하는 것이다.
    - chatGPT 요청을 위한 message -> 응답받기 -> 응답을 parsing 순의 과정이 명확하게 드러난다.

```java
public RecipeUserResponseDTO getRecipeResponse(RecipeUserRequestDTO recipeUserRequestDTO) {
    //CHATGPT에 필요한 system, user message를 생성
    RecipeChatGPTRequestDTO recipeChatGPTRequestDTO = createMessageForChatGPTRequest(recipeUserRequestDTO);

    try {
        String jsonString = getResponseFromChatGPT(recipeChatGPTRequestDTO);
        RecipeUserResponseDTO recipeUserResponseDTO = parseResponseFromChatGPT(jsonString, recipeUserRequestDTO.getRecipeConditionDTO());
            
        return recipeUserResponseDTO;

    } catch (Exception exception) {
        throw new RuntimeException("알 수 없는 에러가 발생하였습니다");
    }
}
```


- public은 controller에서 써야하는 것만 public으로 준다.
- 내부에서만 사용하는 것들은 private으로 만들어준다.

```java
private RecipeChatGPTRequestDTO createMessageForChatGPTRequest(RecipeUserRequestDTO recipeUserRequestDTO) {
    List<MessageChatGPTDTO> messages = List.of(MessageChatGPTDTO.TOUSERMESSAGE(recipeUserRequestDTO),
            MessageChatGPTDTO.TOSYSTEMMESSAGE(recipeUserRequestDTO));

    return RecipeChatGPTRequestDTO.TODTO(model, messages, temperature);
}

/**
 * 타이틀, 재료, 조리방법을 parsing. 조리방법은 숫자를 기준으로 parsing.
 */
private RecipeUserResponseDTO parseResponseFromChatGPT(String jsonString, RecipeConditionDTO recipeUserRequestDTO) throws JsonMappingException, JsonProcessingException {
    JsonNode jsonNode = objectMapper.readTree(jsonString);
    JsonNode choicesNode = jsonNode.get("choices");
    if (choicesNode.isArray()) {
        String content = jsonNode.get("choices").get(0).get("message").get("content").asText();
        String firstElement = "타이틀: ";
        String secondElement = "재료: ";
        String thridElement = "조리방법: ";
        String title = content.substring(content.indexOf(firstElement) + firstElement.length(), content.indexOf(secondElement));
        String[] ingredients = content.substring(content.lastIndexOf(secondElement) + secondElement.length(), content.indexOf(thridElement)).split(",");
        String cookingMethod = content.substring(content.lastIndexOf(thridElement) + thridElement.length());

        Pattern pattern = Pattern.compile("\\d+\\.\\s*(.*?)(?=\\s*\\d+\\.|$)");
        Matcher matcher = pattern.matcher(cookingMethod);
        List<String> steps = new ArrayList<>();
        while (matcher.find()) {
            steps.add(matcher.group(1).trim());
        }
        String[] cookingMethods = steps.toArray(new String[0]);

        return RecipeUserResponseDTO.TODTO(title, ingredients, cookingMethods, recipeUserRequestDTO);
    } else {
        throw new RuntimeException("choiesNode가 array형태가 아닙니다.");
    }
}

private String getResponseFromChatGPT(RecipeChatGPTRequestDTO recipeChatGPTRequestDTO) {
    return webClient.post()
                    .uri("/v1/chat/completions")
                    .header("Authorization", "Bearer " + apiKey)
                    .bodyValue(recipeChatGPTRequestDTO)
                    .retrieve()
                    .bodyToMono(String.class)
                    .block();
}
```




## <span style="color:#802548">_chatGPT에서 request 보내고 response 받기- controller_</span>

- chatGPT에 request를 보내면 그 값을 session에 저장한다.
- session에 저장하는 이유는 redirect.addFlashAttribute는 새로고침 시 값이 날라가버리기 때문이다.

```java
@PostMapping("/recipe/chatGPT")
public String showExample(@RequestBody RecipeUserRequestDTO recipeUserRequestDTO, HttpSession session) {

    RecipeUserResponseDTO response = recipeService.sendRequestToChatGPT(recipeUserRequestDTO);
    
    session.setAttribute("recipe", response);

    return "redirect:/recipe/recommend/output";
}
```

- session에 들어온 값을 매번 model로 다시 넣어준다.
- session이 만료되었을 때의 조치도 필요하다.
    - 따라서 null check를 해준다.
    - null check시 null이라면 empty()인 경우의 값을 반환한다.

```java
@GetMapping("/recipe/recommend/output")
public String viewRecipeRecoomendOutput(Model model, HttpSession session) {
    RecipeUserResponseDTO recipeResponse = session.getAttribute("recipe") != null ? (RecipeUserResponseDTO) session.getAttribute("recipe") : RecipeUserResponseDTO.empty();
    model.addAttribute("recipe", session.getAttribute("recipe"));

    return "recipe/recommend_result";
}
```

- empty인 경우의 builder를 추가해준다.
- session이 시간이 지나 무효화된 경우, 어떻게 처리할 지도 backend에서 정하는 형태다.
- string message는 전부 enum으로 나중에 변환이 필요하다.

```java
public class RecipeUserResponseDTO  {
    private String title;
    private String[] ingredients;
    private String[] cookingMethods;
    private RecipeConditionDTO recipeConditionDTO;

    public static RecipeUserResponseDTO TODTO(String title, String[] ingredients, String[] cookingMethods, RecipeConditionDTO recipeConditionDTO) {
        return RecipeUserResponseDTO.builder()
                                    .title(title)
                                    .ingredients(ingredients)
                                    .cookingMethods(cookingMethods)
                                    .recipeConditionDTO(recipeConditionDTO)
                                    .build();
    }

    public static RecipeUserResponseDTO empty() {
        return RecipeUserResponseDTO.builder()
                                    .title("session이 만료되었습니다.")
                                    .ingredients(new String[]{"session이 만료되었습니다."})
                                    .cookingMethods(new String[]{"session이 만료되었습니다."})
                                    .recipeConditionDTO(RecipeConditionDTO.empty())
                                    .build();
    }
}

@ToString
@Builder
@AllArgsConstructor
@Setter
@Getter
@NoArgsConstructor
public class RecipeConditionDTO {
    private String usage;
    private String menu;
    private String taste;
    private String level;

    public static RecipeConditionDTO TODTO(String usage, String menu, String taste, String level) {
        return RecipeConditionDTO.builder().usage(usage)
                                            .menu(menu)
                                            .taste(taste)
                                            .level(level)
                                            .build();
    }

    public static RecipeConditionDTO empty() {
        return RecipeConditionDTO.builder().usage("")
                                            .menu("")
                                            .taste("")
                                            .level("")
                                            .build();
    }
}
```

- string message는 앞으로 session 만료에 따른 기본 default message로 작동하게 된다.
- 여러군데 분산되어 string으로 관리하면 전부 한꺼번에 찾아내기가 매우 어렵다.

```java
public static RecipeUserResponseDTO empty() {
        return RecipeUserResponseDTO.builder()
                                    .title("session이 만료되었습니다.")
                                    .ingredients(new String[]{"session이 만료되었습니다."})
                                    .cookingMethods(new String[]{"session이 만료되었습니다."})
                                    .recipeConditionDTO(RecipeConditionDTO.empty())
                                    .build();
}
```

- enum을 활용해 전체적으로 한군데에서 관리하기 쉽게 변경한다.
- global 폴더 안에 놔두면 더 편리해진다.

```java
@Getter
@RequiredArgsConstructor
public enum ErrorEnum {
    SESSION_INVALIDATE("session이 만료되었습니다.");

    private final String message;
}

public static RecipeUserResponseDTO empty() {
        return RecipeUserResponseDTO.builder()
                                    .title(ErrorEnum.SESSION_INVALIDATE.getMessage())
                                    .ingredients(new String[]{ErrorEnum.SESSION_INVALIDATE.getMessage()})
                                    .cookingMethods(new String[]{ErrorEnum.SESSION_INVALIDATE.getMessage()})
                                    .recipeConditionDTO(RecipeConditionDTO.empty())
                                    .build();
    }
```



- session에서 너무 오래 쥐고 있으면 부담이 간다.
- 다시 추천페이지에 들어왔다면, 이전에 저장해둔 정보를 활용할 이유가 없다. 
- 따라서 그냥 지워준다.

```java
@GetMapping("/recipe/recommend")
public String viewRecipeRecoomend(HttpSession session) {
    session.removeAttribute("recipe");
    return "recipe/recommend";
}
```


## <span style="color:#802548">_user의 request 화면 구성하기- recommend.html_</span>

- recommend.js는 defer로 놓아 load 이전에 읽히지 않게 방지한다.
- 매번 동일하게 적용되는 header는 전부 fragment로 만들어준다.
- 첫번째 처럼 만들면 local에서만 작동한다.
    - 반드시 두번째처럼 만들어줘야 배포했을 때도 정상 작동한다.
    - 두번째가 context-relative resolution이기 때문이다. 
    - 경로가 바뀌어도 정상 작동하게 하려면 두번째 것으로 하자.

```html
<div class="header" th:replace="/fragment/header :: top-menu"></div>
<div class="header" th:replace="~{fragment/header :: top-menu}"></div>
```

```html
<!DOCTYPE html>
<html lang="ko" xmlns:th="http://thymeleaf.org">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>레시피 추천</title>

        <!-- 내부 import -->
        <link rel="stylesheet" th:href="@{/css/globalstyles.css}" />
        <link rel="stylesheet" th:href="@{/css/recommend.css}" />
        <script src="/js/recommend.js" defer></script>

        <!-- jquery-3.6.0.min.js 나중에 경로 바꿔주세요-->
        <script th:src="@{/js/jquery-3.7.1.min.js}"></script>

        <!-- 외부 alert import -->
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/sweetalert2@11.16.0/dist/sweetalert2.min.css" />
        <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11.16.0/dist/sweetalert2.all.min.js"></script>
    </head>
    <body>
        <div class="container">
            <div class="header" th:replace="~{fragment/header :: top-menu}"></div>
            <div class="recommend-content">
                <!-- 왼쪽: 재료 검색 + 리스트 -->
                <div class="ingredients-section">
                    <div class="search-box">
                        <input type="text" id="search-input" placeholder="재료 검색..." onkeyup="filterIngredients()" />
                        <button class="search-btn">검색</button>
                    </div>
                    <div class="ingredients-list" id="ingredients-list">
                        <!-- JavaScript로 동적으로 추가 -->
                    </div>
                </div>
                <!-- 오른쪽: 장바구니 + 옵션 선택 + 버튼 -->
                <div class="options-section">
                    <form id="recommend-form" th:action="@{/recipe/chatGPT}" method="post">
                        <div class="cart-box" id="cart-box">장바구니<br /></div>
                        <input type="hidden" name="ingredients" id="ingredients-input" />

                        <div class="option-container">
                            <div class="option-box">
                                <h2>어떤 음식을 추천해드릴까요?</h2>
                                <div class="button-group">
                                    <label class="option-label">
                                        <input
                                            type="radio"
                                            name="usage"
                                            value="일반식"
                                            class="option-checkbox"
                                            checked
                                        />
                                        <span class="option-btn selected">일반식</span>
                                    </label>
                                    <label class="option-label">
                                        <input type="radio" name="usage" value="다이어트" class="option-checkbox" />
                                        <span class="option-btn">다이어트</span>
                                    </label>
                                    <label class="option-label">
                                        <input type="radio" name="usage" value="도시락" class="option-checkbox" />
                                        <span class="option-btn">도시락</span>
                                    </label>
                                </div>
                            </div>
                            <div class="option-box">
                                <h2>어떤 메뉴로 추천해드릴까요?</h2>
                                <div class="button-group">
                                    <label class="option-label">
                                        <input type="radio" name="menu" value="한식" class="option-checkbox" checked />
                                        <span class="option-btn selected">한식</span>
                                    </label>
                                    <label class="option-label">
                                        <input type="radio" name="menu" value="중식" class="option-checkbox" />
                                        <span class="option-btn">중식</span>
                                    </label>
                                    <label class="option-label">
                                        <input type="radio" name="menu" value="일식" class="option-checkbox" />
                                        <span class="option-btn">일식</span>
                                    </label>
                                    <label class="option-label">
                                        <input type="radio" name="menu" value="양식" class="option-checkbox" />
                                        <span class="option-btn">양식</span>
                                    </label>
                                </div>
                            </div>
                            <div class="option-box">
                                <h2>기호에 맞는 요리를 선택하세요</h2>
                                <div class="button-group">
                                    <label class="option-label">
                                        <input
                                            type="radio"
                                            name="taste"
                                            value="매운맛"
                                            class="option-checkbox"
                                            checked
                                        />
                                        <span class="option-btn selected">매운맛</span>
                                    </label>
                                    <label class="option-label">
                                        <input type="radio" name="taste" value="순한맛" class="option-checkbox" />
                                        <span class="option-btn">순한맛</span>
                                    </label>
                                    <label class="option-label">
                                        <input type="radio" name="taste" value="건강식" class="option-checkbox" />
                                        <span class="option-btn">건강식</span>
                                    </label>
                                    <label class="option-label">
                                        <input type="radio" name="taste" value="건강식" class="option-checkbox" />
                                        <span class="option-btn">건강식</span>
                                    </label>
                                </div>
                            </div>
                            <div class="option-box">
                                <h2>조리 난이도을 선택하세요</h2>
                                <div class="button-group">
                                    <label class="option-label">
                                        <input type="radio" name="level" value="초보" class="option-checkbox" checked />
                                        <span class="option-btn selected">초보</span>
                                    </label>
                                    <label class="option-label">
                                        <input type="radio" name="level" value="중수" class="option-checkbox" />
                                        <span class="option-btn">중수</span>
                                    </label>
                                    <label class="option-label">
                                        <input type="radio" name="level" value="고수" class="option-checkbox" />
                                        <span class="option-btn">고수</span>
                                    </label>
                                </div>
                            </div>
                        </div>
                        <button class="recommend-btn" type="submit">레시피 추천받기</button>
                    </form>
                </div>
            </div>
        </div>
    </body>
</html>
```

## <span style="color:#802548">_user의 request 화면 구성하기- recommend.js_</span>

- 처음에 화면에 들어가면 재료가 전부 보이게끔 해줘야 한다.
- 재료는 버튼 형태로 생성되는데, 생성되게 되면 onClick eventListener를 달아준다.
    - click을 하면 장바구니에 해당 keyword가 담긴다.

```js
displayIngredients();

function displayIngredients(filter = '') {
    ingredientsList.innerHTML = '';
    ingredients
        .filter((item) => item.includes(filter))
        .forEach((item) => {
            let btn = document.createElement('button');
            btn.textContent = item;
            btn.classList.add('ingredient-btn');

            if (document.querySelector(`[data-item="${item}"]`)) {
                btn.classList.add('selected');
            }

            btn.onclick = () => toggleCartItem(btn, item);
            ingredientsList.appendChild(btn);
        });
}
```

- 장바구니에 들어오게 되면, selected 상태가 된다.
- 장바구니의 X자 버튼을 누르면 지울 수 있다.

```js
function toggleCartItem(button, ingredient) {
    const cartBox = document.getElementById('cart-box');
    let existingItem = cartBox.querySelector(`[data-item="${ingredient}"]`);

    if (existingItem) {
        existingItem.remove();
        button.classList.remove('selected');
    } else {
        if ($('#cart-box button').length >= maxCartItems) {
            Swal.fire({
                icon: 'warning',
                title: '장바구니가 가득 찼습니다',
                text: '최대 10개의 재료만 선택할 수 있습니다',
                confirmButtonColor: '#ff7f50',
                confirmButtonText: '확인',
            });
            return;
        }

        button.classList.add('selected');
        const btn = document.createElement('button');
        btn.innerHTML = `${ingredient} <span class='remove-btn'>✖</span>`;
        btn.setAttribute('data-item', ingredient);
        btn.classList.add('cart-item');

        btn.onclick = function () {
            this.remove();
            button.classList.remove('selected');
            updateCartData();
        };

        cartBox.appendChild(btn);
    }

    updateCartData();
}
```

- 장바구니에 재료가 선택되면 css 처리하는 과정과 별도로, data도 연계한다.
- 배열로 보유하지 않고, 선택된 값들은 쉼표로 전부 이어준다.

```js
function updateCartData() {
    let selectedIngredients = [];
    $('#cart-box button').each(function () {
        selectedIngredients.push($(this).attr('data-item'));
    });
    $('#ingredients-input').val(selectedIngredients.join(','));
}
```

- 재료가 아닌 condition을 선택했을 때 eventListener 처리다.

```js
$(document).on('click', '.option-btn', function () {
    let group = $(this).data('group');
    $(`.option-btn[data-group="${group}"]`).removeClass('selected');
    $(this).addClass('selected');

    //selected 연결
    let optionDom = $(this).siblings("input[type='radio']");
    optionDom.prop('checked', true);

});
```

- submit event가 일어나는 것을 감지해서 실제로는 ajax event를 발동시킨다.
- xxx/url-form-encoded로는 js 상에서 객체를 보내기가 어렵다.
- 따라서 json 형태로 바꿔서 보내기 위해 ajax event로 바꾸는 것이다.


```js
$('#recommend-form').on('submit', function (e) {
    e.preventDefault(); // 폼 제출 방지
    if (!$('#ingredients-input').val().trim()) {
        Swal.fire({
            icon: 'error',
            title: '재료를 선택해주세요',
            text: '최소 1개의 재료를 입력해주세요',
            confirmButtonColor: '#ff7f50',
            confirmButtonText: '확인',
        });
        return;
    }

    const ingredients = document.querySelector("input[name='ingredients']").value;
    const usage = document.querySelector("input[name='usage']:checked").value;
    const menu = document.querySelector("input[name='menu']:checked").value;
    const taste = document.querySelector("input[name='taste']:checked").value;
    const level = document.querySelector("input[name='level']:checked").value;
    const sendData = {
        ingredients,
        recipeConditionDTO: {
            usage,
            menu,
            taste,
            level
        }
    }

    fetch('/recipe/chatGPT', {
        body:JSON.stringify(sendData),
        headers:{
            'content-type':'application/json'
        },
        method:'post'
    }).then((response) => {
        if(!response.ok) {
            Swal.fire({
                icon: 'error',
                title: '오류가 발생하였습니다.',
                text: '네트워크가 연결되어있는지 확인해주세요',
                confirmButtonColor: '#ff7f50',
                confirmButtonText: '확인',
            });
            return;
        }

        location.href='/recipe/recommend/output';
    })
    .catch((error)=> console.log(error));

});
```

- chatGPT와의 통신은 수 초간 지속되는 경우가 많으므로 loading 이미지를 넣어준다.

```html
<div  class="loader" style="display: none;">
    <img class="loader-bar" th:src="@{/image/cook-load.gif}"></img>
</div>
```

- css는 인터넷에 있는 거 그냥 가져왔다.

```css
.loader {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0,0,0,0.5);
    z-index: 9999;
    opacity: 70%;
  }
  
.loader-bar {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 300px;
    height: 300px;
    margin-top: -5px;
    margin-left: -50px;
    background-color: white;
}
  
@keyframes loader-bar {
    0% {
        transform: translateX(-50%) scaleX(0);
    }
    50% {
        transform: translateX(-50%) scaleX(1);
    }
    100% {
        transform: translateX(-50%) scaleX(0);
    }
}
```

- js로 loading bar를 control한다.

```js
function showLoader() {
  loader.style.display = 'block';
}

function hideLoader() {
  loader.style.display = 'none';
}

$('#recommend-form').on('submit',  function (e) {
    //data-collect
    showLoader();
    fetch('/recipe/chatGPT', {
        body:JSON.stringify(sendData),
        headers:{
            'content-type':'application/json'
        },
        method:'post'
    }).then((response) => {
        hideLoader();
        if(!response.ok) {
            Swal.fire({
                icon: 'error',
                title: '오류가 발생하였습니다.',
                text: '네트워크가 연결되어있는지 확인해주세요',
                confirmButtonColor: '#ff7f50',
                confirmButtonText: '확인',
            });
            return;
        }

        location.href='/recipe/recommend/output';
    })
    .catch((error)=> hideLoader());
});
```

## <span style="color:#802548">_user의 request api key 암호화_</span>

- open api key가 노출되면 악용될 수 있다.
- 따라서 application-properties에 암호화된 값을 올려야 한다.
- 그 때 Spring Valut를 쓰기도 하는데, Vault는 서버를 하나 따로 만들어야한다.
- 우리는 서버를 더 추가했다간 RAM이 박살나서 포기했다.
- 대신 흔하게 쓰이는 jascyprt로 암호화된 값을 넣어준다.

```java
implementation 'com.github.ulisesbocchio:jasypt-spring-boot-starter:3.0.5'
```

- 이름은 Encryptor긴 하지만, decrypt 로직으로도 쓰인다.
- 아래 encryptKey는 system 변수인데, 구동할 때 받는 JVM option을 의미한다.
    - application.properties에 있는 값이 아니다.
    - application에 password를 노출하면 복호화가 그대로 뚫려버리는 것이므로 JVM option으로 줘야한다.

```java
@Configuration
public class PropertiesEncryptConfig {

    @Value("#{systemProperties['jasypt.encryptor.password']}")
    private String encryptKey;

    @Bean(name = "JasyptEncoder")
    public PooledPBEStringEncryptor stringEncryptor() {
        PooledPBEStringEncryptor encryptor = new PooledPBEStringEncryptor();
        SimpleStringPBEConfig config = new SimpleStringPBEConfig();
        config.setPassword(encryptKey);
        config.setAlgorithm("PBEWithMD5AndDES");
        config.setKeyObtentionIterations("1000");
        config.setPoolSize("1");
        config.setSaltGeneratorClassName("org.jasypt.salt.RandomSaltGenerator");
        config.setStringOutputType("base64");
        encryptor.setConfig(config);
        return encryptor;
    }
}
```

- JVM option으로 아래와 같은 형태다.
- JVM의 최적화 용도로도 쓰지만, Spring boot server에 관한 옵션으로도 쓴다.

```sh
java -Djasypt.encryptor.password=나는비밀키야 ~~~.jar
```

- vscode에서 local에서 위의 JVM 변수 값을 주려면 아래와 같이 준다.
- 아래 파일은 launch.json인데, 좌측 side bar에 run and debug 메뉴를 누르면 create a launch.json이 뜬다.
- 거기서 vmArgs를 넣어주면 된다. -D를 붙이는 형태다. 
- JVM option으로 두가지 값을 넣어도 그냥 띄어쓰기하고 그대로 이어준다. 배열같은데 구분해서 넣지 않는다.

```json
{
    "version": "0.0.2",
    "configurations": [
        {
            "type": "본인거",
            "name": "본인거",
            "request": "본인거",
            "mainClass": "본인거",
            "projectName": "본인거",
            "vmArgs": "-Dspring.profiles.active=local -Dspring.profiles.active=local"
        },
    ]
}
```

- 암호화된 값을 알아내는 법은 두 가지다.
- 하나는 그냥 test에서 돌리는 방법이다.
- test에서 돌릴 때는 서버의 JVM option을 가져오지 못하므로 test class에서 직접 박아 넣는다.
    - @BeforeAll이다.

```java
@SpringBootTest
class ProjectApplicationTests {

	@Test
	void contextLoads() {
	}

	@BeforeAll
    static void setup() {
        System.setProperty("jasypt.encryptor.password", "나는비밀키야");
        System.setProperty("jasypt.encryptor.bean", "JasyptEncoder");
    }

	@Test
    @DisplayName("1. Encrypt Test")
    void test1() {
        String secretKey = "나는비밀키야";

        String targetText = "나는암호화해야할중요한값";

        // Using Jasypt
        StandardPBEStringEncryptor encryptor = new StandardPBEStringEncryptor();
        encryptor.setPassword(secretKey);
		encryptor.setAlgorithm("PBEWithMD5AndDES");
        String encryptedText = encryptor.encrypt(targetText);
        System.out.println("encryptedText = " + encryptedText);

        String decryptedText = encryptor.decrypt(encryptedText);
        System.out.println("decryptedText = " + decryptedText);

        Assertions.assertThat(decryptedText).isEqualTo(targetText);
    }
}
```


- 다른 하나는 jar를 다운로드하여 java -cp를 돌리는 방법이다.
- chatGPT에 jasypt-1.9.3.jar를 다운로드 받는 link를 달라고 하자.
- 위에서 우리가 받은 것은 boot-starter 버전인데 왜 jasypt-1.9.3인지 의문이 드는 사람도 있을 수 있다.
    - 하지만 spring-boot-starter 버전에는 CLI 용이 포함되어 있지 않다.
    - 따라서 jasypt-1.9.3.jar를 다운로드 받아야한다.
    - https://mvnrepository.com/artifact/org.jasypt/jasypt/1.9.3 참고 링크다.
    - 여기서 Files를 보면 jar가 보인다. 이걸 다운로드 받자.
        - java HOME 경로를 우선 등록하여 system 변수로 활용할 수 있게끔 해주는 선행작업이 필요하다.
        - 그 후에 cmd를 켜서 다운로드 받은 경로에 들어가서 아래 명령어를 실행하면 된다.
        - 그냥 아무대서나 실행하면 오류가 뜬다. 해당 jar가 없기 떄문이다.


```sh
java -cp jasypt-1.9.3.jar org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI input="open.api.key"  password="ㄹㄹㄹ" algorithm=PBEWithMD5AndDES
```

- 그럼 아래와 같이 메시지가 뜨게된다. output이 암호화된 값이다.

```
C:\>java -cp jasypt-1.9.3.jar org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI input="open.api.key"  password="ㄹㄹㄹ" algorithm=PBEWithMD5AndDES

----ENVIRONMENT-----------------

Runtime: Oracle Corporation OpenJDK 64-Bit Server VM 17.0.2+8-86



----ARGUMENTS-------------------

input: open.api.key
password: ㄹㄹㄹ
algorithm: PBEWithMD5AndDES



----OUTPUT----------------------

egwSedXojdGIdVc9p+XXoA==
```

- 이제 암호화된 값을 ENC()의 괄호 안에 넣어준다.
- decode하는 역할은 bean의 이름을 주어야 한다.
    - 위에 PropertiesEncryptConfig에서 만든 bean의 이름을 넣자.
        - name = ""에 써져있는 JasyptEncoder다.
        - 사람마다 다르니까 알아서 잘 넣어주자.
        - 안 넣으면 decrypt가 안 돼서 오류난다.

```sh
open.api.key=ENC(egwSedXojdGIdVc9p+XXoA==)
jasypt.encryptor.bean=JasyptEncoder
```

- 이제 서버가 구동될 떄마다 JasyptEncoder라는 bean이 decode를 하여 properties 값을 해석해 @Value에 넣어준다.


## <span style="color:#802548">_recipe의 history 저장하기- ERD_</span>

- recipe 따로, output 따로, input keyword를 따로 저장한다.
- recipe의 경우는 대리키를 두고, 어떤 user에 속했는 지 확인하기 위해 user_seq를 fk로 둔다.

```sql
Create table recipe
(
	recipe_seq bigint auto_increment,
	user_seq bigint not null, 
	recipe_output_content varchar(5000) not null,
	created_at datetime default current_timestamp,
	constraint recipe_PK primary key (recipe_seq),    
	constraint recipe_user_fk foreign key (user_seq) references user(user_seq) 
);
```

- keyword는 복수개를 집어넣어야 한다.
- 따라서 batch insert가 필요하므로 auto_increment를 걸지 않고 compound로 PK를 만든다.

```sql
CREATE TABLE recipe_input_keyword
(
    recipe_seq BIGINT NOT NULL,
    keyword VARCHAR(30) NOT NULL,
    CONSTRAINT recipe_input_keyword_PK PRIMARY KEY (recipe_seq, keyword),
    CONSTRAINT recipe_input_keyword_FK FOREIGN KEY (recipe_seq) REFERENCES recipe(recipe_seq) ON DELETE CASCADE
);
```


- recipe_output_content는 strict한 identifying relationship을 만들까 고민을 했다.
- 우리는 recipe 1개에 recipe-content가 하나였기 떄문이다.
- 근데 확장성을 위해서는 대리키를 쓰는게 좋다고 하여, 일단은 대리키를 사용했다.
    - 한 마디로 나중에 1 : n으로 바꿀 떄는 대리키가 아니면 안 된다는 의미다. 

```sql
CREATE TABLE recipe_output_content
(
    recipe_output_content_seq BIGINT AUTO_INCREMENT,
    recipe_seq BIGINT NOT NULL,
    recipe_title varchar(100) NOT NULL,
    output_content longtext NOT NULL,
    CONSTRAINT recipe_output_content_PK PRIMARY KEY (recipe_output_content_seq),
    CONSTRAINT recipe_output_content_recipe_FK FOREIGN KEY (recipe_seq) REFERENCES recipe(recipe_seq) ON DELETE CASCADE
);

```

- 만약 대리키를 사용하지 않았다면 strict한 identifying relationship이 됐을 것이다.

```sql
CREATE TABLE recipe_output_content
(
    recipe_seq BIGINT NOT NULL,
    recipe_title varchar(100) NOT NULL,
    output_content longtext NOT NULL,
    CONSTRAINT recipe_output_content_PK PRIMARY KEY (recipe_seq),
    CONSTRAINT recipe_output_content_recipe_FK FOREIGN KEY (recipe_seq) REFERENCES recipe(recipe_seq) ON DELETE CASCADE
);
```

- entity도 아래와 같이 만들어졌을 것이다.

```java
@Entity
@Getter
@Setter
@Table(name = "recipe_output_content")
public class RecipeOutputContent {

    @Id
    @Column(name = "recipe_seq")
    private Long recipeSeq;

    @OneToOne
    @MapsId("recipeSeq")  // This makes recipe_seq both the primary key and foreign key
    @JoinColumn(name = "recipe_seq")
    private RecipeEntity recipeEntity;

    @Column(name = "recipe_title", length = 100, nullable = false)
    private String recipeTitle;

    @Column(name = "output_content", columnDefinition = "LONGTEXT", nullable = false)
    private String outputContent;
}
```



## <span style="color:#802548">_recipe의 history 저장하기- entity_</span>

- 하나의 user가 여러 개의 recipe를 가지게 된다.
- user : recipe = 1: m이며, m인 recipe가 연관관계의 주인이다.
    - 따라서 userEntity가 아닌 RecipeEntity가 @ManyToOne을 갖게된다. 
- 반면에 recipe : inpuyKeyword = 1: m이기에, m인 inputKeyWord가 연관관계의 주인이다.
    - 따라서 RecipeEntity가 mappedBy를 갖게 된다.

```java
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity
@Table(name = "recipe")
public class RecipeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "recipe_seq")
    private Long recipeSeq;

    @ManyToOne
    @JoinColumn(name = "user_seq", referencedColumnName = "user_seq")
    private UserEntity userEntity;

    @Column(name = "created_at")
    @CreationTimestamp
    private LocalDateTime createdAt;

    @OneToMany(mappedBy = "recipeEntity", cascade = CascadeType.REMOVE)
    @Builder.Default
    private List<RecipeInputKeywordEntity> recipeInputKeywordEntity = new ArrayList<>();

    public static RecipeEntity TOENTITY(UserEntity userEntity) {
        return RecipeEntity.builder()
                            .userEntity(userEntity)
                            .build();
    }
}
```

- RecipeInputKeyword의 설계는 복합키로 되어있다.
- recipe_seq와 keyword가 합쳐져서 복합키가 된다.
    - 따라서 Embeddable 속성을 달아준다.
    - 복합키를 구성하는 column을 아래 적어준다.

```java
@Embeddable
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
@Builder
@Getter
public class RecipeComplicatedPK {

    @Column(name = "recipe_seq")
    private Long recipeSeq;

    @Column(name = "keyword")
    private String keyword;
    
    public static RecipeComplicatedPK of(Long recipeSeq, String keyword) {
        return RecipeComplicatedPK.builder()
                                    .recipeSeq(recipeSeq)
                                    .keyword(keyword)
                                    .build();
    }
    
}
```


- 구성한 복합키는 @EmbeddedId로 가져올 수 있다.
- 그런데, 복합키의 요소로 쓰인 recipe_seq는 fk이다.
    - fk이며 동시에 pk의 요소인 경우, @MapsId속성을 사용해야만 한다. java의 field명 기준으로 적어준다.
    - recipeInputKeyword : recipe = m : 1 이므로 연관관계의 주인인 recipeInputKeyword가 @ManyToOne이다.

```java
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity
@Table(name = "recipe_input_keyword")
public class RecipeInputKeywordEntity {

    @EmbeddedId
    private RecipeComplicatedPK recipeComplicatedPK;

    @MapsId("recipeSeq")
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "recipe_seq", referencedColumnName = "recipe_seq")
    private RecipeEntity recipeEntity;

    public static RecipeInputKeywordEntity TOENTITY(RecipeEntity recipeEntity, String keyword) {
        return RecipeInputKeywordEntity.builder().recipeComplicatedPK(RecipeComplicatedPK.of(recipeEntity.getRecipeSeq(), keyword))
                                                    .recipeEntity(recipeEntity)
                                                    .build();
    }

}
```

- recipeOutputContent는 별도의 auto_increment pk를 갖게 한다.
- 해당 table도 recipe entity를 참조한다. 
    - 그런데 recipe는 1개의 output만 가지므로 1 : 1 관계다.
    - FK가 있는 recipeOutput이 연관관계의 주인이다.

```java
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity
@Table(name = "recipe_output_content")
public class RecipeOutputEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "recipe_output_content_seq")
    private Long recipeOutputContentSeq;

    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "recipe_seq", referencedColumnName = "recipe_seq")
    private RecipeEntity recipeEntity;

    @Column(name = "recipe_title")
    private String recipeTitle;

    @Column(name = "output_content")
    private String outputContent;

    public static RecipeOutputEntity TOENTTIY(RecipeEntity recipeEntity, String recipeTitle, String outputContent) {
        return RecipeOutputEntity.builder()
                                    .recipeEntity(recipeEntity)
                                    .recipeTitle(recipeTitle)
                                    .outputContent(outputContent)
                                    .build();
    }

    
}
```

## <span style="color:#802548">_recipe의 history 저장하기- requestDTO_</span>

- RecipeHistroyRequsetDTO는 별다를 게 없다. 
- recipeInputKeyword 저장에 필요한 recipeCondition class
- recipeOutputContent 저장에 필요한 outputContent와 title를 받는다.

```java
@ToString
@NoArgsConstructor
@Builder
@AllArgsConstructor
@Setter
@Getter
public class RecipeHistroyRequsetDTO {
    private String title;
    private String outputContent;
    private RecipeConditionDTO recipeCondition;

    public List<String> getAllConditions() {
        return Arrays.asList(recipeCondition.getUsage(), 
                                recipeCondition.getMenu(), 
                                recipeCondition.getTaste(), 
                                recipeCondition.getLevel()
                            );
    }

}
```

## <span style="color:#802548">_recipe의 history 저장하기- service_</span>

- result를 저장할 때는 복수개의 entity를 저장하게 된다.
    - recipe -> recipeInput -> recipeOutput 순이다.
- 따라서 @Transactional이 필요하다.
- 

```java
@Service
@RequiredArgsConstructor
public class RecipeHistoryService {
    
    private final UserRepository userRepository;
    private final RecipeInpuKeywordRepository recipeInpuKeywordRepository;
    private final RecipeOutputRepository recipeOutputRepository;
    private final RecipeRepository recipeRepository;

    @Transactional
    public Long saveRecipeHisotry(RecipeHistroyRequsetDTO recipeHistroyRequsetDTO) {
        LoginUserDetails loginUser = (LoginUserDetails) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        String userId = loginUser.getUserId();
        UserEntity user = userRepository.findByUserId(userId)
                .orElseThrow(() -> new RuntimeException("no such user"));
        
        RecipeEntity recipeEntity = recipeRepository.save(RecipeEntity.TOENTITY(user));

        if (recipeEntity == null) {
            throw new RuntimeException("recipe is not made due to error");
        }

        List<RecipeInputKeywordEntity> recipeInputKeywords = recipeHistroyRequsetDTO.getAllConditions()
                                                                                .stream()
                                                                                .map(condition -> RecipeInputKeywordEntity.TOENTITY(recipeEntity, condition))
                                                                                .collect(Collectors.toList());

        recipeInpuKeywordRepository.saveAll(recipeInputKeywords);
        
        recipeOutputRepository.save(RecipeOutputEntity.TOENTTIY(
                recipeEntity, 
                recipeHistroyRequsetDTO.getTitle(), 
                recipeHistroyRequsetDTO.getOutputContent()
            ));
            
        return recipeEntity.getRecipeSeq();
    }
}
```

- 하지만 saveALl의 경우, 불필요한 여러 select query를 마주하게 된다.
- 이는 saveAllAndFlush를 해도 마찬가지다.

```
Hibernate: 
    select
        ue1_0.user_seq,
        ue1_0.created_at,
        ue1_0.is_deleted,
        ue1_0.roles,
        ue1_0.updated_at,
        ue1_0.user_email,
        ue1_0.user_id,
        ue1_0.user_name,
        ue1_0.user_password 
    from
        user ue1_0 
    where
        ue1_0.user_id=?
Hibernate: 
    insert 
    into
        recipe
        (created_at, user_seq) 
    values
        (?, ?)
Hibernate: 
    select
        rike1_0.keyword,
        rike1_0.recipe_seq 
    from
        recipe_input_keyword rike1_0 
    where
        (
            rike1_0.keyword, rike1_0.recipe_seq
        ) in ((?, ?))
Hibernate: 
    select
        rike1_0.keyword,
        rike1_0.recipe_seq 
    from
        recipe_input_keyword rike1_0 
    where
        (
            rike1_0.keyword, rike1_0.recipe_seq
        ) in ((?, ?))
Hibernate: 
    select
        rike1_0.keyword,
        rike1_0.recipe_seq 
    from
        recipe_input_keyword rike1_0 
    where
        (
            rike1_0.keyword, rike1_0.recipe_seq
        ) in ((?, ?))
Hibernate: 
    select
        rike1_0.keyword,
        rike1_0.recipe_seq 
    from
        recipe_input_keyword rike1_0 
    where
        (
            rike1_0.keyword, rike1_0.recipe_seq
        ) in ((?, ?))
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_output_content
        (output_content, recipe_seq, recipe_title) 
    values
        (?, ?, ?)
```

- 아래 select는 굳이 필요하지 않은 select다.

```
select
        rike1_0.keyword,
        rike1_0.recipe_seq 
    from
        recipe_input_keyword rike1_0 
    where
        (
            rike1_0.keyword, rike1_0.recipe_seq
        ) in ((?, ?))
```


- 또한 insert 문도 batch insert로 적용되지 않는다.

```
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
```

- 해당 문제는 save자체의 문제에서 비롯된다.
- save는 entity가 영속화되지 않은 경우, 반드시 그 존재 여부를 select를 통해서 확인하기 때문이다.
- 있는 것이 명확한 경우에는 불필요한 select를 제외하기 위해 EntityManager를 통한 persist - flush를 사용한다.

```java
@Repository
public class RecipeInputBatchInSertRepository {

    @PersistenceContext
    EntityManager entityManager;

    public void bulkInsertRecipeKeywords(List<RecipeInputKeywordEntity> keywords) {
        for (int i = 0; i < keywords.size(); i++) {
            entityManager.persist(keywords.get(i));
        }
        entityManager.flush(); // Final flush
        entityManager.clear();
    }

}
```

- saveAll()대신 RecipeInputBatchInSertRepository를 DI하여 bulkInsertRecipeKeywords()로 사용하자.
- saveAll을 쓸 때 있었던 불필요한 select 문이 사라졌음을 알 수 있다.

```
Hibernate: 
    select
        ue1_0.user_seq,
        ue1_0.created_at,
        ue1_0.is_deleted,
        ue1_0.roles,
        ue1_0.updated_at,
        ue1_0.user_email,
        ue1_0.user_id,
        ue1_0.user_name,
        ue1_0.user_password 
    from
        user ue1_0 
    where
        ue1_0.user_id=?
Hibernate: 
    insert 
    into
        recipe
        (created_at, user_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_output_content
        (output_content, recipe_seq, recipe_title) 
    values
        (?, ?, ?)
```

- 위와 같은 방식을 해도 batch insert는 개선되지 않는다.
- batch insert는 우선 @Id가 Identity로 선언되면 안된다.
    - idenetity는 일단 insert된 뒤에 그 id 값을 JPA에서 인지할 수 있기 떄문에 batch insert를 지원하지 않는다.
    - 내 경우에는 identity가 아닌 복합키 방식이므로 batch insert가 가능했다.
- 그래서 안 되는 이유를 찾아보니, 특별한 설정이 요구되었다.
    - url에는 rewriteBatchedStatements=true를 추가한다.

```sh
spring.datasource.url=jdbc:mysql://localhost:3306/project?rewriteBatchedStatements=true
```

- 위의 설정을 추가해도 hibernate 상으로는 insert문이 4번 나간다.

```
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
```


- 하지만 실제 SQL과 hibernate의 sql은 다르게 나갈 수 있다.
    - hibernate에서 생성한 sql을 mysql driver가 rewrite하기때문에, 실제 native sql이 어떻게 진행되는지가 중요하다.
    - 관련해선 https://stackoverflow.com/questions/21530112/no-matter-what-i-cant-batch-mysql-insert-statements-in-hibernate  참고하자.
- 실제 natvie sql을 살펴보고 싶다면 driver url에 아래 내용을 추가해야 한다.
    - &profileSQL=true

```sh
spring.datasource.url=jdbc:mysql://localhost:3306/project?rewriteBatchedStatements=true&profileSQL=true
```

- 실제 natvie sqlmf 살펴봐도 insert 문은 4개가 나가고 있다.
- batch insert가 아닌 것이다.

```
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Sun Mar 16 21:02:17 KST 2025 INFO: [QUERY] insert into recipe_input_keyword (keyword,recipe_seq) values ('일반식',18) [Created on: Sun Mar 16 21:02:17 KST 2025, duration: 1, connection-id: 2776, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)]
ProxyPreparedStatement.java:61
Sun Mar 16 21:02:17 KST 2025 INFO: [FETCH]  [Created on: Sun Mar 16 21:02:17 KST 2025, duration: 0, connection-id: 2776, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)]
ProxyPreparedStatement.java:61
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Sun Mar 16 21:02:17 KST 2025 INFO: [QUERY] insert into recipe_input_keyword (keyword,recipe_seq) values ('한식',18) [Created on: Sun Mar 16 21:02:17 KST 2025, duration: 2, connection-id: 2776, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)]
ProxyPreparedStatement.java:61
Sun Mar 16 21:02:17 KST 2025 INFO: [FETCH]  [Created on: Sun Mar 16 21:02:17 KST 2025, duration: 0, connection-id: 2776, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)]
ProxyPreparedStatement.java:61
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Sun Mar 16 21:02:17 KST 2025 INFO: [QUERY] insert into recipe_input_keyword (keyword,recipe_seq) values ('매운맛',18) [Created on: Sun Mar 16 21:02:17 KST 2025, duration: 0, connection-id: 2776, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)]
ProxyPreparedStatement.java:61
Sun Mar 16 21:02:17 KST 2025 INFO: [FETCH]  [Created on: Sun Mar 16 21:02:17 KST 2025, duration: 1, connection-id: 2776, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)]
ProxyPreparedStatement.java:61
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Sun Mar 16 21:02:17 KST 2025 INFO: [QUERY] insert into recipe_input_keyword (keyword,recipe_seq) values ('초보',18) [Created on: Sun Mar 16 21:02:17 KST 2025, duration: 0, connection-id: 2776, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)]
ProxyPreparedStatement.java:61
Sun Mar 16 21:02:17 KST 2025 INFO: [FETCH]  [Created on: Sun Mar 16 21:02:17 KST 2025, duration: 0, connection-id: 2776, statement-id: 0, resultset-id: 0,	at com.zaxxer.hikari.pool.ProxyPreparedStatement.executeUpdate(ProxyPreparedStatement.java:61)]
```


- batch insert를 위해선 option이 한 가지 더 필요한 것이다.
- 바로 batch size이다.

```sh
spring.jpa.properties.hibernate.jdbc.batch_size = 50
```

- hibernate 상으로는 insert문이 여러개지만, natvieSQL로는 batch insert가 이뤄지고 있다.

```
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        recipe_input_keyword
        (keyword, recipe_seq) 
    values
        (?, ?)
Sun Mar 16 20:58:13 KST 2025 INFO: [QUERY] insert into recipe_input_keyword (keyword,recipe_seq) values ('일반식',17),('한식',17),('매운맛',17),('초보',17)
```


## <span style="color:#802548">_recipe history 저장하기- controller_</span>
- result를 저장해서 성공하면, recipeSeq를 return한다.
- return 한 recipeSeq 값이 있다면 성공하여 저장된 것으로 간주한다.
- 이 recipeSeq를 js closure의 free variable로 보유하여 2번 저장을 방지하는 형태다.

```java
@Controller
@RequiredArgsConstructor
@Slf4j
public class RecipeHistoryController {

    private final RecipeHistoryService recipeHistoryService;
    
    @PostMapping("/recipe/history/save")
    @ResponseBody
    public ResponseEntity<Long> saveRecipeHistory(@RequestBody RecipeHistroyRequsetDTO recipeHistroyRequsetDTO) throws InterruptedException {

        return ResponseEntity.ok(recipeHistoryService.saveRecipeHisotry(recipeHistroyRequsetDTO));
    }
}
```


## <span style="color:#802548">_recipe의 history 화면 구성하기- recommend-result.html_</span>


- 결과 화면은 별표를 누르면 저장이 되게 된다.
- 별도의 페이지네이션이 불필요하기 때문에 그냥 thymeleaf로 parsing해서 구성한다.

```js
<!DOCTYPE html>
<html lang="ko" xmlns:th="http://thymeleaf.org">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>추천된 레시피</title>
        <!-- 내부 import -->
        <link rel="stylesheet"  th:href="@{/css/globalstyles.css}" />
        <link rel="stylesheet"  th:href="@{/css/recommend_result.css}" />
        <script th:src="@{/js/recommend_result.js}" defer></script>
        <!-- 결과 페이지 스타일 -->
        <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11.16.0/dist/sweetalert2.all.min.js"></script>
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/sweetalert2@11.16.0/dist/sweetalert2.min.css" />
    </head>
    <body>
        <div class="container">
            <div class="header" th:replace="~{fragment/header :: top-menu}"></div>

            <!-- ✅ 레시피 결과 컨테이너 -->
            <div class="recipe-container">
                <h1>추천된 레시피</h1>
                <!-- ✅ 레시피 카드 -->
                <div class="recipe-card">
                    <!-- ⭐ 즐겨찾기 버튼 (우측 상단 배치) -->
                    <button class="bookmark-btn">
                        <img th:src="@{/image/star1.png}" alt="즐겨찾기" id="bookmarkIcon" />
                    </button>

                    <div class="recipe-info">
                        <h2 th:text="${recipe.title}" class="recipe-title"></h2>

                        <h3>🍽️ 재료</h3>
                        <ul class="recipe-ingredients">
                            <li th:each="item, status : ${recipe.ingredients}" th:text="${item}"></li>
                        </ul>

                        <h3 class="recipe-steps-header">👨‍🍳 만드는 방법</h3>
                        <ol class="recipe-steps">
                            <li th:each="item, status : ${recipe.cookingMethods}" th:text="${item}"></li>
                        </ol>

                        <input type="hidden" th:value="${recipe.recipeConditionDTO.usage}" name="usage" id="usage">
                        <input type="hidden" th:value="${recipe.recipeConditionDTO.menu}" name="menu" id="menu">
                        <input type="hidden" th:value="${recipe.recipeConditionDTO.taste}" name="taste" id="taste">
                        <input type="hidden" th:value="${recipe.recipeConditionDTO.level}" name="level" id="level">
                    </div>
                </div>

                <!-- ✅ 다시 추천받기 버튼 -->
                <div class="footer">
                    <button onclick="window.location.href='/'" class="recommend-btn">
                        메인페이지
                    </button>
                    <button onclick="window.location.href='/recipe/recommend'" class="recommend-btn">다시 추천받기</button>
                    <button onclick="window.location.href='/mypage/recipeSave'" class="recommend-btn">
                        나의 레시피
                    </button>
                </div>
            </div>
        </div>
    </body>
</html>
```

## <span style="color:#802548">_recipe의 history 화면 구성하기- recommend-result.js_</span>

- 별표를 누르면 서버에서 다시 응답을 받아올 때까지 버튼이 눌려서는 안된다.
    - 따라서 이를 closure로 처리한다.
    - 더불어 한 번 저장된 것은 다시 저장되면 안되므로 그것또한 clsoure의 free variable으로 처리한다.
    - 저장은 ajax인데, fetch - then이 아닌 async await로 구성된다.

```js
function createToggleBookmark() {
    let isClicked = false; // This will persist between clicks
    let recipeSeq = 0; // Initially, there's no recipe sequence

    return async function toggleBookmark() {
        let bookmarkBtn = document.querySelector('.bookmark-btn');
        let bookmarkIcon = document.getElementById('bookmarkIcon');


        if (!bookmarkBtn.classList.contains('bookmarked')) {
            bookmarkBtn.classList.add('bookmarked');
            bookmarkIcon.src = '/image/star2.png'; 
        }

        console.log(isClicked);
        if (isClicked) {
            Swal.fire({
                icon: 'error',
                title: '이미 클릭하셨습니다.',
                text: '서버에서 프로세스가 진행중입니다.',
                confirmButtonColor: '#ff7f50',
                confirmButtonText: '확인',
            });
            return;
        }

        // Block further requests if recipeSeq is already set (meaning data has been saved)
        if (recipeSeq !== 0) {
            Swal.fire({
                icon: 'warning',
                title: '이미 저장된 데이터입니다.',
                text: '저장은 이미 완료되었습니다.',
                confirmButtonColor: '#ff7f50',
                confirmButtonText: '확인',
            });
            return;
        }

        isClicked = true; // Disable further clicks until the API is done
        const title = document.querySelector(".recipe-title").textContent;
        const outputContent = document.querySelector(".recipe-info").innerHTML; // HTML itself as a string
        const usage = document.querySelector("input[name='usage']").value;
        const menu = document.querySelector("input[name='menu']").value;
        const taste = document.querySelector("input[name='taste']").value;
        const level = document.querySelector("input[name='level']").value;

        const data = {
            title,
            recipeCondition: { usage, menu, taste, level },
            outputContent
        };

        try {
            const response = await fetch('/recipe/history/save', {
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(data),
                method: 'POST'
            });

            if (!response.ok) {
                removeBoomark();
                throw new Error('API Error');
            }

            recipeSeq = await response.json(); // Store the server response (recipeSeq)

            Swal.fire({
                icon: 'success',
                title: '저장 성공!',
                text: '저장이 완료되었습니다.',
                confirmButtonColor: '#ff7f50',
                confirmButtonText: '확인',
            });

        } catch (error) {
            removeBoomark();
            console.error("Error:", error);
        } finally {
            isClicked = false; // Re-enable clicking after the response is handled (success or failure)
        }
    };
}

// Attach the function to the button event
const toggleBookmark = createToggleBookmark();
document.querySelector('.bookmark-btn').addEventListener('click', toggleBookmark);

function removeBoomark() {
    document.querySelector('.bookmark-btn').classList.remove('bookmarked');
    document.getElementById('bookmarkIcon').src = '/image/star1.png';
}
```


## <span style="color:#802548">_recipe의 history 저장하기- replay attack 방지용 nonce_</span>
- replay attack, 새로고침을 통한 무한 저장 등을 방지하기 위해 nonce 값을 도입하였다.
- 우선 UUID로 이뤄진 nonce를 session에 저장한다.

```java
@PostMapping("/recipe/chatGPT")
@ResponseBody
public void viewRecipeOutput(@RequestBody RecipeUserRequestDTO recipeUserRequestDTO, HttpSession session) {

    RecipeUserResponseDTO response = recipeService.getRecipeResponse(recipeUserRequestDTO);
    
    session.setAttribute("recipe", response);
    String newUUID = UUID.randomUUID().toString(); 
    session.setAttribute("nonce", newUUID);
}
```

- model로 보낸 nonce는 html에서 hidden으로 보관한다.

```html
<input type="hidden" th:value="${nonce}" name="nonce" id="nonce">
<input type="hidden" th:value="${recipe.recipeConditionDTO.usage}" name="usage" id="usage">
<input type="hidden" th:value="${recipe.recipeConditionDTO.menu}" name="menu" id="menu">
<input type="hidden" th:value="${recipe.recipeConditionDTO.taste}" name="taste" id="taste">
<input type="hidden" th:value="${recipe.recipeConditionDTO.level}" name="level" id="level">
```

- 보관한 것을 js에서 보낸다.

```js
 const title = document.querySelector(".recipe-title").textContent;
const outputContent = document.querySelector(".recipe-info").innerHTML; // HTML itself as a string
const usage = document.querySelector("input[name='usage']").value;
const menu = document.querySelector("input[name='menu']").value;
const taste = document.querySelector("input[name='taste']").value;
const level = document.querySelector("input[name='level']").value;
const nonce = document.querySelector("input[name='nonce']").value;

const data = {
    title,
    recipeCondition: { usage, menu, taste, level },
    outputContent,
    nonce
};
```


- nonce를 확인하여 저장하고, 저장이 성공하면 nonce 값을 지워 replay, refresh를 통한 무한 저장을 방지한다.

```java
@PostMapping("/recipe/history/save")
@ResponseBody
public ResponseEntity<Long> saveRecipeHistory(@RequestBody RecipeHistroyRequsetDTO recipeHistroyRequsetDTO, HttpSession session) throws InterruptedException {
    String userNonce = (String) session.getAttribute("nonce");

    if (!StringUtils.hasText(userNonce) || !recipeHistroyRequsetDTO.getNonce().equals(userNonce)) {
        throw new RuntimeException("nonce 값이 존재하지 않습니다.");
    }

    Long savedRecipeSeq = recipeHistoryService.saveRecipeAndReturnSavedPK(recipeHistroyRequsetDTO);
    if(savedRecipeSeq != 0) {
        session.removeAttribute("nonce");   //recipe save 성공 시에는 즉시 session에서 nonce 값 제거하여 데이터 정합성 확보
    }
    
    return ResponseEntity.ok(savedRecipeSeq);
}
```


## <span style="color:#802548">_recipe의 history 다시보기- controller_</span>


- recipe의 history를 다시보기 위한 page를 만든다.
- 처음에는 아래처럼 thymeleaf를 통한 fetch를 생각했었다.

```java
@Controller
@RequiredArgsConstructor
public class RecipeMyPageController {

    private final RecipeMyPageService recipeMyPageService;

    @GetMapping("/mypage/recipeSave")
    public String recipeSave(Model model, @AuthenticationPrincipal LoginUserDetails loginUserDetails) {
        model.addAttribute("recipes", recipeMyPageService.findAllByUser(loginUserDetails.getUserSeq()));
        return "mypage/recipeSave_mypage"; 
    }


    
}
```

- 그러나 pagination에 따라 매번 page가 reload되는 것이 UX 상 별로라 생각되었다.
- 따라서 페이지 내에서 ajax를 날리는 식으로 바꿨다.

```java
@GetMapping("/mypage/getRecipeSave")
@ResponseBody
public Page<RecipeMyPageResponse> recipeSave(Model model, @AuthenticationPrincipal LoginUserDetails loginUserDetails, 
                                    @RequestParam(name = "page", required = false) int page) {
    Page<RecipeMyPageResponse> recipe = recipeMyPageService.findAllRecipeByUser(loginUserDetails.getUserSeq(), page);

    return recipe; 
}


@GetMapping("/mypage/recipeSave")
public String viewRecipeSave() {

    return "mypage/recipeSave_mypage";
}
```

## <span style="color:#802548">_recipe의 history 다시보기- dto_</span>

- Page에 필요한 response는 아래정도다.

```java
@Builder
public record RecipeMyPageResponse(
                                    Long id,
                                    String title,
                                    String outputHTML,
                                    List<String> inputKeywords,
                                    LocalDateTime createdDateTime
                                ) {

    public static RecipeMyPageResponse toDTO(RecipeEntity recipeEntity) {
		return RecipeMyPageResponse.builder()
				.id(recipeEntity.getRecipeSeq())
                .title(recipeEntity.getRecipeOutputEntity().getRecipeTitle())
                .outputHTML(recipeEntity.getRecipeOutputEntity().getOutputContent())
                .inputKeywords(recipeEntity.getRecipeInputKeywordEntityList().stream().map((entity) -> entity.getRecipeComplicatedPK().getKeyword()).toList())
                .createdDateTime(recipeEntity.getCreatedAt())
				.build();
	}
    
}
```

- Page를 그대로 return하여 보냈더니 아래와 같은 오류가 뜬다.

```java
@GetMapping("/mypage/getRecipeSave")
@ResponseBody
public Page<RecipeMyPageResponse> recipeSave(Model model, @AuthenticationPrincipal LoginUserDetails loginUserDetails, 
                                    @RequestParam(name = "page", required = false) int page) {
    Page<RecipeMyPageResponse> recipe = recipeMyPageService.findAllRecipeByUser(loginUserDetails.getUserSeq(), page);

    return recipe; 
}
```

```
2025-03-12T23:45:51.514+09:00  WARN 5612 --- [project] [nio-9005-exec-5] ration$PageModule$WarningLoggingModifier : Serializing PageImpl instances as-is is not supported, meaning that there is no guarantee about the stability of the resulting JSON structure!
	For a stable JSON structure, please use Spring Data's PagedModel (globally via @EnableSpringDataWebSupport(pageSerializationMode = VIA_DTO))
	or Spring HATEOAS and Spring Data's PagedResourcesAssembler as documented in https://docs.spring.io/spring-data/commons/reference/repositories/core-extensions.html#core.web.pageables.
```

- 따라서 별도의 DTO로 변환하여 사용하기로 결정했다.
- Content의 type은 무엇이 될지 알 수 없기 때문에 generics를 사용한다.

```java
@ToString
@Builder
@AllArgsConstructor
@Setter
@Getter
@NoArgsConstructor
public class PageDTO<T> {
    private List<T> content;
    private int pageNumber;
    private int totalPages;
    private long totalElements;
}
```

- builder를 통한 TODTO를 만들지 않는다면 아래와 같이 직접 가져오면 된다.
- generics를 주었기 때문에, builder() 앞에 어떤 type인지 명시해준다.

```java
 @GetMapping("/mypage/getRecipeSave")
@ResponseBody
public PageDTO<RecipeMyPageResponse> recipeSave(Model model, @AuthenticationPrincipal LoginUserDetails loginUserDetails, 
                                    @RequestParam(name = "page", required = false) int page) {
    Page<RecipeMyPageResponse> recipePage = recipeMyPageService.findAllRecipeByUser(loginUserDetails.getUserSeq(), page);

    PageDTO<RecipeMyPageResponse> recipe = PageDTO.<RecipeMyPageResponse>builder().content(recipePage.getContent())
                                                            .pageNumber(recipeContent.getNumber())
                                                            .totalElements(recipeContent.getTotalElements())
                                                            .totalPages(recipeContent.getTotalPages())
                                                            .build();
    

    return recipe; 
}
```

- TODTO를 만들게 되면 아래와 같은 형태로 만들어준다.
- genercis여도 parameter를 받아서 T를 미리 compile부터 알 수 있게 된다.
- 그럼 runtime이 아니라 compile time에 해석가능하기에 generics를 써도 non-static reference error가 나지 않는다.

```java
public class PageDto<T> {
    private List<T> content;
    private int pageNumber;
    private int totalPages;
    private long totalElements;

    public static <T> PageDTO<T> TODTO(Page<T> page) {
        return PageDTO.<T>builder().content(page.getContent())
                                    .pageNumber(page.getNumber())
                                    .totalElements(page.getTotalElements())
                                    .totalPages(page.getTotalPages())
                                    .build();
    }
}


@GetMapping("/mypage/getRecipeSave")
@ResponseBody
public PageDTO<RecipeMyPageResponse> recipeSave(Model model, @AuthenticationPrincipal LoginUserDetails loginUserDetails, 
                                    @RequestParam(name = "page", required = false) int page) {
    Page<RecipeMyPageResponse> recipeContent = recipeMyPageService.findAllRecipeByUser(loginUserDetails.getUserSeq(), page);
    PageDTO<RecipeMyPageResponse> recipes = PageDTO.TODTO(recipeContent);

    return recipes; 
}
```


## <span style="color:#802548">_recipe의 history 다시보기- repository_</span>

- 처음에는 아래처럼 query method를 활용하려고 시도했으나, 역시 무리라고 생각이 들었다.
- 특히 pagenation까지 하게 되면 알아보기 어렵다는 생각이 들었다.

```java
public interface RecipeMyPageRepository extends JpaRepository<RecipeEntity, Long>{

    List<RecipeEntity> findByUserEntity_UserSeqAndRecipeInputKeywordEntityListIsNotNullAndRecipeOutputEntityIsNotNull(Long userSeq);
    
} 
```

- 따라서 아래처럼 JPQL을 사용하게 바꿨다.
- distinct가 없으면 row가 구별되지 않으므로 entity관계의 OneToOne 등의 관계를 만들기 어렵다.

```java
public interface RecipeMyPageRepository extends JpaRepository<RecipeEntity, Long>{
    
    @Query("""
        SELECT DISTINCT r FROM RecipeEntity r
        JOIN r.recipeInputKeywordEntityList k
        WHERE r.userEntity.userSeq = :userSeq
        AND r.recipeOutputEntity IS NOT NULL
    """)
    Page<RecipeEntity> findRecipesWithPagination(
        @Param("userSeq") Long userSeq,
        Pageable pageable
    );
    
} 
```

- OneTOne과 oneToMany에 따른 여러 entity들이 같이 조회되는 모습이다.
- 사실 redundant한 query가 많기 떄문에 한번에 가져와서 Java 내에서 parsing하는 게 필요해 보인다.
- 보면 count query가 없는데 count query도 자동으로 날라간 것이 보인다. 
    - Page interface는 자동으로 count query를 날린다.
    - totalPage를 알기 위해서 자동으로 날리게끔 구성되어 있다.

```sql
Hibernate: 
    select
        distinct re1_0.recipe_seq,
        re1_0.created_at,
        re1_0.user_seq 
    from
        recipe re1_0 
    join
        recipe_input_keyword rikel1_0 
            on re1_0.recipe_seq=rikel1_0.recipe_seq 
    left join
        recipe_output_content roe1_0 
            on re1_0.recipe_seq=roe1_0.recipe_seq 
    where
        re1_0.user_seq=? 
        and roe1_0.recipe_output_content_seq is not null 
    order by
        re1_0.created_at 
    limit
        ?

Hibernate: 
    select
        roe1_0.recipe_output_content_seq,
        roe1_0.output_content,
        roe1_0.recipe_seq,
        roe1_0.recipe_title 
    from
        recipe_output_content roe1_0 
    where
        roe1_0.recipe_seq=?

Hibernate: 
    select
        ue1_0.user_seq,
        ue1_0.created_at,
        ue1_0.is_deleted,
        ue1_0.roles,
        ue1_0.updated_at,
        ue1_0.user_email,
        ue1_0.user_id,
        ue1_0.user_name,
        ue1_0.user_password 
    from
        user ue1_0 
    where
        ue1_0.user_seq=?

Hibernate: 
    select
        roe1_0.recipe_output_content_seq,
        roe1_0.output_content,
        roe1_0.recipe_seq,
        roe1_0.recipe_title 
    from
        recipe_output_content roe1_0 
    where
        roe1_0.recipe_seq=?

Hibernate: 
    select
        roe1_0.recipe_output_content_seq,
        roe1_0.output_content,
        roe1_0.recipe_seq,
        roe1_0.recipe_title 
    from
        recipe_output_content roe1_0 
    where
        roe1_0.recipe_seq=?

Hibernate: 
    select
        count(distinct re1_0.recipe_seq) 
    from
        recipe re1_0 
    join
        recipe_input_keyword rikel1_0 
            on re1_0.recipe_seq=rikel1_0.recipe_seq 
    left join
        recipe_output_content roe1_0 
            on re1_0.recipe_seq=roe1_0.recipe_seq 
    where
        re1_0.user_seq=? 
        and roe1_0.recipe_output_content_seq is not null

Hibernate: 
    select
        rikel1_0.recipe_seq,
        rikel1_0.keyword 
    from
        recipe_input_keyword rikel1_0 
    where
        rikel1_0.recipe_seq=?

Hibernate: 
    select
        rikel1_0.recipe_seq,
        rikel1_0.keyword 
    from
        recipe_input_keyword rikel1_0 
    where
        rikel1_0.recipe_seq=?

Hibernate: 
    select
        rikel1_0.recipe_seq,
        rikel1_0.keyword 
    from
        recipe_input_keyword rikel1_0 
    where
        rikel1_0.recipe_seq=?
```

- redundant한 query가 덜 나게끔 한번에 받아오게 한다.
- EntityGraph를 추가하면 된다. 그럼 한꺼번에 받아오게 변경된다.

```java
@EntityGraph(attributePaths = {"recipeOutputEntity", "recipeInputKeywordEntityList", "userEntity"})
@Query("""
    SELECT DISTINCT r FROM RecipeEntity r
    JOIN r.recipeInputKeywordEntityList k
    WHERE r.userEntity.userSeq = :userSeq
    AND r.recipeOutputEntity IS NOT NULL
""")
Page<RecipeEntity> findRecipesWithPagination(
    @Param("userSeq") Long userSeq,
    Pageable pageable
);
```

- 대신 query를 보면 알 수 있듯, recipe_output_content에는 recipe_seq index를 걸어야 한다.
- recipe_input_keyword에도 마찬가지다. join 조건절이 해당 column을 사용하기 때문이다.

```sql
select
    distinct re1_0.recipe_seq,
    re1_0.created_at,
    rikel1_0.recipe_seq,
    rikel1_0.keyword,
    roe1_0.recipe_output_content_seq,
    roe1_0.output_content,
    roe1_0.recipe_seq,
    roe1_0.recipe_title,
    ue1_0.user_seq,
    ue1_0.created_at,
    ue1_0.is_deleted,
    ue1_0.roles,
    ue1_0.updated_at,
    ue1_0.user_email,
    ue1_0.user_id,
    ue1_0.user_name,
    ue1_0.user_password 
from
    recipe re1_0 
join
    recipe_input_keyword rikel1_0 
        on re1_0.recipe_seq=rikel1_0.recipe_seq 
left join
    recipe_output_content roe1_0 
        on re1_0.recipe_seq=roe1_0.recipe_seq 
left join
    user ue1_0 
        on ue1_0.user_seq=re1_0.user_seq 
where
    re1_0.user_seq=? 
    and roe1_0.recipe_output_content_seq is not null 
order by
    re1_0.created_at

Hibernate: 
select
    count(distinct re1_0.recipe_seq) 
from
    recipe re1_0 
join
    recipe_input_keyword rikel1_0 
        on re1_0.recipe_seq=rikel1_0.recipe_seq 
left join
    recipe_output_content roe1_0 
        on re1_0.recipe_seq=roe1_0.recipe_seq 
where
    re1_0.user_seq=? 
    and roe1_0.recipe_output_content_seq is not null
```

- is not null도 필요가 없는 조건이다. recipe를 만드는 순간, 무조건 recipe_output_content를 같이 생성하기 때문이다.
- 따라서 빼준다. 빼주면서 동시에 join 조건으로서 recipeOutputEntity는 유지되어야 하므로 join 문에 추가해준다.
- join도 헷갈릴 우려가 있으니 INNER JOIN이라고 명시해주자.

```java
@EntityGraph(attributePaths = {"recipeOutputEntity", "recipeInputKeywordEntityList", "userEntity"})
@Query("""
    SELECT DISTINCT r FROM RecipeEntity r
    INNER JOIN r.recipeInputKeywordEntityList k
    INNER JOIN r.recipeOutputEntity j
    WHERE r.userEntity.userSeq = :userSeq
""")
Page<RecipeEntity> findRecipesWithPagination(
    @Param("userSeq") Long userSeq,
    Pageable pageable
);
```

- sql 상에서는 여전히 join이라고 뜨지만, on keyword가 붙어있기 때문에 cross join은 아니다.
- 따라서 inner join이라고 생각하면 된다.

```sql
select
    distinct re1_0.recipe_seq,
    re1_0.created_at,
    rikel1_0.recipe_seq,
    rikel1_0.keyword,
    roe1_0.recipe_output_content_seq,
    roe1_0.output_content,
    roe1_0.recipe_seq,
    roe1_0.recipe_title,
    ue1_0.user_seq,
    ue1_0.created_at,
    ue1_0.is_deleted,
    ue1_0.roles,
    ue1_0.updated_at,
    ue1_0.user_email,
    ue1_0.user_id,
    ue1_0.user_name,
    ue1_0.user_password 
from
    recipe re1_0 
join
    recipe_input_keyword rikel1_0 
        on re1_0.recipe_seq=rikel1_0.recipe_seq 
join
    recipe_output_content roe1_0 
        on re1_0.recipe_seq=roe1_0.recipe_seq 
left join
    user ue1_0 
        on ue1_0.user_seq=re1_0.user_seq 
where
    re1_0.user_seq=? 
order by
    re1_0.created_at
Hibernate: 
select
    count(distinct re1_0.recipe_seq) 
from
    recipe re1_0 
join
    recipe_input_keyword rikel1_0 
        on re1_0.recipe_seq=rikel1_0.recipe_seq 
left join
    recipe_output_content roe1_0 
        on re1_0.recipe_seq=roe1_0.recipe_seq 
where
    re1_0.user_seq=? 
```

- 그런데 보면 user는 left join을 수행하고 있다.
- user가 탈퇴가 아니라 아예 DB상에서 삭제된 경우에도 보여주는 recipe service가 존재한다면, user가 null이어도 값을 가져야 한다. 따라서 left join이 맞다.
- 그런데 mypage에서 보여주는 것은, user가 무조건 있어야 접근 가능한 페이지기 때문에, left join을 쓰면 성능만 구려진다.
- EntityGraph에 UserEntity를 넣으면, userEntity는 무조건 보유하게 되는 조건이 아니기 때문에 아마 null일 경우를 대비해 left join으로 만드는 것으로 보인다.
- 무조건 user가 있다는 전제가 성립하기 때문에 여기서는 EntitityGraph에서 UserEntity를 지워준다.

```java
@EntityGraph(attributePaths = {"recipeOutputEntity", "recipeInputKeywordEntityList"})
@Query("""
    SELECT DISTINCT r FROM RecipeEntity r
    INNER JOIN r.recipeInputKeywordEntityList k
    INNER JOIN r.recipeOutputEntity j
    INNER JOIN r.userEntity u
    WHERE u.userSeq = :userSeq
""")
Page<RecipeEntity> findRecipesWithPagination(
    @Param("userSeq") Long userSeq,
    Pageable pageable
);
```

- 그럼 user entity의 내용도 거의 안가져오고, join도 left join에서 inner join으로 바뀐다.

```sql
select
    distinct re1_0.recipe_seq,
    re1_0.created_at,
    rikel1_0.recipe_seq,
    rikel1_0.keyword,
    roe1_0.recipe_output_content_seq,
    roe1_0.output_content,
    roe1_0.recipe_seq,
    roe1_0.recipe_title,
    re1_0.user_seq 
from
    recipe re1_0 
join
    recipe_input_keyword rikel1_0 
        on re1_0.recipe_seq=rikel1_0.recipe_seq 
join
    recipe_output_content roe1_0 
        on re1_0.recipe_seq=roe1_0.recipe_seq 
join
    user ue1_0 
        on ue1_0.user_seq=re1_0.user_seq 
where
    ue1_0.user_seq=? 
order by
    re1_0.created_at

select
    count(distinct re1_0.recipe_seq) 
from
    recipe re1_0 
join
    recipe_input_keyword rikel1_0 
        on re1_0.recipe_seq=rikel1_0.recipe_seq 
join
    recipe_output_content roe1_0 
        on re1_0.recipe_seq=roe1_0.recipe_seq 
join
    user ue1_0 
        on ue1_0.user_seq=re1_0.user_seq 
where
    ue1_0.user_seq=?
```


- 실제로 sql을 받아서 적어넣어보면, distinct keyword는 필요하지 않다.
- distinct가 있든 없든 sql 상의 결과값은 동일하다.
- DISTINCT를 JPQL에 넣은 이유는 JPQL의 distinct는 native sql의 distinct만이 아니라 다른 역할도 하기 때문이다.
    - duplicated된 entity들을 제거하여 JAVA Object 관점에서 뽑기 좋게 만들어준다.
    - 극한의 성능을 위해서라면 distinct를 빼고 JPA의 도움 없이 Java단에서 직접 entity들을 제거하고 합치는 작업을 해야한다.

//TODO
```java

```


## <span style="color:#802548">_recipe의 history 다시보기- service_</span>

- Pagination이 들어가기 때문에 pageable도 넣어준다.

```java
@Service
@RequiredArgsConstructor
public class RecipeMyPageService {

    private final RecipeMyPageRepository recipeMyPageRepository;

    public Page<RecipeMyPageResponse> findAllRecipeByUser(Long userSeq, int currentPage) {
        Pageable pageable = PageRequest.of(currentPage, 3, Sort.by(Sort.Direction.ASC, "createdAt"));
        
        return recipeMyPageRepository.findRecipesWithPagination(userSeq, pageable)
                                        .map(RecipeMyPageResponse::toDTO);
    };
    
}
```

- 참고로 Page로 return한 것은 위에서 만든 PageDTO로 다시 감싸서 front에 보내야 한다.


## <span style="color:#802548">_recipe의 history 다시보기- html_</span>

- js로 recipe가 fetch 되는 영역을 그려준다.
- 그를 위해 recipe-fetch-region을 따로 만들어줬다.

```html
<!DOCTYPE html>
<html lang="ko" xmlns:th="http://thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>마이페이지</title>
    <link rel="stylesheet"  th:href="@{/css/globalstyles.css}">
    <link rel="stylesheet"  th:href="@{/css/mypage.css}">
    <link rel="stylesheet"  th:href="@{/css/recipeSave_mypage.css}">
    <script th:src="@{/js/recipeSave_mypage.js}" defer></script>

</head>

<body>
    <div class="container">
        <div class="header" th:replace="~{fragment/header :: top-menu}"></div>

        <div class="mypage-container">
            <div th:replace="~{fragment/sidebar :: sidebar-menu}"></div>

            <!-- ✅ 메인 컨텐츠 -->
            <div class="mypage-content">
                <h1>레시피 저장 리스트</h1>
                <div class="recipe-list">
                    <ul id="recipe-fetch-region">

                    </ul>
                </div>

                <div class="pagination-container">
                    <div id="pagination">
                        <!-- AJAX로 페이지네이션 버튼 생성 -->
                    </div>
                </div>

                <div id="recipeModal" class="recipe-modal">
                    <div class="modal-content">
                        <span class="close-btn">&times;</span>
                        <div class="recipe-card">
                            <div class="recipe-info">

                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</body>

</html>
```


## <span style="color:#802548">_recipe의 history 다시보기- js_</span>

- 페이지를 누를 때마다 fetch가 이뤄져야 한다.
- HTML을 처음 만들 때부터 fetch가 이뤄지게 HTML 내에 click listener를 달았다.

```js
// 모달창 요소
const modal = document.getElementById("recipeModal");
const closeBtn = document.querySelector(".close-btn");

// 모달창 닫기 버튼
closeBtn.addEventListener("click", function () {
    modal.style.display = "none";
});

// 모달창 외부 클릭 시 닫기
window.addEventListener("click", function (event) {
    if (event.target === modal) {
        modal.style.display = "none";
    }
});

async function fetchRecipes(currentPage) {
    try {
        const response = await fetch(`/mypage/getRecipeSave?page=${currentPage}`);
        const data = await response.json(); // Assuming JSON response

        renderRecipes(data.content);
        generatePagination(data);

    } catch (error) {
        console.error("Error fetching recipes:", error);
    }
}

function renderRecipes(recipes) {
    const recipeHTML = document.querySelector("#recipe-fetch-region");
    recipeHTML.innerHTML = ""; // Clear previous content
    let html = '';

    recipes.forEach(recipe => {
        const outerHTML = escapeHTML(recipe.outputHTML);
        const createdDateTime = new Date(recipe.createdDateTime).toISOString().split('T')[0];
        html += `<li>
                        <a href="#" class="recipe-item" data-recipe-id="${recipe.id}" data-recipe-output="${outerHTML}">
                            <span class="recipe-name">${recipe.title}</span>
                            <div>
                            `;
        recipe.inputKeywords.forEach(keyword => {
            html += `
                                    <span class="option">${keyword}</span>
                        `;
        })
        html += `
                                    <span class="date">${createdDateTime}</span>
                            </div>
                        </a>
                    </li>
                `;
    });
    recipeHTML.innerHTML = html;
    handlerRecipeItemClick();
}

function generatePagination(resp) {
    let pagination = '';
    let currentPage = resp.number;
    let totalPages = resp.totalPages;
    let groupSize = 10;
    let startPage = Math.floor(currentPage / groupSize) * groupSize;
    let endPage = Math.min(startPage + groupSize, totalPages);

    if (startPage > 0) {
        pagination += `<button onclick="fetchRecipes(${startPage - 1})">◀ 이전</button>`;
    }

    for (let i = startPage; i < endPage; i++) {
        pagination += `<button onclick="fetchRecipes(${i})" class="${i === currentPage ? 'active' : ''}">${i + 1}</button>`;
    }

    if (endPage < totalPages) {
        pagination += `<button onclick="fetchRecipes(${endPage})">다음 ▶</button>`;
    }

    document.querySelector('#pagination').innerHTML= pagination;
}

function handlerRecipeItemClick() {
    for (const eventListener of document.querySelectorAll(".recipe-item")) {
        eventListener.addEventListener("click", function (event) {
            event.preventDefault(); // 페이지 이동 방지
            modal.style.display = "flex";

            const buttonTag = `<div class="footer">
                                    <button onclick="window.location.href='/board/boardWrite'">게시글 작성하기</button>
                                </div>`;
            document.querySelector(".recipe-info").innerHTML = this.dataset.recipeOutput + buttonTag;
        })
    }
}

// Load first page
fetchRecipes(0);

function escapeHTML(str) {
    return str.replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#039;");
}

```

- 그 중 주목해야 할 js는 아래의 js다.
- 아래 js에서 groupSize는 page에 보여줄 size가 아니다.
- page에서 이전과 다음 버튼이 나오게 할 분수령의 pageNumber다.
    - 따라서 총 10 page가 넘어가야 다음 버튼이 생긴다.
    - 다음 버튼을 눌러 11 page 쪽을 도달하면, 이전 버튼이 생긴다.
    - 이러한 형태가 매 10page씩 반복된다.

```js
function generatePagination(resp) {
    let pagination = '';
    let currentPage = resp.number;
    let totalPages = resp.totalPages;
    let groupSize = 10;
    let startPage = Math.floor(currentPage / groupSize) * groupSize;
    let endPage = Math.min(startPage + groupSize, totalPages);

    if (startPage > 0) {
        pagination += `<button onclick="fetchRecipes(${startPage - 1})">◀ 이전</button>`;
    }

    for (let i = startPage; i < endPage; i++) {
        pagination += `<button onclick="fetchRecipes(${i})" class="${i === currentPage ? 'active' : ''}">${i + 1}</button>`;
    }

    if (endPage < totalPages) {
        pagination += `<button onclick="fetchRecipes(${endPage})">다음 ▶</button>`;
    }

    document.querySelector('#pagination').innerHTML= pagination;
}
```