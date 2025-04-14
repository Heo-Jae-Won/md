---
title: カードアプリの高度化 
published: true
---



## <span style="color:#802548">_最適化されないポップアップ_</span>

- アプリではポップアップのタイプがBOTTOMとLAYERがあるのでそれを分けて情報を送るようにしました。
- 最初はデータをまとめないでそのまま送りました。

```java
String[] param = {"M","B","P","F"};
String[] outputKey = {"Mine","Beneficial","Paying","Financical"};
List<HashMap<String,Object>> infoList = new ArrayList<>();
HashMap<String,Object> outputMap = new HashMap<>();

List<HashMap<String,Object>> subPopupList = new ArrayList<>();
String type = "";

for(int i = 0; i < param.length; i++) {
    HashMap<String,Object> inputParamMap = new HashMap<>();
    HashMap<String,Object> listElementMap = new HashMap<>();
    inputParamMap.put("LOC",param[i]);
    subPopupList = Dao.selectList(inputParamMap); //SELECT

    if(subPopupList.isEmpty()){
        continue;
    }

    for(int j = 0; j < subPopupList.size(); j++) {
        listElementMap.put("EXPS_YN", subPopupList.get(j).get("EXPS_YN"));
        listElementMap.put("EXPS_OS_TP", subPopupList.get(j).get("EXPS_OS_TP"));
        listElementMap.put("type", subPopupList.get(0).get("PUP_TP"));
        infoList.add(listElementMap);
    }

    outputMap.put("infoList", infoList); 
}

return outputMap;
```


## <span style="color:#802548">_共通点が見えるようにしたポップアップ_</span>

- 共通のデータはまとめて送るようにしました。

```java
String[] param = {"m","B","P","F"};
String[] outputKey = {"m","b","p","f"};
HashMap<String,Object> decodeXssMap = new HashMap<>();
HashMap<String,Object> decodeXssList = new HashMap<>();
HashMap<String,Object> outputMap = new HashMap<>();

List<HashMap<String,Object>> subPopupList = new ArrayList<>();
String type = "";

for(int i = 0; i < param.length; i++) {
    HashMap<String,Object> inputParamMap = new HashMap<>();
    
    inputParamMap.put("LOC",param[i]);
    subPopupList = Dao.selectList(inputParamMap); //SELECT

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

    HashMap<String,Object> infoPerType = new HashMap<>();　 // 共通点はパット見えるように
    infoPerType.put("type", type);
    infoPerType.put("list", decodeXssList);
    outputMap.put(outputKey[i], infoPerType);
}

return outputMap;
```

## <span style="color:#802548">_SQLを最小限コールするポップアップ_</span>

- SQLを4回コールする形になって効率的に良くないと感じました。
- 性能の問題になるかもしれないと思ってSQLは一回でSQLでデータを作るようにすることにしました。

```java
HashMap<String,Object> outputMap = new HashMap<>();

try{
    List<HashMap<String,Object>> subPopupList =  Dao.selectList();
    String[] param = {"M","B","P","F"};
    String[] outputKey = {"m","b","p","f"};
    String type = "";
    String notDisplayingDate = "";
    for(int i = 0; i < param.length; i++){
        HashMap<String,Object> infoPerType = new HashMap<>();
        HashMap<String,Object> infoElement = new HashMap<>();
        List<HashMap<String,Object>> infoList = new ArrayList<>();
        for (int j = 0; j < subPopupList.size(); j++) {
            if (param[i].equals(subPopupList.get(j).get("BLTN_LOC"))) {
                infoElement.put("EXPS_YN",subPopupList.get(j).get("EXPS_YN"));
                infoElement.put("EXPS_OS_TP",subPopupList.get(j).get("EXPS_OS_TP"));
                /*
                    １つの領域では必ず１つのタイプしか選択できません。アプリのUXのため制限を設けようと企画者と打ち合わせしました。
                 */
                type = "B".equals(subPopupList.get(j).get("PUP_TP")) ? "bottom" : "layer";
                infoList.add(infoElement);
            }
        }

        if (infoList.isEmpty()) {
            continue;
        }

        for (int k = 0; k < infoList.size(); k++) {
            break;
        }

        infoPerType.put("type", type);
        infoPerType.put("list", infoList);
        outputMap.put(outputKey[i], infoPerType); 
    }
}catch(Exception e){
        throw e;
}

return outputMap;
```

## <span style="color:#802548">_重要ではないものはEXCEPTION使わない_</span>

- TO-BE画面に変更する過程でエラーが発生しました。
- 私がすでに作った管理者画面は、企画が最大5つまで来て5つにしました。

```java
//既存
for(int i = 0; i < infoList.size(); i++){
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
for (int k = 0; k < infoList.size(); k++) {
    if (k > 5){
        break;
    }
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
	for (int k = 0; k < infoList.size(); k++) {
        if (k > 5){
            break;
        }
    }
}catch(Exception e){
        throw e;
}


//変更
try{
	.
	.
	.
	for (int k = 0; k < infoList.size(); k++) {
        if (k > 5){
            break;
        }
    }
}catch(Exception e){
        outputMap.put("type","L");
        outputMap.put("list",Collections.emptyList());
        return outputMap;
}
```


## <span style="color:#802548">_配布を最小限にするポップアップ_</span>

- テスト段階で管理者のポップアップ管理画面で修正要求がありました。
    - その内容でユーザーサーバーに影響を与えられることは管理者が作るポップアップの制限がないようにすることでした。
    - 管理者サーバーでの修正がユーザーサーバーでの配布にならないように処理しました。


```java
for (int k = 0; k < infoList.size(); k++) {
    if (k > 5){
        break;
    }
}
```