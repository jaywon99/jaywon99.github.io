---
title: Gallery and Take photo - v0.1.0
cover-image: hipster.jpg
categories: Ionic4
tags: [Ionic4, Secure Gallery App]
summary: 첫번째 태스크는 ionic 4를 이용하여 initial repo를 만들고, gallery를 만드는 것입니다. 참고페이지는 [Build First Ionic App]이지만, 해당 페이지는 Tab을 기반으로 만든 것이고, 여기서는 sidemenu를 이용해서 만들 예정입니다.

---

첫번째 태스크는 ionic 4를 이용하여 initial repo를 만들고, gallery를 만드는 것입니다. 참고페이지는 [Build First Ionic App](https://ionicframework.com/docs/angular/your-first-app)이지만, 해당 페이지는 Tab을 기반으로 만든 것이고, 여기서는 sidemenu를 이용해서 만들 예정입니다.

## 초기 프로젝트 생성

```
ionic start secure-gallery sidemenu
cd secure-gallery
ionic serve
```

원래 에디터로 vim을 썼지만, 많은 부분에서 vs code를 ionic 4 project에서 쓴다하여, 이번에는 vs code를 써보기로 했습니다. 생각보다 좋더군요. 아래의 extension을 설치해서 사용중입니다.

* [Visual Studio Code](https://code.visualstudio.com/download)
* Ionic 4 Snippets extension
* Ionic Extension Pack extension
* rxjs-snippets extension
* TSLint extension

### Cordova iOS/Android platform 추가
만들려는 것이 모바일 어플리케이션이고, 일단 cordova engine을 이용할 예정이므로, ios/android용 cordova project를 추가합니다.

```
ionic cordova platform add ios
ionic cordova platform add android
```

## 목표 화면
만들려는 화면은 다음과 같습니다.

![Screenshot](/img/2019-08-29-secure-gallery-home.png)

## 단계별 진행
수정된 파일은 다음과 같습니다.

### list page 제거 (scaffold가 만들어주는 것으로 필요없어서 삭제)

#### list 디렉토리 삭제
```
rm -rf src/app/list
```

#### Sidemenu에서 제거: src/app/app.component.ts
```diff
diff --git a/src/app/app.component.ts b/src/app/app.component.ts
index c013bef..2497dc5 100644
--- a/src/app/app.component.ts
+++ b/src/app/app.component.ts
@@ -12,14 +12,9 @@ import { StatusBar } from '@ionic-native/status-bar/ngx';
 export class AppComponent {
   public appPages = [
     {
-      title: 'Home',
+      title: 'Gallery',
       url: '/home',
-      icon: 'home'
-    },
-    {
-      title: 'List',
-      url: '/list',
-      icon: 'list'
+      icon: 'albums'
     }
   ];
```

#### routing에서 제거: src/app/app-routing.module.ts
```diff
diff --git a/src/app/app-routing.module.ts b/src/app/app-routing.module.ts
index b464e7d..c05b328 100644
--- a/src/app/app-routing.module.ts
+++ b/src/app/app-routing.module.ts
@@ -10,10 +10,6 @@ const routes: Routes = [
   {
     path: 'home',
     loadChildren: () => import('./home/home.module').then(m => m.HomePageModule)
-  },
-  {
-    path: 'list',
-    loadChildren: () => import('./list/list.module').then(m => m.ListPageModule)
   }
 ];
```

### Photo Service의 추가 및 사진 찍기
#### Camera Dependencies 추가 (CLI)
```
npm install @ionic-native/camera
ionic cordova plugin add cordova-plugin-camera
```

#### App Module에 Camera 추가
```diff
diff --git a/src/app/app.module.ts b/src/app/app.module.ts
index 9e6994c..8d8fbdc 100644
--- a/src/app/app.module.ts
+++ b/src/app/app.module.ts
@@ -5,6 +5,7 @@ import { RouteReuseStrategy } from '@angular/router';
 import { IonicModule, IonicRouteStrategy } from '@ionic/angular';
 import { SplashScreen } from '@ionic-native/splash-screen/ngx';
 import { StatusBar } from '@ionic-native/status-bar/ngx';
+import { Camera } from '@ionic-native/camera/ngx';

 import { AppComponent } from './app.component';
 import { AppRoutingModule } from './app-routing.module';
@@ -20,6 +21,7 @@ import { AppRoutingModule } from './app-routing.module';
   providers: [
     StatusBar,
     SplashScreen,
+    Camera,
     { provide: RouteReuseStrategy, useClass: IonicRouteStrategy }
   ],
   bootstrap: [AppComponent]
```

#### photo service generate with CLI
```
ionic g service services/Photo
```

#### photo service에 photo 관련 정보 추가
```diff
diff --git a/src/app/services/photo.service.ts b/src/app/services/photo.service.ts
index 11963f2..37d0495 100644
--- a/src/app/services/photo.service.ts
+++ b/src/app/services/photo.service.ts
@@ -1,9 +1,18 @@
 import { Injectable } from '@angular/core';

+// ADDED
+class Photo {
+  name: string;
+  addedAt = '';
+  data: any;
+}
+
 @Injectable({
   providedIn: 'root'
 })
 export class PhotoService {

+  public photos: Photo[] = []; // ADDED
+
   constructor() { }
 }
```
Photo class의 name은 파일명을 위해서, addedAt는 추가된 일자를 위해서 추가합니다. 추후에 해당 값으로 sorting할 수 있어서 넣었습니다.

#### 사진 찍기 기능의 photo service에 추가
사진찍기를 위해서는 먼저 Camera plugin을 추가하고, 또한 CameraOptions를 추가합니다.. 그리고, constructor에 해당 plugin의 사용 등록(?), 그리고 필요 method를 추가합니다.

```diff
diff --git a/src/app/services/photo.service.ts b/src/app/services/photo.service.ts
index 37d0495..6115812 100644
--- a/src/app/services/photo.service.ts
+++ b/src/app/services/photo.service.ts
@@ -1,4 +1,5 @@
 import { Injectable } from '@angular/core';
+import { Camera, CameraOptions } from '@ionic-native/camera/ngx';

 // ADDED
 class Photo {
@@ -14,5 +15,29 @@ export class PhotoService {

   public photos: Photo[] = []; // ADDED

-  constructor() { }
+  constructor(private camera: Camera) { }
+
+  takePicture() {
+    const options: CameraOptions = {
+      quality: 100,
+      destinationType: this.camera.DestinationType.DATA_URL,
+      encodingType: this.camera.EncodingType.JPEG,
+      mediaType: this.camera.MediaType.PICTURE
+    };
+
+    this.camera.getPicture(options).then((imageData) => {
+      // Add new photo to gallery
+      this.photos.unshift({
+        name: new Date().toISOString(), // generate random name (timetstring is good random name)
+        addedAt: new Date().toISOString(),
+        data: 'data:image/jpeg;base64,' + imageData
+      });
+
+    }, (err) => {
+     // Handle error
+     // TODO: Open Error Dialog or something
+     console.log('Camera issue: ' + err);
+    });
+  }
+
 }
```

  Camera plugin의 자세한 사용법은 [https://ionicframework.com/docs/native/camera](Camera) 참조

### 찍은 사진의 저장
일단 여기서는 sqlite 스토리지에 저장하기로 합니다. secure gallery이기 때문에 저장한 이미지를 다른 곳에서 빼가기 어렵게 하려는 의도가 있습니다. (사실 file storage를 쓸지, sqlite storage를 쓸지 아직 고민중이고, tutorial이 sqlite를 쓰도록 되어있어서 그대로 진행합니다.)

#### SQLite storage 사용 준비 (cli install)
```diff
ionic cordova plugin add cordova-sqlite-storage
npm install --save @ionic/storage
```

#### module의 추가
```diff
diff --git a/src/app/app.module.ts b/src/app/app.module.ts
index 8d8fbdc..150b1a9 100644
--- a/src/app/app.module.ts
+++ b/src/app/app.module.ts
@@ -6,6 +6,7 @@ import { IonicModule, IonicRouteStrategy } from '@ionic/angular';
 import { SplashScreen } from '@ionic-native/splash-screen/ngx';
 import { StatusBar } from '@ionic-native/status-bar/ngx';
 import { Camera } from '@ionic-native/camera/ngx';
+import { IonicStorageModule } from '@ionic/storage';

 import { AppComponent } from './app.component';
 import { AppRoutingModule } from './app-routing.module';
@@ -16,7 +17,8 @@ import { AppRoutingModule } from './app-routing.module';
   imports: [
     BrowserModule,
     IonicModule.forRoot(),
-    AppRoutingModule
+    AppRoutingModule,
+    IonicStorageModule.forRoot()
   ],
   providers: [
     StatusBar,
```

#### PhotoService에 읽기/저장하기 추가
```diff
diff --git a/src/app/services/photo.service.ts b/src/app/services/photo.service.ts
index 6115812..f07d30b 100644
--- a/src/app/services/photo.service.ts
+++ b/src/app/services/photo.service.ts
@@ -1,5 +1,6 @@
 import { Injectable } from '@angular/core';
 import { Camera, CameraOptions } from '@ionic-native/camera/ngx';
+import { Storage } from '@ionic/storage';

 // ADDED
 class Photo {
@@ -15,7 +16,7 @@ export class PhotoService {

   public photos: Photo[] = []; // ADDED

-  constructor(private camera: Camera) { }
+  constructor(private camera: Camera, private storage: Storage) { }

   takePicture() {
     const options: CameraOptions = {
@@ -33,6 +34,7 @@ export class PhotoService {
         data: 'data:image/jpeg;base64,' + imageData
       });

+      this.storage.set('photos', this.photos); // TODO: Error Handler
     }, (err) => {
      // Handle error
      // TODO: Open Error Dialog or something
@@ -40,4 +42,9 @@ export class PhotoService {
     });
   }

+  loadPhotos() {
+    this.storage.get('photos').then((photos) => {
+      this.photos = photos || [];
+    });
+  }
 }
```

  저장하기는 기존의 코드에 추가하고, 읽기는 method를 나누었습니다. 현재는 굳이 나눌 필요가 없어서이며, refactoring은 필요할 때 한다는 주의에 따라 진행하였습니다.추후에는 addPhoto라는 메쏘드가 생기리라 예상됩니다.

### home의 변경
필요한 기능은 다 추가하였으니, 이제는 실제 화면에 연동시켜 보도록 하겠습니다.

#### grid의 추가
  대부분의 gallery는 grid view를 가지고 있습니다. ionicframework에서는 2개의 column을 잡았으나, 개인적으로 2개는 사진이 좀 커 보여서, 4개로 정했습니다.

```xml
<ion-header>
  <ion-toolbar>
    <ion-buttons slot="start">
      <ion-menu-button></ion-menu-button>
    </ion-buttons>
    <ion-title>
      Secure Gallery
    </ion-title>
  </ion-toolbar>
</ion-header>

<ion-content>
  <ion-grid>
    <ion-row>
      <ion-col size="3" *ngFor="let photo of photoService.photos">
        <img [src]="photo.data" />
      </ion-col>
    </ion-row>
  </ion-grid>
</ion-content>
```

  기존 코드 (처음 만들어진 template)가 길어서, 수정 코드를 다시 만들었습니다.
  하나의 ion-row는 ion-col을 12를 가질 수 있는데, size를 이용하여 각 column의 크기를 정할 수 있습니다. size=3이면, 4개의 컬럼을, size=6이면 2개의 column을, size=4이면 3개의 column을 넣을 수 있습니다.
  ion-grid에 대한 자세한 설명은 [https://ionicframework.com/docs/api/grid](ion-grid)를 참조하세요.

#### 사진 찍기 버튼 추가 (ion-fab)
```diff
diff --git a/src/app/home/home.page.html b/src/app/home/home.page.html
index f4f828e..a14bd8f 100644
--- a/src/app/home/home.page.html
+++ b/src/app/home/home.page.html
@@ -17,4 +17,9 @@
       </ion-col>
     </ion-row>
   </ion-grid>
+  <ion-fab vertical="bottom" horizontal="center" slot="fixed">
+    <ion-fab-button (click)="photoService.takePicture()">
+      <ion-icon name="camera"></ion-icon>
+    </ion-fab-button>
+  </ion-fab>
 </ion-content>
```

  ion-grid 하단에 ion-fab을 추가하여, floating action button을 추가합니다.
  FAB은 Floating Action Button을 의미하는데, 보통 떠다니는 버튼 (타이틀바, 탭바, 목록, 양식등에 있는 것이 아니라, 하단이나, 우상단 등에 떠있는 둥그런 버튼)을 의미합니다.
  ion-fab에 대한 자세한 설명은 [https://ionicframework.com/docs/api/fab](ion-fab)을 참조하세요.

#### photo service 연동
```diff
diff --git a/src/app/home/home.page.ts b/src/app/home/home.page.ts
index 83522d5..beaa9a9 100644
--- a/src/app/home/home.page.ts
+++ b/src/app/home/home.page.ts
@@ -1,4 +1,5 @@
 import { Component } from '@angular/core';
+import { PhotoService } from '../services/photo.service';

 @Component({
   selector: 'app-home',
@@ -7,6 +8,9 @@ import { Component } from '@angular/core';
 })
 export class HomePage {

-  constructor() {}
+  constructor(public photoService: PhotoService) {}

+  ngOnInit() {
+    this.photoService.loadPhotos();
+  }
 }
```

  page module에 photoService를 추가합니다.

## 현 버젼 개발 완료
이상으로 현 버젼을 개발 완료했습니다. 본 기능은 컴퓨터로는 테스트 할 수가 없으므로, 폰으로 테스트 해야 하는데, 첫 단계부터 폰으로 넣기에는 조금 이른 것 같아, [ionic devapp](https://ionicframework.com/docs/appflow/devapp)을 이용하여 테스트 하였습니다.

ionic devapp은 폰에 설치되는 앱으로, 해당 앱에서 방금 만든 프로그램을 띄워볼 수 있으며, 하드웨어 또한 직접 연동이 됩니다.

개발하고 나니 추가할 것이 보입니다.

1. grid의 모양이 안 이쁩니다. padding을 조절해야겠습니다.
1. grid의 모양이 안 이쁩니다. 정사각형 thumbnail을 넣어야 겠습니다.
1. 사진을 저장하는 데 시간이 많이 걸립니다. 아무래도 한번에 모두 넣어서 그런것 같습니다.

### 진행상황

이번 버젼은 0.1.0으로 명명하였습니다. 이번 버젼에 추가한 내역은 다음과 같습니다.
```
[X] 갤러리 + 사진찍기 (v0.1.0)
```

앞으로 개발할 큰 내역은 다음과 같습니다.
```
[ ] 사진 크게 보기 (디테일 포함 - 파일명, 시각)
[ ] 목록 스타일 (Gallery / List)
[ ] Import from Photo
[ ] 이름 수정 + 소팅 (이름순/등록순)
[ ] 비밀번호 설정 (Settings화면) - 하단에 about 추가??
[ ] Splash Screen
[ ] Fingerprint Security / Face-ID / etc - 디바이스 보안
[ ] AdMob 추가
[ ] Inapp Purchase
[ ] Export or Sync via WIFI/Internet
[ ] Export/Import via iTunes
[ ] Albums 기능
```

그리고, 작은 업데이트 예정은 다음과 같습니다.
```
[ ] grid의 모양이 안 이쁩니다. padding을 조절해야겠습니다.
[ ] grid의 모양이 안 이쁩니다. 정사각형 thumbnail을 넣어야 겠습니다.
[ ] 사진을 저장하는 데 시간이 많이 걸립니다. 아무래도 한번에 모두 넣어서 그런것 같습니다.
```


- - -

