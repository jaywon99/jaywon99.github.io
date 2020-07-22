---
layout: post
title: vue.js 공부하며 정리
date: 2020-07-22 18:31
category: Study
author: ["Jongpil Won"]
tags: [Study]
cover-image: hipster.jpg
summary: vue.js를 공부하면서 정리하기
---

# FYI - vue.js

Refer: [https://kr.vuejs.org/v2/guide/](https://kr.vuejs.org/v2/guide/)

Study repo: [https://github.com/jaywon99/vue-study.git](https://github.com/jaywon99/vue-study.git)

### 설치 및 초기화 - vue-cli를 사용하면 프로젝트 만들기 쉬움
    1. install vue-cli (npm install -g @vue/cli @vue/cli-service-global)
    2. create project (vue create folder)
    3. install sass (npm install sass-loader node-sass --save-dev)
    4. add plugin router (vue add router vuex)
    5. build project!!!

### vue-router
- &lt;router-view&gt;가 필요 - 일반적으로 App.vue에 등록 (sidebar등과 같이)
- router table이 필요 - router/index.js 등에 등록해서 main.js에서 불러서 등록

    ```jsx
    import TodoList from '@/App-TodoList.vue'
    import HelloWorld from '@/App-HelloWorld.vue'

    Vue.use(Router);

    const routes = [
        { path: '/', component: TodoList },
        { path: '/hello', component: HelloWorld }
    ]

    export default new Router({
        mode: 'history',
        routes: routes,
    });
    ```

    ```jsx
    import Vue from 'vue'
    import App from './App.vue'
    import router from '@/router/index'

    ...

    new Vue({
        render: h => h(App),
        router,
    }).$mount('#app')
    ```

- vue-router에 링크를 다는 방법
    - HTML: `<router-link to="/foo">Go to Foo</router-link>`
    - Javascript: `this.$router.push('/')`

### 디렉토리 구조
- Component별로 dot-vue 파일을 만들어서 사용 - https://kr.vuejs.org/v2/guide/single-file-components.html
- vuexy등 기존의 template을 이용하면, 디렉토리 구조등을 그대로 쓸 수 있음. 그렇지 않으면 (내 기준)

    ```
    + config
    + public
    | + index.html & ...
    + vue.config.js (...)
    + package.json 
    + src
    +-+ api (외부 api 연동을 여기에)
      + app (main.js & App.vue)
      + assets (외부 asset들) - 여기에 icon을 넣을 방법을 찾아보자.
      + components (공동 사용 components)
      + config (config)
      + router (routing 정보)
      + services (vuexy에서 이용 - 개인적인 것은 잘 모르겠음)
      + store (vuex용)
      + utils (util들)
      + views 
      +-+ index.vue (home)
        + sub-folder (namespace?)
        +-+ index.vue (namespace home)
        | + modules (본 페이지에서만 사용하는 component들)
        | + sub-folder (본 namespace아래의 namespace?)
        + ...
    ```

### 기타
- axios를 이용해서 api를 연동 (이게 쉬움)
- 실제 개발을 들어가기 전에 화면을 그려놓으면 일하기 편함. 그렇지 않으면 이동이 아주 잦을거라 예상됨
    - 필요하면, 종이 대신에 펜으로 그리는 노트 이용 - capture가 편한지 모르겠음.
- vscode이용시 vetur랑 tsconfig.json / jsconfig.json 을 쓰면, import가 쉬워짐

    ```json
    {
      "compilerOptions": {
        "baseUrl": ".",
        "paths": {
          "*": ["src/*"],
          "test/*": ["test/*"]
        }
      }
    }
    ```

### Extra
- vue-cli 공부 및 build방법
- 배포시 docker VS s3 bucket - CORS 조심
    - [https://webruden.tistory.com/115](https://webruden.tistory.com/115)
-
