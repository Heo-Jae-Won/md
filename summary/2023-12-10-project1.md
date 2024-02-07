## <span style="color:#802548">_이중 for문, 이중 if문 활용하여 data 가공하기_</span> 


- 만약 data가 없는 경우에도 data 구조를 만들어야 한다면 아래와 같이 필수 값만 넣는다. 
- 다만 향후 db에 data가 있으면 언제든 값이 존재하게끔 동시에 설계해야 한다.
- 그 경우 아래와 같이 배열을 이용할 수 있다. npc위치와 npc유형을 기준으로 값을 가져온다고 가정한다면 아래와 같이 쓸 수 있다. 
- npc위치가 큰 분류고, npc유형이 작은 분류라고 한다면 아래와 같이 배열을 만든다.

```java
String[] npc위치= {"a","b","c","d"};
```

- 그리고 db에서 가져왔던 list에서 값을 가져온다. 그러기 위해선 아래와 같은 이중 for문이 필요하다.
- db에서 가져온 List data type의 값을 꺼내서 재구조화하기 위해 필요한 절차다.

```java
for(int i=0;i<npc위치.length;i++){
    for(int j=0;j<gameList.size();j++){
   
    }
}
```

- 새롭게 가공한 data로 list를 return하기 위해 아래와 같이 newNpcList 변수를 초기화한다. 
- 그리고 newNpcList에 들어가는 element를 채워넣기 위해 바깥 for문에 tempMap 변수를 초기화한다.

```java
List<HashMap<String,Object>> newNpcList=new ArrayList<>();
String[] npc위치= {"a","b","c","d"};
for(int i=0;i<npc위치.length;i++){
        HashMap<String,Object> tempMap=new HashMap<String,Object>;
        for(int j=0;j<gameList.size();j++){   
```

- 이제 if문을 통해 원하는 data를 넣을 수 있게 제어한다.

```java
List<HashMap<String,Object>> newNpcList=new ArrayList<>();
String[] npc위치= {"a","b","c","d"};

for(int i=0;i<npc위치.length;i++){
        HashMap<String,Object> tempMap=new HashMap<String,Object>;
        for(int j=0;j<gameList.size();j++){  
               if(npc위치[i].equals(gameList.get(j).get("npc위치"))){
                      
               }
        }
}
```
- npc위치에 맞는 값을 가져오고 그 중에서도 npc유형이 다를 수 있기에 이중 if문으로 npc유형까지 확인해서 data를 받아온다.

```java
List<HashMap<String,Object>> newNpcList=new ArrayList<>();
String[] npc위치= {"a","b","c","d"};

for(int i=0;i<npc위치.length;i++){
        HashMap<String,Object> tempMap=new HashMap<String,Object>;
        for(int j=0;j<gameList.size();j++){  
               if(npc위치[i].equals(gameList.get(j).get("npc위치"))){
                   if(gameList.get(j).get("npc유형").equals("A")){   

                   }else if(gameList.get(j).get("npc유형").equals("B")){

                   }
               }
        }
}
```
- 아래와 같이 이제 값을 넣어준다.


```java
List<HashMap<String,Object>> newNpcList=new ArrayList<>();
String[] npc위치= {"a","b","c","d"};

for(int i=0;i<npc위치.length;i++){
        HashMap<String,Object> tempMap=new HashMap<String,Object>;
        for(int j=0;j<gameList.size();j++){  
               if(npc위치[i].equals(gameList.get(j).get("npc위치"))){
                   if(gameList.get(j).get("npc유형").equals("A")){   
                       tempMap.put("Atype_npc위치",npc위치[i]);
                       tempMap.put("Atype_npc유형",gameList.get(j).get("npc유형"));
                       tempMap.put("Atype_npc노출여부",gameList.get(j).get("npc노출여부"));
                       tempMap.put("Atype_npc노출일자",gameList.get(j).get("npc노출일자"));
                       tempMap.put("Atype_npc노출종료일자",gameList.get(j).get("npc노출종료일자"));
                   }else if(gameList.get(j).get("npc유형").equals("B")){
                        tempMap.put("Ltype_npc위치",npc위치[i]);
                        tempMap.put("Btype_npc유형",gameList.get(j).get("npc유형"));
                        tempMap.put("Btype_npc노출여부",gameList.get(j).get("npc노출여부"));
                        tempMap.put("Btype_npc노출일자",gameList.get(j).get("npc노출일자"));
                        tempMap.put("Btype_npc노출종료일자",gameList.get(j).get("npc노출종료일자"));
                   }
               }
        }
}
```
- group by로 결과를 가져오게 되면 count를 한번에 가져올 수가 없다. 
- 아래와 같이 += 대입자로 계속 더해주어야 한다. Oracle db에서 가져온 Number는 Java에서 BigDecimal로 저장되며 이는 int가 아니다. 
- 따라서 String 형변환 이후 다시 int로 형변환한다.

```java
List<HashMap<String,Object>> newNpcList=new ArrayList<>();
int registeredAtypeCount=0;
int displayedAtypeCount=0;
int registeredBtypeCount=0;
int displayedBtypeCount=0;
String[] npc위치= {"a","b","c","d"};

for(int i=0;i<npc위치.length;i++){
        HashMap<String,Object> tempMap=new HashMap<String,Object>;
        for(int j=0;j<gameList.size();j++){  
               if(npc위치[i].equals(gameList.get(j).get("npc위치"))){
                   if(gameList.get(j).get("npc유형").equals("A")){   
                       tempMap.put("Atype_npc위치",npc위치[i]);
                       tempMap.put("Atype_npc유형",gameList.get(j).get("npc유형"));
                       tempMap.put("Atype_npc노출여부",gameList.get(j).get("npc노출여부"));
                       tempMap.put("Atype_npc노출일자",gameList.get(j).get("npc노출일자"));
                       tempMap.put("Atype_npc노출종료일자",gameList.get(j).get("npc노출종료일자"));

                       registeredAtypeCount +=Integer.parseInt(String.valueOf(gameList.get(j).get("npc등록숫자")));
                       displayedAtypeCount +=Integer.parseInt(String.valueOf(gameList.get(j).get("npc노출숫자")));

                       tempMap.put("Atype_npc등록숫자",registeredAtypeCount);
                       tempMap.put("Atype_npc노출숫자",displayedAtypeCount);
                   }else if(gameList.get(j).get("npc유형").equals("B")){
                        tempMap.put("Ltype_npc위치",npc위치[i]);
                        tempMap.put("Btype_npc유형",gameList.get(j).get("npc유형"));
                        tempMap.put("Btype_npc노출여부",gameList.get(j).get("npc노출여부"));
                        tempMap.put("Btype_npc노출일자",gameList.get(j).get("npc노출일자"));
                        tempMap.put("Btype_npc노출종료일자",gameList.get(j).get("npc노출종료일자"));

                        registeredBtypeCount+=Integer.parseInt(String.valueOf(gameList.get(j).get("npc등록숫자")));
                        displayedBtypeCount+=Integer.parseInt(String.valueOf(gameList.get(j).get("npc노출숫자")));

                        tempMap.put("Btype_npc등록숫자",registeredAtypeCount);
                        tempMap.put("Btype_npc노출숫자",displayedAtypeCount);
                   }
               }
        }
}
```
- tempMap에 데이터를 다 채워넣었으니 이제 이 element를 newNpcList에 넣어준다.

