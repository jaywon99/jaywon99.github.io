---
title: Named Routes - v0.1.1
cover-image: hipster.jpg
categories: Flutter
tags: [Flutter, Routines App]
summary: AboutPage를 StatefulWidget에서 StatelessWidget으로 변경하고, 불필요한 코멘트 제거하고, 이와 함께 Named Routes로 기본 라우팅을 변경하였습니다.

---

## 오늘의 목차

1. 필요없는 코멘트 제거
1. Named Route의 지정
1. sidemenu 코드의 수정

### 필요없는 코멘트 제거

v0.1.0에서 about.dart를 main.dart를 복제해서 만들었더니, AboutPage가 StatefulWidget으로 만들어졌습니다. 그런데 굳이 그럴 필요가 없다고 하네요.. (사실 StatefulWidget과 StatelessWidget의 차리를 아직 잘 모르겠습니다.)

그래서 하는 김에 StatefulWidget을 StatelessWidget으로 바꾸면서 코멘트를 정리하였습니다.

소스코드: [lib/widgets/about.dart](https://github.com/jaywon99/routines/blob/bfc868133b811b0b6864a76e14feda0c0229ebb6/lib/widgets/about.dart)

### Named Route의 지정

메인페이지에서 Named 라우트를 추가하였습니다.
[https://flutter.dev/docs/cookbook/navigation/named-routes](https://flutter.dev/docs/cookbook/navigation/named-routes) 을 참고하여 작업하였으나, 리로드시에 오류가 나왔습니다.

#### 1차시도
```
class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Routines App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      initialRoute: '/',
      routes: {
        // When navigating to the "/" route, build the MyHomePage widget.
        '/': (context) => MyHomePage(title: 'Routines!'),
        // When navigating to the "/second" route, build the AboutPage widget.
        '/about': (context) => AboutPage(),
      },
    );
  }
}
```

발생오류
```
 ══╡ EXCEPTION CAUGHT BY WIDGETS LIBRARY ╞═══════════════════════════════════════════════════════════
flutter: The following assertion was thrown building Builder(dirty):
flutter: The builder for route "/" returned null.
flutter: Route builders must never return null.
flutter:
```

원인을 찾아보았으나, 가장 그럴듯한 설명이 **왜 당신은 StatefulWidget을 쓰죠** 였습니다. StatefulWidget을 쓰는 이유는 처음에 그렇게 만들어져서 쓴건데 말이에요.. 그래서, 이리 저리 고민하다가, 원 코드에는 **home:** 이 있는 것을 보고, 왠지 이 **home:** 이 라우팅의 "/"을 대신할 수 있지 않을까 하면서, 새로운 시도를 해 봤습니다.

#### 2차시도
```
class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Routines App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      initialRoute: '/',
      routes: {
        // When navigating to the "/second" route, build the AboutPage widget.
        '/about': (context) => AboutPage(),
      },
      // When navigating to the "/" route, it will launch home property
      home: MyHomePage(title: 'Routines!'),
    );
  }
}
```

오류코드는 아래와 같습니다.
```
 ══╡ EXCEPTION CAUGHT BY WIDGETS LIBRARY ╞═══════════════════════════════════════════════════════════
flutter: The following assertion was thrown building MaterialApp(dirty, state: _MaterialAppState#c443b):
flutter: If the home property is specified, the routes table cannot include an entry for "/", since it would
flutter: be redundant.
flutter: 'package:flutter/src/widgets/app.dart':
flutter: Failed assertion: line 172 pos 10: 'home == null ||
flutter:          !routes.containsKey(Navigator.defaultRouteName)'
flutter:
```

뭔가 오류가 바뀌면서 풀 수 있을 듯 해져서 다음과 같이 수정하였습니다.

#### 3차시도:
```
class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Routines App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      routes: {
        // When navigating to the "/second" route, build the AboutPage widget.
        '/about': (context) => AboutPage(),
      },
      // When navigating to the "/" route, it will launch home property
      home: MyHomePage(title: 'Routines!'),
    );
  }
}
```

오류가 발생하지 않았으며, 문제 없이 잘 동작합니다!

소스코드: [lib/main.dart](https://github.com/jaywon99/routines/blob/bfc868133b811b0b6864a76e14feda0c0229ebb6/lib/main.dart)

### sidemenu 코드의 수정

v0.1.0에서 언급한 것처럼 sidemenu의 Navigator를 한줄로 수정하였습니다.

Before:
```
onTap: () => {_navigateAbout(context)},
```

After:
```
onTap: () => {Navigator.popAndPushNamed(context, '/about')},
```

소스코드: [lib/widgets/sidemenu.dart](https://github.com/jaywon99/routines/blob/bfc868133b811b0b6864a76e14feda0c0229ebb6/lib/widgets/sidemenu.dart)

### Milestones

```
[X] v0.1.0 Empty Home + Menu List + About 화면
[X] v0.1.1 Named Routes의 추가 및 필요없는 코드 제거
[ ] v0.2.X (+) Button + Routine 관리 (색상 + 타이틀글 + subtitle 정도)
[ ] v0.3.X Home 화면 - Routine 체크 화면
[ ] v0.4.X 진행상황보기 - 달력으로 실행 내역 보기 (주간 레포트를 볼 수 있지 않을까?)
[ ] v0.5.X Ads 추가
[ ] v0.6.X InApp Purchase 추가
[ ] v1.0.X AppStore, PlayStore에 등록하기
[ ] v1.1.X Routine 프로퍼티 추가 - 매일 하는 일이 아닌 특정 요일 하는 일만 표시
[ ] v1.2.X Routing 프로퍼티 추가 - 숫자 입력하기 (예: 횟수, 시간, 등등)
```

