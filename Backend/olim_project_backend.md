## Node.js로 백엔드 서버 만들기

- next에서 프론트 서버를 돌릴 때 노드를 사용함, 백엔드 서버를 노드로 만들 수 있다.

- 노드는 자바스크립트 런타임. (서버로 사용가능한 것이지 서버만 실행할 수 있는 것 x )

예시

```js
const http = require('http');
const server = http.createServer((req, res) => {
  console.log(req.url, req.method);
    
  res.write('hello my first node')
  res.end('Hello node');
});
server.listen(3065);
```

```
$ node app.js
```

localhost:3065 접속 시, hello my first node와 Hello node가 출력되어 있다.



- 노드에서는 import/export를 사용하지 않고, **require/module.export를 사용해 모듈을 관리**한다.

- node runtime이 이 코드를 실행시켜서 **노드에서 제공하는 http 모듈이 작동해 서버 역할**을 해줌. (노드가 서버인 것이 아닌, 노드가 제공해주는 http 모듈이 서버를 제공해주는 것이다.)

- 데이터가 여러개인 경우, 한번 요청에 여러 데이터를 묶어서 한번에 응답을 줄 수도 있고, 여러번 요청을 나눠 보내고 그에 맞게 데이터를 응답해주는 방법이 있다.

  **요청과 응답은 1:1**로 이뤄져야 한다.



## Express로 라우팅하기

- **라우팅을 보다 편리하게 하기 위해서 Express 사용**

  ```
  $ yarn add express
  ```

  

  ```js
  // Express 없이 라우팅
  const http = require('http');
  const server = http.createServer((req, res) => {
    console.log(req.url, req.method);
      if(req.method === 'GET') {
          if(req.url === '/post') {
              ...
          }
      } else if(req.method === 'POST'){
          ...
      }
      ...
    res.write('hello my first node');
    res.end('Hello node');
  });
  server.listen(3065);
  ```

  **라우팅: URL(URI) 요청에 따라 어플리케이션이 응답하는 방법을 결정하는 것**

  

- Express도 내부적으로 http를 사용하기 때문에, 서버를 돌릴 수 있는 것이다.

  ```js
  // Express로 편리하게 라우팅
  const express = require('express');
  
  const app = express();
  
  app.get('/', (req, res) => {
    res.send('hello express');
  });
  
  app.get('/api', (req, res) => {	
    res.send('hello express2222');
  });
  
  app.listen(3065, () => {
    console.log('서버 실행 중');
  });
  ```

  

- REST API

  (주소창에 치는 것은 GET)

  ```
  app.get -> 가져오기
  app.post -> 생성하기
  app.put -> 전체 수정
  app.delete -> 제거
  app.patch -> 부분 수정
  app.options -> 찔러보기(확인용?)
  app.head -> 헤더만 가져오기(원래 헤더/바디 다 가져옴)
  ```



- Express 라우터 분리

  ```js
  // routes/post.js
  
  const express = require('express');
  
  const router = express.Router();
  
  router.post('/', (req, res) => {
    res.json({ id: 1, content: 'test' });
  });
  
  router.delete('/', (req, res) => {
    res.json({ id: 1 });
  });
  
  module.exports = router;
  ```

  ```js
  // app.js
  
  const express = require('express');
  
  const postRouter = require('./routes/post');
  
  const app = express();
  
  app.use('/post', postRouter);
  
  app.listen(3065, () => {
    console.log('서버 실행 중');
  });
  ```

  



## MySQL과 Node.js 연결

MySQL과 MySQL workbench 설치 후 진행

```
$ yarn add sequelize sequelize-cli mysql2
$ yarn add -D ts-node @types/express @types/express-session @types/node
$ npx sequelize init
```

- `mysql2`: node와 mysql을 연결해주는 드라이버 (sequelize가 내부적으로 mysql2 사용함)

- `sequelize`: 자바스크립트로 sql 조작할 수 있게 해주는 라이브러리

- `npx sequelize init`: sequelize 셋팅 (config, models, seeders, migrations 디렉토리 생김)

  

- 생성된 config/config.ts에서 mysql 연결 정보를 입력한다.

  (비밀번호는 보안을 위해 dotenv 라이브러리를 이용한다. .env 파일안에 넣어주고 .gitignore에 .env 파일을 추가해 git에 올라가는 것을 방지할 수 있다.)

  ```js
  // config/config.ts
  
  import dotenv from 'dotenv';
  
  dotenv.config();
  
  type ConfigType = {
    database: string;
    username: string;
    password: string;
    dialect: 'mysql';
    host: string;
  };
  
  interface ConfigGroup {
    development: ConfigType;
    test: ConfigType;
    production: ConfigType;
  }
  
  const config: ConfigGroup = {
    development: {
      database: 'olim',
      username: 'root',
      password: process.env.DB_PASSWORD!,
      dialect: 'mysql',
      host: '127.0.0.1',
    },
    test: {
      database: 'olim',
      username: 'root',
      password: process.env.DB_PASSWORD!,
      dialect: 'mysql',
      host: '127.0.0.1',
    },
    production: {
      database: 'olim',
      username: 'root',
      password: process.env.DB_PASSWORD!,
      dialect: 'mysql',
      host: '127.0.0.1',
    },
  };
  
  export default config;
  
  ```

  development, test, production 모드에 따라 다른 db를 사용할 수 있음

  