```java
List<HashMap<String,Object>> newNpcList=new ArrayList<>();
int registeredAtypeCount=0;
int displayedAtypeCount=0;
int registeredBtypeCount=0;
int displayedBtypeCount=0;
String[] npc위치= {"M","P","B","F"};

for(int i=0;i<npc위치.length;i++){
        HashMap<String,Object> tempMap=new HashMap<String,Object>;
        for(int j=0;j<gameList.size();j++){
                if(npc위치[i].equals(gameList.get(j).get("npc위치"))){
                        if(gameList.get(j).get("npc유형").equals("A")){
                                tempMap.put("Atype_npc위치",npc위치[i]);
                                tempMap.put("Atype_npc유형",gameList.get(j).get("npc유형"));
                                tempMap.put("Atype_npc노출여부",gameList.get(j).get("npc노출여부"));
                                tempMap.put("Atype_npc노출일자",gameList.get(j).get("npc노출일자"));
                                tempMap.put("Atype_npc노출종료일자",gameList.get(j).get("npc노출종료일자"));

                                registeredAtypeCount +=Integer.parseInt(String.valueOf(gameList.get(j).get("npc등록숫자")));
                                displayedAtypeCount +=Integer.parseInt(String.valueOf(gameList.get(j).get("npc노출숫자")));

                                tempMap.put("Atype_npc등록숫자",registeredAtypeCount);
                                tempMap.put("Atype_npc노출숫자",displayedAtypeCount);
                        }else if(gameList.get(j).get("npc유형").equals("B")){
                                tempMap.put("Ltype_npc위치",npc위치[i]);
                                tempMap.put("Btype_npc유형",gameList.get(j).get("npc유형"));
                                tempMap.put("Btype_npc노출여부",gameList.get(j).get("npc노출여부"));
                                tempMap.put("Btype_npc노출일자",gameList.get(j).get("npc노출일자"));
                                tempMap.put("Btype_npc노출종료일자",gameList.get(j).get("npc노출종료일자"));

                                registeredAtypeCount +=Integer.parseInt(String.valueOf(gameList.get(j).get("npc등록숫자")));
                                displayedAtypeCount +=Integer.parseInt(String.valueOf(gameList.get(j).get("npc노출숫자")));

                                tempMap.put("Btype_npc등록숫자",registeredAtypeCount);
                                tempMap.put("Btype_npc노출숫자",displayedAtypeCount);
                                }
                }
        } 
        newNpcList.add(tempMap);
}
```
- 마지막으로 tempMap은 for문 밖으로 빼서 instance를 생성하고 for문 안에서는 새로운 값을 할당하는 식으로만 한다. 
- instance는 여러개를 만들지 않고 1개만 만들어 값을 초기화하고 채워넣는 식이다.

```java
List<HashMap<String,Object>> newNpcList=new ArrayList<>();
int registeredAtypeCount=0;
int displayedAtypeCount=0;
int registeredBtypeCount=0;
int displayedBtypeCount=0;
String[] npc위치= {"M","P","B","F"};

HashMap<String,Object> tempMap=new HashMap<String,Object>;
for(int i=0;i<npc위치.length;i++){
        tempMap=new HashMap<String,Object>;
        for(int j=0;j<gameList.size();j++){
                if(npc위치[i].equals(gameList.get(j).get("npc위치"))){
                        if(gameList.get(j).get("npc유형").equals("A")){
                                tempMap.put("Atype_npc위치",npc위치[i]);
                                tempMap.put("Atype_npc유형",gameList.get(j).get("npc유형"));
                                tempMap.put("Atype_npc노출여부",gameList.get(j).get("npc노출여부"));
                                tempMap.put("Atype_npc노출일자",gameList.get(j).get("npc노출일자"));
                                tempMap.put("Atype_npc노출종료일자",gameList.get(j).get("npc노출종료일자"));

                                registeredAtypeCount +=Integer.parseInt(String.valueOf(gameList.get(j).get("npc등록숫자")));
                                displayedAtypeCount +=Integer.parseInt(String.valueOf(gameList.get(j).get("npc노출숫자")));

                                tempMap.put("Atype_npc등록숫자",registeredAtypeCount);
                                tempMap.put("Atype_npc노출숫자",displayedAtypeCount);
                        }else if(gameList.get(j).get("npc유형").equals("B")){
                                tempMap.put("Ltype_npc위치",npc위치[i]);
                                tempMap.put("Btype_npc유형",gameList.get(j).get("npc유형"));
                                tempMap.put("Btype_npc노출여부",gameList.get(j).get("npc노출여부"));
                                tempMap.put("Btype_npc노출일자",gameList.get(j).get("npc노출일자"));
                                tempMap.put("Btype_npc노출종료일자",gameList.get(j).get("npc노출종료일자"));

                                registeredAtypeCount +=Integer.parseInt(String.valueOf(gameList.get(j).get("npc등록숫자")));
                                displayedAtypeCount +=Integer.parseInt(String.valueOf(gameList.get(j).get("npc노출숫자")));

                                tempMap.put("Btype_npc등록숫자",registeredAtypeCount);
                                tempMap.put("Btype_npc노출숫자",displayedAtypeCount);
                                }
                }
        } 
        newNpcList.add(tempMap);
}
```

## <span style="color:#802548">_Jquery로 동적 형성 html을 다루기_</span> 


html을 동적으로 형성하려면 js로 만들어야 한다. 아래와 같이 html을 형성할 수 있다. 아래는 변수를 넣은 형태다.

