## <span style="color:#802548">_SQL 최적화_</span>

- 아래 같이 하면 db 조회가 4번이라 오버헤드가 많다.

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

- 좀 더 현대적으로 하는 방법은 아래와 같다.

```java
import java.util.*;
import java.util.stream.Collectors;

public Map<String, Object> getPopupOutputMap() throws Exception {
    Map<String, Object> outputMap = new HashMap<>();
    
    List<Map<String, Object>> subPopupList = Dao.selectList();
    if (subPopupList == null || subPopupList.isEmpty()) {
        return outputMap;
    }

    String[] params = {"M", "B", "P", "F"};
    String[] outputKeys = {"My", "Benefit", "Payment", "Finance"};

    Map<String, List<Map<String, Object>>> groupedByLoc = subPopupList.stream()
            .filter(row -> row.get("BLTN_LOC") != null)
            .collect(Collectors.groupingBy(row -> row.get("BLTN_LOC").toString()));

    for (int i = 0; i < params.length; i++) {
        String currentParam = params[i];
        List<Map<String, Object>> matchedRows = groupedByLoc.get(currentParam);

        if (matchedRows == null || matchedRows.isEmpty()) {
            continue;
        }

        Map<String, Object> firstRow = matchedRows.get(0);
        String type = "B".equals(firstRow.get("PUP_TP")) ? "bottom" : "layer";
        Object notDisplayingDate = firstRow.get("PUP_NO_USE_TP");

        List<Map<String, Object>> decodeXssList = matchedRows.stream()
                .limit(5) 
                .map(row -> {
                    Map<String, Object> cleanMap = new HashMap<>();
                    cleanMap.put("EXPS_YN", row.get("EXPS_YN"));
                    cleanMap.put("EXPS_OS_TP", row.get("EXPS_OS_TP"));
                    return cleanMap;
                })
                .collect(Collectors.toList());

        Map<String, Object> elementMap = new HashMap<>();
        elementMap.put("type", type);
        elementMap.put("notDisplayingDate", notDisplayingDate);
        elementMap.put("list", decodeXssList);

        outputMap.put(outputKeys[i], elementMap);
    }

    return outputMap;
}

```



- 이걸 데이터베이스에서 하게 되면 훨씬 더 자바 코드의 가독성이 높아진다.
- union all을 한다고 해도, table scan이 늘어나는 게 없기 때문에 마구마구 해도 된다.
- 최대 허용 갯수를 보여줄 때도 ROW_NUMBER에서 조절하면 돼서 더 간단해진다

```sql
WITH TargetParams AS (
    SELECT 'M' AS LOC, 'My' AS KEY_NAME UNION ALL
    SELECT 'B' AS LOC, 'Benefit' AS KEY_NAME UNION ALL
    SELECT 'P' AS LOC, 'Payment' AS KEY_NAME UNION ALL
    SELECT 'F' AS LOC, 'Finance' AS KEY_NAME
),
RankedPopups AS (
    SELECT 
        tp.KEY_NAME,
        pm.BLTN_LOC,
        pm.EXPS_YN,
        pm.EXPS_OS_TP,
        pm.PUP_NO_USE_TP,
        -- Calculate UI Type directly inside the database
        CASE WHEN pm.PUP_TP = 'B' THEN 'bottom' ELSE 'layer' END AS CALCULATED_TYPE,
        -- Rank elements within each code partition group to enforce boundaries
        ROW_NUMBER() OVER (
            PARTITION BY pm.BLTN_LOC 
            ORDER BY pm.REG_DT DESC -- Assuming you want the 5 newest records
        ) as row_num
    FROM POPUP_MASTER pm
    -- Inner join screens out anything that isn't M, B, P, or F instantly
    INNER JOIN TargetParams tp 
    ON pm.BLTN_LOC = tp.LOC
)
SELECT 
    KEY_NAME,
    BLTN_LOC,
    EXPS_YN,
    EXPS_OS_TP,
    PUP_NO_USE_TP,
    CALCULATED_TYPE
FROM RankedPopups
-- Discard any records exceeding the threshold restriction ceiling
WHERE row_num <= 5;
```

- 이전 java 소스코드보다 훨씬 간단해진 것을 살펴볼 수 있다.
- index의 유용성, stream의 연산에 들어가는 많은 CPU 소요 등을 생각하면 db 단에서 처리할 수 있다면 db단에서 처리하는 것이 가장 바람직하다.

