# Babel(바벨)

### 1. 배경

### 1-1 크로스 브라우징

- 브라우져마다 사용하는 언어가 달라서 프론트엔드 코드는 일관적이지 못할 때가 많다.
- 스팩과 브라우저가 개선되고 있지만 여전히 인터넷 익스플로러는 프로미스를 이해하지 못한다.
- 작년까지만 해도 사파리 최신 브라우져는 Promise.prototype.finally 메소드를 사용할 수 없었다.
- 프론트 엔드 개발에서 크로스브라우징 이슈는 코드의 일관성을 해치고 초심자를 불안하게 만든다.
- **크로스브라우징**을 해결하기 위해 **바벨**이 등장
- ECMAScript2015+로 작성된 코드를 모든 브라우져에서 동작하도록 호환성을 지켜준다.
- 타입스크립트 , JSX처럼 다른 언어로 분류되는 것도 포함한다.



### 1-2 트랜스파일과 빌드

- 변화하는 것을 **트랜스파일** 한다라고 표현한다.
- 변환 전후의 추상화 수준이 다른 빌드와는 달리 트랜스파일은 추상화 수준을 유지한 상태로 코드를 변환한다.
- 타입스크립트 => 자바스크립트, JSX => 자바스크립트 처럼 트랜스파일 후에도 여전히 코드를 읽을 수 있다.

- ECMAScript 2015 이상의 코드를 적당한 하위버전으로 바꾸는 것이 주된 역할

```shell
npm install -D @babel/core @babel/cli
```

- 설치 완료후 바벨 명령어

```
npx babel app.js
```

```javascript
// app.js
const alert = msg => window.alert(msg);
```

```js
// 출력 : const alert = msg => window.alert(msg);
```

- 바벨은 파싱과 출력만 담당하고 변환 작업은 다른 녀셕이 처리하는데 이것을 **플러그인**이라고 부른다.
  - 변환하려면 **플러그인** 필수

- 바벨은 세 단계로 빌드를 진행
  - 파싱 : 코드를 받아서 각 토큰별로 분해
  - 변환 : es6코드를 es5로 변환
  - 출력 : 변환된 코드를 출력



### 1-3 커스텀 플러그인

- my-babel-plugin.js 파일 생성
  - 커스텀 플러그인 만들 때 `visitor`라는 객체를 가지고 있는 객체를 반환해줘야 한다.
  - `Identifier`는 `path`라는 객체를 받고 `path.node.name`으로 파싱된 결과물을 접근할 수 있다.

```javascript
module.exports = function myBabelPlugin() {
  return {
    visitor: { 
      Identifier(path) {
        const name = path.node.name 

        // 바벨이 만든 AST 노드를 출력한다
        console.log("Identifier() name:", name)

        // 변환작업: 코드 문자열을 역순으로 변환한다
        path.node.name = name.split("").reverse().join("")
      },
    },
  }
}
```

```shell
// 플러그인으로 바벨 실행
npx babel app.js --plugins './my-babel-plugin.js'
```

- 출력 결과

```shell
Identifier() name: alert
Identifier() name: msg
Identifier() name: window
Identifier() name: alert
Identifier() name: msg
// 파딩되어진 토큰들이 다 역순으로 뒤집힙니다.
const trela = gsm => wodniw.trela(gsm);
```

- const를 var로 변경해보자

```javascript
module.exports = function myBabelPlugin() {
  return {
    visitor: { 
      VariableDeclaration(path) {
        console.log("VariableDeclaration() kind:", path.node.kind) // 출력 : const
        
        //cosnt => bar 변환
        if (path.node.kind === "const") {
          path.node.kind = "var"
        }
      },
    },
  }
}
```

```javascript
// 출력 결과
var alert = msg => window.alert(msg);
```



### 1-4. 플러그인 사용해보기