```javascript
var vHtml="";
vHtml += "<div id=persisted data-display-order='+dispOd+'>";
vHtml += "<input type=text name=npc위치'+dispOd+' id=npc위치'+dispOd+'>";

$("button:button[id='append']").on('click',function(){
   $("div[id='npc메인창"+dispOd+"']").append(vHtml);
});
```
위와 같이 동적으로 형성하여 vHtml을 넣어주었을 때, 처음 document에는 없기 때문에 이벤트를 달아줄 수 없다. 그럴 때 아래와 같이 document에 해당 요소 event를 위임한다.

```javascript
$(document).on('click','.datepicker',function(){
        $(".datepicker").datepicker();
})
```
event는 document에 위임한다고 해도, 동적으로 형성된 element 자체를 활용하려면 결국 그것이 존재하는 시기에 eventListener를 달아줘야 한다. 예를 들어, 처음엔 없지만, 저장버튼을 누르면 동적으로 형성되는 html이라고 하자. 그럼 아래와 같이 버튼을 눌렀을 때 해당 요소를 사용할 수 있다.

```javascript
$("button:button[id='append']").on("click",function(){
        var id=$("thead[id='notPersisted']").attr("id");
});
```
## <span style="color:#802548">_jsp 태그 라이브러리로 형 변환하기_</span>  


우선 날짜 형변환을 알아보자. db에는 char나 date type중 하나로 저장되는 편이다. char로 db에 저장된 날짜의 경우, fmt 태그로 Date형으로 바꿀 수 있다. 여기서 now는 Java에서 보낸 new Date().now()다.

```javascript
<fmt:parseDate value = "${now}" pattern = "yyyy-MM-dd HH:mm:ss" var = "date"/>
```
Date형으로 바꿨다면 아래와 같이 string형태로 바꿀 수 있다.

```javascript
<fmt:formatDate pattern="yyyy-MM-dd" value="${date}"/>
```
pattern=yyyy-MM-dd, yyyyMMddHHmm, yyyyMMdd HH:mm 등 다양한 방식으로 날짜를 보여줄 수 있다.


그런데 분과 초를 select하는 경우는 좀 다르다. forEach로 분과 초를 만들 경우 i가 int형이기 때문이다. 따라서 Date를 Number로 바꿔주어야 한다. Date를 Number로 바꾸는 작업은 Date --> String -->Number의 과정이다. 물론 나는 char라서 String -->Number의 과정만 거치면 된다. 그런데 분과 초는 날짜의 일부만 가져와야 해서 fn태그도 활용이 필요하다. 위와 같이 fn태그를 활용하여 필요한 String만 가져온다. 날짜는 년월일시분초를 yyyyMMddHHmmss로 적은 14자리다.

```javascript
<c:set var="dateStr" value="${map.DATE}"/>
<c:set var="hourStr" value="fn:subString(dateStr,8,10)"/>
<c:set var="minuteStr" value="fn:subString(dateStr,10,12)"/>
```
받아오고 나서 아래와 같이 하면 비교를 하려고 하면 오류가 난다.

```javascript
<select>
   <option value="${i}" <c:if test="${i eq minuteStr}"> selected </c:if> />
</select>
```
c태그에서 비교의 경우 반드시 type이 모두 동일해야 한다. char type과 String type을 비교할 수 없고String type과 int type을 비교할 수 없다. 아래와 같이 쓰면 오류가 나게 된다.

```javascript
<c:if test="${abc eq '1'}"/>
<c:if test="${abc eq 7}"/>
```
그럼 이걸 해결하기 위해선 어떻게 할까? 간단하다 형변환으로 맞춰준다.

```javascript
<c:if test="${abc eq '1'}"/>
<c:if test="${abc eq '7'}"/>
```
그럼 string에서 number로 어떻게 바꿀까? 아래와 같이 바꾼다. 참고로 여기서 minuteStr이 '05'여도 5로 취급된다.

```javascript
<fmt:parseNumber var="minute" value="${minuteStr}" />
```
이제 아래와 같이 바꿔써주면 시와 분까지 db에서 받아온 값으로 selected 처리가 가능하다.

```javascript
<c:set var="dateStr" value="${map.DATE}"/>
<c:set var="hourStr" value="fn:subString(dateStr,8,10)"/>
<c:set var="minuteStr" value="fn:subString(dateStr,10,12)"/>
<fmt:parseNumber var="minute" value="${minuteStr}" />

<select>
        <option value="${i}" <c:if test="${i eq minute}"> selected </c:if> />
</select>
```
selected처리가 아니라 date 간의 비교자체가 필요한 경우도 있을 수 있다. 그 경우에 int와 같이 >, <와 같은 비교 연산자를 활용하면 된다. 다만 비교할 pattern을 똑같게 맞춰주고서 날짜를 비교해야한다.

```javascript
<fmt:parseDate var="today" value="${now}" pattern="yyyyMMddHHmmss"/>
<fmt:parseDate var="startDateTime" value="${startDate}" pattern="yyyyMMddHHmmss"/>
<fmt:parseDate var="endDateTime" value="${endDate}" pattern="yyyyMMddHHmmss"/>
<span>
   <c:if test="${today > startDateTime and today< endDateTime}" >노출중 </c:if>
   <c:if test="${today < startDateTime}" >노출예정 </c:if>
   <c:if test="${today < endDateTime}" >노출종료 </c:if>
</span>
```
여기서 누군가는 c태그에 fmt로 바로 format을 하면 되지 않는가라는 이야기를 할 수도 있다. 안타깝게도 c 태그는 value 안에 fmt태그를 인식하지 못한다. fn태그와 Java에서 model로 보낸 object만 인식할 수 있다. 따라서 아래처럼 변수를 설정하면 그 value값이 계산되지 않고 <fmt:~~~>라는 string이 value가 되어버린다.

```javascript
<c:set var="minuteStr" value="<fmt:formatNumber pattern='00' value='${minuteStr}' />" />
```
대신 fmt태그는 html에서 활용될 수 있다. 아래와 같이 쓰면 실제 브라우저의 html에서는 아래와 같이 값을 인식하여 그 해당 값으로 대체된다.

```javascript
<div id="imagePath<fmt:formatNumber pattern='00' value='${minuteStr}' />" />
<div id="imagePath05" />
```
참고로 c:if에서 list는 ne null이 아니라 not empty로 비교한다. list 자체는 new로 instance가 생성되면 빈 값이어도 null이 아니기 때문이다.


