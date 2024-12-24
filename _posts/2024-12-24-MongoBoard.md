---
layout: single
title: "[Dreamhack] MongoBoard write-up"
date: 2024-12-24 11:13 +0900
categories: 
    - WEB-write-up
tag:
typora-root-url: ../
toc: true
toc_sticky: true
toc_label: 목차
author_profile: true
comment: true
sidebar:
    nav: "docs"
---

## [Dreamhack] MongoBoard

<br>

### 문제 설명

- node와 mongodb로 구성된 게시판입니다.
- 비밀 게시글을 읽어 FLAG를 획득하세요.
  - MongoDB < 4.0.0

<br>

### 풀이

<br>

문제 페이지에 접속하면 일단 admin 계정에 FLAG가 저장되어 있는 것 같다.

![{8772C583-03AB-4984-84F8-AAD8BF9D5585}](/images/2024-12-24-MongoBoard/{8772C583-03AB-4984-84F8-AAD8BF9D5585}.png)

<br>

소스 코드를 보면 모든 요청에 대해서 CORS 설정을 추가하였고, 아래 router 설정 하는 부분에서 router 디렉토리에서 라우팅 파일을 불러오고 있다.

```javascript
var express     = require('express');
var app         = express();
var bodyParser  = require('body-parser');
var mongoose    = require('mongoose');
var path        = require('path');

// Connect to MongoDB
var db = mongoose.connection;
db.on('error', console.error);
db.once('open', function(){
    console.log("Connected to mongod server");
});
mongoose.connect('mongodb://localhost/mongoboard');

// model
var Board = require('./models/board');

// app Configure
app.use('/static', express.static(__dirname + '/public'));
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

app.all('/*', function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Methods", "POST, GET, OPTIONS, PUT");
  res.header("Access-Control-Allow-Headers", "Content-Type");
  next();
});

// router
var router = require(__dirname + '/routes')(app, Board);
app.get('/', function(req, res) {
    res.sendFile(path.join(__dirname + '/index.html'));
});

// run
var port = process.env.PORT || 8080;
var server = app.listen(port, function(){
 console.log("Express server has started on port " + port)
});

```

<br>

route 디렉토리의 index.js 파일에서는 /api/board 경로에서 get과 put 메소드가 존재하고, /api/board/:{board_id} 경로에서도 board_id라는 파라미터를 받아와 주는 get 메소드도 존재한다.

```javascript
module.exports = function(app, MongoBoard){
    app.get('/api/board', function(req,res){
        MongoBoard.find(function(err, board){
            if(err) return res.status(500).send({error: 'database failure'});
            res.json(board.map(data => {
                return {
                    _id: data.secret?null:data._id,
                    title: data.title,
                    author: data.author,
                    secret: data.secret,
                    publish_date: data.publish_date
                }
            }));
        })
    });

    app.get('/api/board/:board_id', function(req, res){
        MongoBoard.findOne({_id: req.params.board_id}, function(err, board){
            if(err) return res.status(500).json({error: err});
            if(!board) return res.status(404).json({error: 'board not found'});
            res.json(board);
        })
    });

    app.put('/api/board', function(req, res){
        var board = new MongoBoard();
        board.title = req.body.title;
        board.author = req.body.author;
        board.body = req.body.body;
        board.secret = req.body.secret || false;

        board.save(function(err){
            if(err){
                console.error(err);
                res.json({result: false});
                return;
            }
            res.json({result: true});

        });
    });
}

```

<br>

/api/board 경로에 접속해보면 아래와 같은 json 데이터들이 존재하고, 이전에 생성해둔 test 글도 보인다.

![{943FFB85-A5DD-4C9F-81C8-F42741641607}](/images/2024-12-24-MongoBoard/{943FFB85-A5DD-4C9F-81C8-F42741641607}.png)

<br>

put 메소드로 게시글 생성도 가능하기 때문에 일단 아래와 같이 Content-Type 헤더에 `application/json` 헤더를 추가해주고 secret 값을 true로 변경해서 생성해보았다.

![{34F69D6F-14FE-45F0-8164-FBF1E0B62538}](/images/2024-12-24-MongoBoard/{34F69D6F-14FE-45F0-8164-FBF1E0B62538}.png)

<br>

/api/board/:board_id 경로로 title이 Mongo인 id를 입력하니 아래와 같이 데이터가 잘 출력되고 있다. 하지만 admin의 id는 null로 되어 있다. 따라서 MongoDB의 ObjectID가 어떤 구조로 이루어져있는지 알아야 한다.

![{C5E480F4-5271-45AB-9D12-98B0C8A37605}](/images/2024-12-24-MongoBoard/{C5E480F4-5271-45AB-9D12-98B0C8A37605}.png)

<br>

no의 구조를 잘 살펴보면 중간 부분에 있는 `63218cc9dc` 는 프로세스당 한 번 생성되는 임의의 값으로 똑같은 걸 볼 수 있고 뒤에 있는 6자리는 counter를 뜻하는 자리로서 1씩 증가되는 것을 알 수 있다. 앞에 있는 8자리는 timestamp 값이다.

![{3E617924-984D-4D14-8467-3AD0541A7D66}](/images/2024-12-24-MongoBoard/{3E617924-984D-4D14-8467-3AD0541A7D66}.png)

<br>

timestamp 값을 비교해보기 위해서 계산해보면 2가 나온다.

```python
a = "676a184b"
b = "676a1849"

c = int(a, 16)
d = int(b, 16)

time = c - d

print(time)
```

<br>

서버를 재생성 하였더니 timestamp 값이 바뀌였고 두 번째 값과 3초가 차이가 났기 때문에 아래와 같이 timestamp 값을 3 더해주고 counter 부분에 1 더해서 `/api/board/:board_id` 경로에 조회하면 flag 값을 획득할 수 있다.

```python
a = "676a2833"

# timestamp 값
b = int(a, 16) + 3
c = hex(b)
d = str(c[2:])

# 임의값 + counter
counter = "406996628988db77"
print(d + counter)
```

![{A246D829-8942-4CE2-9740-DA93692432EE}](/images/2024-12-24-MongoBoard/{A246D829-8942-4CE2-9740-DA93692432EE}.png)