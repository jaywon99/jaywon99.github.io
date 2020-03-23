---
title: Memory DAO / Bloc / Named route with parameter
author: ["Jongpil Won"]
cover-image: hipster.jpg
categories: Flutter
tags: [Flutter, Routines App]
summary: 원래 sqlite를 추가한 persistence layer를 붙이려고 했으나, 그 전에 bloc에 대해 먼저 얘기하고 적용하는 게 좋다고 판단하여, 이번에는 bloc의 적용 및 in-memory style persistence layer를 추가하였습니다. 그와 함께 약간의 refactoring 및 named route에 parameter를 이용하는 것을 추가하였습니다.

---

## 오늘의 목차
1. Memory DAO 만들기
1. Bloc 적용 - 위에서 만든 DAO를 적용
1. LoadingWidget 만들기
1. refactoring - widget 이름들 수정
1. editform의 named route로 면경

### Memory DAO 만들기
Memory DAO는 특별한 것은 아니고, 전에 routines/page.dart에 들어있던 \_routines 로컬 변수를 마치 persistence layer인 것처럼 사용하기 위해 바깥으로 빼서 사용할 수 있도록 만들었습니다. 위치는 lib/dao/memory/routines.dart 이고, class명은 RoutinesMemoryDAO입니다. (path의 역순입니다.)

