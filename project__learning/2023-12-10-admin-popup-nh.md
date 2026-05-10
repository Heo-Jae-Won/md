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

## <span style="color:#802548">_네이티브에 뿌리는 내용_</span>

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

```java
import java.util.*;
import java.util.stream.Collectors;

public Map<String, Object> getPopupOutputMap() throws Exception {
    Map<String, Object> outputMap = new HashMap<>();
    
    // 1. Single DB Round-trip
    List<Map<String, Object>> subPopupList = Dao.selectList();
    if (subPopupList == null || subPopupList.isEmpty()) {
        return outputMap;
    }

    // Mapping arrays
    String[] params = {"M", "B", "P", "F"};
    String[] outputKeys = {"My", "Benefit", "Payment", "Finance"};

    // 2. Pre-group database rows by their BLTN_LOC location identifier for O(1) lookups
    Map<String, List<Map<String, Object>>> groupedByLoc = subPopupList.stream()
            .filter(row -> row.get("BLTN_LOC") != null)
            .collect(Collectors.groupingBy(row -> row.get("BLTN_LOC").toString()));

    // 3. Process each configuration location parameter
    for (int i = 0; i < params.length; i++) {
        String currentParam = params[i];
        List<Map<String, Object>> matchedRows = groupedByLoc.get(currentParam);

        // Skip processing if no database records exist for this code type
        if (matchedRows == null || matchedRows.isEmpty()) {
            continue;
        }

        // 4. Safely extract metadata properties from the primary baseline entry (Index 0)
        Map<String, Object> firstRow = matchedRows.get(0);
        String type = "B".equals(firstRow.get("PUP_TP")) ? "bottom" : "layer";
        Object notDisplayingDate = firstRow.get("PUP_NO_USE_TP");

        // 5. Generate unique individual list entries up to a maximum threshold limit of 5
        List<Map<String, Object>> decodeXssList = matchedRows.stream()
                .limit(5) 
                .map(row -> {
                    Map<String, Object> cleanMap = new HashMap<>();
                    cleanMap.put("EXPS_YN", row.get("EXPS_YN"));
                    cleanMap.put("EXPS_OS_TP", row.get("EXPS_OS_TP"));
                    return cleanMap;
                })
                .collect(Collectors.toList());

        // 6. Assemble isolated final data structure node object
        Map<String, Object> elementMap = new HashMap<>();
        elementMap.put("type", type);
        elementMap.put("notDisplayingDate", notDisplayingDate);
        elementMap.put("list", decodeXssList);

        outputMap.put(outputKeys[i], elementMap);
    }

    return outputMap;
}

```





```sql
WITH TargetParams AS (
    -- This inner table serves as a virtual join table providing your specific filter targets
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
    INNER JOIN TargetParams tp ON pm.BLTN_LOC = tp.LOC
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