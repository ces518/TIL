- node , npm 설치되어있어야함.
- mkdir babel-compile
    - 폴더생성
- npm init
    - npm 초기화
- npm install --save-dev @babel/core @babel/cli
    - babel/core , babel/cli 설치
- npm install --save-dev @babel/preset-env
    - babel 사용하기위한 preset-env 설치
    - preset-env 는 babel 사용시 필요한 플러그인들을 모아논 것 
- npm install --save-dev @babel/plugin-proposal-class-properties
- .babelrc
    - babel 설정 파일
    - 프로젝트 루트에 설치한다. (babel-compile/ 하위)
```javascript
{
  "presets": ["@babel/preset-env"]
  "plugins": ["@babel/plugin-proposal-class-properties"]
}
```

- 편한 빌드를 위해 package.json 에 명령어를 추가한다.
    - "build": "babel src/js -w -d dist/js"
    - npm build 실행시 빌드됨
```java
{
  "name": "babel-compile",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "babel src/js -w -d dist/js"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "@babel/cli": "^7.5.5",
    "@babel/core": "^7.5.5",
    "@babel/preset-env": "^7.5.5"
  }
}
```
