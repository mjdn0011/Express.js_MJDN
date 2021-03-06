# Middleware
*`express().use`*

* 요청 핸들러와 유사하지만(요청 수신 - 응답 전송),
미들웨어는 하나의 핸들러라기보다는 순차적으로 여러 번의 처리를 수행할 수 있음

* 하나의 함수를 통해서만 요청을 처리하는 순수한 Node.j와 대조적으로,
Express.js는 사실상 함수 배열인 **미들웨어 스택**을 가짐

* Express.js 스택의 거의 모든 조각이 어떤 점에서는 미들웨어의 영향을 받은 것

* 미들웨어 함수 작성: 3개의 인수를 갖는 함수
* 에러 처리 미들웨어 작성 및 사용: 4개의 인수를 갖는 함수
* 로깅용 Morgan 및 정적 파일 서비스용 express.static과 같은 오픈 소스 미들웨어 함수 사용하기 

프레임워크가 없는 Node.js는 전체 앱에 대한 하나의 큰 요청 핸들러 함수를 작성하게 한다.

* 웹 애플리케이션 서버는 
요청을 수신 대기(listening)하고, 
이들 요청을 분석하며, 
응답을 보냄

1. Node 런타임은 요청을 받아서 원시 바이트를 처리할 수 있는 두 개의 JS 객체(request, response)로 전환
2. 이 두 객체는 JS 함수(요청 핸들러)로 전달
3. 사용자가 원하는 것을 확인하기 위해 req를 분석하고 res를 다뤄서 응답을 준비
4. 응답 작성이 끝날 때 `res.end`를 호출 
   이 호출은 응답이 모두 준비되었고, 네트워크를 통해 보낼 수 있다는 신호를 줌
5. Node 런타임은 응답 객체에 수행한 것을 확인하고, 응답 객체를 한 뭉치의 바이트로 바꿔 인터넷을 통해 요청한 사람에게 전송

Node.js에서 요청, 응답 객체는 단 하나의 함수로 전달됨
Express.js에서 요청, 응답 객체는 함수의 배열, 즉 미들웨어 스택이라는 것을 통해 전달됨

Express로 동작할 때, 하나의 요청 핸들러 함수는 미들웨어 함수의 스택으로 대체됨

##### 미들웨어 스택의 모든 함수는 3개의 인수를 가짐
```js
/* Middleware Function Signature */

function(req, res, next)

// 1. req: HTTP 요청을 나타내는 객체
// 2. res: HTTP 응답을 나타내는 객체
// 3. next: 호출될 때 다음 미들웨어로 보내는 함수
//          함수 자체이며, 관례상 next 라고 함
```
* 스택에서 이들 함수 중 하나는 요청을 끝내는 `res.end`를 호출해야 함
* 미들웨어 스택의 함수는 모두 res.end를 호출할 수 있지만,
한 번만 실행해야 하며, 그렇지 않으면 에러가 발생함

* 미들웨어 함수가 끝나면 다음 두 가지 중 하나를 수행해야 함
   1. 해당 요청에 대한 응답을 끝내야 함
      1. `res.end`
      2. Express의 `res.send`나 `res.sendFile`과 같은 
      편의 메서드 중 하나 사용
   2. 미들웨어 스택에서 다음 함수로 계속하기 위해 `next`를 호출


```js
var app = express();

app.use(function(req, res, next) {
  res.end('Hello world!');

  next();  // next() 호출은 체인의 다음 미들웨어를 수행함
});
```

##### 미들웨어는 다양한 애플리케이션을 가짐
* 모든 요청을 기록하는 미들웨어
* 각 요청에 대해 특정 HTTP 헤더를 설정하는 미들웨어
* 등등

#### # 미들웨어 스택
> ~~ONE request handler~~
**ARRAY** of reqeust handler

Express는 미들웨어를 통해 ~~함수 하나~~가 아니라 <u>함수들의 배열</u>을 실행하도록 함

##### 미들웨어의 고수준 동작 방식
* Node.js의 HTTP 서버에서 모든 요청은 <u>하나의 큰 함수(**마스터 요청 함수**)</u>를 통해 처리됨
* Express의 미들웨어를 사용하면 요청은 하나의 함수를 거치기보다는,
**미들웨어 스택**이라는 **함수의 배열**을 통과하게 됨

서버를 시작하고 요청을 받으면 최상위 미들웨어에서 작업을 시작해 아래로 내려가면서 수행함

#### # 패시브 미들웨어
기본적으로 각 미들웨어 함수는 요청이나 응답을 수정할 수 있음
패시브 미들웨어는 <u>응답이나 요청을 바꾸지 않는 미들웨어</u>
ex) 로깅 미들웨어

#### # 정적 미들웨어 *`express.static()`*
`express.static`은 정적 파일 서비스를 도와줌.
파일을 전송하는 간단한 동작에도 생각해볼만한 <u>경계 값 테스트 문제</u>와 <u>성능 고려사항</u>이 많음 -> 실제로는 많은 작업이 필요
```js
// path 모듈 사용하여 path 설정
var publicPath = path.resolve(__dirname, 'public');
// publicPath 디렉터리에서 정적 파일 전송
app.use(express.static(publicPath));
```

#### # 서드파티 미들웨어 라이브러리
* *`MORGAN`*: logging
* *`connect-ratelimit`*: 시간당 특정 요청 수에 대한 요청 수 조절
  누군가 너무 많은 요청을 보낸다면, 사이트가 다운되는 것을 막기 위해 요청 보낸 곳에 에러 표시할 수 있음
* *`Helmet`*: HTTP 헤더 추가
  HTTP 헤더를 추가함으로써 앱을 특정 종류의 공격으로부터 더 안전하게 만들어 줌
* *`cookie-parser`*: 브라우저 쿠키 분석
* *`response-time`*: 애플리케이션 성능 디버그
  X-Response-Time 헤더를 전송하기 때문에 애플리케이션의 성능을 디버그할 수 있음
> Express와 같은 Connect라는 다른 프레임워크가 있음 (Connect는 미들웨어 기능만 수행함)
Connect 미들웨어는 Express와 호환됨
따라서 "Connect middleware"로 검색하여도 Express에서 사용할 수 있는 미들웨어를 발견할 수 있음
  