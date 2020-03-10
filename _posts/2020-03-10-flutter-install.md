---
title: Flutter install
cover-image: hipster.jpg
summary: 기존에 만들던 Ionic 4 Framework 기반의 앱은 만들다 말고, 주변의 의견에 따라 이번에는 Flutter용 앱을 개발해보고자 합니다. 그것을 위해 필요한 flutter 환경을 설치/설정하는 글입니다.
---

기존에 만들던 Ionic 4 Framework 기반의 앱은 만들다 말고, 주변의 의견에 따라 이번에는 Flutter용 앱을 개발해보고자 합니다.

본 프로젝트는 *Routines!* 라는 앱으로, 매일 매일 할 일을 알려주고, 실행 후 표시하고, 진행상황을 보기좋게 표현하여, 매일매일 루틴을 만드는 일을 도와줄 앱입니다. 이 역시 많이 나와 있고, 팔고도 있지만, 공부삼아 만들어 보려고 합니다. 또한 소스도 공개할 예정입니다.

소스는 [Github Repo - jaywon99/routines](https://github.com/jaywon99/routines.git)을 통해 공개할 예정이며, 매 진행마다 commit을 할 예정입니다. 이번에는 쉬지 않고 잘 진행되기를 기원합니다.

### Miltstones

```
[ ] v0.1.X Empty Home + Menu List + About 화면
[ ] v0.2.X (+) Button + Routine 관리 (색상 + 타이틀글 + subtitle 정도)
[ ] v0.3.X Home 화면 - Routine 체크 화면
[ ] v0.4.X 진행상황보기 - 달력으로 실행 내역 보기 (주간 레포트를 볼 수 있지 않을까?)
[ ] v0.5.X Ads 추가
[ ] v0.6.X InApp Purchase 추가
[ ] v1.0.X AppStore, PlayStore에 등록하기
[ ] v1.1.X Routine 프로퍼티 추가 - 매일 하는 일이 아닌 특정 요일 하는 일만 표시
[ ] v1.2.X Routing 프로퍼티 추가 - 숫자 입력하기 (예: 횟수, 시간, 등등)
```

### Flutter 설치

참고페이지 - [Flutter Install](https://flutter.dev/docs/get-started/install)

위의 페이지에 시스템 별 링크를 따라가면 설치방법이 너무 잘 되어있어서, 그대로 하면 됩니다. 제 경우는 IDE로 Visual Code를 선택하여 설치하였습니다. 본인의 취향에 맞춰 진행하면 됩니다.

저는 OSX를 사용중인데, flutter 설치 및 업그레이드를 제 취향에 맞춰 /opt/flutter 디렉토리에 설치하였습니다.

```
sudo mkdir -p /opt/flutter
sudo chown [my-account] /opt/flutter
cd /opt/flutter
unzip [downloaded-flutter.zip]
export PATH=$PATH:/opt/flutter/bin
```

또한 .zshrc에 PATH를 추가하였습니다. 

### vs code의 extension 설치

참고페이지 - [Set up an editor](https://flutter.dev/docs/get-started/editor?tab=vscode)

위의 페이지에 Editor 및 extension을 설치하는 것도 잘 되어있습니다. 다만, 저는 제 취향으로 아래의 extension을 추가로 하였습니다. (참고로 제 theme은 dracula입니다.)

* Dart extension
* Flutter extension 
* indent-rainbow - indentation을 색으로 표시해줍니다. (취향에 따라 색조정 가능하며, indentation을 일목요연하게 보여줍니다.)
* Rainbow Brackets - 이것은 소괄호'(/)'/중괄호'{/}'를 서로 다른 색상으로 보여줍니다. indent-rainbow처럼 색상을 수정할 수 있으면 더 좋았을 것 같습니다.

## 자 그럼 출발!

- - -