## <span style="color:#802548">_Jquery each와 function의 차이_</span>   

아래는 fucntion giveMeSth()의 내용이다. index가 5가되면 return이 false이기에 function의 결과가 false로 return될 것처럼 보인다. 그러나 그렇지 않다. Jquery each문 안의 return false는 each라는 함수의 false이기 때문에 원하는 함수의 결과가 false로 return되는 것이 아니다. undefined가 return된다.

```javascript
function giveMeSth(){
        $("#give").each(function(index){
                if(index==5){
                        return false;
                }
        })
}
```
즉 아래와 같은 변수는 늘 undefined가 되어있는 것이다.

```javascript
var isValid=giveMeSth();
```
그럼 위의 giveMeSth()는 index가 5가 되었을 때 false를 return하지 않고 each가 return false를 하고 giveMeSth 자체는 undefined를 return한다. 따라서 giveMeSth가 false를 return하게 하려면 flag변수를 return해야한다.

```javascript
function giveMeSth(){
        var isGiven=true;
        $("#give").each(function(index){
                if(index==5){
                        isGiven=false;
                        return false;
                }
        })

        return isGiven
}
```
## <span style="color:#802548">_서버단 data 재가공 없이 table merge하기_</span>


Java ㅡ> Jstl ㅡ>Html ㅡ>Js(Jquery도 여기 포함) 순으로 작동한다. 즉, Jstl은 Java에서 model의 object로 보낸 값을 참조할 수 있지만, Js에서 만들어진 값은 참조할 수 없다.

```javascript
var html=${abcd}; //가능
var number=5; ${number} //불가능
```

Html은 Jstl의 변수값을 참조할 수 있지만, Js에서 만들어진 값은 참조할 수 없다. Html이 Js에서 만들어진 값을 참조할 수 있는 경우는 Js로 형성되는 Html이어야 한다. 기존 body안에서는 모두 Jstl을 통해 값을 참조해야 한다. 그래도 html에 영향을 미치는 건 Js로도 가능하다. html이 만들어진 후 js가 최종적으로 html을 건드리기 때문이다.

특히 event를 등록할 때나, 일정 조건을 만족하면 css나 속성을 바꾸는 등의 행위는 js를 많이 사용한다. 물론 초기화면에 제약을 걸 때는 c:if 등으로 html에 직접 jstl을 넣는 것이 더 많이 사용되는 편이다.


또한 저 위의 순서 때문에 c:forEach로 생성한 html을 js로 조정할 수 있는 것이기도 하다. 최종 html은 js적용까지 마친 html이 우리 눈에 보이기 때문에 c:forEach 전에는 cell이 merge되지 않더라고 js로 cell을 merge 시키면 merge된 table로 첫화면에 뜨게 된다. ssr이기 때문에 빈 화면이 먼저 뜨지 않고 js까지 다 적용된 이후에 화면이 뜨기 때문에 그러하다.


예를 보자. 아래와 같이 forEach를 활용해 html을 형성한다. Java에서 model에 값을 담아 보낸다.

```java
model.addAttribute("list",npcList);
```
c:forEach는 jstl의 일종으로 Java에서 보낸 값을 받아서 쓴다. c:forEach기 때문에 element만큼 html을 반복하여 생성한다.

```javascript
<c:forEach var="map" items="list">
   <table>
      <tr>
         <td class="rowspan">${map.npc위치}</td>
         <td>${map.npc유형}</td>
         <td>${map.npc노출일자}</td>
         <td>${map.npc노출종료일자}</td>
      </tr>
   </table>
</c:forEach>
```
만약 element가 3개라면 아래와 같은 구조가 html이 만들어진다.

```javascript
 <table>
      <tr>
         <td class="rowspan">a</td>
         <td>A</td>
         <td>20230803</td>
         <td>20230910</td>
      </tr>
   </table>
 <table>
      <tr>
         <td class="rowspan">b</td>
         <td>A</td>
         <td>20230910</td>
         <td>20231124</td>
      </tr>
   </table>
 <table>
      <tr>
         <td class="rowspan">c</td>
         <td>B</td>
         <td>20231130</td>
         <td>200240105</td>
      </tr>
   </table>
```
그럼 Js를 통해서 해당 table을 원하는 형태로 merge한다.

```javascript
$("td[class='row']").each(function(){
   var tdList = $(".rowspan:contains('로그인 이후')");
	if(tdList.length > 1){
		tdList.eq(0).attr("rowspan",tdList.length);
		tdList.not(":eq(0)").remove();
	}
}
```
그럼 List라는 형태로 그대로 Java에서 return을 받으면서 UI만 control할 수 있다. 즉 서버단에서 data를 재가공할 필요가 없어지는 것이다.

## <span style="color:#802548">_네이티브에 뿌리는 내용_</span>
```java
String[] param={"M","B","P","F"};
String[] outputKey={"My","Benefit","Payment","Finance"};
HashMap<String,Object> inputParamMap = new HashMap<>();
HashMap<String,Object> elementMap = new HashMap<>();
HashMap<String,Object> decodeXssMap = new HashMap<>();
HashMap<String,Object> decodeXssList = new HashMap<>();
HashMap<String,Object> outputMap = new HashMap<>();

List<HashMap<String,Object>> subPopupList = new ArrayList<>();
String type = "";

for(int i =0; i <param.length; i++){
        inputParamMap = new HashMap<>();
        inputParamMap.put("BLTN_LOC",param[i]);
        subPopupList = Dao.selectList(inputParamMap);
        elementMap = new HashMap<>();

        if(subPopupList.isEmpty()){
                continue;
        }

        for(int j=0; j<subPopupList.size();j++){
                decodeXssMap.put("EXPS_YN",subPopupList.get(j).get("EXPS_YN"));
                decodeXssMap.put("EXPS_OS_TP",subPopupList.get(j).get("EXPS_OS_TP"));
                decodeXssList.add(decodeXssMap);
        }

        if(subPopupList.get(0).get("PUP_TP").equals("bottom")){
                type = "bottom";
        }else{
                type = "layer";
        }

        elementMap.put("type", type);
        elementMap.put("list", decodeXssList);
        elementMap.put("notDisplayingDate", subPopupList.get(0).get("PUP_NO_USE_TP"))
        outputMap.put(outputKey[i], elementMap); // 좀 깔끔하게 구조가 잡힘. 공통은 key별로 따로 추출한 형태.
}

return outputMap;
```

