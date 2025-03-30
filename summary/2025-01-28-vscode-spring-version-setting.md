## <span style="color:#802548">_vscode lanuage supoort java 사용하기_</span>

- lanugage support java extension은 auto-update를 꺼놔야 한다.
- 아니면, 그냥 jdk 요구 버전 자체가 올라간다.
- 이 jdk는 extension이 돌아가는 jdk 버전이기 때문에 무조건 맞춰줘야 한다. 
- 1.39.0버전부터는 jdk 21이상을 반드시 요구하게끔 되어있다.
    - open jdk를 깔기만 하면 알아서 인식된다.
    - 그 뒤로는 각 프로젝트 마다 java runtime을 setting해서 8부터 21까지 쓸 수 있다.
    - 공통 버전의 경우,   "java.jdt.ls.java.home" 라는 값을 user settings에 넣는다.
    - 아래처럼 설정하면, lanuage support in java extension의 JAVA_HOME 경로가 된다.
```sh
# user settings
"java.jdt.ls.java.home": "c:\\Users\\user\\Downloads\\openlogic-openjdk-17.0.9+9-windows-x64",
```

- 위는 기본 default가 17로 처리된다는 의미이다. 
- 여기서 workspace 별로 jdk를 다르게 가져가고 싶다면?
    - 매 workspace setting마다 값을 주어야 한다.
    - workspace setting을 열어서 아래처럼 값을 준다.
    - workspace setting마다 jdk는 하나기 때문에 하나만 딱 명확하게 명시하자.
```sh
# workspace settings
"java.configuration.runtimes": [
    {
        "name": "JavaSE-17",
        "path": "C:\\Program Files\\Java\\jdk-17",
    }
]
```

- jdk 21을 깔기 전까지는 java 관련 명령어가 모두 제대로 인식되지 않는다.
    - java: Configure Java Runtime
    - Java: Clean Java Language Server Workspace
- 모두 먹통이 되어버린다.



## <span style="color:#802548">_현재 project가 사용중인 jdk version check_</span>

- 그럼 이제 runtime이 어떻게 설정되었는지는 아래처럼 확인한다.
- 아까 name으로 주었던 것이 jdk runtime에 그대로 노출된다.

```sh
java: Configure Java Runtime
```

- classpath도 위의 명령어에서 source를 확인하면 된다.



## <span style="color:#802548">_project에 local, dev 알려주기_</span>

- project가 어떤 profile을 갖고 있는지를 명시하려면 launch.json에 넣어야 한다.
- launch.json은 ctrl + shift + d해서 create a launch.json 파일을 해주면 만들어진다.
- args는 아래처럼 넣어주면 된다.

```sh
"args": ["--spring.profiles.active=local"],
```


## <span style="color:#802548">_java clean_</span>
- 가끔 workspace가 문제가 일 때는 아래 명령어를 활용한다.


```sh
Java: Clean Java Language Server Workspace
```

## <span style="color:#802548">_debug console로 옮기기_</span>
- Terminal에서는 진짜 cmd나 git bash 등의 역할을 하게 하기 위해 debug console로 옮겨주자.
- user settings에서 아래처럼 추가해준다. 그럼 debug console에서 error message가 출력되게 된다.
- 아래 옵션을 주석처리하면 terminal에서 출력되게 된다.

```sh
"java.debug.settings.console": "internalConsole",
```

- 그 다음 debug 색깔을 더 알록달록하게 볼 수 있게 하기 위해 application.properties에서 아래처럼 설정해준다.
- 다만 운영에서는 이렇게 하면 안된다.

```sh
spring.output.ansi.enabled= ALWAYS
```

- 색깔을 더 알록달록하게 하기 위해서 아래처럼 user settings에 설정해준다.

```
  "workbench.colorCustomizations": {
    "debugConsole.infoForeground": "#4982cc",
    "debugConsole.sourceForeground": "#a04798",
    "debugConsole.warningForeground": "#ffffff",
  },
```

- error trace에 관해서는 알록달록하게 만들어주는 설정이 따로 존재하지 않는다.

## <span style="color:#802548">_다른 버전 jdk 설치 이후 error_</span>
- jdk 21버전을 설치하고 부터 계속 gradle build가 error가 난다. 
- 분명히 JAVA_HOME 경로는 jdk 17로 잘 설정되어 있다.
- java --version으로 했었을 때도 17이 나왔다.
- 그런데 ouput에 계속 아래와 같이 Java home이 잡혔다. 
- 프로젝트의 jdk는 17인데, gradle의 jdk는 21이니까 build가 안되는 것이다.

```
[info] Java Home: C:\Users\user\.vscode\extensions\redhat.java-1.40.2025013108-win32-x64\jre\21.0.6-win32-x86_64
```

- 나중에 java --version을 해보니까 21로 나와서 뭐지했더니 시스템 환경변수에서 JAVA_HOME은 그대로였다.
- 그런데 Path의 맨위 상단에 아래와 같이 jdk21이 떡하니 박혔다.

```
C:\Program Files\OpenLogic\jdk-21.0.6.7-hotspot\bin
```

- 그래서 이게 문제구나 직감하고 지웠다.
- 하지만 여전히 문제는 똑같았다. 문제는 Java Lanugae Extension pack이었다.
- 이 extension pack을 지우니까 JAVA_HOME을 제대로 인식하기 시작했다.
    - 여기서 뭔가 문제가 생긴게 틀림없었다.
    - 살펴보니 lanugage support의 settings에 gradle 쪽이 따로 있었다.

```
Java › Import › Gradle › Java: Home
The location to the JVM used to run the Gradle daemon.
```

- 이제 gradle은 문제없이 build가 된다.
- 아니면 user setting.json에 아래처럼 추가하자.

```sh
"java.import.gradle.java.home": "C:\\JDK\\jdk-17.0.2",
```

- 하지만 run을 때리면 여전히 문제가 발생했다.

```
org.hibernate.exception.JDBCConnectionException: unable to obtain isolated JDBC connection [Communications link failure

The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.] [n/a]
```

## <span style="color:#802548">_실제 error의 원인_</span>
- 그래서 communication link가 왜 failure가 나는 지 살펴보니, url이 달랐다.
- 학원에선 3306 port였는데, 집에선 3366 port였다.
- 그래서 해당 부분을 수정했더니, 잘 run 된다.

- 그래서 궁금해져서 gradle build option인 "java.import.gradle.java.home"을 지워봤는데, 그래도 잘 된다.
- vscode에서 gradle build가 fail되도 spring server를 run하는데는 문제가 없는 것으로 보인다. 왜 그런지는 모르겠다.
- 하지만 gradle build가 실패하면 grade extension에서 dependency를 아무것도 볼 수 없게 된다.