- models/index.js에서 sequelize 객체 생성.

  ```js
  // models/index.ts (node.js와 MySQL 연결만)
  
  import { Sequelize } from 'sequelize';
  import config from '../config/config';   // config에서 env 부분 불러오기
  
  type EnvType = 'production' | 'test' | 'development';
  
  const env = (process.env.NODE_ENV as EnvType) || 'development';
  const db = {};
  
  // sequelize가 node와 mysql 연결, sequelize에 연결정보 담겨있음
  const sequelize = new Sequelize(
    config[env].database,
    config[env].username,
    config[env].password,
    config[env]
  );
  
  // 아직 db안에 테이블이 존재하지 않음
  
  db.sequelize = sequelize;	// 연결정보를 db에 넣기
  
  module.exports = db;
  ```

  이렇게 mysql과 node.js의 연결을 성공했으면 mysql의 테이블을 생성해서 db에 추가해 주어야한다. 이는 sequelize를 이용해서 만들 수 있다. 

  

- MySQL에서 테이블을 만드는 것이 아닌, Sequelize를 이용해서 MySQL의 테이블을 만들 수 있다. 

  ```js
  // models/user.js (users 테이블 만들기)
  
  import { Model, DataTypes, Sequelize } from 'sequelize';
  
  interface UsersAttributes {
    loginId: string;
    password: string;
    userName: string;
    name: string;
  }
  
  class User extends Model<UsersAttributes> {
    public readonly id!: number;
    public loginId!: string;
    public password!: string;
    public userName!: string;
    public name!: string;
  
    public readonly createdAt!: Date;
    public readonly updatedAt!: Date;
  
    static initModel(sequelize: Sequelize) {
      return User.init(
        {
          loginId: {
            type: DataTypes.STRING(30),
            allowNull: false,
            unique: true,
          },
          password: {
            type: DataTypes.STRING(100),
            allowNull: false,
          },
          userName: {
            type: DataTypes.STRING(30),
            allowNull: false,
            unique: true,
          },
          name: {
            type: DataTypes.STRING(30),
            allowNull: false,
          },
        },
        {
          modelName: 'User',
          tableName: 'users',
          charset: 'utf8',
          collate: 'utf8_general_ci', // 한글 저장
          sequelize,
        }
      );
    }
  
    static associate(db: any) {
      db.User.hasMany(db.Post);
      db.User.hasMany(db.Comment);
      db.User.belongsToMany(db.Post, { through: 'Like', as: 'Liked' });
      db.User.belongsToMany(db.User, {
        through: 'Follow',
        as: 'Followers',
        foreignKey: 'FollowingId',
      });
      db.User.belongsToMany(db.User, {
        through: 'Follow',
        as: 'Followings',
        foreignKey: 'FollowerId',
      });
    }
  }
  
  export default User;
  
  ```
  
  - type 종류: STRING, TEXT, BOOLEAN, INTEGER, FLOAT, DATETIME 등
  - 관계형 테이블에 대하여
    - `through`: 테이블명, `as`: 별칭(in JS), `foreignKey`: 컬럼명 설정 (동일한 테이블 내에서 n:n 발생할 경우 사용)
    - `hasMany`: 내 기준 1:n
    - `belongsTo`: 내 기준 n:1
    - `belongsToMany`: 내 기준 n:n



- models/index.js에서 위에서 만든 테이블들로 db를 구성한다.

  ```js
  // models/index.ts
  
  import { Sequelize } from 'sequelize';
  
  import Comment from './comment';
  import Hashtag from './hashtag';
  import Image from './image';
  import Post from './post';
  import User from './user';
  import config from '../config/config';
  
  type EnvType = 'production' | 'test' | 'development';
  type DBType = {
    Comment: typeof Comment;
    Hashtag: typeof Hashtag;
    Image: typeof Image;
    Post: typeof Post;
    User: typeof User;
    [key: string]: any;
  };
  
  const env = (process.env.NODE_ENV as EnvType) || 'development';
  
  const sequelize = new Sequelize(
    config[env].database,
    config[env].username,
    config[env].password,
    config[env]
  );
  
  const db: DBType = { Comment, Hashtag, Image, Post, User };
  
  Object.keys(db).forEach((model) => {
    db[model].initModel(sequelize);
  });
  
  Object.keys(db).forEach((model) => {
    if (db[model].associate) {
      db[model].associate(db);
    }
  });
  
  db.sequelize = sequelize;
  
  export default db;
  ```

  

- Express에서 sequelize를 등록해서 서버 실행될 때 db 연결도 같이 진행되도록 만든다.

  ```js
  // app.ts
  
  import express from 'express';
  
  import userRouter from './routes/user';
  import db from './database/models/index';
  
  const app = express();
  
  // db 연결
  db.sequelize
    .sync()
    .then(() => {
      console.log('db연결 성공');
    })
    .catch(console.error);
  
  app.use('/user', userRouter);
  
  app.listen(3065, () => {
    console.log('서버 실행 중');
  });
  ```

  

- 위 코드를 완성한 후에, MySQL에 데이터베이스를 생성해준다.

  ```
  $ npx sequelize db:create
  ```

 

- 매번 코드를 수정할 때마다 node app을 재실행 시키기 귀찮으므로 nodemon 라이브러리를 이용

```
$ yarn add nodemon -D

// package.json에서 아래와 같이 변경 후, yarn dev 실행
  "scripts": {
    "dev": "nodemon --exec ./node_modules/.bin/ts-node ./src/app.ts"
  },
```



## Backend 와 Frontend 연결

우선 frontend에서 api 요청을 보낸다. (예를 들어 saga or toolkit의 action)

front에서 넘겨받은 데이터를 사용하기 위해서는 back/src/app.ts에서 express관련 설정을 추가해야 한다. (다른 설정보다 위에)

```ts
app.use(express.json());	// json 형식으로 데이터 받으면 req.body에 넣어줌
app.use(express.urlencoded({ extended: true }));	// form했을 때 데이터 처리??
```

그리고 위에서 간단한게 만들었던 routes 디렉토리 안의 파일에서 요청 api에 대한 처리를 진행해주면 된다.

