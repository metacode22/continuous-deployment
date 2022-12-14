## AWS S3와 Github Actions를 이용해 자동으로 배포되는 환경 구축해보기

<div>AWS S3를 이용해 정적 웹 사이트 호스팅</div>
<div>Github Actions를 이용해 CD 구축</div>

</br>

### 작성한 yml 파일
```yml
name: deploy

on:
  push:
    branches:
    - main

jobs:
  deploy:
    # 우분투 가장 최신 버전에서 돌아간다.
    runs-on: ubuntu-latest
    steps:
    # 이 밑에 - <이름> 들은 모두 하나하나의 step이다.
    # main으로 checkout
    - uses: actions/checkout@master
    # npm clean install
    - run: npm install
    - run: npm run build
    # 이제부터 하나의 step이 deploy to s3 bucket이라는 이름으로 실행될 거다.
    - name: deploy to s3 bucket
      # 실질적으로는 누가 만들어놓은 이걸 사용하겠다.
      uses: jakejarvis/s3-sync-action@master
      # 이런 인자를 넣을 것이다.
      # S3의 버켓을 비워야 하니 delete라는 옵션을 준다.
      with:
        args: --delete
      env:
        # 밑의 3개는 환경 변수로 노출되지 않아야 하는 것들이다.
        # 이는 Github Repository의 Settings - Secrets - Actions에서 설정한다.
        # S3 버킷 이름
        AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}
        # aws에도 내가 접근한다는 것을 알려야 한다. 아이디와 비밀번호라고 보면 된다.
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        # 밑의 2개는 노출되어도 상관없다.
        # region은 서울
        AWS_REGION: 'ap-northeast-2'
        # 소스 폴더는 build이다.
        SOURCE_DIR: 'build'
```