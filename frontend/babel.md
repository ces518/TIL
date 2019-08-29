# Babel
- Babel은 ES6+ 코드를 ES5 이하의 버전으로 트랜스파일링 한다.

```javscript
[1,2,3].map(n => n * n);
```

위 코드는 ES6의 화살표 함수를 사용하고 있다.
IE는 물론 구형 브라우저에서 지원하지 않기때문에 ES6+ 코드를 ES5이하의 버전으로 변환이 필요하다.

Babel을 사용하면 위 코드를 다음과 같이 ES5 이하의 버전으로 트랜스파일링이 가능하다.
```javascript
"use strict";
[1, 2, 3].map(function (n) {
  return n * n;
});
```

#### Babel CLI
- [NPM](https://poiemaweb.com/nodejs-basics#2-install) 을 활용하여 BabelCLI 를 설치
- 전역 설치보단 로컬 설치를 권장한다.

```javascript
mkdir babel-compile
npm init
npm install --save-dev @babel/core @babel/cli
```

설치 완료 이후 package.json
```javascript
{
  "name": "babel-compile",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {

  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "@babel/cli": "^7.5.5",
    "@babel/core": "^7.5.5",
    "@babel/plugin-proposal-class-properties": "^7.5.5",
    "@babel/preset-env": "^7.5.5"
  },
  "dependencies": {
    "@babel/polyfill": "^7.4.4"
  }
}
```

#### .babelrc
- Babel 을 사용하려면 먼저 [@babel/preset-env](https://babeljs.io/docs/en/babel-preset-env/) 를 설치 해야한다.
- @babel/preset-env 는 할께 사용해야할 Babel 플러그인을 모아둔것이다. [Babel 프리셋](https://babeljs.io/docs/en/presets) 이라고 부른다.
- Babel이 제공하는 공식 Babel 프리셋
    - [@babel/preset-env](https://babeljs.io/docs/en/babel-preset-env)
    - [@babel/preset-flow](https://babeljs.io/docs/en/babel-preset-flow)
    - [@babel/preset-react](https://babeljs.io/docs/en/babel-preset-react)
    - [@babel/preset-typescript](https://babeljs.io/docs/en/babel-preset-typescript)
- @babel/preset-env도 공식 프리셋중 하나이며 필요한 플러그인들을 프로젝트 지원 환경에 맞춰 동적으로 결정 해준다. 
- 프로젝트 지원환경은 [Browserslist](https://github.com/browserslist/browserslist) 형식이다.
- .browserslistrc 파일에 상세히 설정할 수 있다. 프로젝트 지원 환경 설정 작업을 생략하면 기본값으로 설정된다.

babel/preset-env 를 설치
```javascript
npm install --save-dev @babel/preset-env
```

설치 완료 후 package.json
```java
{
  "name": "es6-project",
  "version": "1.0.0",
  "devDependencies": {
    "@babel/cli": "^7.2.3",
    "@babel/core": "^7.2.2",
    "@babel/preset-env": "^7.4.5"
  }
}
```

프로젝트 루트에 .babelrc 파일을 생성한다.
- 현재 프로젝트에 @babel/preset-env을 사용하는 설정이다.
```java
{
  "presets": ["@babel/preset-env"]
}
```
