---
title:  "React Router"
excerpt: React Router
categories:
  - react
---

> ## React-Router란?

- 리액트는 SPA(Single Page Application) 방식
- 여러 페이지를 사용하며, 새로운 페이지를 로드하는 기존의 MPA(Multi Page Application) 방식과 달리 새로운 페이지를 로드하지 않고 하나의 페이지 안에서 필요한 데이터만 가져오는 형태를 가짐

<img src="https://cys779988.github.io/assets/img/react-1.png">  
  
  
<img src="https://cys779988.github.io/assets/img/react-2.png">  


- MPA 방식은 현재의 페이지를 완전히 다시 로드해서 새로 구성
- MPA 방식에서 다른 페이지로 이동하기 위해선 주로 \<a href=""\> 태그를 사용하거나 window.location.href 등의 메소드를 사용
- SPA 방식은 필요한 데이터만 가지고 와서 재로드 없이 렌더링
- SPA 방식에서는 오직 하나의 페이지만 가지고 있기 때문에 MPA 방식으로의 페이지 이동이 불가능 (html이 아닌 JS파일로 구성 되어 있기 때문)
- React-Router는 페이지를 새로 불러오지 않는 상황에서 각각의 선택에 따라서 선택된 데이터를 하나의 페이지에서 렌더링 해주는 라이브러리

> ## React-Router 적용

```javascript
yarn add react-router-dom
//npm install react-router-dom
```  

```javascript
// src/App.js

import {BrowserRouter, Route} from 'react-router-dom';

...

```  

- src/inc 디렉토리 생성
- inc/home.js, inc/test.js 파일 생성


```javascript
// src/inc/home.js

import React, {Component} from 'react';

class home extends Component{
    render(){
        return(
            <div>
                <h3>This is YeongSu's Blog</h3>
            </div>
        );
    }

}

export default home;
```  

```javascript
// src/inc/test.js

import React, {Component} from 'react';

class test extends Component{
    render(){
        return(
            <div>
                <h3>This is test page</h3>
            </div>

        );
    }
}

export default test;
```  

```javascript
import React, {Component} from 'react';
import './App.css';
import {BrowserRouter, Route} from 'react-router-dom';

import Home from './inc/home.js';
import Test from './inc/test.js';

class App2 extends Component{
    constructor(props){
        super(props)
        this.state = {

        }
    }

    render(){
        return(
            <div className='App'>
                <BrowserRouter>
                    <Route path="/" component={Home} />
                    <Route path="/test" component={Test} />
                </BrowserRouter>
            </div>
        );
    }
}

export default App2;
```  

```javascript
// src/App2.js

import React, {Component} from 'react';
import './App.css';
import {BrowserRouter, Route} from 'react-router-dom';

import Home from './inc/home.js';
import Test from './inc/test.js';

class App2 extends Component{
    constructor(props){
        super(props)
        this.state = {

        }
    }

    render(){
        return(
            <div className='App'>
                <BrowserRouter>     // React-Route를 시작하는 코드. <Route> 코드는 반드시 <BrowserRouter> 안에서 실행되어야함
                    <Route path="/" component={Home} />         // Route 경로를 설정해주는 코드
                    <Route path="/test" component={Test} />     // path는 Route의 속성 값으로 해당 경로의 url을 설정 ("/"는 제일 첫번째 경로의 값)
                </BrowserRouter>                                // component는 설정한 path의 경로로 이동했을 때 실행되는 컴포넌트를 설정
            </div>
        );
    }
}

export default App2;
```  

- test.js 를 실행하는 "/test"의 경로 중 "/" 가 home.js 를 실행하는 "/"와 중복되기 때문에 home.js, test.js 둘 다 실행됨
- 이러한 중복을 막기 위해 home.js route가 설정된 태그에 아래 코드 추가

```javascript
<Route path="/" component={Home} exact />
```  


### 한번의 코드로 해당 디렉토리 모든 파일 불러오기

- 'src/inc/index.js' 생성

```javascript
// src/inc/index.js

export {default as Home} from './home';
export {default as Test} from './test';
```  

```javascript
// src/App.js

import {Home, Test} from './inc'
```  