- 위와 같이 하면 db 조회가 4번이라 오버헤드가 많다.
- db 조회는 1회로 하고 아래와 같이 자바에서 필터링을 하자.

```java
HashMap<String,Object> outputMap = new HashMap<>();

try{
        List<HashMap<String,Object>> subPopupList =  Dao.selectList();
        String[] param={"M","B","P","F"};
        String[] outputKey={"My","Benefit","Payment","Finance"};
        HashMap<String,Object> elementMap = new HashMap<>();
        HashMap<String,Object> decodeXssMap = new HashMap<>();
        HashMap<String,Object> decodeXssList = new ArrayList<>();
        String type = "";
        String notDisplayingDate = "";
        for(int i = 0; i < param.length; i++){
                for(int j=0; j < subPopupList.size();j++){
                        if(param[i].equals(subPopupList.get(j).get("BLTN_LOC"))){
                                decodeXssMap.put("EXPS_YN",subPopupList.get(j).get("EXPS_YN"));
                                decodeXssMap.put("EXPS_OS_TP",subPopupList.get(j).get("EXPS_OS_TP"));
                                type = "B".equals(subPopupList.get(j).get("PUP_TP")) ? "bottom" : "layer";
                                notDisplayingDate = (String)subPopupList.get(j).get("PUP_NO_USE_TP");
                                decodeXssList.add(decodeXssMap);
                        }
                }

                if(decodeXssList.isEmpty()){
                        continue;
                }

                for(int k = 0; k < decodeXssList.size(); k++){
                        if(k > 4){
                                decodeXssList.remove(k);
                        }
                }

                elementMap.put("type", type);
                elementMap.put("notDisplayingDate", notDisplayingDate);
                elementMap.put("list", decodeXssList);
                outputMap.put(outputKey[i], elementMap); // 좀 깔끔하게 구조가 잡힘. 공통은 key별로 따로 추출한 형태.
        }
}catch(Exception e){
        throw e;
}



return outputMap;
```

- 4 초과 remove도 5 미만 list에 add로 바꾸면 remove하는 데 들이는 시간을 줄일 수 있다.
```java
HashMap<String,Object> outputMap = new HashMap<>();

try{
        List<HashMap<String,Object>> subPopupList =  Dao.selectList();
        String[] param={"M","B","P","F"};
        String[] outputKey={"My","Benefit","Payment","Finance"};
        HashMap<String,Object> elementMap = new HashMap<>();
        HashMap<String,Object> decodeXssMap = new HashMap<>();
        HashMap<String,Object> decodeXssList = new ArrayList<>();
        String type = "";
        String notDisplayingDate = "";
        for(int i = 0; i < param.length; i++){
                HashMap<String,Object> finalList = new ArrayList<>();
                for(int j=0; j < subPopupList.size();j++){
                        if(param[i].equals(subPopupList.get(j).get("BLTN_LOC"))){
                                decodeXssMap.put("EXPS_YN",subPopupList.get(j).get("EXPS_YN"));
                                decodeXssMap.put("EXPS_OS_TP",subPopupList.get(j).get("EXPS_OS_TP"));
                                type = "B".equals(subPopupList.get(j).get("PUP_TP")) ? "bottom" : "layer";
                                notDisplayingDate = (String)subPopupList.get(j).get("PUP_NO_USE_TP");
                                decodeXssList.add(decodeXssMap);
                        }
                }

                if(decodeXssList.isEmpty()){
                        continue;
                }

                for(int k = 0; k < decodeXssList.size(); k++){
                        if(k < 5){
                                finalList.add(decodeXssList.get(k));
                        }
                }

                elementMap.put("type", type);
                elementMap.put("notDisplayingDate", notDisplayingDate);
                elementMap.put("list", finalList);
                outputMap.put(outputKey[i], elementMap); // 좀 깔끔하게 구조가 잡힘. 공통은 key별로 따로 추출한 형태.
        }
}catch(Exception e){
        throw e;
}



return outputMap;
```

## <span style="color:#802548">_정규식_</span>
```javascript
const regExp = /[^ㄱ-ㅎ|ㅏ-ㅣ|가-힣|]+$/gi; //허용하는 패턴. 하지만 이는 키보드에서만 작동.
const regExp = /[ㄱ-ㅎ|ㅏ-ㅣ|가-힣|]+$/gi; //허용하지 않는 패턴. 모바일에서 작동 안 함. 모바일에는 가운데 점(middle dot)이라는 특수문자가 있기 떄문.
const regExp = /[^ㄱ-ㅎ|ㅏ-ㅣ|가-힣|\u3180\u119E\u11A2\u2022\u2025a\u00B7\uFE55]+$/gi; //모바일에 있는 가운데 점(middle dot)이라는 특수문자도 포함하여 한글을 허용

//아래와 같이 해주면 입력 시 한글을 쓰면 저절로 지워짐.
if(regExp.test(event.target.value)){
        event.target.value = event.target.value.replace(regExp, '');
}

const regExp = /[^가-힣|0-9|_]+$/gi; // 허용하는 패턴
//아래와 같이 써주면 입력 시 한글, 숫자, _를 제외한 모든 입력이 지워짐.
if(regExp.test(event.target.value)){
        event.target.value = event.target.value.replace(regExp, '');
}

const regExp = /[^ㄱ-ㅎ가-힣ㅏ-ㅣ0-9\_\-\u318D\u2025a\u00B7\uFE55]+$/gi; // 허용하는 패턴은 한글, _, -, 천지인키보드 미들닷
 const regExp = /(?:[^\w\s\uAC00-\uD7A3\u3131-\u314E\u314F-\u3163\u318D\u119E\u11A2\u2022\u2025a\u00B7\uFE55]|_)+/g; 
 //위와 똑같음. 다만 유니코드로 변경된 것일 뿐임
 /*  uAC00-\uD7A3 : 가-힣 (음계)
   * \u3131-\u314E : ㄱ-ㅎ (자음)
   * \u314F-\u3163 : ㅏ-l (모음)
   * \u318D\u119E\u11A2\u2022\u2025a\u00B7\uFE55 : 천지인 키보드 문자
   * '가-힣'으로 할경우 euc-kr 에서는 작동하지 않음.
   * /

```

