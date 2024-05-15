## <span style="color:#802548">_좋은 error message 만들기_</span>
- 좋은 오류메시지 구성요소는 다음과 같다.
  - 오류가 생긴 원인
  - 어떤오류인지
  - 오류를 극복한 시도
- 실제 예시로 들면 아래와 같다.
- 아래같이 오류가 생긴 원인은 알았다.
- 그러나 해당 메시지로는 어떤 오류인지는 알기 어렵다.
```
couldnt parse config file
```

- 어떤 파일 때문에 오류가 났는지 명확하게 적는다.
```
couldnt parse config file ---> couldnt parse config file: /etc/sample-config.properties
```

- 해당 파일의 어떤 부분 때문에 오류가 났는지 구체적으로 적는다.
```
couldnt parse config file: /etc/sample-config.properties---> couldnt parse config file: /etc/sample-config.properties; given snapshot mode 'nevr' isn't valid
```


- 오류를 고치기 위해서 할 수 있는 간단한 solution도 추가해준다.
```
couldnt parse config file: /etc/sample-config.properties; given snapshot mode 'nevr' isn't valid---> couldnt parse config file: /etc/sample-config.properties; given snapshot mode 'nevr' isn't valid(must be one of initial, always, never)
```

- 서버에서 뿌려줄 때 메시지만이 아니라 코드도 같이 주는 게 훨씬 좋다.
- 메시지는 변경될 수 있기 때문에 코드와 묶어주는 게 좋다.
  - ORA-01476: divisor is equal to zero

- 오류메시지를 쓰는 법은 아래와 같다.
  - 태는 통일한다.
    - couldnt parse config file (능동태)
    - config file couldnt be parsed (수동태)
    - 하나 골라서 모든 오류 메시지를 동일한 태로 쓴다.
  - 민감한 정보는 노출하면 안 된다.
  - 오류는 가장 빨리 일어난 시점에서 출력되어야 한다. 실패시점에 바로 던지는게 좋다.

<br/>

- 오류 메시지를 쓰는 법을 추천 예시와 비추천 예시를 통해 살펴보자.
- 오류 원인을 정확히 파악하는 메시지가 좋다.
```
not recommended
bad directory

recommended
the specified directory exists but is not writable. To add files to this directory the directory must be writable.
```

- 사용자의 유효하지 않은 입력을 파악해서 알려주는 메시지가 좋다.
```
not recommended
invalid postal code

reommended
The postal code for the US must consist of either five or nine digits. The psecified postal code (input) contained sevent digits.
```

- 요구사항과 제약 사항을 정확하게 명세하는 메시지가 좋다.
```
not recommended
The combined size of the accatchments is too big.

recommended
The combines size of the attachments (14MB) exceeds the allowed limit (10MB). 
```
- 예제를 제공하는 메시지가 좋다.
```
Not recommended
Invalid email address.

recommended
The specified email address (robin) is missing an @Sign and a domain name. For example: robin@example.com
```
- 오류메시지는 간결해야한다.
```
Not recommended
The resource was not found and cannot be differentiated. What you selected doesnt exists in the cluster.

recommended
Resource<name> isn't in cluseter <name>.
```

- 이중부정은 읽기 어려우니 쓰지 말자.
- 아래같이 excep와 unless는 같이 쓰지 않는다.
```
Not reco 
The App Engine Service account must have permissions on the image, except the Storage Object Viewer role, unless the Storage Object Admin role is available.

recommended

the App Engine service account must have one of the following roles:
 1. Storage Object Admin
 2. Storage Object Creator
```
- 사용자에 맞는 맞춤형 오류가 필요
```
not recommended
A server dropped ur clients' request because the server farm is running at 92% CPU capacity. Retry in finve minutes.

recommended
So many People are shopping right now that our system can't complete ur purchase. Dont worry-- we won't lose ur shopping cart. Plz retry ur purchase in five minutes.
```
- 오류 메시지는 일관되게 사용한다.
```
Not recommended
Cant connect to cluster at 127.0.0.1:56. Check whether minikube is running.

recommended
Cant connect to minikube at 127.0.0.1:56. Check whether minikube is running.
```
- 긍정적 표현을 사용하자
```
Not recommended
you entered an invalid postal code.

recommended
Enter a valid postal code.
```
- 과도한 사과를 피하자
```
Not recommended
we're sorry, a server error occurred and we're temporarilty unable to load ur spreadsheet. we apologize for the inconvenience. plz wait a while and try again.

recommended
Google Docs is temporarily unable to open ur spreadsheet. In the meantime, try right-clicking the spreadsheet in the doc list to download it
```
- 사용자를 비난하지말자
```
Not recommended
You specifed a printert that's offline

recommended
The specified printer is offline.
```
 