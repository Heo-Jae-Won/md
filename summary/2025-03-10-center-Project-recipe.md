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
    
    // ObjectMapper 빈 등록

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
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
- Serializable도 implements해줘야 한다.

```java
@ToString
@Builder
@AllArgsConstructor
@Setter
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class ChatGPTRequestDTO implements Seriablizable {
    private String model;
    private List<MessageDTO> messages;
    private Double temperature;
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
public class RecipeChatGPTResponseDTO implements Serializable {
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
- 

```java
@ToString
@Builder
@AllArgsConstructor
@Setter
@Getter
@Slf4j
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class MessageChatGPTDTO implements Serializable {

    private String role;
    private String content;

    public static MessageChatGPTDTO TOSYSTEMMESSAGE() {
        return MessageChatGPTDTO.builder()
                .role("system")
                .content("너는 이제 음식 레시피를 추천해주는거야")
                .build();

    }
```

- 재료, 음식용도, 메뉴, 맛, 난이도의 변수를 받아올 수 있는 DTO를 만든다.
- 아래와 같이 간편한 방식은 Java 15이상에서만 가능하다. 

```java
.
.
.
public class MessageChatGPTDTO implements Serializable {
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

- recipeConditionDTO는 추가될 수 있기 때문에 따로 class를 만들어 관리한다.

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
 // //GPT-4
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
- session이 시간이 지나 무효화된 경우, 어떻게 처리할 지는 이제 front에 달렸다.
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
    - 경로가 바껴도 정상 작동하게 하려면 두번째 것으로 하자.

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
$(document).on('submit', '#recommend-form', function (e) {
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

//TODO. properties 파일 암호화 관련 추가 --jascyprt

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



```sql

```



//TODO. recipe-output table

```sql

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
@EqualsAndHashCode
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
@Builder
@Getter
public class RecipeComplicatedPK implements Serializable {

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


//TODO bulk insert 적용하기
//TODO 쓸데없는 쿼리문 줄이기


## <span style="color:#802548">_recipe history 저장하기- controller_</span>


- result를 저장해서 성공하면, recipeSeq를 return한다.
- return 한 recipeSeq 값이 있다면 성공하여 저장된 것으로 간주한다.
- 이 recipeSeq를 free variable로 보유하여 2번 저장을 방지하는 형태다.

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

## <span style="color:#802548">_recipe의 history 저장하기- requestDTOs_</span>

- 

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
//TODO replay attack을 방지하는 nonce 값 추가



## <span style="color:#802548">_recipe의 history 다시보기- controller_</span>



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


```java
public interface RecipeMyPageRepository extends JpaRepository<RecipeEntity, Long>{

    List<RecipeEntity> findByUserEntity_UserSeqAndRecipeInputKeywordEntityListIsNotNullAndRecipeOutputEntityIsNotNull(Long userSeq);
    
} 
```

```java
@Service
@RequiredArgsConstructor
public class RecipeMyPageService {

    private final RecipeMyPageRepository recipeMyPageRepository;


    public List<RecipeMyPageResponse> findAllByUser(Long userSeq) {
        return recipeMyPageRepository.findByUserEntity_UserSeqAndRecipeInputKeywordEntityListIsNotNullAndRecipeOutputEntityIsNotNull(userSeq)
                                        .stream().map(RecipeMyPageResponse::toDTO).toList();
         
    };
    
    
}
```


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
                    <ul>
                        <li th:each="recipe : ${recipes}">
                            <a href="#" class="recipe-item" th:data-recipe-id="${recipe.id}" th:data-recipe-output="${recipe.outputHTML}">
                                <span class="recipe-name" th:text="${recipe.title}">레시피 이름</span>
                                <div>
                                    <span class="option" th:each="tag : ${recipe.inputKeywords}" th:text="${tag}">카테고리</span>
                                    <span class="date" th:text="${#temporals.format(recipe.createdDateTime, 'yyyy-MM-dd')}">2025-02-12</span>
                                </div>
                            </a>
                        </li>

                        <th:block th:if="${recipes.size() == 0}">
                            <p>레시피가 없습니다.</p>
                        </th:block>

                    </ul>
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

```js
const recipeItems = document.querySelectorAll(".recipe-item");

// 모달창 요소
const modal = document.getElementById("recipeModal");
const closeBtn = document.querySelector(".close-btn");

recipeItems.forEach(item => {
    item.addEventListener("click", function (event) {
        event.preventDefault(); // 페이지 이동 방지
        modal.style.display = "flex";
    })
});

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

for (const eventListener of document.querySelectorAll(".recipe-item")) {
    eventListener.addEventListener("click", function () {
        const buttonTag = `<div class="footer">
                                <button onclick="window.location.href='/board/boardWrite'">게시글 작성하기</button>
                            </div>`;
        document.querySelector(".recipe-info").innerHTML = this.dataset.recipeOutput + buttonTag;
    })
}
```