```ts
import express from 'express';

import User from '../database/models/user';	// 테이블 불러오기

const router = express.Router();

router.post('/signup', async(req, res, next) => {
  try {
    await User.create({
        name: req.body.name,
        userId: req.body.userId,
        password: req.body.password,	// 비밀번호 그냥 저장!!!
        userName: req.body.userName,
    })
    res.status(201).send('ok');
  } catch (error) {
    console.log(error);
    next(error);    // Express가 에러 처리 (status 500)
  }
});

router.delete('/', (req, res) => {
  res.json({ id: 1 });
});

export default router;

```

요청/응답은 헤더(상태, 용량, 시간, 쿠키)와 바디(데이터)로 구성되어 있다.



### 문제 발견) 비밀번호 그대로 저장

bcrypt라는 암호화 라이브러리 이용

```
$ yarn add bcrypt
```

```ts
import bcrypt from 'bcrypt';

const hashedPasswrod = await bcrypt.hash(req.body.password, 12);	// 12는 보안 강도
```



## CORS

CORS는 브라우저(3060)에서 다른 도메인 서버(3065)에 요청을 보내게 되면 **브라우저에서 차단**을 한다. Response의 Headers에 '**Access-Control-Allow-Origin**'을 넘겨주지 않는 방식을 이용해 차단한다. Access-Control-Allow-Origin를 backend 서버에서 추가하거나 요청 방식을 서버끼리는 문제가 생기지 않으므로, (브라우저->백엔드)를 (브라우저->프론트->백엔드)로 보내서 해결한다. 이를 **프록시(Proxy)**라고 함.

추가하는 함수를 구현하거나, cors 라이브러리를 이용해서 해결

```js
// app.js

app.use(
  cors({
    origin: true, // 나중에는 진짜 주소 넣어주기
    credentials: false, // 나중에 true로 해주기(쿠키, false가 default)
  })
```

Method가 OPTIONS인 request가 바로 Access-Control-Allow-Origin이 존재하는 확인하는 것.



## 로그인 passport로 구현

**클라이언트가 서버에 요청할 자격이 있는지 인증할 때에 passport 미들웨어를 사용**

```
$ yarn add passport passport-local
```

`passport-local`: 이메일/비밀번호 or 아이디/비밀번호로 로그인을 할 수 있도록 도와줌



- app.ts에 아래 코드를 추가한다.

```ts
app.use(passport.initialize());
app.use(passport.session());
```

`passport.initialize()` 메서드는 서버의 요청(req 객체)에 passport 폴더의 index.ts에 작성한 설정들을 입력한다. passport를 미들웨어로 사용하겠다는 선언의 의미로 보면된다.

`passport.session()` 메서드는 `req.session`에 passport의 정보들을 저장한다. `req.session`은 `express-session`을 통해 생성되니, `express-session` 미들웨어 아래에 위치해야 한다.



- passport 디렉토리안에 index.ts(passport의 설정)와 local.ts(전략) 파일 생성

- 우선 local.ts에서 로그인 전략을 세운다.

```ts
// back/src/passport/local.ts

import passport from 'passport';
import { Strategy as LocalStrategy } from 'passport-local';
import bcrypt from 'bcrypt';
import User from '../database/models/user';

export default () => {
  passport.use(
    new LocalStrategy(
      // req.body에 대한 설정
      {
        usernameField: 'userName', //  req.body.userName
        passwordField: 'password', //  req.body.password
      },
      // 로그인 전략
      async (userId, password, done) => {
        try {
          // 아이디 존재 여부 확인
          const user = await User.findOne({	
            where: { userId },
          });
          if (!user) {
            // done(error 여부, 결과값, 실패정보)
            return done(null, false, { message: '존재하지 않는 아이디입니다.' });
          }
			
          // 비밀번호 확인
          const result = await bcrypt.compare(password, user.password);
          if (result) {
            return done(null, user);	// 성공, 유저 정보 전부 포함
          }
          return done(null, false, { message: '비밀번호가 틀렸습니다.' });
        } catch (error) {
          // 서버 에러
          console.error(error);
          return done(error);
        }
      }
    )
  );
};
```

passport.use의 첫번째 인자에서는 LocalStrategy의 usernameField와 passwordField의 value 값들에 각각 req.body의 속성명을 담아줘야 한다.

passport.use의 두번째 인자에서는 실제 전략을 수행하는 로직을 작성합니다. 유저가 존재하는지, 비밀번호는 일치하는지 등의 로직들을 작성할 수 있다.



- 그리고 위에서 세운 전략을 routes에서 불러와서 사용할 수 있다.

```ts
// back/src/routes/user.ts
import passport from 'passport'
...

router.post('/login', passport.authenticate('local', (isError, user, errInfo) => {
    // 오류 처리
    if (isError) {
      console.error(isError);
      return next(isError);
    }
    if (errInfo) {
      return res.status(401).send(errInfo.message);
    }

    // passport 로그인
    return req.login(user, async (loginErr) => {  // 알아서 쿠키 보내주고 세션 연결 
      if (loginErr) {
        // passport 로그인 오류
        console.error(loginErr);
        return next(loginErr);
      }
      // res.setHeader('Cookie', 'asdfasdf')	// 쿠키 (코드 추가 안해도 자동으로 생김) 
      // 로그인 성공
      const allUserData = await User.findOne({
        where: { id: user.id },
        attributes: { exclude: ['password'] },
        include: [
          {
            model: Post,
            attributes: ['id'],
          },
          {
            model: User,
            as: 'Followings',
            attributes: ['id'],
          },
          {
            model: User,
            as: 'Followers',
            attributes: ['id'],
          },
        ],
      });
      return res.status(200).json(allUserData);
    });
}));

export default router;
```

