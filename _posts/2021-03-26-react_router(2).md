---
title:  "React Router(2)"
excerpt: React Router(2)
categories:
  - react
---

> ## React-Route 링크 추가하기

- html의 \<a href="" \> 태그와 같이 React에서 페이지 전환이 가능한 \<Link\>태그 사용
- react-router-dom 선언된 코드에 Link 태그 추가

```javascript
// src/App.js

import { BrowserRouter, Route, Link } from 'react-router-dom';
```  


```javascript
// src/App.js

   <BrowserRouter>
      <Route path="/" component={Home} exact />
      <Route path="/test" component={Test} />
      <ul>
          <li> <Link to='/'> Home </Link> </li>
          <li> <Link to='/test'> Test </Link> </li>
      </ul>
  </BrowserRouter>
```  

> ## URL Parameter 넘기기

### 1. params로 데이터 넘기기

```javascript
// src/App.js

<Route path="/test/:data" component={Test} />
```  

```javascript
// src/inc/test.js

import React, {Component} from 'react';

class test extends Component{
    constructor(props){
        super(props)
        console.log(props);
    }
    
    render(){
        return(
            <div>
                <h3>My name is {this.props.match.params.data}</h3>
            </div>

        );
    }
}

export default test;
```  

-   ```http://localhost:3000/test/ys```   경로로 이동하면 'data : ys' 값이 전달됨
-   ```{this.props.match.params.data}```   의 값으로 가져옴


### 2. query-string 사용하기

- query-string 라이브러리를 사용하면 url로 넘기는 data를 객체 형태로 받아볼 수 있음

```javascript
yarn add query-string
//npm install query-string
```  

```javascript
// src/inc/test.js

import React, {Component} from 'react';
import queryString from 'query-string';

class test extends Component{
    constructor(props){
        super(props)
        console.log(this.props);
    }

    render(){
        return(
            <div>
                <h3>My name is {this.props.location.search}</h3>
            </div>

        );
    }
}

export default test;
```  

- location의 search에 값이 담겨서 전달됨
- search로 전달된 데이터 값은 문자열의 형태로만 사용
- 이 문자열을 query-string 라이브러리를 사용해 객체 형태로 변환

```javascript
const qry = queryString.parse(this.props.location.search);
```  

-   ```http://localhost:3000/test?name=ys```   {name:"ys"} 객체 형태로 변환됨


```javascript
 render() {
    const qry = queryString.parse(this.props.location.search);

    return (
        <div>
          <h3> My name is {qry.name} </h3>
        </div>
    );
  }

```  

- {qry.name}으로 값을 받아올 수 있음
- 두 개의 값을 전달할 때는 uri 사이에 '&'을 덧붙임

- index.js 코드를 수정하여 중복적으로 \<BrowserRouter\> 태그를 선언하거나 App에서 호출되지 않을 때는 index.js 에서 \<BrowserRouter\> 태그 선언

```javascript
// src/index.js

import { BrowserRouter } from 'react-router-dom';

ReactDOM.render(
    <BrowserRouter>
        <App />
    </BrowserRouter>, 
    document.getElementById('root')
);
```  


> ## Switch 태그

```javascript
  <Route path="/test" component={Test} />
  <Route path="/test/:data" component={Test} />
```  

- Route의 path에 설정된 중복 문제 ('/test'가 중복)
- exact 태그속성으로 중복을 피할 수 있지만 Switch 태그를 사용하면 좀 더 체계적이고 가독성 있게 관리할 수 있음

```javascript
// src/App.js

import {BrowserRouter, Route, Link, Switch} from 'react-router-dom';
```  

- 위 코드를 추가한 후 src/App.js render 부분의 코드 수정
- 주의할 점은 params 값을 가지는 Route를 먼저 선언

```javascript
    render(){
        return(
            <div className='App'>
                <BrowserRouter>
                    <Route path="/" component={Home} exact />

                    <Switch>
                        <Route path="/test/:data" component={Test} />
                        <Route path="/test" component={Test} />
                    </Switch>
                <ul>
                    <li> <Link to='/'> Home </Link> </li>
                    <li> <Link to='/test'> Test </Link> </li>
                </ul>
                </BrowserRouter>

            </div>
        );
    }
```  

- Switch 는 Route 태그의 path에서 params 를 가지는 Route 를 먼저 출력하고 만약 params 값이 없다면 그 다음 Route를 출력함
