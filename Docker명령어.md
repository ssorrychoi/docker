

### Docker container run 명령

-d : background에서 돌게 하기 위한 옵션

-p : port번호를 지정하기 위한 옵션

-it : 해당 컨테이너와 interaction하기 위한 옵션

 ls -a : 중지돼 있는 컨테이너까지 볼 수 있는 명령어

-q : Container ID만 가져오기

#### - Docker container run 명령

`~# docker container run -p 8888:8080 -d example/echo:latest`

`~# curl http://localhost:8888`    =>호스트에서 컨테이너의 서비스를 실행

#### - Docker attach 명령

`~# docker attach echo`

#### - Docker container stop 명령

~# docker stop ~~containerID~~

#### - Docker Tag

동일한 이미지명으로 빌드할 경우 이전 태그가 사라지기 때문에 ((<none>으로 표기됨.))

태그를 입력해 주어야 한다.

`~# docker image tag example/echo:latest ssorrychoi/hello:latest` 

### Docker status 

- 실행
- 명령
- 삭제





### Docker container / image / 싹다지우기

`~# docker container rm -f $(docker container ls -q)` 

`~# docker image rm $(docker image ls -q)`

`~# docker system prune`



### container안에서의 명령어 실행

#### exec 명령어 사용

`~/docker # docker container exec ~~컨테이너 ID~~ ls`

`~/docker # docker container cp ./test ~~컨테이너 ID~~:/`



### Docker Compose

docker-compose.yaml 파일을 작성하는데 주의할 점은 <Tab>키를 써서는 안된다. space키만 사용 가능.

--scale ~~이미지name~~=n 이미지를 n개만큼 한꺼번에 생성

`docker-compose up --scale echo=10`



### Docker Image 생성

##### commit 명령어 : container에서 image 만들기

`~# docker container commit --author "ssorrychoi" --message  "my first docker image" ~~컨테이너name~~ dockerhubID/imagename:latest`

##### inspect 명령어 : 세부정보 알아보기

`~# docker image inspect dockerhubID/imagename:latest`



### Container Volume 

####  - 공유 디렉토리 만들기

`~# docker container run -d -v /root/test:/usr/share/nginx/html -p 8888:80 nginx:latest`



### Data volume container

데이터 저장을 폭표로 하는 Container다.

**busybox**는 base image로 많이 쓰인다. 최소한의 경량 os만 제공한다.