`passport.authenticate`의 첫번째 인자는 passport 미들웨어의 전략들 중 local을 사용하겠다는 것이고, 두번째 인자는 콜백함수인데 local 전략에서 `done` 함수의 결과값을 차례로 받아온 것이다.

하지만 위처럼 만들면 사용자가 입력한 데이터를 받아오거나 출력해주고, 에러 처리를 해주는 (req, res, next)를 사용할 수 없게 된다. 원래 `passport.authenticate()`는 원래 (req, res, next)를 사용할 수 없는 미들웨어이지만, 미들웨어 확장 기능을 이용하여 문제를 해결해줄 수 있다.

```ts
// 로그인
router.post('/login', async (req, res, next) => {
  passport.authenticate('local', (isError, user, errInfo) => {
	...
  })(req, res, next)
});
```



이제  passport/index.ts 설정을 해보자. `passport.authenticate('local', ... )`이 실행되면 passport/local.ts에서 만들어놓은 로그인 전략이 실행되고, 로그인 성공 시에 `req.login(user, ...)`이 실행되면서 passport/index.ts에 있는 세션과 관련된 전략들이 실행된다. 

```js
import passport from 'passport';
import local from './local';
import User from '../database/models/user';

export default () => {
  // session에 user.id만 저장
  passport.serializeUser((user, done) => {	// 현재 user에는 모든 정보 포함
    done(null, user.id); 					// 
  });

  // session에 저장된 id로 User 정보 불러오기
  passport.deserializeUser(async (id, done) => {	// id: 세션쿠키를 통해 메모리에 저장된 id
    try {
      const user = await User.findOne({ where: { id } });
      done(null, user); // (서버 에러, 성공), req.user에 넣어줌
    } catch (error) {
      console.error(error);
      done(error);
    }
  });

  local();
};
```

serialize는 직렬화, deserialize는 역직렬화

직렬화라는 것은 어떤 데이터를 다른 곳에서 사용할 수 있게 다른 포맷의 데이터로 바꾸는 것을 의미(선택한 user.id값을 직렬화를 통해 세션에 저장될 수 있는 포맷으로 바꾼 뒤 req.session(세션) 객체에 저장), 로그인시에 딱 한번 실행 

역직렬화는 다른 포맷의 데이터로 바뀐 데이터를 원래 포맷으로 복구 (req.session(세션)에 저장된 id를 통해 메모리에 저장된 id를 찾아서 가져오고, 해당 id와 연결된 user.id를 이용해 user 정보 불러와 반환), 조회된 결과값은 req.user에 담김, 페이지를 이동할때마다 (passport.session()이 실행될때마다) 호출

req.login시에 serializeUser가 호출, done이 호출(req.session 객체에 저장)될 때 res.setHeader가 쿠키 저장 



> #### 로그인 시 쿠키와 세션
>
> 로그인 할때마다 백엔드 서버에서 개인의 모든 정보(id, 비번,게시글, 댓글, 좋아요, 팔로우, 팔로워 등)를 브라우저로 보내면 보안에 취약해진다. 그래서 개인정보 대신에 랜덤한 토큰(**쿠키**)를 브라우저에 보내놓고, 서버 쪽에서 해당 토큰이 어떤 저장된 데이터들(**세션**)과 연결되어있는지 저장해놓는다. 어떤 데이터를 서버에 요청하거나 보낼 때마다 쿠키를 백엔드 서버에 함께 보내 개인정보를 인식할 수 있게 만드는 것이다. 
>
> 백엔드에서도 항상 모든 데이터들을 다 들고 있으면 너무 무거워지므로, 로그인 쿠키와 user.id 정보만 매칭해놓고, 후에 필요할때 user.id로 DB에서 데이터를 불러와 사용한다. 



- 쿠키와 세션 설정

```
$ yarn add express-session cookie-parser
```

back/src/app.ts에서 쿠키와 세션에 대한 미들웨어를 설정해준다.

```ts
// back/src/app.ts
...
app.use(cookieParser(process.env.COOKIE_SECRET)); // cookie & session
app.use(
  session({								// req.session 생성
    saveUninitialized: false,
    resave: false,
    secret: process.env.COOKIE_SECRET!,
    cookie: {
      httpOnly: true,
      secure: false, // https이면 true
      // domain: process.env.NODE_ENV === 'production' && '.주소.com'
    },
  })
);

app.use(passport.initialize())
app.use(passport.session())
...
```



#### 미들웨어로 라우터 검사

사용자가 라우터를 가지고 문제를 발생시킬 수 있으므로(로그인 안했는데 로그아웃 요청 보내기 등), 미들웨어로 라우터를 검사해야한다.

미들웨어 파일을 routes 디렉토리에 생성하고, `req.isAuthenticated()` 로 로그인 여부 검증하는 코드를 만든 다. 그 다음 다른 파일들(user)의 router 코드에 해당 미들웨어를 추가해주어 오류를 방지한다.



#### 미들웨어

`app.use()` 내부 코드와 `app.get/post/...`의 (`req`,`res`,`next`)를 활용한 콜백함수 모두 미들웨어다.

- **`next` 사용법**

  - `next`안에 값이 들어있으면 error 처리

  - `next`안에 값이 없으면 다음 미들웨어 진행

- `next`로 보내준 **error처리 미들웨어**는 코드 `app.listen()` 상단에 내부적으로 존재함(직접 만들 수 있음)

- 요청이 오면 app.js 위에서 아래로 실행되다가 일치하는 라우터 발견 시 실행

  (ex. /user에 대한 요청 들어오면 userRouter가 실행된다.)



## Routes 동적 주소 할당

```js
// POST /post/{data.postId}/comment
router.post('/:postId/comment', isLoggedIn, async (req, res, next) => {
  try {
    const comment = await Comment.create({
      content: req.body.content,
      PostId: req.params.postId,
      UserId: req.user.id,
    });
    res.status(201).json(comment);
  } catch (error) {
    console.error(error);
    next(error);
  }
});
```