```javascript
//radio 버튼중에 value가 Y인 것이 checked된다.
$("input:radio[name='owner']:input[value='Y']").prop('checked',true);

$("thead:not([class])") // class가 없는 thead html
```

## <span style="color:#802548">_sql 예약어 피하기_</span>
```javascript
function checkSearchedWor(text){
        if(text.length >0){
                var expText = /[%=><]/;
                if(expText.test(text) == true){
                        alert("특문 입력 불가합니다");
                        text = text.split(expText).join(""); // text = text.replace(expTest, "");
                        return false;
                }
                var sqlArray = new Array("OR","SELECT","INSERT","DELETE","UPDATE","CREATE","EXEC","UNION","FETCH","DECLARE","TRUNCATE");
                var regex;
                for(var i = 0; i<sqlArray.length; i++){
                        regex = new regExp(sqlArray[i],"gi");
                        if(regex.test(text)){
                                alert("\" + sqlArray[i] + "\"와 같은 특정문자열은 검색이 불가합니다.");
                                text = text.replace(regex, "");
                                return false;
                        }
                        
                        return true;
                }
        }
}
```
## <span style="color:#802548">_select 대신 뜨는 팝업과 select validation 연동하기_</span>
- 참고로 아래 로직은 해당 팝업의 html 구성에서 쓸 수 있는 것이다.
- select 값을 대변하는 팝업 값 중 아무것도 선택하지 않고 팝업 닫기 버튼을 누르면 팝업이 닫히며 validation이 뜨게 한다.
- 해당 select를 누르면 html이 생성되는 형식이라 동적으로 형성되는 html event를 control하는 방식을 택한다.
```javascript
$(document).on('click','a[class="pop-close ui-pop-close"]',function(){
        var selectedCategory = $(this).parent()[0].firstElementChild.innerText();
        var selectedDomList = $(this).parents().find("#container select");
        var selectedDom = "";
        for(var i = 0;i<selectedDomList.length;i++){
                if(selectedDomList[i].outerText == selectedCategory){
                        selectedDom = $(selectedDomList[i]).find("select");
                }
        }
        displayTextWhenValidationFails(selectedDom);
})
```

- select 값을 대변하는 팝업 값 중 아무것도 선택하지 않고 팝업 바깥 영역 버튼을 누르면 팝업이 닫히며 validation이 뜨게 한다.
- 해당 select를 누르면 html이 생성되는 형식이라 동적으로 형성되는 html event를 control하는 방식을 택한다.
```javascript
$(document).on('click','div[id="uiSelectLayer"]',function(){
        var selectedCategory = $(this).children()[0].firstElementChild.innerText();
        var selectedDomList = $(this).parent().find("#container select");
        var selectedDom = "";
        for(var i = 0;i<selectedDomList.length;i++){
                if(selectedDomList[i].outerText == selectedCategory){
                        selectedDom = $(selectedDomList[i]).find("select");
                }
        }
        displayTextWhenValidationFails(selectedDom);
})
```

```javascript
function displayTextWhenValidationFails(obj){
        var value = $(obj).val();
        if(value != ''){
                $(obj).parents('div.fm_cont').children('.error_txt').hide();
        }else{
                $(obj).parents('div.fm_cont').children('.error_txt').show();
        }
}
```

- 그런데 여기서 IOS의 경우와 AOS의 경우가 띄어쓰기 공백을 인식하는 방식이 다르기 때문에, 반드시 trim으로 공백을 지워주고 비교해야 한다
```javascript
$(document).on('click','div[id="uiSelectLayer"]',function(){
        var selectedCategory = $(this).children()[0].firstElementChild.innerText();
        var selectedDomList = $(this).parent().find("#container select");
        var selectedDom = "";
        for(var i = 0;i<selectedDomList.length;i++){
                if(selectedDomList[i].outerText.trim() == selectedCategory.trim()){
                        selectedDom = $(selectedDomList[i]).find("select");
                }
        }
        displayTextWhenValidationFails(selectedDom);
})
```

## <span style="color:#802548">_중요하지 않은 내용은 throw Exception하지 않기_</span>
- 내가 하는 메인팝업 업무는 appMain에서 중요도가 매우 낮았다
- 그런데 to-be화면으로 바꾸는 과정에서 오류가 났다.
- 내가 이미 만든 관리자 화면은 기획이 5개 최대로 와서 5개로 했다.
- 그런데 통테 기간에 이걸 바꿔달라고 왔고, 그래서 바꿨다
- 기존에는 아래와 같았다.
```java
for(int i = 0; i<subPopupList.size(); i++){
        ....
}
```
- 그걸 아래와 같이 바꿨다.

```java
for(int i = 0; i<5; i++){
        ....
}
```

- 그러자 오류가 났다.
- indexOutOf 오류가 계속 났다.

이 때 알았는데, 삭제만이 아니라 원래도 없는 index의 요소를 읽어오려고 하면 indexOutOfRange exception이 나는 것이었다.
- 그래서 해당 오류가 나지 않게 바꿨다.

```java
for(int i = 0; i<subPopupList.size(); i++){
        for(i>5){
                break;
        }
        ...
}
```

- 그런데 그 외에도 다른 이유로 오류가 나면 오류가 나지 않게끔 exception처리가 아니라 빈 더미 데이터를 반환하는 게 나을 것 같다고 했다.
- 내 메인 팝업은 앱메인에서 중요하지 않기 때문이다
- 그래서 아래와 같이 바뀌었다.
```java
try{

}catch(Exception e){
        throw e;
}
```

```java
try{

}catch(Exception e){
        outputMap.put("type","L");
        outputMap.put("notDisplayingDate","1");
        outputMap.put("list",Collections.emptyList());
        return outputMap;
}
```

## <span style="color:#802548">_핸드폰 붙여넣기를 포함한 모든 input의 변화를 감지하기_</span>
```javascript
$(selector).on('propertychange change keyup paste input')
//focusout에도 필요하다면 focusout도 추가해주자.
```

## <span style="color:#802548">_alert 대신 html toast띄우기_</span>



```javascript
function toastBox(txt, delayTime, cssBottom){
        const className = '.toast_box';
        const closeTime = 500;
        let delayTime = 3_000;

        let boxHtml = '';

        botHtml +='<div class=' + className.substring(1)+ ' ' + 'ty2';
        botHtml +='    <div class=txt>' + txt + '<//div'>;
        botHtml +='<//div'>;

        $('body').after(boxHtml);

        const toast = $(className).last();
        if($('.fixed_space'.length)){
                toast.removeClass('off');
                toast.css('bottom',cssBottom);
        }

        toast.addRemoveClass('on',0,delayTime,function(){
                setTimeout(function(){
                        const toast = $('.toast_box');
                        toast.remove();
                },closeTime);
        })
}
```

