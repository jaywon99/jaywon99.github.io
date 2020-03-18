---
title: Routines List 관리 화면
cover-image: hipster.jpg
categories: Flutter
tags: [Flutter, Routines App]
summary: Routines의 목록 관리 (추가, 수정, 삭제)를 제공하는 화면을 사용하였습니다. 사용한 위젯은 (ListTile, ListView, TextFormField, ColorPicker) 등입니다.

---

## 오늘의 목차

1. 자료 구조 정리
1. 리스트에서 Item 보여줄 형식 정하기 (ListTile Widget)
1. 리스트 형식으로 보여주기 (ListView Widget)
1. Swipe로 삭제하기 (Dismissible)
1. Routine 입력/수정 화면
1. 화면간 자료의 주고 받음
1. ColorPicker를 위한 Widget

### 자료 구조의 정리

만들고자 하는 "Routines!" App의 경우, Routine을 설명하는 항목으로, 먼저 제목(title), 소제목(subtitle), 색상(color) 이 세가지를 선택하였습니다.

```
class Routine {
  int id;
  String title;
  String subtitle;
  Color color;

  Routine({this.id, this.title, this.subtitle, this.color});  
}
```

참조: (/lib/models/routines.dart)[https://github.com/jaywon99/routines/blob/57f1d144a6fd6aa17e1eed1065e24855669c95a5/lib/models/routines.dart]

### 리스트에서 Item 보여줄 형식 정하기 (ListTile Widget)

ListTile Widget은 ListView Widget 사용시, 내부에 보여줄 Child의 형식 중 가장 보편적인 것을 만들어 놓은 Widget으로, 앞에 아이콘, 제목, 소제목, 그리고 오른편에 아이콘 형식의 뷰를 제공하도록 이미 만들어진 Widget입니다. 좀 더 복잡한 것이 필요한 분은, 스스로 Widget을 만들어서 사용하면 됩니다만, 여기서 제가 원하는 것은 이정도면 충분한 것이어서 이를 사용하였습니다.

ListTile (https://api.flutter.dev/flutter/material/ListTile-class.html)[https://api.flutter.dev/flutter/material/ListTile-class.html]
* title: 메인 텍스트
* subtitle: 서브 텍스트 - 메인텍스트 아래에 좀 작은 글씨로 적힘
* leading: 텍스트 앞에 보여줄 아이콘 (또는 Widget)의 공간
* trailing: 텍스트 뒤에 보여줄 아이콘 (또는 Widget)의 공간
* onTap: Tile 탭 시에 호출되는 callback

![화면 #1:](/img/0.2.0-routines-color-rect.png)

코드
```
  Widget build(BuildContext context) {
    return ListTile(
      title: Text('${routine.title}'),
      subtitle: Text('${routine.subtitle}'),
      leading: Container(
        width: 20.0,
        color: routine.color,
      ),
    );
  }
```

처음에는 Color Rectangle을 썼으나, AvatarCircle도 모양이 이쁜 것 같아서 그냥 사용하기로 하였습니다.

![화면 #2:](/img/0.2.0-routines-color-circle.png)

코드
```
  Widget build(BuildContext context) {
    return ListTile(
      title: Text('${routine.title}'),
      subtitle: Text('${routine.subtitle}'),
      leading: CircleAvatar(backgroundColor: routine.color,),
    );
  }
```

### 리스트 형식으로 보여주기 (ListView Widget)
위에 만들어진 ListTile을 하나로 모아서 보여주는 ListView Widget을 만듭니다. ListView는 혼자서 독립적일 수 있으나, 현재는 이를 하나의 페이지로 놓을 예정이어서, Scaffold를 만들고, 거기레 ListView를 추가, 그리고 FloatingActionButton, SnackBar까지 추가합니다.

FloatingActionButton 및 Scaffold는 이미 시작부터 써 보셨을 테니, 여기서는 Skip하고 ListView의 사용만 설명합니다.

ListView를 쓰는 방법은 자체 Child를 넣는 방법과 itemBuilder를 쓰는 방법, 두가지가 있는 것으로 보입니다.

```
class RoutineList extends StatefulWidget {
  @override
  _RoutineListState createState() => _RoutineListState();
}

class _RoutineListState extends State<RoutineList> {
  List<Routine> routines = <Routine>[...];
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      key: _scaffoldKey,
      appBar: AppBar(
        title: Text('Routines Management'),
      ),
      body: Center(
        child: ListView.builder(
          itemCount: routines.length,
          itemBuilder: (context, index) {
            return RoutineListItem(
              routine: routines[index],
              onDismissedAction: _dismissRoutineItem,
              onEditAction: _editRoutineItem,
            );
          },
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _newRoutineItem,
        tooltip: 'New Routine',
        child: Icon(Icons.add),
      ),
    );
  }

  void _newRoutineItem() {
    Navigator.push(
      context,
      MaterialPageRoute(builder: (context) => RoutineItemEditForm()),
    );
  }

  void _editRoutineItem(Routine routine) {
    Navigator.push(
      context,
      MaterialPageRoute(builder: (context) => RoutineItemEditForm()),
    );
  }

}
```

위의 코드는 ListView의 builder를 쓰는 경우로, itemBuilder 및 itemCount를 쓸 경우, widget 자체의 생성을 다른 builder를 이용하여 만들도록 합니다. 다른 방법으로는, children으로 즉시 만들어 넣는 방법도 있습니다.

```
        child: ListView.builder(
          itemCount: routines.length,
          itemBuilder: (context, index) {
            return RoutineListItem(
              routine: routines[index],
              onDismissedAction: _dismissRoutineItem,
              onEditAction: _editRoutineItem,
            );
          },
        ),
``` 
이 부분을 아래와 같이 바꿀 수도 있습니다.
```
        child: ListView(
          children: routines.map((routine) {
            return RoutineListItem(
              routine: routine,
              onDismissedAction: _dismissRoutineItem,
              onEditAction: _editRoutineItem,
            );
          }).toList(),
        ),
```

### Swipe로 삭제하기 (Dismissible)
다음으로 List에서 하나의 item을 삭제하기 위해, swipe 방식으로 삭제를 하려고 이리저리 고민하다가, 검색 first!를 이용하여 찾아보니 Dismissible이라는 Widget이 있으며, 아주 쉽게 지원한다고 합니다.

Dismissible (https://api.flutter.dev/flutter/widgets/Dismissible-class.html)[https://api.flutter.dev/flutter/widgets/Dismissible-class.html]
* key: 삭제용 Key
* background: swipe시에 보여줄 화면
* secondaryBackground: swipe right시에 보여줄 화면, 없을 경우 background 사용
* dismissThresholds: 얼마나 swipe를 해야 동작하는 지 설정
* onDismissed: (direction) swipe시에 호출되는 callback 함수
* child: 원래의 ListTile Widget

위를 적용한 RoutineListItem의 Build는 다음과 같습니다.
```
    return Dismissible(
      key: Key(routine.id.toString()),
      background: Container(
        color: Colors.red,
        padding: EdgeInsets.symmetric(horizontal: 20),
        alignment: AlignmentDirectional.centerStart,
        child: Row(
          children: <Widget>[
            Icon(Icons.delete_outline,
                color: Colors.white, semanticLabel: "Delete"),
            Text('Delete', style: TextStyle(color: Colors.white)),
          ],
          crossAxisAlignment: CrossAxisAlignment.center,
          mainAxisSize: MainAxisSize.min,
        ),
      ),
      // https://medium.com/flutter-community/an-in-depth-dive-into-implementing-swipe-to-dismiss-in-flutter-41b9007f1e0
      secondaryBackground: Container(
        color: Colors.red,
        padding: EdgeInsets.symmetric(horizontal: 20),
        alignment: AlignmentDirectional.centerEnd,
        child: Row(
          children: <Widget>[
            Text('Delete', style: TextStyle(color: Colors.white)),
            Icon(Icons.delete_outline,
                color: Colors.white, semanticLabel: "Delete"),
          ],
          crossAxisAlignment: CrossAxisAlignment.center,
          mainAxisSize: MainAxisSize.min,
        ),
      ),
      dismissThresholds: const {
        DismissDirection.startToEnd: 0.2,
        DismissDirection.endToStart: 0.2,
      },
      onDismissed: (direction) =>
          onDismissedAction(routine), // only one type dismissed support
      child: ListTile(
        title: Text('${routine.title}'),
        subtitle: Text('${routine.subtitle}'),
        leading: CircleAvatar(backgroundColor: routine.color,),
        onTap: () => onEditAction(routine),
      ),
    );
  }
```

### Routine 입력/수정 화면

다음으로는 기존 값의 수정 및 새로운 입력을 위한 양식 화면을 만들고자 합니다. 먼저, 순수하게 빈 양식 화면을 만들어 보겠습니다.

참조: https://flutter.dev/docs/cookbook/forms

먼저, 하나의 textfield를 추가하는 양식을 만드는 것은 다음의 페이지를 참조바랍니다. (// https://flutter.dev/docs/cookbook/forms/retrieve-input)[// https://flutter.dev/docs/cookbook/forms/retrieve-input]

본 화면에서는 3개의 입력 필드 (2개는 텍스트, 1개는 색상선택)가 필요합니다.
색상선택필드는 나중에 다시 하기로 하고, 먼저 두개의 텍스트 필드를 만듭니다.
단순 입력창의 경우, Stateful이던 Stateless이던 상관없으나, 나중에 자료를 주고 받으려면, Stateful이어야 할 것 같습니다.

여기서 새로 사용하게 된 Widget은 Form, TextFormField 입니다.
Form은 입력 폼들을 묶어 놓기 위한 Widget으로, 현재 사용하는 것은 validation밖에 없습니다. 좀 더 공부가 필요합니다.
TextFormField는 Text입력 Widget입니다.

TextFormField (https://api.flutter.dev/flutter/material/TextFormField-class.html)[https://api.flutter.dev/flutter/material/TextFormField-class.html]
* decoration: InputDecoration
 * labelText: 입력창의 타이틀(?) 텍스트
 * hintText: 입력창 힌트 텍스트 (원래 뒤에 보였다가, 입력하면 사라지는 placeholder)
* validator: FormFieldValidator(value) - 입력 내용이 잘 입력되었는 지 확인하는 callback
* initialValue: 초기값
* autovalidate: validator를 매 수정마다 호출할지 flag
* enabled: 수정 가능 여부
* 기타 fields, methods는 위 문서를 참조하세요.

```
class RoutineItemEditForm extends StatefulWidget {
  RoutineItemEditForm({Key key, this.routine}) : super(key: key);
  final Routine routine;

  @override
  \_RoutineItemEditFormState createState() => \_RoutineItemEditFormState();
}

class \_RoutineItemEditFormState extends State<RoutineItemEditForm> {
  final \_formKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("New/Edit Routine")),
      body: Form(
        key: _formKey,
        child: Container(
          margin: EdgeInsets.all(15.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: <Widget>[
              TextFormField(
                decoration: InputDecoration(
                  labelText: 'Title',
                  hintText: 'Enter Routine Title',
                ),
                validator: (value) {
                  if (value.isEmpty) {
                    return 'Please enter title text';
                  }
                  return null;
                },
              ),
              TextFormField(
                decoration: InputDecoration(
                  labelText: 'SubTitle',
                  hintText: 'Enter Routine SubTitle',
                ),
                validator: (value) {
                  if (value.isEmpty) {
                    return 'Please enter sub text';
                  }
                  return null;
                },
              ),
              Container( // 단순히 버튼을 오른쪽으로 보내고 싶었습니다
                padding: const EdgeInsets.symmetric(vertical: 16.0),
                alignment: Alignment.centerRight,
                child: RaisedButton(
                  onPressed: () {
                    if (_formKey.currentState.validate()) { // validator 호출
                      Navigator.of(context).pop();
                    }
                  },
                  child: Text('Submit'),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
```

### 화면간 자료의 주고 받음

입력 화면을 만들었으면, 
1. 입력된 내용을 목록에 반영하고, 
1. 수정시에 목록의 내용을 입력화면에 가져와야 합니다. 

#### 입력된 내용을 목록에 반영
입력된 내용을 목록에 반영하는 방법은 여러가지가 있겠으나 (status에서 자체 수정하거나, 입력된 내용을 호출한 화면에 전달하는 방법 등), 그 중에서 입력된 내용을 후출한 화면에 전달하는 방법에 대해 알아보겠습니다.

참조: https://flutter.dev/docs/cookbook/navigation/returning-data

작업은 크게 두가지로, 입력화면의 종료시 action과 호출화면에서의 결과 수집으로 나뉘게 됩니다.

##### 입력화면 종료시 action
입력화면의 종료시 action은 위의 화면에서는 RaisedButton의 OnPressed에 해당하는 부분으로, 화면이 닫힐 때 `Navigator.of(context).pop()`에 파라메터 란에 값을 넣어서 보내주는 것입니다. 이렇게요. 
```
  Navigator.of(context).pop(
    Routine(
      id: -1,
      title: titleController.text,
      subtitle: subtitleController.text,
    ),
  );
```

여기에 못보던 코드들이 들어있는데요, `titleController`, `subtitleController`등이 그것입니다.

TextFormField의 값을 가져올 방법은 여러가지가 있을 수 있는데요, 제일 좋은 방법은 controller를 쓰는 방법으로 보입니다. controller는 TextFormField와 직접적으로 엑세스 할 수 있는 좋은 방법인데요, 다른 개발 환경에서는 TextFormField 자체를 variable로 설정하고 엑세스 하나, flutter에서는 controller를 만들고, 그걸 통해서 서로 control하고 있습니다.

먼저 State 클래스 상단에
```
class \_RoutineItemEditFormState extends State<RoutineItemEditForm> {
  final titleController =
      TextEditingController(); // https://flutter.dev/docs/cookbook/forms/retrieve-input
  final subtitleController = TextEditingController();
```
이런식으로 두개의 controller를 설정합니다.

다음으로, 실제 TextFormField에 `controller: titleController`와 같이 추가해줍니다.

```
  TextFormField(
    controller: titleController,
    decoration: InputDecoration(
```

그리고, 실제로 사용하는 곳 (예: 저장시)에 위의 코드처럼 `titleController.text`와 같이 입력값을 사용하면 됩니다.

백버튼의 경우에는 Pop시에 파라메터를 넣지 않으므로, null값을 전송하게 됩니다.

##### 호출화면의 수정
그리고 결과를 받을 화면 (여기서는 widget/routines/page.dart)에서 실제 editform을 호출한 부분 \_newRoutineItem 부분을 다음과 같이 수정합니다.

```
  Future<void> \_newRoutineItem() async {
    final resultRoutine = await Navigator.push(
      context,
      MaterialPageRoute(builder: (context) => RoutineItemEditForm()),
    );
    if (resultRoutine != null) {
      resultRoutine.id = routines.last.id + 1;
      setState(() {
        routines.add(resultRoutine);
      });

      _scaffoldKey.currentState
        ..removeCurrentSnackBar()
        ..showSnackBar(
            SnackBar(content: Text("${resultRoutine.title} Added.")));
    }
  }
```

여기에 새로이 등장하는 것이 async와 await, Future<T>인데요, 아주 단순하게 얘기하면, await를 결과물이 바로 return되지 않을때, async는 자신 함수 내부에 await (또는 그에 준하는 함수)를 호출하였을 때 사용하면 됩니다. 여기서는 Navigator.push는 새로운 화면을 띄우는 함수로, 그 화면이 닫힐때까지 결과가 나오지 않으므로, await을 쓰는 게 맞고, 안에서 await를 호출하니 async를 쓰고, 또한 async이니, return 타입이 Future<T>입니다.

다음으로 SnackBar 관련 부분이 있는데, SnackBar는 앱 하단에 빼꼼 올라오는 정보창입니다. SnackBar가 동작하기 위해서는 ScaffoldKey가 필요한데, 이건 처음 페이지를 만들었을 때 넣었습니다.

#### List Item의 내역을 입력 화면에 전달
ListItem을 입력화면에 전달하는 부분 역시 두 곳의 수정이 필요합니다. 

1. 호출부분의 수정
1. 입력화면의 수정

##### 호출 부분의 수정
호출 부분의 수정 부분은 굉장히 쉽습니다. 화면을 생성할 때 파라메터를 주며 생성하면 됩니다.

```
  Future<void> \_editRoutineItem(Routine routine) async {
    final resultRoutine = await Navigator.push(
      context,
      MaterialPageRoute(
          builder: (context) => RoutineItemEditForm(routine: routine)),
    );
    if (resultRoutine != null) {
      setState(() {
        // 1. 찾아서 바꿔끼자.
        int idx = routines.indexWhere((item) => item.id == resultRoutine.id);
        routines[idx] = resultRoutine;
      });

      _scaffoldKey.currentState
        ..removeCurrentSnackBar()
        ..showSnackBar(
            SnackBar(content: Text("${resultRoutine.title} Edited.")));
    }
  }
```

아래의 resultRoutine 부분은 입력화면에서 자료를 받기 위해 미리 만들어 둔 부분이고, 실제 수정된 부분은 `builder: (context) => RoutineItemEditForm(routine: routine)),` 입니다. routine을 파라메터로 전송하는 거죠. 이것으로 끝!

##### 입력화면의 수정
입력화면의 수정은 크게 두 부분에서 변경됩니다. 파라메터를 받아야 하니 Widget 생성자가 수정되어야 하고, 입력받은 파라메터의 값을 입력창에 넣어줘야 하는 부분이 변경됩니다.

생성자 수정 부분은 아래와 같습니다.
```
class RoutineItemEditForm extends StatefulWidget {
  RoutineItemEditForm({Key key, this.routine}) : super(key: key);
  final Routine routine;
```

입력창에 텍스트를 넣는 것은 처음에는 initialValue에 넣는 방법과 controller를 사용하는 두 방법이 있습니다. 처음에는 initialValue를 사용했었는데, 다른 이유 때문에 (해당 Widget에 setState를 호출할 필요가 있었는데, 그 때 자료가 리셋됨) 실패해서, controller를 쓰는 방법으로 변경하였습니다. 여기서는 State class의 initState부분을 추가해서 해결하였습니다.

```
  @override
  void initState() {
    titleController.text = widget.routine?.title ?? "";
    subtitleController.text = widget.routine?.subtitle ?? "";
    super.initState();
  }
```

여기서 widget.routine? 를 쓴 이유는, 추가 요청의 경우 routine값이 null이 들어오기 때문에, null에 대한 비교를 할 필요가 있습니다.

### ColorPicker를 위한 Widget
마지막으로 routine의 입력시 표시를 이쁘게 하기 위해 색상을 넣고자 했습니다. 이에 어떻게 하면 색상을 입력할 수 있을까 고민하다가, 아래의 plugin을 찾았습니다. 설치 방법은 아래의 url을 참조하세요.

참조: https://github.com/mchome/flutter_colorpicker

위의 color picker는 widget type의 color picker로서, 저는 색상버튼/바을 넣고, 해당 버튼을 누르면 popup이 되어 색상을 선택하는 방식으로 동작하고 싶었습니다. 거기에 TextEditingController와 같이 controller 기반으로 자료를 접근하고도 싶었습니다. 그래서 새로운 widget을 만들고자 시도하였습니다.

이리 저리 여러 소스코드를 보면 쥐어짜다가 아래와 같은 Widget으로 완료하였습니다.

소스 참조: [/lib/widget/common/colorpicker.dart](https://github.com/jaywon99/routines/blob/57f1d144a6fd6aa17e1eed1065e24855669c95a5/lib/widgets/common/colorpicker.dart)

상단에 color리스트를 원래의 colorpicker코드에서 가져와 다시 선언한 이유는, 랜덤 색상 선택때문에 가져왔습니다. 

본 코드에서 ColorPickingController를 만들기 위해 TextEditingController 소스코드와 TextFormField 소스 코드를 공부하였고, 그와 함께 ValueNotifier 소스 코드도 공부하였습니다. 좋은 경험이었습니다.

[TextEditingController 소스코드](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/editable_text.dart)
[TextFormField 소스코드](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/material/text_form_field.dart)
[ValueNotifier 소스코드](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/foundation/change_notifier.dart)

이 코드에는 좋은 아이디어가 있다면, 언제든지 환영합니다.

또한, ColorPicker Widget의 View에 대해 고민이 있었으나, 단순하게 RaisedButton의 입력값을 그대로 받도록 했으며, Button의 액션만 다르게 하였습니다. 그래서 onLongPress같은 건 그냥 쓸 수 있도록 했습니다. 원래는 보조 색상도 그에 맞춰 줄 수 있으면 좋겠으나, 일단은 null로 해서, 시스템이 알아서 설정하도록 하는 것이 제일 좋을 듯 합니다.

[RaisedButton 소스코드](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/material/raised_button.dart)

관련 form에 넣은 부분은 (/lib/widgets/routines/editform.dart)[https://github.com/jaywon99/routines/blob/57f1d144a6fd6aa17e1eed1065e24855669c95a5/lib/widgets/routines/editform.dart]를 참조하세요.

---

v0.2.0 최종본은 (https://github.com/jaywon99/routines/tree/57f1d144a6fd6aa17e1eed1065e24855669c95a5)[https://github.com/jaywon99/routines/tree/57f1d144a6fd6aa17e1eed1065e24855669c95a5]에 있습니다.

위와 같이 routine들을 신규/수정/목록 그리고 색상 선택까지 마쳤습니다. 이번 글은 좀 길었습니다. 한 3개쯤 나눠서 썼어야 하는 건 아니었나 싶습니다. 다음에는 이번에 만든 routine을 local에 저장하는 기능을 만들예정입니다.

### Milestones

```
[X] v0.1.0 Empty Home + Menu List + About 화면
[X] v0.1.1 Named Routes의 추가 및 필요없는 코드 제거
[X] v0.2.0 (+) Button + Routine 관리 (색상 + 타이틀글 + subtitle 정도)
[ ] v0.2.X Routine의 저장 (persistence)
[ ] v0.3.X Home 화면 - Routine 체크 화면
[ ] v0.4.X 진행상황보기 - 달력으로 실행 내역 보기 (주간 레포트를 볼 수 있지 않을까?)
[ ] v0.5.X Ads 추가
[ ] v0.6.X InApp Purchase 추가
[ ] v1.0.X AppStore, PlayStore에 등록하기
[ ] v1.1.X Routine 프로퍼티 추가 - 매일 하는 일이 아닌 특정 요일 하는 일만 표시
[ ] v1.2.X Routing 프로퍼티 추가 - 숫자 입력하기 (예: 횟수, 시간, 등등)
```