#### GET에 유동적인 주소 만들기

get 방식으로 axios 데이터를 불러오면, axios.get()의 두번째 인자에 crudentials이 들어가므로 data를 주소에 못넣어준다.

주소를 넣어주고 싶다면, 쿼리문을 이용하면 된다.

```js
function loadPostsAPI(lastId) {
  return axios.get(`/posts?lastId=${lastId}`);
}
```

get은 데이터 캐싱이 가능하다 >>??



## 쿠키 전달하기

도메인이 다른 경우 CORS가 발생하며, 쿠키도 전달되지 않는다.

방법1: Proxy, **방법2: CORS 모듈 활용**

```js
// backend/app.js

app.use(
  cors({
    origin: true,		// 나중에는 진짜 주소 넣어주기
    credentials: true,  // 쿠키 전달
  })
```

```js
// frontend/sagas/post.js
// isLoggedIn, isNotLoggedIn을 사용하는 routes들을 포함한 axios에 모두 적용

function addCommentAPI(data) {
  return axios.post(`/post/${data.postId}/comment`, data, {
    withCredentials: true,	// 쿠키 전달
  });
}
```

front의 axios에서 `withCredentials: true`, back의 cors에서 `credentials: true`를 넣어주면 서로 쿠키를 주고받을 수 있다.

credentials를 `true`로 해주면, `access-control-allow-credentials`가 `true`가 된다.

매번 다 넣어주기 힘드니, sagas/index.js에서 default로 적용시킬 수 있다.

```js
// sagas/index.js

axios.defaults.baseURL = 'http://localhost:3065';
axios.defaults.withCredentials = true;
```

추가로 `credentails: true`이면, 쿠키라는 민감한 정보를 서로 주고받으므로, cors의 origin의 값을 `"*"` 으로 주게 되면 위험해지므로 정확한 주소를 전달해야한다. (정확한 주소가 없을 시, true로 주어도 해결된다.)



#### db에서 원하는 데이터 불러오기

예를 들어 로그인을 하는 경우, 아이디와 비밀번호를 backend에 전달하게 된다. backend에서는 받은 데이터를 이용해 검증을 하고 로그인 처리를 하는데, 로그인이 성공되면 팔로잉, 팔로우, 나의 게시글 등 개인 데이터를 받아와야 한다. 물론 로그인의 경우 매번 모든 데이터를 주고받게 되면 데이터 용량이 너무 커지므로, passport에서 serializerUser와 deserializerUser 를 이용해 쿠키값과 아이디만 전달한다. 어찌됐든, 그 이후에 사용할 데이터들을 가져와야 하는데, 비밀번호는 가져오지 않고, 아까 위에서 말한 데이터를 가지고 오고 싶다.

그럴 경우, 테이블에 접근해서 데이터를 가져오는 쿼리문을 날리면 된다.

```js
// back/routes/user.js

...
router.post('/login', isNotLoggedIn, (req, res, next) => {
  passport.authenticate('local', (servErr, user, cliErr) => {
    if (servErr) {
      ...
    }
    if (cliErr) {
      ...
    }
    return req.login(user, async (loginErr) => {
      if (loginErr) {
        ...
      }

      const fullUserWithoutPassword = await User.findOne({
        where: { id: user.id },
        attributes: { exclude: ['password'] }, // attributes: ['id', 'nickname', 'email']
        include: [
          {
            model: Post,
          },
          {
            model: User,
            as: 'Followings',
          },
          {
            model: User,
            as: 'Followers',
          },
        ],
      });
      return res.status(200).json(fullUserWithoutPassword);
    });
  })(req, res, next);
});
```



## 쿠키를 서버로 전달해서 로그인 유지하기

개발자 도구 창에서 Application의 Cookies를 들어가보면 connect.sid가 있다.

이것이 passport와 Express Session에서 만들어낸 쿠키이다.

쿠키만 있으면 서버쪽에서 로그인 여부를 알 수 있는데, 쿠키 값이 있음에도 새로고침 시에 로그아웃되는 이유는 쿠키가 서버쪽으로 전달이 되지 않기 때문이다.

애초에 브라우저에 접속해서 서버에서 데이터를 받아올 때 로그인 유지를 하고싶으면 SSR이 필요하다. SSR적용 전에는 CSR 방식으로 데이터 통신이 일어나기 때문에, 처음 CSR로 받아왔을 때 로그아웃 된것처럼 뜨다가 데이터 요청해서 로그인이 유지된다.

사용자 정보 불러오는 api를 작성

```ts
// src/routes/user.ts
router.get('/myinfo', async (req, res, next) => {
  try {
    if (req.user) {
      const allUserData = await User.findOne({
        where: { id: req.user.id },
        attributes: { exclude: ['password'] },
      });
      res.status(200).json(allUserData);
    } else {
      res.status(200).json(null);
    }
  } catch (error) {
    console.error(error);
    return next(error);
  }
});
```

이를 모든 화면에서(로그인,회원가입 등 제외) 불러오도록 만들어주면 된다.



## 요청과 응답 기록하기

morgan 라이브러리를 이용해 프론트에서 백엔드에 어떤 요청을 보냈는지 확인할 수 있다.



## 고차함수 사용

반복문에 컴포넌트가 들어가고, 그 안에 onClick과 같은 이벤트가 있는데, 각 이벤트마다 고유의 값을 주어야한느 경우 사용

원래는 onClick={onCancel} 처럼 함수 자체를 넘겨주는데, onClick={onCancel(post.id)}와 같은 경우를 생각하면 된다.

위의 경우, onCancel 함수를 고차함수로 정의해 사용할 수 있다.