## <span style="color:#802548">_뒤로 가기로 접근한 페이지인지 알고 싶을 때_</span>

```javascript
window.onPageShow = function(event){
        if(event.persisted  || window.performance.navigation.type ===2 /* 이건 후자는 AOS용. 전자는 IOS용*/){


        }
}

window.onPageShow = function(){
        if(window.performance.getEntriesByType('navigation')[0].type == "back_forward"){//true면 뒤로가기로 접근한 페이지

        }
}
```
## <span style="color:#802548">_평범한 중복클릭 방지 플래그_</span>
```javascript
let requestSendingFlag = false;

$("button[id=click]").on('click',function(){
        dataModeling();

        if(requestSendingFlag == false){
                ajax();
        }

        requestSendingFlag = true;
        //setTimeout("reqFlag = false", 1000); 문자열이 아니라 callback fn으로 넣기
})
```
## <span style="color:#802548">_validation과 같이 가는 중복클릭 방지 플래그_</span>
- 클로저를 사용하지 않고 전역변수를 이용한다
```javascript
var reqFlag = false;
function reqCheck(){
        if(reqCheck == false){
                reqFlag = true;
                return false;
        }

        setTimeout(function(){
                reqFlag = false;
        },3000)

        return true;
}

function sendKYCData(){
        var isDisabled = $(this).hasClass("disabled");
        if(isDisabled){
                return false;
        }

        if(reqCheck()){
                return false;
        }
        
}
```
- 클로저를 사용하면 전역변수를 쓰지 않는다. 함수안에 집어넣는다.
```javascript
function sendKYCData(){
        var isDisabled = $(this).hasClass("disabled");
        if(isDisabled){
                return false;
        }
        var reqCheck = (function(){
                var reqFlag=false;  
                
                return function(){
                        if(reqFlag == false){
                                reqFlag = true;
                                return false;
                        }
                        setTimeout(function(){
                                reqFlag = false;
                        },3000)

                         return true;
                }
        })();

        if(reqCheck()){
                return false;
        }
}
```
## <span style="color:#802548">_이전 페이지 없이 바로 해당 url로 들어오는 것을 방지하기_</span>
```javascript
if(document.referrer == ""){
        closeWebView();
}else{
        history.back();
}
```

## <span style="color:#802548">_focus에 따른 전화번호 hypen 붙였다 뗴기_</span>
```javascript
$("#emailSelfInput").on("focusout",function(){
        const value = $(this).val();
        if(value.length == 8 ){
                $(this).val(value.replace(/^(\d{4})(\d{4})$/,'$1-$2'));
        }else if(value.length == 7 ){
                $(this).val(value.replace(/^(\d{3})(\d{4})$/,'$1-$2'));
        }
})

$("#emailSelfInput").on("focus",function(){
        const value = $(this).val();
       $(this).val(value.replace(/-/,""));
})
```
## <span style="color:#802548">_앱이 설치되었는 지 검사_</span>
```javascript
if(!document.webkitHidden && !document.hidden){
        if(confirm("앱을 설치하시겠습니까?") != 0){
                location.href = "/appdownload";
        }
}
```


## <span style="color:#802548">_문자열을 가지고 뭘 할 땐 상수부터 앞으로_</span>
- 그래야 NPE를 회피할 수 있다. 만약 앞이 변수라면 null일 경우, null이라는 참조변수에 접근하기 때문에 NPE가 발생한다.
```java
String str = req.getParameter();
if(str.equals("Y")); //X
if("Y".equals(str)); //O

if(str.indexOf("Y") > -1) //X
if("Y".indexOf(str) > -1) //O

if(str.contains("Y")) //X
if("Y".contains(str)) //O
```

## <span style="color:#802548">_post와 get 이동하기_</span>
- @RequestMapping에 Http method를 명시하지 않으면 모든 method를 사용할 수 있다.
- 따라서 아래는 get과 post 모두 이동가능하다.
```java
@RequestMapping(value="/uri")
```

```javascript
location.href = "/uri"; //get 이동

var form = document.transferData;
form.method = '/post';
form.action = '/uri';
form.submit(); //post 이동

<form id = "transferData" name = "transferData">
        <input name = "payload" value="${payload}">
</form>
```

## <span style="color:#802548">_String 첫자리만 비교할 경우__</span>
- 기존에는 array에 넣고 list로 바꾸고 list의 method인 contains를 활용했다.

```java
String[] strArr = {"3","4"};
List<String> strList = Arrays.asList(strArr);
boolean isMillenium = strList.contains(rlno.substring(6,7));
```

- 그를 아래와 같이 간단하게 바꿀 수 있다. 연산도 줄어든다.
```java
boolean isMillenium = (rlno.charAt(0) == '3' || rlno.charAt(0) == '4');
```

## <span style="color:#802548">_화면 load 순서와 setTimeout__</span>
- window.load와 $(function())이 있다.
- onload는 문서에 포함된 모든 콘텐츠가 로드된 후에 실행되기에 불필요한 로딩시간이 추가될 수 있음.
- 특히 이미지의 경우, 이미지가 load된 이후에나 script를 읽어들이기 시작.

 ```javascript
 window.onload = function(){

 }



window.onload = function(){ //마지막 선언된 것만 읽힘. 그러나 body에 onload가 있으면 body것만 읽음.
        //UI rendering function
 }

 window.onload = function(){ //떄로 브라우저가 HTML을 채 그리지 못했는데 UI logic이 발생한다면 아래와 같이 setTimeout에 집어넣기도 한다.
                                //setTimeout으로 UI rendering 함수를 task queue 후위로 미뤄버려 다른 일이 먼저 진행되게 한다.
        setTimeout(function(){
                //UI rendering function.
        },0 /*300 등 다양함*/)
 }

  <body onload ="">
```

- 반면에 $()는 DOM만 load되면 바로 script를 읽어들임.
- 더 빠르게 화면을 띄울 수 있다.
```javascript
 $(function(){

 }

  $(function(){ //순차적으로 모두 읽힘

 }
)
```

