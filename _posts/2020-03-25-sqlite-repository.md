---
title: SQLite Data Provider - v0.2.2
author: ["Jongpil Won"]
cover-image: hipster.jpg
categories: Flutter
tags: [Flutter, Routines-App]
summary: 이번에는 지난번에 만들어 놓은 bloc에 data provider로서 memory대신에 sqlite를 넣는 개발을 진행하였습니다. 관련 refactoring 및 provider를 추가하였습니다.

---

## 오늘의 목차
1. sqlite package 추가
1. repository로 refactoring
1. sqlite용 provider만들기

### sqlite package 추가
여기서는 flutter용 sqlite package중 sqflite라는 패키지를 추가할 예정입니다. 추가하는 방법은 /pubspec.yaml에 sqflite package를 추가하면 됩니다.

```
dependencies:
  flutter:
    sdk: flutter

  # The following adds the Cupertino Icons font to your application.
  # Use with the CupertinoIcons class for iOS style icons.
  cupertino_icons: ^0.1.2

  flutter_colorpicker: ^0.3.2
  sqflite: ^1.3.0
```

그리고, ide를 쓰는 경우에는 저장만 하면 되고, 그렇지 않으면 터미널에서 `flutter package get`을 하시면 됩니다.

많은 인터넷 페이지에서 path_provider 패키지를 추가하여 사용하고 있어서, 해당 패키지도 추가하였었으나, 이상하게도 오류가 나서 해당 패키지를 빼고 사용하였습니다.  그래서, DB용 디렉토리로는 getDatabaseDirectory를 사용할 예정입니다.
- https://github.com/flutter/flutter/issues/46679

### repository로 refactoring
이전 차수의 RoutinesMemoryDAO 라는 클래스는 독립적이어서 그냥 만들어 사용했으면 되었으나, 이번에는 sqlite용으로 새로 만들어야 해서, 몇가지 refactoring을 진행하였습니다. 

먼저 dao 디렉토리를 provider 디렉토리로 변경하였습니다. dao를 그대로 써도 되지만, 혹시라도 network 대역을 넣는다던가, firebase를 넣는다던가 등을 대비하여, 좀 더 제너럴한 명칭으로 변경하였습니다. 다만, 나중에 provider가 많아 진다면, 다시 dao로 넘어갈 수도 있겠죠..

다음으로, 해당 DAO에서 외부 메쏘드를 빼내어 abstract class를 만들어 repository로 이전하였습니다.

```
import 'package:routines/models/routine.dart';

abstract class RoutinesRepository {
  // find all routines
  Future<List<Routine>> findAll();

  // find routine by id.
  Future<Routine> findById(int id);

  // add or update routine.
  // if not exist, add it, else update it.
  Future<int> update(Routine routine);

  // Remove routine.
  Future<int> remove(Routine routine);
}
```

다른 인터넷 코드들을 보면, Repository를 만들고, Repository에 implementation을 inject해서 사용하는데, 그렇게 하고 보니, bloc에 repository를 inject하고, repository에 provider를 inject하는 모양이 되고, repository는 아무일도 안하고 provider를 호출만 하는 것 같아서, 굳이 그럴 필요가 있나 싶어서 그냥 abstract class로 만들었습니다. 물론, 각각 (ui, bloc, repository, provider)의 R&R이 지금과 달라지면 다시 고민해야 하겠지만, 지금 상태로서는 굳이 나눌 필요가 없어 보였습니다.

참고로 현재의 R&R은 다음과 같습니다.

* model:
* widgets: user interface와 직접적인 연결을 가지는 action만 핸들 
* blocs: ui와 model간의 인터페이스 
* repositories: abstract class만 존재함 (특별한 R&R없음. bloc에 바로 inject)
* providers: 실제 data storage와 business logic간의 조율
* data storage: data 읽고 쓰는 것과 직접적인 연관이 있는 부분 

참고로, RoutineMemoryDAO의 경우 `List<Routine> _routines`가 data storage에 해당하고, sqlite에서는 sqflite package + db file를 data storage로 봅니다.

그리고, 아래와 같이 코드를 넣어볼까 합니다.

