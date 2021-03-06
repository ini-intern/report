 ## Summer Internship 1st Project

 <h3> wordpress를 이용한 컨텐츠 업로드 + 관리 페이지 제작 </h3>

 <div style="text-align: right"> 작성자 : 김지희 </div>


 #### 목차

##### 1. 프로젝트 개요

##### 2. video list 페이지 구현 정보

##### 3. 구현 결과

##### 4. Issue 발생 및 해결 방안

##### 5. 프로젝트 후기

<br>

-----------

 ### 1. 프로젝트 개요

- 프로젝트 기간
 <br> 2018.07.02 ~ 2018.07.17

- 개발 환경 및 requirements
   - OS : ubutu 16.04
   - php==7.1
   - MySQL==5.7
   - apache2
   - wordpress==4.9.8
   - ffmpeg==3.3.4-2
   - wordpress theme = snaptube
   - installed_plugin = PHP code snippets (Insert PHP), WP Mail SMTP

- 목적
    <Br> backend 로 php를 frontend 로 wordpress를 이용하고 업로드 페이지에서 업로드 된 동영상에 대한 정보를 MySQL database와 서버로부터 받아와서 table 형식으로 확인할 수 있는 페이지를 제작

<br>

###  2.  video list 페이지 구현 정보

#####  기존의 wordpress database에 uploaded_video table 생성
<center><img src="https://i.imgur.com/FfkngQB.png" width=50%/></center>

 video_id 가 primary key 이고 file 업로더인 author 가 wp_users 의 user_login 칼럼을 참조하며 status 에 defualt로 'uploaded' 를 할당


##### PDO 를 이용하여 file upload 시 database에 insert
1. form data로 받아 온 file 정보를 parsing한 후 파일 이름, 파일이 저장되는 위치, 파일 크기, 업로더, 업로드 한 시간, 파일 형식 정보를 새로 생성한 table 에 저장
2. Database 연결 및 실행은 PHP database access object 인 PDO 를 이용하고 stmt 변수에 insert query문을 준비한 뒤 file이 업로드 될 때마다 실행되도록 구현

##### upload 된 video 의 thumbnail 생성
1. thumbnail 생성을 위해 ffmpeg 프로그램 설치
insert query 를 실행힌 후 해당 비디오의 재생 후 5초의 이미지를 썸네일로 만들어 비디오와 같은 위치에 같은 이름으로 확장자만 png 로 변경하여 저장
 <img src = "https://i.imgur.com/TkOPyN7.png" />


##### database 정보를 받아와 table 생성
1. cookies 조회 결과 cookie name 이 wordpress_logged_in으로 시작하는 cookie 의 value를 파싱하여 현재 로그인 되어있는 username을 알아낼 수 있음을 확인

2. username으로 database의 wp_users table에서 user name에 해당하는 ID 쿼리(위의 ERD 참고)

3. wp_uploaded_video 에서 author과 username이 동일한 rows 쿼리

4. php 코드로 table을 생성하기 위해 echo 이용
 ```PHP
 echo '<table>';
   echo '<tr>';
     echo '<td> <strong>thumbnail</strong> </td>';
     echo '<td><strong> title </strong></td>';
     echo '<td><strong> size </strong></td>';
     echo '<td> <strong>uploaded_time</strong> </td>';
     echo '<td> <strong> type </strong></td>';
     echo '<td> <strong>delete </strong></td>';
     echo '</tr>';    
            .
          <생략>
            .
            .
 ```



### 3. 구현 결과

##### MySQL database 출력 결과
화면 에는 현재 접속한 user가 올린 video 중 status가 'delete'가 아닌 것만 출력

<center><img src = "https://i.imgur.com/xR9sfh4.png"> </center>

##### cookie 를 이용 해당 user가 올린 비디오 리스트 출력
<center><img src="https://i.imgur.com/67I9Akj.png" width=70%/></center>
<center><br><h5>user: user1</h5><br></center>
<center><img src="https://i.imgur.com/1su4TM2.png" width=70%/></center>
<center><br><h5>user: jihee <br></h5></center>

