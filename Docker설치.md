# DOCKER

### Docker 설치

#### - HTTP 통신에 사용되는 패키지 및 공개키 설치

`~# apt-get update`

`~# apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common`

`~# apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D`

#### - Docker Repository 추가

`~# vi /etc/apt/sources.list`

파일 열어서 맨 밑에 `deb https://apt.dockerproject.org/repo ubuntu-xenial main`  추가

`~# apt-get update`

#### - linux-image extra 및 docker-engine 패키지 설치

`~# apt-get install linux-image-extra-$(uname -r)`

`~# apt-get install docker-engine`

#### - Docker version 확인

`/etc/apt# docker version` 

 #### - 작업 Directory 생성 및 main.go 파일 작성

`~# mkdir docker`

`~# cd docker`

`~/docker# vi main.go`

<main.go>

```go
package main 
import (
			"fmt"
			"log"
			"net/http"
)
func main(){
		http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request){
				log.Println("received request")
				fmt.Fprintf(w, "Hello Docker!")
		})
		log.Println("start server")
		server := &http.Server{Addr: ":8080"}
		if err := server.ListenAndServe(); err != nil{
				log.Println(err)
		}		
}
```

#### -main.go Test

`~/docker# apt install golang-go`

`~/docker# go run main.go`

=> 서버 실행됨.

#### -Dockerfile 작성

`~/docker# vi Dockerfile`

```
FROM golang:1.9

RUN mkdir /echo

COPY main.go /echo

CMD [ "go", "run", "/echo/main.go" ] 
```

#### - Docker image build하기

docker image build -t ~~imagename:대그명~~ Dockfile_path

`~/docker# docker image build -t example/echo:latest .` 

=> Dockerfile이 실행되면서 

1) 베이스이미지인 golang을 pull해오고

2) echo디렉토리를 생성하고

3) main.go 를 /echo에 복사한 후

4) main.go를 실행한다.