```js
const onCancel = (id) => () => {
    dispatch({
      type: UNFOLLOW_REQUEST,
      data: id
    })
  }
```



## multi-part form 처리 방법

multi-part로 올릴 수 있는 데이터: 파일, 이미지, 비디오 등

```ts
// src/app.ts

// axios로 받은 데이터 처리, json 형식으로 데이터 받으면 req.body에 넣어줌
app.use(express.json()); 
// 일반 form의 데이터 처리 (not multi-part form)
app.use(express.urlencoded({ extended: true })); 
```

위 코드는 axios로 받은 데이터와 일반 form 데이터를 처리해줄 수 있다. 하지만 multi-part data는 위의 데이터 처리 코드만으로 받을 수 없어, backend에서 multi-part data를 받을 수 없게된다. 그러므로 multi-part를 처리해주는 코드를 추가해주어야 한다. (-> multer 라이브러리 사용)

multer 미들웨어를 app.ts에다가 장착할 수 있지만, 그렇게 하면 모든 라우터에 장착되므로, 주로 라우터마다 장착한다. 

multer는 폼마다 형식이 달라 router마다 별도로 셋팅을 해줘야 한다.(이미지 안올릴수도 있고, 이미지 1개, 여러개 다 폼이 다름)

```js
// 하드디스크에 저장하는 방식 (추후에 AWS의 S3로 변경)
const upload = multer({
  storage: multer.diskStorage({
    destination(req, file, done) {
      done(null, 'uploads'); // uploads 라는 폴더에 저장
    },
    // path는 node.js에 기본 탑재
    filename(req, res, done) {
      // ex) test.png
      const ext = path.extname(file.originalname); // 확장자 추출(.png)
      const basename = path.basename(file.originalname, ext); // 파일명 추출(test)
      done(null, basename + new Date().getTime() + ext); // 중복 제거
    },
  }),
  limits: { fileSize: 20 * 1024 * 1024 }, // 20MB
});

// 미들웨어 추가 
// (이미지 한 장이면 upload.single, 없으면 upload.none()
// 만약 file input이 여러개면 upload.fields)
router.post('/images', isLoggedIn, upload.array('image'), (req, res, next) => {
  console.log(req.files);	// 업로드 정보
  res.json(req.files.map((v) => v.filename)); // 어디로 업로드 되었는지 파일명 반환
});
```

PostForm.js의 image input에서 올린게, 그대로 array로 전달된다.



multiform 데이터를 routes에서 받아올 때, array로 받았으면 `req.files`로 받고, single or none이면 `req.body.변수명`으로 받는다.



나중에 AWS의 S3와 같은 클라우드에 저장 

이유: 컴퓨터의 하드디스크에 저장하면, 나중에 백엔드 서버 스케일링(복사) 시에 이미지가 같이 복사되어 넘어가 서버에 쓸데없는 공간을 차지하게 된다.



#### 용량이 큰 파일 처리

동영상과 같이 용량이 큰 파일은 서버를 거치지 않는 것이 좋다.

프론트 -> 클라우드로 바로 올리는 것을 제공하므로 그것을 사용하는 것이 좋다.

(나중에 대규모 서비스 되면 처리하자..)



#### 파일 시스템 관리

fs 라이브러리 사용(내재되어 있음)

```js
// uploads 폴더 생성
try {
  fs.accessSync('uploads');	// uploads 폴더 유무 확인
} catch (error) {
  console.log('uploads 폴더 없어 생성함');
  fs.mkdirSync('uploads');
}
```

Express가 uploads 폴더를 프론트에 제공할 수 있도록 설정

```js
// app.js
...
app.use('/', express.static(path.join(__dirname, 'uploads')))

'/': localhost:3065 (백엔드 서버 주소)
```

`express.static(__dirname + '/uploads')` 하지 않는 이유는, Mac이냐 Window냐에 따라 주소의 /와 \가 달라지기 때문이다.

프론트에서는 서버쪽 폴더 구조를 모르고, / 로 접근하기 때문에 보안에 유리하다.



#### 동기 액션: 이미지 지우기

이미지를 지우는 액션은 동기 액션으로, type이 REMOVE_IMAGE 뿐이다.

이미지는 업로드 후에, 서버에서 지우지 않기로 결정했기 때문에 화면 상(Redux)에서만 지우므로 동기 액션으로 처리하는 것이다.

만약 이미지를 서버에서도 지우고 싶다면, 다른 액션과 동일하게 백엔드 서버에서도 지워지게 구현하면 된다.



#### 로직

- input 태그에 type="file"을 추가해주고, 이쁘게 만들기 위해 Button 태그와 연결한다. (useRef와, .current.click() 이용)
- 추가 및 변경되는 이미지는 FormData를 만들어서, 이미지 전역 state에 추가
- 이미지 전역 state(링크?)와 text를 새로운 FormData에 넣어서 post 전역 state와 db에 추가





# Next 11버전에서 SSR 사용하기

next에서 SSR 구현하는 메서드 4개가 있으나, 모두 redux와 호환성 문제가 있어 next-redux-wrapper가 제공하는 것이 더 나음.

SSR 적용 전 문제점은 첫 화면이 로딩될 때, 페이지가 불러와지고나서 useEffect로 사용자 정보, 게시글이 로딩되는 것이다. SSR을 이용해 사용자 정보와 게시글이 먼저 로딩되게 만들 것이다.

- page.`getInitialProps` (없어질 가능성 잇음)

