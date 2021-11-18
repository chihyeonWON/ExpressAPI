# 익스프레스로 API 서버 생성

## 실습 프로젝트 구성

이번 프로젝트는 앞선 MongoDB 프로젝트 구조에서 웹 서버를 구현하기 위해 express 패키지를 추가로 사용한다.
이번 절에서 추가로 설치해야 할 패키지는 다음과 같다.
```typescript
npm i -S express body-parser cors
npm i -D @types/express @types/body-parser @types/cors

이번 절에서 사용하는 프로젝트의 package.json 파일로서, script 항목에 start 명령을 추가하였다.
```typescript
{
  "name": "expressapi",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "ts-node src"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "body-parser": "^1.19.0",
    "chance": "^1.1.8",
    "cors": "^2.8.5",
    "express": "^4.17.1",
    "mkdirp": "^1.0.4",
    "mongodb": "^4.1.4",
    "rimraf": "^3.0.2"
  },
  "devDependencies": {
    "@types/body-parser": "^1.19.2",
    "@types/chance": "^1.1.3",
    "@types/cors": "^2.8.12",
    "@types/express": "^4.17.13",
    "@types/mkdirp": "^1.0.2",
    "@types/mongodb": "^4.0.7",
    "@types/node": "^16.11.7",
    "@types/rimraf": "^3.0.2",
    "ts-node": "^10.4.0",
    "typescript": "^4.4.4"
  }
}
```

마지막으로 mongodb 프로젝트의 src 디렉토리에 있는 코드를 복사하면 준비가 끝난다.

## REST 방식 서버

REST(Representational State Transfer)는 HTTP 프로토콜의 주요 저자인 로이 필딩(Roy Fielding)의 2000년 박사 학위 논문에서 소개되었으며 1990년 말에 표준이 되었다.
REST는 웹 서버 소프트웨어 구조의 한 형식이다. REST 서버는 HTTP 프로토콜의 GET, POST, PUT, DELETE와 같은 메서드를 사용해 웹 클라이언트와 통신하는 서버이다.

## 익스프레스 프레임워크

노드제이에스 환경에서 API 서버는 대부분 익스프레스(express) 프레임워크를 사용한다.
익스프레스 프레임워크를 사용하면 다음 코드처럼 웹 서버를 쉽게 만들 수 있다.
```typescript
import express from 'express'
const app = express(), port = 4000

app
  .get('/', (req, res) => res.json({message: 'Hello world!'}))
  .listen(port, () => console.log('http://localhost:${port} started... '))
```

앞서 package.json 파일에 start 명령을 정의했으므로 터미널에서 간단하게 다음 명령으로 웹 서버를 동작시킬 수 있다.
```typescript
npm start
```

웹 서버가 제대로 동작하는지 확인하고자 웹 브라우저를 열고 http://localhost:4000 주소로 접속한다.
앞 코드에서 05행이 실행되어 다음 화면처럼 JSON 포맷 데이터가 출력된다.
![4000](https://user-images.githubusercontent.com/58906858/142161596-044ba973-9d24-480e-bc8a-89a2f98d9543.png)

## 라우팅 기능 구현

웹 브라우저의 주소 창에서 http://localhost:4000/users/1/2 와 같은 경로로 접속했을 대 이를 처리하려면 코드를 다음과 같은 구조로 작성한다.
```typescript
app.get('/users/:skip/:limit', 라우터 콜백 함수)
```

skip과 limit 앞에 콜론(:)은 다음 코드로 경로에서 1과 2와 같은 값을 얻기 위해 사용한다.
```typescript
const {skip, limit} = req.params
```

index.ts 파일의 내용을 다음처럼 수정한다.

src/index.ts
```typescript
import express from 'express'
const app = express(), port = 4000

app
  .get('/', (req, res) => res.json({message: 'Hello world!'}))
  .get('/users/:skip/:limit', (req, res) => {
    const {skip, limit} = req.params
    res.json({skip, limit})
  })
  .listen(port, () => console.log(`http://localhost:${port} started... `))
```

skip과 limit 실제 값이 다음과 같이 출력된다.
![router](https://user-images.githubusercontent.com/58906858/142371710-acf0e034-e860-4ace-923a-8fccc60e5922.jpg)

## 익스프레스 미들웨어 추가

REST 방식의 API 서버들은 웹 페이지의 본문 내용을 분석하려고 할 때 bodyParser와 cors라는 패키지를 use메서드를 사용해 다음처럼 작성해야 한다.

src/index.ts
```typescript
import bodyParser from 'body-parser'
import cors from 'cors'

app
  .use(bodyParser.urlencoded({extended: true})
  .use(cors())
```

## 몽고DB 연결

몽고DB에 저장된 데이터를 실제로 서비스하려면, 다음 코드에서 보듯 몽고DB 서버에 접속하는 코드를 만들어야한다. 
이 코드에서는 runServer를 구현하는 것이 관건이다.
```typescript
import {connect} from './mongodb/connect'
import {runServer} from './runServer'

connect()
  .then( async(connection) => {
    const db = await connection.db('mongodb')
    return db
  })
  .then(runServer)
  .catch((e: Error) => console.log(e.message))
```

앞 코드에서 runServer 함수는 다음처럼 구현한다.

src/runServer.ts
```typescript
import express from 'express'
import bodyParser from 'body-parser'
import cors from 'cors'

export const runServer = (mongodb) => {
    const app = express(), port = 4000

    app
        .use(bodyParser.urlencoded({extended: true}))
        .use(cors())
        .get('/', (req, res) => res.json({message: 'Hello world!'}))
        .get('/users/:skip/:limit', async (req, res) => {
            const {skip, limit} = req.params
            
            const usersCollection = await mongodb.collection('users')
            const cursor = await usersCollection
                .find({})
                .sort({name: 1})
                .skip(parseInt(skip))
                .limit(parseInt(limit))
            const result = await cursor.toArray()
            res.json(result)
        })
        .listen(port, () => console.log(`http://localhost:${port} started ...`))
}
```

이 서버가 이제 API 서버로 동작하고 있음을 알 수 있다.

이 프로젝트는 리액트와 부트스트랩으로 프런트엔드 웹 프로젝트로 이어진다. 
