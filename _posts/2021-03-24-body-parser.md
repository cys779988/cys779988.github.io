---
title:  "nodejs body-parser"
excerpt: nodejs body-parser
categories:
  - nodejs
---

> ## body-parser 모듈 사용이유

- body-parser 는 post로 요청된 body를 쉽게 추출할 수 있는 모듈
- 추출된 결과는 request 객체에 body 속성으로 저장됨.
- http 모듈로만 post body를 파싱하려면   ```req.on('data', function(chunk) { body += chunk; });```  와 같이 이벤트를 등록한 후 인코딩 처리를 해줘야함
- body-parser 를 사용하면   ```bodyParser.urlencoded()```  를 등록하면, 자동으로 req에 body속성이 추가되고 저장됨.
- 만약 urls에 접근하고 싶다면 req.body.urls, 인코딩도 default로 UTF-8로 해줌. 이벤트 등록할 필요가 사라짐

> ## urlencoded() 의 옵션

- 만약 아무 옵션을 주지 않았다면 body-parser deprecated undefined extended: provide extended option 같은 문구가 뜸
- API 문서를 보면,   ```.use(bodyParser.urlencoded({ extended: true or false })); ```  로 쓰라고 함.
- extended 는 중첩된 객체표현을 허용할지 말지를 정하는 것. 객체 안에 객체를 파싱할 수 있게하려면 true

### extended 옵션

- 내부적으로 true를 하면 qs 모듈을 사용하고, false 면 query-string 모듈을 사용
- 두 모듈간의 차이에서 중첩객체 파싱여부가 갈림

참조 : https://stackoverflow.com/questions/29960764/what-does-extended-mean-in-express-4-0/45690436#45690436