- getServerSideProps / getStatic? / ??

  ```js
  // front/pages/index.js
  
  // wrapper: configureStore.js에서 가져온 wrapper
  // SSR
  export const getServerSideProps = wrapper.getServerSideProps(
    (store) =>
      async ({ req }) => {
        // 쿠키 backend에 전달
        const cookie = req ? req.headers.cookie : '';
  
        axios.defaults.headers.Cookie = ''; //쿠키 지우기
        if (req && cookie) {
          axios.defaults.headers.Cookie = cookie; // 쿠키를 써서 요청보낼 때
        }
  
        // action 끝까지
        store.dispatch({
          type: LOAD_MY_INFO_REQUEST,
        });
        store.dispatch({
          type: LOAD_POSTS_REQUEST,
        });
        store.dispatch(END);
        await store.sagaTask.toPromise();
      },
  );
  
  ```

  - `getServerSideProps`에서 `dispatch()`가 실행되면, `store`에 정보가 들어가게 된다.  `reducers/index.js`의 **HYDRATE** action이 실행되면서 그 정보들을 받는다.

  - `getServerSideProps`는 브라우저에서 실행되는 것이 아닌, 프론트 서버에서 실행되는 것이다.

  - <u>브라우저에서 데이터를 보낼 때는 쿠키가 자동으로 보내졌는데, SSR을 하는 경우 위 부분은 프론트 서버에서 데이터를 보내므로 **쿠키**가 정상적으로 전달되지 않는다.</u> (`back/routes/user.js`의 `/` 에서, `console.log(req.headers)`를 찍어보면 쿠키가 존재하지 않는다는 것을 알 수 있다.)

  - `getServerSideProps`를 이용해 쿠키값을 전달하고 로그인 상태를 유지하는 경우, `getServerSideProps` 부분은 서버쪽에서 실행된다는 것을 알아둬야한다. 이 말은 즉, 접속하는 브라우저는 사용자마다 다른데, 서버는 중앙에 하나뿐이므로, <u>코드를 잘못 짜게되면 다른 사람이 내 정보로 로그인</u>되어 있을 수 있다. 그렇기 때문에 서버의 쿠키를 초기화시키고, 쿠키를 써서 요청을 보낼 때만 쿠키가 저장될 수 있도록 만들어야 한다.



- `getStaticProps`
  
  - 사용 방법은 getServerSideProps와 동일
  - 언제 접속해도 데이터가 바뀔 일이 없으면 getStaticProps (블로그 게시글), 접속할 때마다 상황이 바뀌면 getServerSideProps (대부분)
  
- `getStaticPaths`

  - getStaticProps와 같이 사용하는 것으로, 다이나믹 라우팅에서 쓰인다.

  - 아래 코드를 getStaticProps 위에 놓으면 된다.

    ```js
    // front/pages/post/[id].js
    if (router.isFallback) {
        return <div>로딩중</div>
    }
    
    export async function getStaticPaths() {
       	const result = await axios.get('/post/list')	// 게시글 list
    	return {
            paths: [
                { params: {id: '1'}},	// id가 1인 게시글의 페이지 생성, result 동적 적용
                { params: {id: '2'}},
                { params: {id: '3'}},
            ],
            fallback: true,	// paths에 없는 페이지 오류x
        }
    }
    ```

    fallback이 true인 경우, paths에 게시글이 존재하지 않으면 getStaticProps 부분에서 게시글을 서버로부터 받아옴 (CSR 방식?)



#### CSS에 SSR 적용하기

Next 내부적으로 Webpack과 Babel이 돌아간다.

Babel 설정을 임의로 바꿔주어 css SSR 오류 해결해줄 수 있다.

(강좌에서는 styled-component를 사용해 이런 오류가 생기는데, emotion.js의 경우 버전 10 이상부터는 Next에서 SSR을 지원해준다.)





#### _document.js

_app 위에 존재하는 파일 ( _app을 포함한 pages 전체 파일 감쌀 수 있음.)

```js
// pages/_document.js
// 기본 형태

import React from 'react';
import Document, { Html, Head, Main, NextScript } from 'next/document';

export default class MyDocument extends Document {
  static async getInitialProps(ctx) {
    const initialProps = await Document.getInitialProps(ctx);
    return {
      ...initialProps,
    };
  }
  render() {
    return (
      <Html>
        <Head />
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    );
  }
}
```





#### 다이나믹 라우팅

게시글을 공유하는 서비스 제작 시, 게시글 별로 주소가 필요하다.

pages에서 포스트별로 페이지를 다 만들 수 없으므로, post/[id].js로 만들어준다.

```js
// post/[id].js

import {useRouter} from 'next/router';

const Post = () => {
	const router = useRouter();
	const { id } = router.query	// router 주소에 따라 id값이 달라짐.
    
    return (
    	<div>{id}번 게시글</div>
    )
}
```



#### 메타데이터

```js
    <AppLayout>
      <Head>
        <title>
          {singlePost.User.nickname}
          님의 글
        </title>
        <meta name="description" content={singlePost.content} />
        <meta property="og:title" content={`${singlePost.User.nickname}님의 게시글`} />
        <meta property="og:description" content={singlePost.content} />
        <meta property="og:image" content={singlePost.Images[0] ? 									singlePost.Images[0].src : 'https://nodebird.com/favicon.ico'} />
        <meta property="og:url" content={`https://nodebird.com/post/${id}`} />
      </Head>
      <PostCard post={singlePost} />
    </AppLayout>
```



#### Postman 활용

GET으로 주소 링크를 보내면, 검색엔진이 보는 데이터를 볼 수 있다.



#### Pollyfill.io의 활용

최신 문법은 Babel로 바꿔줄 수 있지만, Promise나 set과 같은 것들은 Babel로 해도 추가가 안된다.

babel polyfill을 사용하기도 하는데, 사이즈가 너무 커서 Polyfill.io를 사용하면 좋다.



#### SWR

Next에서 만든 라이브러리 (SSR 지원)

Redux를 대체할 수 있다, fetcher를 다른 걸로 바꾸면 graphQL도 가능하다.

```js
// front/pages/profile

