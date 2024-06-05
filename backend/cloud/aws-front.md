- front를 배포할 때는 S3, CloudFront를 많이 사용한다.
- S3는 origin 저장소라면, CloudFront는 caching 저장소, 임시저장소다.
- S3는 HTTPS 기능이 없어서, HTTPS를 쓰려면 CloudFront를 사용해야 한다.
- 즉 두개 같이 조합해서 front 배포가 필요하다는 의미다.
- user가 cloudfront에 요청해서 있으면 바로 주고, 없으면 S3에 요청해서 유저에게 준다.
- 한번 cloudfront에 저장해두면 계속 캐싱시간동안은 S3까지 가지 않고 캐싱서버인 cloudFront에서 준다.

- S3에서 bucket을 만들어준다.
- Block all public access은 체크 해제한다. 웹페이지라서 public access를 받아야한다.
- 객체가 퍼블릭 상태가 될 수있다는 경고에 대해 알고있다고 체크 해준다.
- 만들고 Permission tab을 눌러 bucket policy를 edit한다.
- add new statement를 하고, 서비스는 s3를 택하고, 하위 수준은 GetObject를 택한다.
- 밑에 add new resources에는 리소스 유형을 object로 고르고, bucketname은 아까 주었던 S3 이름을 준다.
- ObjectName은 *로 적어준다. 모두 다 줄것이기 때문이다. 
- 마지막으로 principal은 {}가 아니라 *로 적어둔다.

<br>

- 이제 properties tab에서 static website hosting을 enable로 바꾼다.
- enable로 바꾸고 index document와 error document를 index.html로 놓는다.
- front project를 build한 결과물들을 모두 upload한다.
- 폴더를 올리는 게아니라 결과물을 올리는 것이다.
- 다 배포했으면 접속이 가능하다.

```
http://srt-front.s3-website.ap-northeast-2.amazonaws.com/
```

<br>

- 그 다음은 cloudFront에 배포할 차례다.
- 원본 도메인은 S3를 의미한다. 아까 올린 srt-front를 클릭해준다.
- default cache에서 Redirect HTTP to HTTPS로 설정해준다.
- 보안 보호는 일단은 비활성화한다. 비활성화해도 보안이 취약한 건 아니다.
- setting에서 Use North America, Europe, Asia, Middle East, and Africa을 택해준다.
- Default root object는 index.html로 적는다.
- 배포가 완료되면 배포 도메인이 노출된다.

```
https://dkl93t7o3pkmq.cloudfront.net
```

<br>

- 클라우드 프론트에 연결하기 전에 인증서를 발급받아야 한다.
- 클라우드 프론트에서 HTTPS를 적용하려면 인증서를 region이 서울이 아니라 미국 동부 버지니아에서 받아야 한다.