## <span style="color:#802548">_Json -> JavaObject(request)/ JavaObject -> JSON(response) parsing__</span>
- JSP에서는 json문자열로 보내고, 이를 JSONParser로 parsing하여 JSONObject로 변환한다.
- 그 뒤 JSONObject를 다시 HashMap으로 변환하여 꺼내와 사용한다.
- 다만 JSP에서는 배열도 일단 object처럼 바꿔서 보내야 한다. 
- 즉 보내고자 하는 데이터가 배열이라도, object 안의 key-value 형태로 배열을 넣어야한다. 직접 배열을 보내선 안 된다.
```java
//공통 method. response용
public String getJsonFromData(HashMap<String,Object> jsonMap){
        String jsonStr = "";
        try{
                JSONObject jsonObject = new JSONObject();
                jsonObject.putAll(jsonMap);
                jsonStr = jsonObject.toJsonString();
        }catch(Exception e){
                throw e;
        }

        return jsonStr;
}

//공통 method. request용
public HashMap<String,Object> getDataFromJson(String jsonStr){
        HashMap<String,Object> jsonMap = new HashMap<>();
        try{
                JSONParser jsonParser = new JSONParser();
                JSONObject jsonObject = (JSONObject)jsonParser.parse(jsonStr);
                jsonMap.putAll(jsonObject); // JsonObject는 HashMap을 extend했기 때문에 다형성에 의해 parameter로 들어갈 수 있다.
        }catch(Exception e){
                throw e;
        }

        return jsonMap;
}

//controller
@RequestMapping(value = "/uri")
public String updateList(@RequestBody jsonStr){
        HashMap<String,Object> outputMap = new HashMap<>();
        HashMap<String,Object> inputMap = new HashMap<>();
        inputMap = getDataFromJson(jsonStr);
        String retStr = null;
        try{
                mainService.update(inputMap);

                outputMap.put("code", 0000);
                outputMap.put("message","SUCCESS")
        }catch(Exception e ){
                ErrorException2 exception2 = new ErrorException2();
                throw exception2;
        }

        retStr = getJsonFRomData(outputMap);

        return retStr;
}

//service
public void update(HashMap<String,Object> inputMap) throw Exception{
        List<HashMap<String,Object>> list = (List<HashMap<String,Object>>) inputMap.get("updateElementList");
        for(int i = 0; i < list.size(); i++){
                mainDao.update(list.get(i).get("SQNO"));
        }
}
```
```javascript
//jsp
var object = {
        updateElementList: []
};
object.updateElementList = element;
var sendData = JSON.stringify(object);
$.ajax({
        url:'/uri',
        method:'post',
        data:sendData,
        dataType:'json' //content-type 명시가 없으면 기본은 x-wwww-urlencoded다. Spring에서는 getParameter로 받을 수 있다.
                        //body라고 해도 key=value&key=value 형식으로 가능하다. query string만 그런게 아니다.
})
```  

- 아래와 같은 실제 예시를 보자.

```javascript
var formData = {};
formData.sqnoList = [];
formData.sqnoList[index] = element;
for(var i = 0; i < formData.sqnoList.length; i++){
        if(formData.sqnoList[i] == null){
                formData.sqnoList.splice(i,1);
                i--; //js는 indexOutOf 오류 안나나?
        }
}

var sendData = JSON.stringify(formData);
$.ajax({
        .
        .
        data:sendData
}),
 success : function(result) { 
        var result = JSON.parse(result);
        var code = result.code;
        if(code == 200){
                location.href = "/uri2";
                //var form = document.transferDataForm;
                //form.action = "/uri2";
                //form.method = "post";
                //form.submit();
        }
 }

```

```java
@RequestMapping(value = "/update")
public String updateSqnoList(@RequestBody jsonStr){
        HashMap<String,Object> inputMap = new HashMap<>();
        inputMap = getDataFromJson(jsonStr);

        updateService.updateSqnoList(inputMap);
}

@Service
public HashMap<String,Object> updateSqnoList(HashMap<String,Object> inputMap){
        List<String> sqnoList = (List<String>)inputMap.get("sqnoList");
        updateDao.updateSqnoList(sqnoList);
}

@Repository
public void updateSqnoList(List<String> sqnoList){
        sqlSession.update(sqnoList);
}

//mapper
delete from list
where in 
<foreach item = "SQNO" collection = "list" open = "(" seperator = ","  close = ")">
        #{SQNO}
</foreach> 
//그럼 ('202301333', '201415551', '2041424214'...)와 같이 where in 문으로 나가서 delete 여러번 안 치고 한꺼번에 삭제 가능
```

- payload안에 무조건 넣어서 가져올 수도 있다.

```javascript
var param= {};
param.OPENBETA = $("#openBeta").val();
param.CLOSEBETA = $("#closeBeta").val();

setAjax(param);

function setAjax(obj){
        var sendData ={};
        sendData.payload = JSON.stringify(obj);

        /*
        sendData:{
                payload:{
                        object:{...},
                        list:[{},{},{}....],
                        string:"",
                        number:0
                }
        }
        */
        $.ajax({
                data:sendData
                dataType:'json'
                .
                .

        })
}
```

```java
@RequestMapping()
public String getData(HttpServletRequest req){
        String payload = req.getParameter("payload");
        HashMap<String,Object> retMap = new HashMap<>();
        JSONObject jsonObject = new JSONObject();
        JSONParser jsonParser = new JSONParser();
        jsonObject = (JSONObject)jsonParser.parse(payload);

        retMap.putALl(jsonObj);
        HashMap<String,Object> map = (HashMap<String,Object>)retMap.get("object");
        List<HashMap<String,Object>> list = (List<HashMap<String,Object>>)retMap.get("list");
        String string = (String)retMap.get("string");
        Long number = (Long)retMap.get("number");
}
```

## <span style="color:#802548">_HashMap 로그 찍기_</span>
```java
public String logMapData(HashMap<String,Object> mapData){
        StringBuffer mapLogger = new StringBuffer();
        if(logger.isDebugEnabled){
                String key = null;
                Set<String> set = mapData.keySet();
                Iterator<String> iterator = set.iterator();
                int size = 0;
                while(iterator.hasNext()){
                        key = (String)iterator.next();

                        mapLogger.append(key);
                        mapLogger.append(":");
                        mapLooger.append(mapData.get(key));
                        mapLogger.append("\n");

                        size ++;
                }
                mapLogger.append("\n mapData size: " + size);
        }

        return mapLogger.toString();
}



