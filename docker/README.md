
# 도커 이미지 생성
### Dockerfile 작성

    FROM <BaseImage:가령 java, python, node 등>
    WORKDIR /app
    EXPOSE <Port 번호>
    COPY <실행시킬 파일> .
    RUN <라이브러리 다운로드>
    CMD <실행 커맨드>

### Dockerfile을 통한 image 생성

    docker build -t najh0528/<이미지명> <도커파일명>

### DockerHub 업로드

    docker login
    docker push najh0528/<이미지명>