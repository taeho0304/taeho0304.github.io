---
layout: post
title: Location 속성 root vs alias [NGINX]
description: >
  [NGINX] root vs alias <br>
  참고 <br>
  - https://stackoverflow.com/questions/10631933/nginx-static-file-serving-confusion-with-root-alias
tags: [NGINX]
---

## Location 속성 root vs alias [NGINX]

## root vs alias

nginx 환경설정에서 location 속성을 이용해 디렉터리 접근 경로를 설정할 때 주로 alias와 root 키워드를 사용한다.

이때 alias와 root 키워드의 차이점을 알아보자.


### root

root는 location으로 넘어온 부분을 root로 설정한 경로에 추가한다.

```bash
location /static/ {
    root /var/www/app/static/;
    autoindex off;
}
```

위와 같이 location을 설정하면 location으로 넘어온 /static/을 경로에 추가해 /var/www/app/static/static/ 에서 찾는다.

### alias
 

alias는 location으로 넘어온 부분을 제거하고, alias로 설정한 경로만 이용한다.

```bash
location /static/ {
    alias /var/www/app/static/;
    autoindex off;
}
```

위와 같이 location을 설정하면 location으로 넘어온 /static/ 부분은 경로에서 제거하고 alias로 설정한 /var/www/app/static/ 에서 찾는다.

## 예시

아래와 같이 /home/ubuntu/images에 있는 파일에 접근해보자.

![](https://taeho0304.github.io/assets/img/NGINX/Root&Alias.png)


nginx 설정파일에 들어가서 아래 두가지 방법중 하나로 location을 설정해 준다.

![](https://taeho0304.github.io/assets/img/NGINX/root&alias_ex.png)

설정을 마친 뒤 본인의 도메인 주소 뒤에 /images/{파일명} 을 입력하면 해당 파일을 찾아볼 수 있다.

![](https://taeho0304.github.io/assets/img/NGINX/root&alias_ex2.png)