* 화면 자체와 관련된 로직 (예: color picker button를 누르면 color picker dialog를 띄운다. list에서 하나를 선택하면 자세한 내역을 보여준다. 등)은 ui에서 해결
* data 자체의 변경이 일어나거나, business logic등이 들어가는 경우 bloc에서 해결
* 하나의 data provider에서만 일어나는 선택 (예: nextId를 뽑아오는 것 - memory에서는 필요하나 sqlite등에서는 sqlite가 직접 해결)은 provider가 해결
* data provider의 둘 이상의 method를 호출해서 문제를 해결해야 하는 경우는 현재는 bloc에서 해결 - (아마 이 부분이 복잡해지면, repository에서 해결해야 될 지도 모르겠음)

### sqlite용 provider 만들기
sqlite용 provider는 만들기가 아주 쉽습니다. 인터넷에 아주 많은 샘플이 있고, 그것을 가져다 사용하면 됩니다.

다만, 저는 몇가지를 바꿔 봤습니다. 하나는 table명 및 버젼관리를 따로 뺀 것이고, 다른 하나는 json to model converter를 model에서 provider로 뺀 것입니다.

table명 및 table version 관리를 DatabaseProvider에서 뺀 이유는, 혹시라도 테이블의 수가 많아지면, 한 곳에서 관리하기 보다 나눠서 관리하는 게 좋지 않을까 라는 생각에서 시작했으며, 개별 table들의 meta 테이블을 따로 하나씩 만들어서 관리하였다고 보시면 됩니다. 여기에 테이블명, 컬럼 등을 추가해볼까 합니다. 

다음으로 sqflite에서는 json을 이용하여 작업하므로, model과 json간의 변환이 발생하는데, 이것을 model에 두지 않고 sqlite provider들 안으로 옮겼습니다. 그 이유는 여러 provider에서 이용하는데, 그 때마다 해당 변환 작업을 model에 둔다면, model 자체가 좀 더 어수선해지지 않을까 생각해서, 혼자 쓸거면 혼자 가져가라는 컨셉으로 이 쪽으로 옮겨두었습니다.

### 고민사항
현재 sqlite를 이용하고, bloc pattern을 이용해서 개발을 진행하고 나니, injection에 대한 고민이 많이 생겼습니다. java spring framework을 사용할때는 config파일이나 annotation을 이용해서 넣었던 것들인데, 지금은 모두 code에 박혀있고, TODO로 Problems에 넣어두었기 때문입니다. 조만간에 injection 부분을 고려해야 할 것 같습니다.

1. DatabaseProvider에 개별 Table Info를 Inject 할 방법 (현재는 생성자)
   * static + annotation으로 하는 방법이 있을지..
1. RoutineBloc에 RoutinesSQLiteProvider를 Inject 할 방법 (현재는 생성자)


---

v0.2.2 최종본은 [https://github.com/jaywon99/routines/tree/37a8a8cf3819a6bcb4a50546b61ed20f8dfbe276](https://github.com/jaywon99/routines/tree/37a8a8cf3819a6bcb4a50546b61ed20f8dfbe276) 에 있습니다.

이와 같이 sqlite를 이용하여, 자료를 저장할 수 있도록 변경하였습니다. 다음에는 Home (Today라고 이름을 바꿀 예정입니다.) 화면을 만들어 보겠습니다. ui, bloc, repository, provider를 어떻게 나눠서 진행하지 고민해보겠습니다.

### Milestones

```
[X] v0.1.0 Empty Home + Menu List + About 화면
[X] v0.1.1 Named Routes의 추가 및 필요없는 코드 제거
[X] v0.2.0 (+) Button + Routine 관리 (색상 + 타이틀글 + subtitle 정도)
[X] v0.2.1 Persistence 분리 (in-memory) + Bloc
[X] v0.2.2 Routine의 저장 (persistence) - sqlite
[ ] v0.3.X Home 화면 - Routine 체크 화면
[ ] v0.4.X 진행상황보기 - 달력으로 실행 내역 보기 (주간 레포트를 볼 수 있지 않을까?)
[ ] v0.5.X Ads 추가
[ ] v0.6.X InApp Purchase 추가
[ ] v1.0.X AppStore, PlayStore에 등록하기
[ ] v1.1.X Routine 프로퍼티 추가 - 매일 하는 일이 아닌 특정 요일 하는 일만 표시
[ ] v1.2.X Routing 프로퍼티 추가 - 숫자 입력하기 (예: 횟수, 시간, 등등)
```





