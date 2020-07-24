---
title: Electron-Vue Study 정리
author: ["Jongpil Won"]
cover-image: hipster.jpg
categories: Study
tags: [Study]
summary: Electron-Vue를 공부하면서 정리하기
---

# FYI - electron-vue

[https://simulatedgreg.gitbooks.io/electron-vue/content/ko/getting_started.html](https://simulatedgreg.gitbooks.io/electron-vue/content/ko/getting_started.html)

- install

    ```bash
    npm install -g vue-cli
    ```

- create project

    ```bash
    vue create myproject
    cd myproject
    vue add electron-builder
    ```

- project 실행

    ```bash
    npm run electron:serve
    ```

- boilerplate 기본 구조

    ```
    Project Home
    +- src
    +- public
    +- package.json
    +- vue.config.js

    src 아래
    +- background.js # 메인 js 파일 - (electron용 js파일)
    +- App.vue       # 메인 App.vue
    +- index.js      # renderer 메인 js 파일 (vue.js용 js파일)
    +- components    # vue.js components 파일
    | +- HelloWorld.vue # hello world vue 파일
    +- assets        # assets

    public아래
    +- index.html    # vue.js - index.html - 특별히 수정할 필요 없음
    +- favicon.ico   # favicon
    ```

- 생각하는 구조

    ```
    +- src
       +- background.js    # main/index.js를 읽어서 실행하는 것 하나 만들기
       +- main
       |  +- index.js      # background.js를 이름 변경하여 옮기기
       +- renderer
          +- app
          |  +- App.vue    # App.vue의 이동
          |     index.js   # index.js의 이동
          +-- ... #vue.js 용 구조를 그대로,   
    ```

- 그걸 위한 실행 명령

    ```bash
    mkdir -p src/main
    mkdir -p src/renderer/app
    mv src/background.js src/main/index.js
    mv src/index.js src/App.vue src/renderer/app
    mv src/components src/renderer
    mv src/assets src/renderer/assets
    echo "require('./main/index.js');" > src/background.js
    ```

- vue.config.js - vue.js용 vue.config.js와 유사

    ```jsx
    module.exports = {
        pages: {
            index: {
                entry: 'src/renderer/app/main.js',
                template: 'public/index.html',
                filename: 'index.html',
                title: 'Electron-Vue Study',
                chunks: ['chunk-vendors', 'chunk-common', 'index'],
            },
        },
        publicPath: '/',
    }
    ```

이제 이걸 기반으로 zipviewer를 만들어보자!

- 추가 Plugins
    - vue-electron - vue ↔  electron 쉽게 연결
- 기타
    - __static 참조 - [https://simulatedgreg.gitbooks.io/electron-vue/content/ko/using-static-assets.html](https://simulatedgreg.gitbooks.io/electron-vue/content/ko/using-static-assets.html)
    - 파일 읽기 - app.getPath(name) 을 이용하여 electron 디렉토리 접근 - [https://www.electronjs.org/docs/api/app#appgetpathname](https://www.electronjs.org/docs/api/app#appgetpathname)
    - db style의 엑세스 - nedb의 사용 - [https://github.com/louischatriot/nedb](https://github.com/louischatriot/nedb)

- 고민할 부분
    - static vs assets - 도대체 어떻게 쓰는 건지