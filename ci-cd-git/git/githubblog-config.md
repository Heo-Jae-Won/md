## <span style="color:#802548">_1. thema 선택_</span>
- https://jekyllthemes.io/github-pages-themes 에서 원하는 thema를 살펴볼 수 있다.
- 원하는 thema를 골랐으면 해당 thema를 clone해서 자신의 github blog로 만들면 된다.
  - 나는 https://github.com/tocttou/hacker-blog 를 골랐다.

## <span style="color:#802548">_2. 깃허브와 연동_</span>
- git hub와 연동하려면 repo를 일단 만들어야한다.
- 주의할 점은 repo의 이름이 반드시 본인이 설정한 이름 + github.io로 해야한다는 것이다.
  - 예로 내가 Heo JaeWon으로 했다면, repo는 Heo-Jae-Won.github.io로 해줘야 한다.
- _posts 폴더에 post를 올릴 때도 반드시 yyyy-mm-dd-파일명.md로 post파일을 올려야 한다.
  - 예를 들면 아래와 같이 commit, push해야 한다는 의미다.
    - 2023-09-16-linux-command.md, 2024-03-07-network-first.md
- 더불어 md 파일내에 published를 true로 설정해야 한다.
  - md 파일의 맨 위에 아래와 같이 제목을 정하고 published를 true로 줘야한다는 의미다.
  - 블로그 상에서 글이 써진 날짜는 파일명에 붙은 날짜로 지정된다. 
```
---
title: Network first week
published: true
---
```


## <span style="color:#802548">_3. 목차 보이게_</span>
- 목차가 보이게 하는 방법은 아래와 같다.
  - _include 폴더에 toc.html을 넣는다.
  - toc.html은 보통 https://github.com/allejo/jekyll-toc/tree/master/_includes 에서 toc.html을 가져온다
  - _layout 폴더에 post.html을 수정한다. 
  - _sass 폴더에 base.css에 css를 추가한다. css이름은 테마마다 다를 것이다.
- 그럼 이제 모든 post마다 상시로 목차가 보인다.
  - 이 목차는 스크롤을 해도 사이드바처럼 계속 달라붙어 있어 매우 편하다.

- _layout/post.html은 아래와 같이 수정해준다.
- 참고로 아래 css와 html 수정은 https://ts.devbj.com/jekyll-toc 에서 가져왔다.
```html
<article>
  <h2>{{ page.title }}</h2>
  <time datetime="{{ page.date | date_to_xmlschema }}" class="by-line">
    {{ page.date | date_to_string }}
  </time>
  <div class="col-md-12 mx-auto">

    <!-- 
    refer from 
    - https://devinlife.com/
    - https://github.com/allejo/jekyll-toc
    -->
    <aside class="sidebar__right sticky" style="height: auto !important;">
    <div class="toc">
      <a href="/">홈으로</a>
      {% include toc.html html=content sanitize=true h_min=1 h_max=2 %}
    </div>
    </aside>
    
    
    {{ content }}
    </div>
</article>
```


- css 수정은 나의 경우, base.css에 추가해야했다.
- 다른 사람은 다른 css일 확률이 높다. thema마다 다르다.
```css
.sidebar__right {
  display: block;
  position: sticky;
  top: 0;
  right: 0;
  width: 200px;
  margin-right: -500px;
  /* padding-left: 1em; */
  z-index: 10;
  clear: both;
  position: -webkit-sticky;
  top: 2em;
  float: right;
}

.toc {
  background-color: #472f2f;
  border: 1px solid #b6b6b6;
  border-radius: 4px;
  -webkit-box-shadow: 0 1px 1px rgba(0,0,0,0.125);
  box-shadow: 0 1px 1px rgba(0,0,0,0.125);
  width: 100%;
  font-size: 1em;
  ul {
    margin: 0;
    padding: 4px;
    list-style: none;
  }
  li {
    padding: 0px 10px;
  }
}
```


## <span style="color:#802548">_4. local 수정가능하게_</span>
- 우선 jekyll은 ruby에서 돌아가므로 ruby를 다운로드 해야한다.
  - https://www.ruby-lang.org/en/downloads/
  - 원도우 인스톨러: https://rubyinstaller.org/downloads/
- 내가 받은 thema는 gemfile이 없었다.
  - 그래서 could not locate a gemFile 오류가 났다.
  - gemfile이 없으면 ruby가 돌아가지 않는다.
  - 그럴 땐 clone 받은 git repo를 git bash로 들어가서 bundle init을 해준다.
  - bundle init을 하면 gemFile이 형성된다.
    - 처음에는 gemFile에 아무것도 없다.
    - 이제 config.yaml에 있는 plugin들을 차례차례 모두 다운로드한다.
    - 나는 jekyll-seo-tag, jekyll-paginate, jekyll-sitemap였다.
    - 그래서 bundle add (plugin)으로 다운로드했다.
    - 3개라서 차례차례 3개를 모두 다운로드했다.
  - 그러고 나서 bundle exec jekyll serve를 하면 local에서도 볼 수 있다.
    - 보통 4000 port number다.

- 마지막으로 .gitignore 파일을 만들어 아래와 같이 추가해주자.
  - 로컬에서 돌리면 post를 바탕으로 html을 생성해주는데, 이 html들을 github 소스에는 불필요하다.
  - 또한 Gemfile.lock도 불필요하다. 어차피 bundle install해주면 되기 때문이다.
  - 따라서 해당 내용들을 git 상에서 무시해주자. 
```
/.history
_site/**
Gemfile.lock
```

