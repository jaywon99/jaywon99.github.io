---
layout: post
title: Electron 공부하면서 정리
date: 2020-07-22 18:42
category: Study
author: ["Jongpil Won"]
tags: [Study]
cover-image: hipster.jpg
summary: Electron 공부하면서 정리하기
---

# FYI - electron

Refer: [https://www.electronjs.org/docs](https://www.electronjs.org/docs)

- node / npm 필요
- install `npm install --save-dev electron`
- update package.json

    ```json
    {
      "name": "your-app",
      "version": "0.1.0",
      "main": "main.js",
      "scripts": {
        "start": "electron ."
      }
    }
    ```

- main.js와 index.html이 start point (sample 참조)
- main javascript과 render javascript간의 ipc 대에 공부 필요
    - main은 system dependent script를 (camera, dialog, bluetooth, etc)
    - renderer는 view 관련 script를
    - 서로간에 ipc로 통신 (event 방식) - 이벤트 설계를 잘 할 필요가 있음
    - 재밌는 것.. FileOpen 등은 main / renderer 모두에서 사용 가능
- main용 API들 - [https://www.electronjs.org/docs/api/structures](https://www.electronjs.org/docs/api/structures)

- 가볍게 만들어 본 zipviewer with bootstrap carousel
    - 리포는 추후 예정

