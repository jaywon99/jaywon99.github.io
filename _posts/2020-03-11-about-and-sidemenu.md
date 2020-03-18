---
title: Sidemenu and About - v0.1.0
author: ["Jongpil Won"]
cover-image: hipster.jpg
categories: Flutter
tags: [Flutter, Routines App]
summary: 두번째 Task는 about page를 만들어 넣고, 페이지 접근을 위해 sidemenu를 넣었습니다.

---

두번째 Task는 초기 생성 패키지에 about page를 만들어 넣고, 해당 페이지 접근을 위해 sidemenu를 넣는 것입니다.

### about page

About 페이지를 만드는 것은 쉽습니다. 일단 about page에 들어갈 내용이 하나도 없기 때문인데요.. 그냥 Title에 About을 넣고, body에 about이라는 정보만 들어갈 예정입니다.

일단 대부분의 page나 widgets을 lib/widgets 폴더에 넣을 예정이므로, lib/widgets 폴더를 만들어 주신 후에, about.dart를 만들어 줍니다.

원래는 처음부터 만들어야 하지만, 처음이므로 main.dart를 복사하고, MyApp부분을 제거하고, MyHomePage를 AboutPage로 이름 바꿔줍니다.
여기에 AboutPageState가 남지만, 현재는 뭐하는 건지 모르니, 그냥 둡니다. 그리고, build 부분에서 Column의 children 영역만 넣고자 하는 형식으로 바꿉니다. 그리고, 페이지 title도 바꿔줍니다. (코멘트도 좀 지워야 하나.. 처음이니..)

Navigation방식은 new page and back style로 할 예정입니다.

소스: [lib/widgets/about.dart](https://github.com/jaywon99/routines/blob/237544d0d5976078ee644b8348cd2b7042e64c41/lib/widgets/about.dart)  
참고: [https://flutter.dev/docs/cookbook/navigation/navigation-basics](https://flutter.dev/docs/cookbook/navigation/navigation-basics)

현재 상태 만으로는 내용을 볼 방법이 없습니다.

### sidemenu 만들기

다음으로는 sidemenu를 만들예정입니다.

처음에는 [https://medium.com/@maffan/how-to-create-a-side-menu-in-flutter-a2df7833fdfb](https://medium.com/@maffan/how-to-create-a-side-menu-in-flutter-a2df7833fdfb) 를 참고하여 공부하였으며, 실제로 그 코드를 많이 차용하였습니다.

처음에는 sitemenu에 넣을 것이, Home과 About밖에 없어서, 아주 가볍게 만들었습니다.

참고 페이지에는 5개의 메뉴가 있으나, 2개만 남기고 다 지우고, 그 두개의 text 및 icon을 바꿔줍니다. (어떤 icon이 있는지는 [https://api.flutter.dev/flutter/material/Icons-class.html](https://api.flutter.dev/flutter/material/Icons-class.html) 참조)

소스: [lib/widgets/sidemenu.dart](https://github.com/jaywon99/routines/blob/237544d0d5976078ee644b8348cd2b7042e64c41/lib/widgets/sidemenu.dart)
다른참조: [https://flutter.dev/docs/cookbook/design/drawer](https://flutter.dev/docs/cookbook/design/drawer)

### sidemenu 추가

main.dart에 sidemenu를 추가합니다. sidemenu를 widget으로 잘만 만들면, main에 붙이는 것은 아주 쉬었습니다. MyHomeWidget의 build부분에서 appBar를 선언하는 바로 다음에 (property여서 순서는 상관없습니다.) drawer라는 프로퍼티로 생성해 넣으면 됩니다.

바꾸는 김에, title이랑 app의 title도 바꿔줍니다.

이제 reload하면, 좌상단에 햄버거 버튼이 나오고, 클릭시 메뉴도 잘 나옵니다.

소스: [lib/main.dart](https://github.com/jaywon99/routines/blob/237544d0d5976078ee644b8348cd2b7042e64c41/lib/main.dart)

### 문제 발생

v0.1.0 소스에서는 문제가 없었으나, 처음 만든 코드에서는 about페이지를 띄우는 데 문제가 있었습니다.

처음 sidemenu의 'About' ListTitle의 onTap은 참조한 페이지처럼

```
onTap: () => {Navigator.of(context).pop()}
```

이었습니다. 이 의미는 마지막에 띄운 widget을 닫으라는 것으로, 우리는 about 페이지를 띄워야 해서 아래와 같이 바꿨습니다.

```
onTap: () => {    
    Navigator.of(context).pop();
    Navigator.push(
      context,
      MaterialPageRoute(builder: (context) => AboutPage()),
    );
  }
```

그런데, 제가 dart를 잘 몰라서 그런지, 위와 같이 두개의 명령을 함께 넣으면 syntax 오류가 납니다. 다른 언어의 anonymous function처럼 사용하면 안되나 봅니다. 
그래서 아래와 같이 수정하였습니다.

```
onTap: () => {_navigateAbout(context)},
```

```
_navigateAbout(BuildContext context) async {
  Navigator.of(context).pop();
  Navigator.push(
    context,
    MaterialPageRoute(builder: (context) => AboutPage()),
  );
}
```

위의 두 명령을 순차적으로 실행하는 하나의 method를 만들고 해당 method를 실행하도록 한거죠. 그랬더니 원하는 방식대로 잘 동작하였습니다.

### 프로젝트 추가
* 위의 navigation을 공부하다보니 Named Route라는 것이 있었습니다. 꽤 재밌을 것 같아서, Milestone에 0.1.1로 Named Milestone을 사용하는 것을 추가하려 합니다.
* about 페이지가 중앙 정렬로 나오다 보니, about페이지로서 좀 이상하게 보입니다. 그래서 중앙정렬, 상단정렬, 상단 마진, scroll이 되도록 수정해보려고 합니다. 아마 v1.0.0때 넣지 않을까 합니다. (무슨 말을 써야 할지 모르겠어서요..)

### Milestones

```
[X] v0.1.0 Empty Home + Menu List + About 화면
[ ] v0.1.1 Named Routes의 추가 및 필요없는 코드 제거
[ ] v0.2.X (+) Button + Routine 관리 (색상 + 타이틀글 + subtitle 정도)
[ ] v0.3.X Home 화면 - Routine 체크 화면
[ ] v0.4.X 진행상황보기 - 달력으로 실행 내역 보기 (주간 레포트를 볼 수 있지 않을까?)
[ ] v0.5.X Ads 추가
[ ] v0.6.X InApp Purchase 추가
[ ] v1.0.X AppStore, PlayStore에 등록하기
[ ] v1.1.X Routine 프로퍼티 추가 - 매일 하는 일이 아닌 특정 요일 하는 일만 표시
[ ] v1.2.X Routing 프로퍼티 추가 - 숫자 입력하기 (예: 횟수, 시간, 등등)
```