### 4. Issue 발생 및 해결 방안

 1. custom page에 snaptube theme 적용하기
  - issue : wordpress에 새로운 page를 추가하기 위해 ``/var/www/html/wp-content/plugins/theme/snaptube/`` 경로에 php 코드를 추가하여 page를 만들면 snaptube theme가 적용되지 않는 issue 발생
  - 해결 방안 : insert_php 플러그인을 설치하여 포스트의 text_area에 직접 코드를 삽입하여 해결


 2. 디렉터리 쓰기 권한 설정
- issue : 파일이 이동해야하는 route가 사용자에게 쓰기 권한이 없을 경우 파일 이동 시 'permission denied' 에러 발생
- 해결 방안 : directory 권한 수정

     ```bash
    foo@bar$ chmod -R 775 upload되는 폴더 Path/
    ```
  file upload는 `755` 로 권한을 수정해도 문제 없지만 이후 삭제 기능 구현을 위해 `775` 로 수정  

 3. 썸네일 생성 시 shell_exec 함수 허용
  - issue : ffmpeg 이용 시 shell 에서 작동하지만 php 파일에서 shell exec 작동이 안되는 에러 발생
  - 해결 방안 : 권한 문제 해결
    ```bash
    foo@bar$ sudo chown www-data:www-data -R 썸네일 만들어지는 폴더 Path/
    foo@bar$ sudo vi /etc/sudoers
    # www-data ALL=NOPASSWD: ALL 주석을 풀어 sudoers 의 마지막 줄에 추가
    # syntax error 주의
    ```

 4. 사용자 화면 수정
  - issue : 리스트에서 사용자가 볼 때 불필요한 정보를 삭제하고 ui 수정
  - 해결 방안 : user name, file path, video 번호 등 사용자에게 불필요한 정보를 삭제하고 file size에 단위 추가

 5. file 업로드 시 file size limit 조정
  - issue : wordpress 에서 media upload filesize limit 이 8M 으로 제한되어 8M 이상의 동영상 업로드 불가
  - 해결 방안 : php.ini 파일에서 post_max_size 수정

     ```
    ; Maximum size of POST data that PHP will accept.
    ; Its value may be 0 to disable the limit. It is ignored if POST data reading
    ; is disabled through enable_post_data_reading.
    ; http://php.net/post-max-size
    post_max_size = 10000M
      ```
     저장 후 apche2 재시작
    ```
      foo@bar$ sudo systemctl restart apache2
    ```
     재시작한 후 wordpress의 dashboard>media>add new 페이지 접속 시

    <center>    <img src = "https://i.imgur.com/p1YMBhR.png" width=80%/> </center>

     다음과 같이 변경된 것 확인 가능
<br>

### 5. 프로젝트 후기
##### 개선할 점
 - 관리페이지의 UI 에서 글씨 크기 확대, 중앙 정렬,  썸네일의 사이즈 등이 개선 필요
 - 프로젝트 일정 관리

##### 느낀점
  처음에 과제를 받았을 때 요구사항을 잘못 이해해서 주어진 기간 중 3분의 1을 제대로 이용하지 못해서 아쉬웠다. 또, wordpress나 php 등 한 번도 접해보지 못한 tool 등을 공부해야해 서 프로젝트 초반에 결과에 비해 학습 시간이나 노력이 많이 소요되었다.
  wordpress에 플러그인을 이용해 코드를 직접 심으면 에러 메시지를 확인할 수 없어서 답답해하기도 하고 내 컴퓨터에서 잘 작동하는 기능이 서버 컴퓨터로 옮기면 또 다른 에러가 나는 등 예상보다 많은 시간이 걸려 결국 기한 내에 마무리하지 못했지만 이를 통해 일정 관리와 자습이 얼마나 중요한 지 깨달았다.(버퍼를 정하고 진행해야 한다)
  또한, Database 를 설계부터 진행해보며 학교에서 배운 내용을 실전에서 적용해보고 새로운 언어나 tool 을 이용해야하는 상황이 왔을 때 어떤 식으로 접근해야 효율적으로 작업할 수 있는지 배울 수 있었던 프로젝트였다.     
