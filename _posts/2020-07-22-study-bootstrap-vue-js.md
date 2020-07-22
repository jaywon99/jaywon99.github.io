---
layout: post
title: Bootstrap-Vue 공부하며 정리
date: 2020-07-22 18:39
category: Study
author: ["Jongpil Won"]
tags: [Study]
cover-image: hipster.jpg
summary: Bootstrap-Vue를 공부하면서 정리하기
---

# FYI - bootstrap-vue

[https://bootstrap-vue.org/docs](https://bootstrap-vue.org/docs)

기본적으로 bootstrap div등의 component를 쓰지 않고, bootstrap-vue용 component 사용

### Install

```bash
npm install bootstrap-vue --save
```

### Put in entry point - main.js

```jsx
import Vue from 'vue'
import { BootstrapVue, BootstrapVueIcons } from 'bootstrap-vue'

import 'bootstrap/dist/css/bootstrap.css'
import 'bootstrap-vue/dist/bootstrap-vue.css'

// Install BootstrapVue
Vue.use(BootstrapVue)
// Optionally install the BootstrapVue icon components plugin
Vue.use(BootstrapVueIcons)
```

Vue CLI 3 plugin - `vue add bootstrap-vue`

### Bootstrap Icon 사용법

일단 위에서 BootstrapVueIcons를 넣었으므로, `<b-icon-arrow-up></b-icon-arrow-up>` 이런식으로 넣으면 됨. `<b-icon-이름></b-icon-이름>`