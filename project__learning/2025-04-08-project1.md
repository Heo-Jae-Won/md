---
title: カードアプリの高度化 
published: true
---


## <span style="color:#802548">_重要ではないものはEXCEPTION使わない_</span>
- TO-BE画面に変更する過程でエラーが発生しました。
- 私がすでに作った管理者画面は、企画が最大5つまで来て5つにしました。

```java
//既存
for(int i = 0; i < list.size(); i++){
        ....
}

//変更
for(int i = 0; i < 5; i++){
        ....
}
```

- すると、indexOutOfエラーが続きました。
- この時知ったのですが、削除だけでなく、元々ないindexの要素を読み込もうとするとindexOutOfRange例外が出るということでした。
- エラーが発生しないようにBREAKを使いました。

```java
for(int i = 0; i < list.size(); i++){
        for(i > 5){
            break;
        }
        ...
}
```

- しかし、それ以外にも他の理由でエラーが発生することもあります。
- 私のメインポップアップはアプリにとってそれほど重要ではないので、メインポップアップが原因でアプリが止まってはいけません。
- その場合も考えて、例外を投げずに空のダミーデータを返す方がアプリケーションのユーザビリティに役立つとアドバイスされました。

```java
//既存
try{
	.
	.
	.
	for(int i = 0; i < list.size(); i++){
        for(i > 5){
            break;
        }
        ...
	}
}catch(Exception e){
        throw e;
}


//変更
try{
	.
	.
	.
	for(int i = 0; i < list.size(); i++){
        for(i > 5){
            break;
        }
        ...
	}
}catch(Exception e){
        outputMap.put("type","L");
        outputMap.put("noView","1");
        outputMap.put("list",Collections.emptyList());
        return outputMap;
}
```

## <span style="color:#802548">_mobileも考えた入力フィルタリング正規表現_</span>

- 日本語は音節単位の言語なので、母音を形成するための中間段階の記号や特殊文字がなくても入力が完了します。
- だから、韓国語のようにキーボードの構造に応じた内部組み合わせのための仮文字(・)のようなものは存在しません。
- 韓国語のモバイルキーボードではは仮文字（・）が必要です。
- 最初はモバイルキーボードとキーボードが違うことを知らなかったのでモバイルで請求表現がうまくいかなかったです。

```javascript
const regExp = /[^ㄱ-ㅎㅏ-ㅣ가-힣]+$/gi;

if(regExp.test(event.target.value)){
        event.target.value = event.target.value.replace(regExp, '');
}
```

- ハングルのモバイルキーボードでは、中央の点という特殊な文字で母音を生成します。 
- 私はそれを知らなかったのでそれも含めたUNICODEを使いました。

```javascript
const regExp = /[^\uAC00-\uD7A3\u3131-\u314E\u314F-\u3163\u318D\u119E\u11A2\u2022\u2025\u00B7\uFE55]+/g; 

```


## <span style="color:#802548">_sql 예약어 피하기_</span>

- JAVAを使うとPREPAREDSTATEMENTのようなパラメーターのバインド(Parameter binding)ができます。
- 

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

for(int i = 0; i < param.length; i++){
        inputParamMap = new HashMap<>();
        inputParamMap.put("BLTN_LOC",param[i]);
        subPopupList = Dao.selectList(inputParamMap);
        elementMap = new HashMap<>();

        if(subPopupList.isEmpty()){
                continue;
        }

        for(int j = 0; j < subPopupList.size(); j++){
                decodeXssMap.put("EXPS_YN", subPopupList.get(j).get("EXPS_YN"));
                decodeXssMap.put("EXPS_OS_TP", subPopupList.get(j).get("EXPS_OS_TP"));
                decodeXssList.add(decodeXssMap);
        }

        if( "bottom".equals(subPopupList.get(0).get("PUP_TP")) ){
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