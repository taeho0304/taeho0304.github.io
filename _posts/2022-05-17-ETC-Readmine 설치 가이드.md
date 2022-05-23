---
layout: post
title: Readmine 설치 가이드

description: >
  참고 <br>
  - Ubuntu 18.04 : Redmine 설치, 구성 방법, 예제, 명령어 : <https://jjeongil.tistory.com/1445><br>
  - HowTo Install Redmine on Ubuntu step by step? : <https://www.redmine.org/projects/redmine/wiki/howto_install_redmine_on_ubuntu_step_by_step#Configuring-Apache><br>
tags: [ETC]
---

## Readmine 설치 가이드

## Readmine 설치 가이드

### 전제조건 : Apache, mod-passenger 및 MySQL 설치

#### Apache, mod-passenger 설치

``` shell
$ sudo apt-get install apache2 libapache2-mod-passenger
```

#### mysql 설치

``` shell
$ sudo apt-get install mysql-server mysql-client 
```

### MySQL 데이터베이스 생성

1. 로그인
``` shell
$ sudo mysql
``` 

2. DB 생성
``` shell
CREATE DATABASE redmine CHARACTER SET utf8mb4;
``` 

3.  MySQL 사용자 계정 생성 및 액세스 권한 부여
``` 
GRANT ALL ON redmine.* TO 'redmine'@'localhost' IDENTIFIED BY 'change-with-strong-password';
``` 

4. 종료
``` 
EXIT;
``` 
### Ruby 설치

다음을 입력하여 Ruby를 설치합니다.
``` shell
$ sudo apt install ruby-full
```
### Ubuntu에 Redmine 설치

먼저 Redmine을 구축하는 데 필요한 종속성을 설치합니다. 

``` shell
$ sudo apt install build-essential libmysqlclient-dev imagemagick libmagickwand-dev
```

1. Redmine을 다운로드
   
다음 컬 명령을 사용하여 Redmine 아카이브를 다운로드합니다.  Redmine의 최신 안정 버전은 4.2.6 버전입니다.

``` shell
$ sudo curl -L http://www.redmine.org/releases/redmine-4.2.6.tar.gz -o /tmp/redmine.tar.gz
```

다운로드가 완료되면 아카이브를 추출하여 /opt 디렉토리로 이동합니다.
``` shell
$ cd /tmp
$ sudo tar zxf /tmp/redmine.tar.gz
$ sudo mv /tmp/redmine-4.2.6 /opt/redmine
```

2. Redmine Database 구성

먼저 Redmine 예제 구성 파일을 복사하십시오.

``` shell
$ sudo cp /opt/redmine/config/database.yml.example /opt/redmine/config/database.yml
```

텍스트 편집기로 파일을 엽니다.

``` shell
$ sudo nano /opt/redmine/config/database.yml
```

프로덕션 섹션을 검색하고 이전에 생성한 MySQL 데이터베이스 및 사용자 정보를 입력합니다.

``` 
# /opt/redmine/config/database.yml

production:
  adapter: mysql2
  database: redmine
  host: localhost
  username: redmine
  password: "change-with-strong-password"
  encoding: utf8
``` 

3. Ruby 종속성 설치

Redmine 디렉토리로 이동하여 번들러 및 기타 Ruby 종속성을 설치합니다.

``` shell
$ cd /opt/redmine/
$ sudo gem install bundler --no-rdoc --no-ri 
$ sudo bundle install --without development test postgresql sqlite
``` 

4. 데이터베이스를 마이그레이션

다음 명령을 실행하여 키를 생성하고 데이터베이스를 마이그레이션합니다.

``` shell
cd /opt/redmine/
sudo bundle exec rake generate_secret_token
sudo RAILS_ENV=production bundle exec rake db:migrate
``` 

5. 올바른 사용 권한을 설정합니다.

다음 Chown 명령을 실행하여 올바른 사용 권한을 설정합니다.

``` shell
$ sudo chown -R www-data: /opt/redmine
``` 

### Apache 구성

1. Passenger 설정 파일에 Defaultuser 추가
``` 
# /etc/apache2/mods-available/passenger.conf

<IfModule mod_passenger.c>
  PassengerDefaultUser www-data # user 설정
</IfModule>
``` 

2. Redmine을 웹 문서 공간에 연결하는 심볼릭 링크 생성
``` shell
$ sudo ln -s /opt/redmine/public /var/www/html/redmine
``` 

3. 심볼릭 링크 연결
```
# /etc/apache2/sites-available/000-default.conf 

ServerAdmin webmaster@localhost
DocumentRoot /var/www/html/redmine
PassengerAppRoot /opt/redmine
``` 

4. Apache의 www-data 사용자가 액세스할 수 있도록 Gemfile.lock 파일의 소유권을 생성하고 설정
``` shell
$ sudo touch /opt/redmine/Gemfile.lock
$ sudo chown www-data:www-data /opt/redmine/Gemfile.lock
``` 

5. Apache 재시작
``` shell
$ sudo service apache2 restart
``` 