```java
public Map<String, Object> getPopupOutputMap() throws Exception {
    Map<String, Object> outputMap = new HashMap<>();
    
    // Single DB round-trip executing the optimized INNER JOIN query above
    List<Map<String, Object>> subPopupList = Dao.selectList();
    if (subPopupList == null || subPopupList.isEmpty()) {
        return outputMap;
    }

    // Group the rows by their target response key ("My", "Benefit", etc.)
    Map<String, List<Map<String, Object>>> groupedByKey = subPopupList.stream()
            .collect(Collectors.groupingBy(row -> row.get("KEY_NAME").toString()));

    for (Map.Entry<String, List<Map<String, Object>>> entry : groupedByKey.entrySet()) {
        String outputKey = entry.getKey();
        List<Map<String, Object>> matchedRows = entry.getValue();

        // Safe baseline extraction from index 0 (all matching records share identical group configurations)
        Map<String, Object> baselineRow = matchedRows.get(0);
        String type = (String) baselineRow.get("CALCULATED_TYPE");
        Object notDisplayingDate = baselineRow.get("PUP_NO_USE_TP");

        // Clean collection mapping
        List<Map<String, Object>> cleanList = matchedRows.stream().map(row -> {
            Map<String, Object> innerMap = new HashMap<>();
            innerMap.put("EXPS_YN", row.get("EXPS_YN"));
            innerMap.put("EXPS_OS_TP", row.get("EXPS_OS_TP"));
            return innerMap;
        }).collect(Collectors.toList());

        // Construct node entry structure safely
        Map<String, Object> elementMap = new HashMap<>();
        elementMap.put("type", type);
        elementMap.put("notDisplayingDate", notDisplayingDate);
        elementMap.put("list", cleanList);

        outputMap.put(outputKey, elementMap);
    }

    return outputMap;
}
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

- 이 때 알았는데, 삭제만이 아니라 원래도 없는 index의 요소를 읽어오려고 하면 indexOutOfRange exception이 나는 것이었다.
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

## <span style="color:#802548">_이전 페이지 없이 바로 해당 url로 들어오는 것을 방지하기_</span>

```javascript
if(document.referrer == ""){
        closeWebView();
}else{
        history.back();
}
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

## <span style="color:#802548">_nativeBridge_</span>

```js
var callbackId=Math.floor(Math.randow()*2000000000);
var callbacks = {}

window.NativeBridge = {
	exec: function(action,param,successCallback,fallCallback){
		var sCallbackId = action + (callbackId++).toString();
		
		callbacks[sCallbackId]={
			success : successCallback,
			fail : failCallback
		}

		var osName = getOSName()
		if(osName=='android'){
			try{
				var paramJsonString = null;
				if(param != null && typeof param =="object"){
					paramJsonString = JSON.stringify(param);
					window["allonepayApp"][action](paramJsonString, sCallbackId)
				}else{
					window["allonepayApp"][action](sCallbackId)
				}
			}catch(e){
				window.nativeBridge.fail(sCallbackId, 'action' + action + ':: nativeBridge exception ' + e);
			}
		}else if(osName =='ios'){
			try{
				var cmd={
					name : action,
					param : param,
					sCallbackId : sCallbackId
				}
				window.webkit.messageHandlers.NHCWebInterface.postMessage(JSON.stringify(cmd))
			}catch(e){
				window.nativeBridge.fail(sCallbackId, 'action' + action + ':: nativeBridge exception ' + e);
			}
		}else{
			console.log('web?');
			window.nativeBridge.success(sCallbackId, action + 'result success!');
		}
	}

	getCallbacks: function(){
		return callbacks;
	}

	sucess: function(callbackId,data){
		window.nativeBridge.callbackFromNative(callbackId,true,data);
	}
	
	fail: function(callbackId,data){
		window.nativeBridge.callbackFromNative(callbackId,false,data):
	}

	callbackFromNative: function(callbackId, isSuccess, data){
		var callback = callbacks[callbackId]
		if(isSuccess){
			callback.sucess && callback.success.apply(null,[data]):
		}else{
			callback.fail && callback.fail.apply(null,[data]);
		}
		delete callbacks[callbackId];
	}

}
```