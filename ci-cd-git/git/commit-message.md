## <span style="color:#802548">_git commit message 남기는 법_</span>
- commit은 how 대신 what과 why로 적는 것이다. 
  - 맨처음은 요약 50글자
    - 그래야 git log --oneline 으로 명령으로 요약이 전부 출력된다.
  - 본문은 70글자 정도..
  - 마지막은 참고사항을 적는다.
- git commit -m "Fix typo in introduction to use guide" 같이 -m 옵션은 쓰면 안된다.
  - vscode는 source control에서 컨트롤 엔터하면 vim처럼 쓸 수 있게 제공해준다.
  - 자동 행갈이가 없으니 enter를 쳐줘야 한다.
- commit은 아래와 같이 구성해서 쓴다.
```
Change default confirm taget from 2 to 6  //1. 요약 50자짜리 commit
//한줄 띄고

//바꾼 이유
Recent discussion(in IRC meetings, and e.g. #8989) has shown a preference for the default confirm target for smartfees to be 6 instead of 2, to avoid overpaying fees for questionable gain.
//한줄 띄고

//참고사항
6 is also a compromise between the GUI's pre-#8989 value of 25 and the bitconid ... 
```



- 구체적인 commit message를 쓰는 사항은 아래와 같다.
  - 주제행 첫글자는 대문자가 되어야 한다.
  - 행 마지막에 점을 찍으면 안 된다
  - 주제 행을 명령문으로 만들어라
    - fixed bug -->fix bug
    - changing behavior -> change behaviour
    - sweet new API -> release version 1.0.0
    - more fix for broken stuff -> remove deprecated methods


- 잘 쓴 예시는 아래와 같다.
```
Convert template to US-ASCII to fix error

I introduced some tests in a feature branch to match the contents of
`/etc/nginx/router_routes.conf`. They worked fine when run with `bundle exec
rake spec` or `bundle exec rspec modules/router/spec`. But when run as
`bundle exec rake` each should block failed with:

    ArgumentError:
      invalid byte sequence in US-ASCII

I eventually found that removing the `.with_content(//)` matchers made the
errors go away. That there weren't any weird characters in the spec file. And
that it could be reproduced by requiring Puppet in the same interpreter with:

    rake -E 'require "puppet"' spec

That particular template appears to be the only file in our codebase with an
identified encoding of `utf-8`. All others are `us-ascii`:

    dcarley-MBA:puppet dcarley$ find modules -type f -exec file --mime {} \+ | grep utf
    modules/router/templates/routes.conf.erb:text/plain; charset=utf-8

Attempting to convert that file back to US-ASCII identified the offending
character as something that looked like a whitespace:

    dcarley-MBA:puppet dcarley$ iconv -f UTF8 -t US-ASCII modules/router/templates/routes.conf.erb 2>&1 | tail -n5
      proxy_intercept_errors off;

      # Set proxy timeout to 50 seconds as a quick fix for problems
      #
    iconv: modules/router/templates/routes.conf.erb:458:3: cannot convert

After replacing it (by hand) the file identifies as `us-ascii` again:

    dcarley-MBA:puppet dcarley$ file --mime modules/router/templates/routes.conf.erb
    modules/router/templates/routes.conf.erb: text/plain; charset=us-ascii

Now the tests work! One hour of my life I won't get back..
```

- commit의 맨 앞에 나오는 전형적인 단어들은 아래와 같다.
  - fix -> 올바르지 않은 동작을 고침
  - add -> 기능, test, 문서를 추가
  - remove -> 코드를 삭제
  - use -> 뭔가를 사용해 구현
  - refactor -> 리팩토링
  - simplify -> refactor보단 약한 수정( if문 위치 옮기기.. field명만 바꾸기.. )
  - update -> 수정/추가/보완
  - improve -> 호환성/테스트커버리지/접근성 향상이 있을 때
  - implement -> 클래스/모듈 단위로 코드를 추가
  - revise -> 문서를 개정
  - correct -> 문법 오류, 타입 변경, 이름 변경
  - ensure  -> 오류처리가 없었는데 추가 등 기능을 보장
  - prevent -> stackoverflow나 out of memory 등이 일어날 수 있는데 이를 막기 위해 코드를 수정한 경우


<br/>


- 파일 수정의 목적이 다르면 각자 개별 commit으로 넣어야 한다.
  - 기능추가
  - 주석보완
  - import정리
    - 위 3개는 모두 파일 수정 목적이 다르다.
    - 개별 commit으로 넣어야 한다..