코드를 보시면 아시겠지만, 비록 in-memory이지만, 미래를 위해 일단은 Future/await/async로 도배를 하였습니다. Future/await/async에 대해서는 많은 인터넷 문서가 있으니 참조하세요. 저는 이 문서가 제 머리속에서 정리가 되더군요. (참고로 Node.js를 쓰면서 promise모델이나 coroutine등에 대해서는 이미 알고 있었습니다.)
[https://www.didierboelens.com/2019/01/futures---isolates---event-loop/](https://www.didierboelens.com/2019/01/futures---isolates---event-loop/)

```
import 'package:flutter/material.dart';
import 'package:routines/models/routine.dart';

// interface style로 만들어야 하지 않을지..
class RoutinesMemoryDAO {
  List<Routine> _routines;
  int _lastId = 0;

  RoutinesMemoryDAO() {
    _routines = List<Routine>();
    // PUT INITIAL DATA for testing
  }

  // get all routines
  Future<List<Routine>> findAll() async {
    // PUT DELAY for TESTING
    // return _routines;
    return Future.delayed(Duration(seconds: 1)).then((onValue) => _routines);
  }

  // find routine by id.
  Future<Routine> findById(int id) async =>
      Future.value(_routines.firstWhere((r) => r.id == id));

  // add or update routine.
  // if not exist, add it.
  // if exist, update it.
  Future<int> update(Routine routine) async {
    // print("before ${routine.id} ${routine.title}");
    if (routine.id == -1 || routine.id == null) {
      // NEW data
      routine.id = (await \_nextId());
    }
    int idx = _routines.indexWhere((item) => item.id == routine.id);
    if (idx == -1) {
      _routines.add(routine);
    } else {
      _routines[idx] = routine;
    }
    // print("after ${routine.id} ${routine.title}");
    return Future.value(routine.id);
  }

  // Remove from list.
  Future<int> remove(Routine routine) {
    _routines.removeWhere((item) => item.id == routine.id);
    return Future.value(1);
  }

  Future<int> \_nextId() async {
    _lastId = \_lastId + 1;
    return Future.value(_lastId);
  }
}
```

위의 코드를 보시면 일부로 Future/async/await를 넣은 것 이외에는 전형적인 코드입니다. 다만, update메쏘드가 좀 다릅니다.
처음에는 update method가 다음과 같았습니다.

실제 위의 update코드의 코멘트 부분의 프린트를 보면, 출력 순서가 다음과 같습니다.
```
flutter: before -1 Routine1
flutter: before -1 Routine2
flutter: before -1 Routine3
flutter: after 1 Routine1
flutter: after 2 Routine2
flutter: after 3 Routine3
```

원래는 after가 모두 0이었는데요, 이걸 수정하기 위해 여러번의 시도 끝네 위와 같은 코드를 만들었습니다.

참고로 시도 1은 오류로, \_maxId 를 리스트를 scan하여 마지막 값 다음 값을 주도록 하였습니다. DB에서 maxId값을 찾아서, maxId+1 을 하도록 만들면, id가 충돌하던 것과 똑같은 현상이 발생한거죠.
```
  Future<int> _maxId() async {
    if (_routines.length == 0) {
      return Future.value(0);
    }
    return Future.value(
        (_routines.reduce((curr, next) => curr.id > next.id ? curr : next)).id);
  }
```

시도2는 update 메쏘를 synchronized라는 plugin을 이용해서 lock을 걸었습니다. 그래서 update 자체를 동시 시행을 막았습니다. 그 결과로는 잘 돌았습니다. 그러면 신규 추가 부분에 동시성이 막혀서 문제를 해결하였죠.

```
  Future<int> update(Routine routine) async {
    return await _lock.synchronized(() async {
... 코드 생략
    };
  }
```

시도 3는 db의 해당 문제를 푸는 방법인 sequence db나 last_id 등의 방법을 써보면 어떨까 해서, maxId를 가져오는 대신에 nextId를 가져오는 방법으로 수정하였습니다. 그 결과로 내부에 최대값을 보관해야 하는 이슈가 있으나, 더 간결하게 문제를 해결할 수 있었습니다.

### Bloc 적용 - memory dao를 이용하도록
Bloc = Business LOgic Component로, BLoC pattern이 있습니다. 인터넷을 찾아보면 아주 많은 문서를 찾아볼 수 있는데요, ( [https://www.raywenderlich.com/4074597-getting-started-with-the-bloc-pattern](https://www.raywenderlich.com/4074597-getting-started-with-the-bloc-pattern) ). 쉽게 말하면 (복잡하게 말하면, 더 다양할 수 있으나) flutter에서는 widget과 widget간의 통신을 없애, widget간의 dependency를 줄이고, 대신에 bloc라는 component를 이용하여 widget간의 연결을 한다는 것입니다. 그렇다면, widget - widget 통신이 아닌 widget - bloc - widget 통신으로 바뀐다는 것인데요, 장점이 무엇일 지 의문이 될 수 있으나, widget - bloc - widget이 asynchronous하게 동작하고 또한 stream을 이용하여 이득을 볼 수 있습니다. 요즘 개발 추세가 가급적이면 객체들간의 dependency를 줄이는 부분이어서, 적용해 볼 만 하다고 봅니다.

그리고, Bloc을 widget간에 어떻게 주고 받을지의 문제가 남는데요, 위의 참조 문서에 보면 bloc_provider.dart라는 방법을 이용합니다. 이럴 경우, main에서 필요한 bloc을 다 넣어준다면, 모든 widget에서 뽑아 사용할 수 있을 것 같습니다.

### LoadingWidget의 추가
그러다 보니, loading시에 시간이 걸릴 수 있어서 Loading Widget을 만들어 추가하였습니다. 비슷하게 FutureBuilder를 이용하는 future_loading_widget등도 만들 수 있지 않을까 생각됩니다. 아! 찾아보니 이미 좋은 샘플이 있네요. [https://api.flutter.dev/flutter/widgets/FutureBuilder-class.html](https://api.flutter.dev/flutter/widgets/FutureBuilder-class.html)

소스코드: [/lib/widget/common/loading\_widget.dart](https://github.com/jaywon99/routines/tree/c841286f1c56389b2a4cff9c383110455bdb5540/lib/widget/common/loading_widget.dart)

### 그와 함께 refactoring - widget 파일명 및 이름 수정
Widget이름을 좀 수정하였습니다. Widget이름과 directory 및 path 이름을 맞춰볼 수 있을 것 같아서, 정리를 좀 하였습니다.

* RoutinesPage이면 /widget/routines/page.dart
* RoutinesEditForm이면 /widget/routines/edit\_form.dart

이런 형식입니다. 좀 더 찾아가기 쉬우리라 생각됩니다. Page와 Body를 나눌 생각도 해 봤으나, 현재로서는 too much라고 생각되어 진행하지 않았습니다.

### named route의 변경 및 editform의 적용
이번에는 named route를 쓰는 방법을 살짝 변경하였습니다. 

1. main에 패스를 텍스트로 바로 적는 문제점의 수정

   named route를 쓸데, main에 path를 적어놓았습니다. 근데, 이 부분이 가급적이면 텍스트를 코드 본문에 적지 말자는 그런 방향성이랑 안 맞아서, 고민하였습니다. 그러다가 몇몇 인터넷 페이지를 보니, widget에 routeName이라는 const를 정의하고, 그것을 이용해서 적용하더군요. 그러나, 이 방법은 Widget과 Widget간의 직접 연결이 생성되는 문제(논란거리?)가 생깁니다. bloc을 쓸데, widget과 widget간의 연결이 싫어서 bloc을 썼는데, routing을 위해서 다시 쓴다고 하니.. 좀 이상해질 것 같았습니다. 

    그래서, 간단하게 const 들로 되어있는 routing name을 쓰기로 했습니다. routing name은 const에서 뽑아쓰고, 연결은 main에서 하는 거죠..

1. edit\_form의 호출을 직접 호출에서 named route로 변경

   edit\_form의 경우 현재 직접 호출이어서, 이것을 어떻게 named route로 변경할 까 하다가, named route with parameter라는 부분을 찾아서 적용하였습니다. [https://flutter.dev/docs/cookbook/navigation/navigate-with-arguments](https://flutter.dev/docs/cookbook/navigation/navigate-with-arguments)

   onGenerateRoute를 쓰게 되면, 코드를 좀 넣어야 했는데, routes를 그냥 쓰는 방법도 있어서, 그것을 써서 해결하도록 하였습니다.

```
      routes: {
        Const.ROUTE_ABOUT_PAGE: (context) => AboutPage(),
        Const.ROUTE_ROUTINE_LIST_PAGE: (context) => RoutineListPage(),
        Const.ROUTE_ROUTINE_EDIT_FORM: (context) =>
            RoutineEditForm(routine: ModalRoute.of(context).settings.arguments),
      },
```

```
    final resultRoutine = await Navigator.pushNamed(
      context,
      Const.ROUTE_ROUTINE_EDIT_FORM,
      arguments: routine, // 여기에 파라메터를 넘긴다.
    );
```

---

v0.2.1 최종본은 [https://github.com/jaywon99/routines/tree/c841286f1c56389b2a4cff9c383110455bdb5540](https://github.com/jaywon99/routines/tree/c841286f1c56389b2a4cff9c383110455bdb5540)에 있습니다.

위와 같이 bloc의 적용, in-memory class를 이용한  persistence의 분리, 그리고 named route with parameter를 적용하였습니다. 다음번에는 sqlite를 적용해 볼 예정입니다.

### Milestones

```
[X] v0.1.0 Empty Home + Menu List + About 화면
[X] v0.1.1 Named Routes의 추가 및 필요없는 코드 제거
[X] v0.2.0 (+) Button + Routine 관리 (색상 + 타이틀글 + subtitle 정도)
[X] v0.2.1 Persistence 분리 (in-memory) + Bloc
[ ] v0.2.X Routine의 저장 (persistence) - sqlite
[ ] v0.3.X Home 화면 - Routine 체크 화면
[ ] v0.4.X 진행상황보기 - 달력으로 실행 내역 보기 (주간 레포트를 볼 수 있지 않을까?)
[ ] v0.5.X Ads 추가
[ ] v0.6.X InApp Purchase 추가
[ ] v1.0.X AppStore, PlayStore에 등록하기
[ ] v1.1.X Routine 프로퍼티 추가 - 매일 하는 일이 아닌 특정 요일 하는 일만 표시
[ ] v1.2.X Routing 프로퍼티 추가 - 숫자 입력하기 (예: 횟수, 시간, 등등)
```