import useSWR from 'swr';

const fetcher = (url) =>	// 설정에 따라 GraphQL로 바꿀 수 있다.
  axios.get(url, { withCredentials: true }).then((result) => result.data);

const Profile = (props) => {
    ...
    
  const { data: followersData, error: followerError } = useSWR(
    'http://localhost:3065/user/followers',
    fetcher,
  );
    
  const { data: followingsData, error: followingError } = useSWR(
    'http://localhost:3065/user/followings',
    fetcher,
  );
    
    ...
    
  if (followerError || followingError) {
    console.error(followerError || followingError);
    return '팔로잉/팔로워 로딩 중 에러 발생';
  }
    ...
    
  <FollowList header="팔로잉" data={followingsData} />	// me.Followings
  <FollowList header="팔로워" data={followersData} />	// me.Followers
}
```

(return이 Hooks보다 위에 존재하면 안된다.)



#### 404 오류 원인 중 하나 (404: 존재x)

routes에서 params를 이용해 backend 주소를 동적으로 만들었다.

미들웨어는 위에서 아래로 주소를 찾는데, 동적인 주소 때문에 원하는 주소를 못찾아 갈 수 있다.

그러므로 params를 이용한 동적 router는 아래로 내려준다.



#### 날짜

moment 라이브러리

data-fns 라이브러리: 불변성

dayjs: 데이터 용량 굳! (moment와 사용법 유사)



#### 빌드하기

빌드를 하면, html,css,js로 결과물이  나오고, 이것들을 웹서버에 올려놓으면 사용자들의 브라우저로 전달된다.



**빌드 결과)**

First Load JS가 1MB가 안넘으면 우리나라에서 운영 가능, 1MB가 넘으면(모바일에서 렉 걸릴 수 있음) 코드 스플리팅을 통해 해결. (React.Lazy,  서스팬딩?)

Page 부분은 페이지의 타입을 말함. 람다: SSR (getServerSideProps), 검은 동그라미: SSR (getStaticProps), 빈 동그라미: 처음부터 HTML

gzip으로 압축해서 html, css, js 압축



빌드 내용 분석 패키지: @next/bundle-analyzer

```
$ npm i @next/bundle-analyzer
```



##### 절차

github에 push -> CI/CD툴을 github과 연결(자동 점검) -> build, test 통과 -> AWS 서버로 배포

빌드한 폴더들은 front의 .next 폴더 안에 들어간다.



#### 커스텀 Webpack, bundle-analyzer

배포 직전에 코드 구성을 완성하고, 파일 크기를 확인 후 조정하는 단계에서 사용

웹팩은 next에 기본 설정이 있기 때문에, 다른 리액트에서 webpack 설정하듯이 하는 것이 아닌, config 통해서 기본 설정을 바꿔주는 방식으로 진행

- compress: gzip 적용
- bundel-analyzer:
- devtool의 hidden-source-map: 소스 코드 숨기기
- plugin의 new webpack.ContextReplacementPlugin(/moment[/\\]locale$/, /^\.\/ko$/) : locale tree shaking(예를 들어 모멘트에서 필요없는 다른 나라의 언어까지 가져오는 것을 지워주는 것.)

```js
// front/next.config.js

const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  compress: true, // gzip
  webpack(config, { webpack }) {
    const prod = process.env.NODE_ENV === 'production';
    return {
      ...config,
      mode: prod ? 'production' : 'development',
      devtool: prod ? 'hidden-source-map' : 'eval', // 소스 코드 노출 방지
      plugins: [
        ...config.plugins,
        new webpack.ContextReplacementPlugin(/moment[/\\]locale$/, /^\.\/ko$/),
      ],
    };
  },
});
```



enabled를 true로 만들기 위해, process.env.ANALYZE를 true로 만든다.

(next build할 때 next.config.js가 읽히는 것을 이용해 추가한다.)

```
// package.json

scripts: {
	"build": "cross-env ANALYZE=true NODE_ENV=production next build"
}
```



위처럼 설정을 완료한 후 다시 build하면 브라우저에 창이 뜨는데 client 부분을 주의깊게 봐야한다.



#### ??

- 쿠키는 어디서 만들어서 어떻게 저장, 어떻게 활용하는 것인가
- 로그인 유지에서 쿠키를 서버에 전달하지 못해서 문제가 생겼던 것인데, 왜
- 왜 팔로워, 팔로잉을 따로 불러오는거지? me에 포함되있는게 아닌가?
- draft 동작 관련 이해하기
- 더미데이터 쓸 때는 이미지 어떻게 띄워줬지?
- multer의 upload.none을 사용하기 위해 PostForm에서 formData를 사용했다고 한다. 이미지가 있는데 이미지가 없으면 formData를 사용할 필요가 없다는게 무슨 소리일까?
- front에서 backend 서버의 폴더에 접근 하는 방법(postForm, postImages, imagesZoom에서 localhost:3065를 앞에 추가해줌)
- 이미지 주소를 db에 추가할 때(routes/post.js), 사용한 방법 이해하기
- 리트윗 동작 방식 이해하기
- backend Express 시퀄라이저 include에 대해 조금 더 깊이 이해하기
- saga 대체: toolkit(툴킷)
- getServerSideProps와 다른것들의 차이
- getServerSideProps 코드 내부의 context가 무엇이고 어떤 용도로 사용되는지
- encodeUri
- 배포는 다시 하자....
- cors, cookie를 위한 도메인 



git commit 관련 정리

리덕스 다시 보고 이해하기(saga까지) 

tsconfig, eslintrc.js, prettier 관련 파일을 서버, 리액트, Next.js 등에 맞게 파일 만들어놓고 git에 올려놓자.

다른 도메인간 제약사항