-  [block-scoping](https://babeljs.io/docs/en/babel-plugin-transform-block-scoping) 플러그인 : const, let 처럼 블록 스코핑을 따르는 예약어를 함수 스코핑을 사용하는 var 변경한다.
-  사용법

```shell
//설치
npm install -D @babel/plugin-transform-block-scoping

//시작
npx babel app.js --plugins @babel/plugin-transform-block-scoping

//출력
var alert = msg => window.alert(msg);
```



- 에로우 함수를 변환하는 플러그인 : arrow-functions

```javascript
// 설치
npm install -D @babel/plugin-transform-arrow-functions

// 실행
npx babel app.js --plugins @babel/plugin-transform-block-scoping --plugins @babel/plugin-transform-arrow-functions

// 출력
var alert = function (msg) {
  return window.alert(msg);
};
```



- **ECMASCript5**에서부터 지원하는 엄격모드를 사용하는 것이 안전하기 때문에 'use strict' 구문을 추가해야 겠다 => strict-mode 플러그인

```javascript
// 설치
npm install @babel/plugin-transform-strict-mode

// 실행
npx babel app.js --plugins @babel/plugin-transform-block-scoping --plugins @babel/plugin-transform-arrow-functions --plugins @babel/plugin-transform-strict-mode

// 출력
"use strict";

var alert = function (msg) {
  return window.alert(msg);
};
```



- 커맨드 라인 명령어가 점점 길어지기 때문에 설정 파일로 분리하는 것이 좋다.
- babel.config.js를 사용한다.

```js
module.exports = {
    plugins:[
        "@babel/plugin-transform-block-scoping",
        "@babel/plugin-transform-arrow-functions",
        "@babel/plugin-transform-strict-mode"
    ]
}
```

```shell
// 실행
npx babel app.js
```

- 기본적으로 babel.config.js파일을 읽어서 플러그인을 적용시킨다.
- 하지만 ES6로 코딩할때 필요한 플러그인을 일일이 설정하는 것인 **비현실적**이다.
- 목적에 맞게 **여러가지 플러그인을 세트로 모아놓은 것을 프리셋**이라고 한다.



### 2. 프리셋

- 커스텀 프리셋
  - my-babel-preset.js 파일 생성

  ```javascript
  module.exports = function myBabelPreset(){
      return {
          plugins:[
              "@babel/plugin-transform-block-scoping",
              "@babel/plugin-transform-arrow-functions",
              "@babel/plugin-transform-strict-mode"
          ]
      }
  }
  ```

  - babel-config.js

  ```javascript
  module.exports = {
      presets: [
          './my-babel-preset.js'
      ]
  }
  ```

- 프리셋 사용해보기
  - preset-env : ECMAScript2015+를 변환할 때 사용한다.
    - 바벨 7 이전 버전에는 연도별로 각 프리셋을 제공했지만(babel-reset-es2015, babel-reset-es2016, babel-reset-es2017, babel-reset-latest) 지금은 env 하나로 합쳐졌다.
  - preset-flow
  - preset-react
  - preset-typescript : 타입스크립트로 작성한 코드를 프리셋을 통해서 es5로 바꿔준다.

- preset-env : 가장 많이 사용한다.

```javascript
// 설치
npm install -D @babel/preset-env

// babel.config.js => es5문법으로 바꿔준다.
module.exports = {
    presets: [
        '@babel/preset-env'
    ]
}
```



### 2-1. 타겟브라우저

- 우리 코드가 크롬 최신버전만 지원한다고 하자. 그렇다면 인터넷 익스플로러를 위한 코드 변환은 불필요하다. 
- target 옵션에 브라우저 버전명만 지정하면 env 프리셋은 이에 맞는 플러그인을 찾아 최적의 코드를 출력한다.

- babel.config.js

```javascript
module.exports = {
  presets: [
    ['@babel/preset-env', {
      targets: {
        chrome: '79', //크롬 79버전을 지원하는 코드를 만든다
        ie: '11' // ie도 지원
      }
    }]
  ]
}
```



### 2-2. 폴리필

```javascript
//app.js
new Promise();

//출력 : 변환되지 않고 나오고 있다...ie는 지원안하는데 ..
"use strict";
new Promise();
```

- 바벨은 ECMAScript2015+를 ECMAScript5 버전으로 변환할 수 있는 것만 빌드한다. 그렇지 못한 것들은 **폴리필**이라고 부르는 코드조각을 추가해서 해결한다.
- 가령 ECMAScript2015의 블록 스코핑은 ECMASCript5의 함수 스코핑으로 대체할 수 있다. 화살표 함수도 일반 함수로 대체할 수 있다. 이런 것들은 바벨이 변환해서 ECMAScript5 버전으로 결과물을 만든다.
- 한편 프로미스는 ECMAScript5 버전으로 대체할 수 없다. 다만 ECMAScript5 버전으로 구현할 수는 있다.
  - `core-js promise`
- env 프리셋은 폴리필을 사용할지 안할지 옵션을 제공한다.
- babel.config.js

```javascript
module.exports = {
  presets: [
    ['@babel/preset-env', {
      targets: {
        chrome: '79', //크롬 79버전을 지원하는 코드를 만든다
        ie: '11' // ie도 지원
      },
      useBuiltIns: 'usage', // 폴리필 사용방식
      corejs: {
        version: 2 // 최신버전은 3
      }
    }]
  ]
}

//출력 결과 : 폴리필 파일을 가저오는 로직이 추가
"use strict";

require("core-js/modules/es6.object.to-string.js");

require("core-js/modules/es6.promise.js");

new Promise();
```



### 3. 웹팩으로 통합

- 실무환경에서는 바벨을 직접 사용하는 것보다는 웹팩으로 통합해서 사용하는 것이 일반적이다.
- 로더 형태로 제공하는 **babel-loader**가 그것이다.
- 웹팩 설정에 로더를 추가한다.

```shell
npm install -D babel-loader
```

```javascript
const path = require('path');
const webpack = require('webpack') 
const HtmlWebpackPlugin = require('html-webpack-plugin');
const {CleanWebpackPlugin} = require('clean-webpack-plugin')
const MiniCssExtreactPlugin = require('mini-css-extract-plugin')

module.exports = {
  mode: 'development',
  entry: { 
      main : './app.js',
  },
  output : { 
      path: path.resolve('./dist'), 
      filename: '[name].js'
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          process.env.NODE_ENV === 'production'
          ? MiniCssExtreactPlugin.loader
          : 'style-loader',
          'css-loader'
        ]
      },
      {
        test: /\.(png|jpg|gif|svg)$/,
        loader: 'url-loader',
        options:{
          // publicPath: './dist/',
          name: '[name].[ext]?[hash]',
          limit: 20000,
        }
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        // 자바스크립트에서 node 모듈에 있는 라이브러리코드도 가져올 수 있는데 이것은 바벨로더가 처리하지 않도록 제외시켜주기
        exclude: /node_modules/  
      }
    ]
  },
  plugins:[
  ]
}
```

- 오류발생 : 폴리필을 동작하는 코드가 추가되었지만 웹팩은 `core.js`가 **node모듈**에 없기 때문에 오류 발생 


```shell
//@2 : 특정버전 설치
npm install core-js@2
//둘중 하나
npm i @babel/core
```

- 바벨은 일관적인 방식으로 코딩하면서, 다양한 브라우져에서 돌아가는 어플리케이션을 만들기 위한 도구다.

- 바벨의 코어는 파싱과 출력만 담당하고 변환 작업은 플러그인이 처리한다.

- 여러 개의 플러그인들을 모아놓은 세트를 프리셋이라고 하는데 ECMAScript+ 환경은 env 프리셋을 사용한다.

- 바벨이 변환하지 못하는 코드는 **폴리필**이라 부르는 코드조각을 불러와 결과물에 로딩해서 해결한다.

- babel-loader로 웹팩과 함께 사용하면 훨씬 단순하고 자동화된 프론트엔드 개발환경을 갖출 수 있다.



### IE에서 웹팩 번들링 작동하게 하기

- 웹팩 설정파일에서 target을 추가해줍니다.

```js
module.exports = {
  target: ['web', 'es5'],
}
```

- `async` `await` 쓰려면 `regenerator-runtime` 설치 필요
  - `core-js@2`로는 불가합니다.
  - `Array.includes()`도 폴리필로 해결해야 합니다.

```
npm install regenerator-runtime
```



### Sass를 babel로 바꿔보자

- app.js

```javascript
// Sass에는 2가지 확장자 존재 : sass, scss
import "./app.scss";
```

- 설치
  - `node-sass` : `sass`코드를 `css`로 컴파일 해주는 코드
  - `sass-loader` : 웹팩이 sass 파일을 만나면 `node-sass`를 돌려주는 역할

```
npm install sass-loader node-sass
```

```javascript
const path = require("path");
const webpack = require("webpack");

const HtmlWebpackPlugin = require("html-webpack-plugin");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

process.env.NODE_ENV = process.env.NODE_ENV || "development";

module.exports = {
  mode: "development",
  entry: {
    main: "./src/app.js"
  },
  output: {
    filename: "[name].js",
    path: path.resolve("./dist")
  },
  module: {
    rules: [
      {
        test: /\.(scss|css)$/, //변경
        use: [
          process.env.NODE_ENV === "production"
            ? MiniCssExtractPlugin.loader 
            : "style-loader", 
          "css-loader",
          "sass-loader" //추가
        ]
      },
    ]
  },
};
```

