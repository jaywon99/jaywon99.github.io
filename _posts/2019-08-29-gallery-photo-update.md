---
title: View Photo - v0.1.1
author: ["Jongpil Won"]
cover-image: hipster.jpg
categories: Ionic4
tags: [Ionic4, Secure Gallery App]
summary: 이번 단계에서는 v0.1.0에서 보기 좋지 않았던 몇가지를 빠르게 검색해서 수정하려했으나, 생각보다 문제가 커졌네요. grid를 거의 재구축했습니다.

---

## 이번 버젼에서 할일
1. grid의 모양이 안 이쁩니다. padding을 조절해야겠습니다.
1. grid의 모양이 안 이쁩니다. 정사각형 thumbnail을 넣어야 겠습니다.

### grid의 패딩 조절 
grid의 패딩 조절은 다음의 참고페이지에 보면 조절 방법이 나옵니다. (*--ion-grid-padding*)
  참고페이지: [https://ionicframework.com/docs/layout/grid#grid-padding](ion-grid Grid Padding)

다만, 어디에다가 해당 내용을 넣느냐인데요, 일단 구글에서 "ionic framework css variables"을 검색 후 나온 첫번째 페이지 (역시 구글!) [https://ionicframework.com/docs/theming/css-variables](CSS Variables)의 Setting Values를 참고하였습니다.

```diff
diff --git a/src/theme/variables.scss b/src/theme/variables.scss
index 4b39b39..72d7f1d 100644
--- a/src/theme/variables.scss
+++ b/src/theme/variables.scss
@@ -74,4 +74,7 @@
   --ion-color-light-contrast-rgb: 0, 0, 0;
   --ion-color-light-shade: #d7d8da;
   --ion-color-light-tint: #f5f6f9;
+
+  /** ion-grid **/
+  --ion-grid-padding: 2px;
 }
```

그런데, 위의 코드를 테스트 해보면, 원하는 데로 나오질 않습니다. 숫자를 이리저리 늘렸다 줄였다 하면서 발견한 것은 ion-grid-padding은 원하던 것이 아니란 걸 발견합니다. 왜냐하면 ion-grid-padding은 ion-grid 자체의 padding (최외곽 padding)인거죠.
그래서 ion-grid-padding대신에 ion-grid-column-padding을 이용해야 합니다.
[https://ionicframework.com/docs/api/col](ion-col) 참조
최종본은 다음과 같습니다.

```diff
diff --git a/src/theme/variables.scss b/src/theme/variables.scss
index 4b39b39..72d7f1d 100644
--- a/src/theme/variables.scss
+++ b/src/theme/variables.scss
@@ -74,4 +74,7 @@
   --ion-color-light-contrast-rgb: 0, 0, 0;
   --ion-color-light-shade: #d7d8da;
   --ion-color-light-tint: #f5f6f9;
+
+  /** ion-grid **/
+  --ion-grid-column-padding: 2px;
 }
```

### grid의 정사각형 thumbnail
ionic framework에 보면 component중에 ion-thumbnail과 ion-img가 있습니다. 이것을 한번 이용해보려고 합니다.

처음에는 ion-thumbnail로 img tag를 감싸안아봤습니다만, 잘 되지를 않습니다. 
```
<ion-thumbnail>
  <img [src]="photo.data"/>
</ion-thumbnail>
```

| ![Screenshot](/assets/images/2019-08-30-strange-thumbnail.png) |
|:--:|
| Screenshot |

다음으로는 img 대신에 ion-img를 넣어봅니다. 이 경우에는 별로 차이가 없습니다.

```
<ion-thumbnail>
  <ion-img [src]="photo.data"></ion-img>
</ion-thumbnail>
```

정사각형 이미지는 나오는데, 원하는 꽉 찬 이미지가 나오지를 않습니다. 그래서 원인을 찾기 위해, tag에 style로 border를 넣어서, 어떤 모양인지를 확인해보았습니다.
```
<ion-col size="3" *ngFor="let photo of photoService.photos" style="border: solid thin purple;">
  <ion-thumbnail  width="100%" style="border: solid thin red;">
    <ion-img [src]="photo.data" width="100%"></ion-img>
  </ion-thumbnail>
</ion-col>
```

| ![Screenshot](/assets/images/2019-08-30-with-border.png) |
|:--:|
| Screenshot |

결국 발견한 것은, 이미지의 정사각형이 넓이에 맞지 않고, 높이에 맞아버렸다는 것을 알았습니다. 그래서 어떻게 하면 정사각형을 넓이에 맞출 수 있을 지 찾아보았습니다.

*그러다가 중요한 것을 발견했습니다.* iOS Photo나 기타 앱들은 폭을 중심으로 정사각형을 만드는 게 아니라, 높이를 중심으로 정사각형을 만들고, 옆으로는 무한히 나열하는 방식이더군요. 그래서, 새로운 리서치를 시작하였습니다.

### GRID 재구축
위의 grid를 재구축하기 위해서 이리저리 찾아봤으나, 마음에 드는 grid가 없어서, 전체적으로 재 작성하기로 하였습니다. 일단 ion-grid에 있는 attribute는 하나도 사용하지 않을 예정이며, 모두 css로 처리하기로 하였습니다. 덕분에 위에서 설정한 padding등도 다 다시 설정하였습니다.

여러가지로 고민하다가, 크게 두가지에 촛점을 맞췄습니다. 하나는 정사각형 썸네일, 다른 하나는 이미지 사이즈를 80px-100px사이로 입니다.

#### home.page.html의 변경
home.page.html에서는 ion-col의 size attribute를 사용하지 않으려 합니다. 왜냐하면, 한 줄에 몇개의 썸네일이 들어갈 지 알수가 없어서입니다.
썸네일 이미지는 이미지 태그 대신에 div의 background-image를 이용하여, 크기 변화에 대응토록 하였습니다.

```diff
diff --git a/src/app/home/home.page.html b/src/app/home/home.page.html
index a83875a..8d6a5e3 100644
--- a/src/app/home/home.page.html
+++ b/src/app/home/home.page.html
@@ -12,8 +12,8 @@
 <ion-content>
   <ion-grid>
     <ion-row>
-      <ion-col size="3" *ngFor="let photo of photoService.photos">
-        <img [src]="photo.data" />
+      <ion-col *ngFor="let photo of photoService.photos">
+        <div class="image-container" [style.background-image]="'url(' + photo.data + ')'"></div>
       </ion-col>
     </ion-row>
   </ion-grid>
```

#### home.page.scss의 재작성
이 scss는 원래의 grid를 다시 만들기 위해 vw unit을 이용하여 다시 작성하였습니다.
내용을 보시면 아시겠지만, media query를 이용하여, max-width기준으로 900px부터 매 100px 단위로 분리하여 폭을 정했습니다. 900px 이상의 경우에는 한 줄에 10개의 썸네일이 들어가도록 지정하였습니다.

```scss
ion-grid {
  padding: 0px;
  ion-col {
    padding: 1px;

    height: 10vw;
    width: 10vw;
    min-width: 10vw;
    min-height: 10vw;
    max-width: 10vw;
    max-height: 10vw;

    @media screen and (max-width: 900px) {
      height: 11.11111111vw;
      width: 11.11111111vw;
      min-width: 11.11111111vw;
      min-height: 11.11111111vw;
      max-width: 11.11111111vw;
      max-height: 11.11111111vw;
    }

    @media screen and (max-width: 800px) {
      height: 12.5vw;
      width: 12.5vw;
      min-width: 12.5vw;
      min-height: 12.5vw;
      max-width: 12.5vw;
      max-height: 12.5vw;
    }

    @media screen and (max-width: 700px) {
      height: 14.2857142857vw;
      width: 14.2857142857vw;
      min-width: 14.2857142857vw;
      min-height: 14.2857142857vw;
      max-width: 14.2857142857vw;
      max-height: 14.2857142857vw;
    }

    @media screen and (max-width: 600px) {
      height: 16.66666666vw;
      width: 16.66666666vw;
      min-width: 16.66666666vw;
      min-height: 16.66666666vw;
      max-width: 16.66666666vw;
      max-height: 16.66666666vw;
    }

    @media screen and (max-width: 500px) {
      height: 20vw;
      width: 20vw;
      min-width: 20vw;
      min-height: 20vw;
      max-width: 20vw;
      max-height: 20vw;
    }

    @media screen and (max-width: 400px) {
      height: 25vw;
      width: 25vw;
      min-width: 25vw;
      min-height: 25vw;
      max-width: 25vw;
      max-height: 25vw;
    }

    .image-container {
      width: 100%;
      height: 100%;
      background-size: cover;
    }
  }
}
```

그 결과로 만들어진 화면은 다음과 같습니다.

| ![Screenshot](/assets/images/2019-08-30-final-thumbnail.png) |
|:--:|
| Screenshot |


### 진행상황

이번 버젼은 0.1.1입니다. 이번 버젼까지 추가한 내역은 다음과 같습니다.
```
[X] 갤러리 + 사진찍기 (v0.1.0)
[X] grid의 모양이 안 이쁩니다. padding을 조절하였습니다. (v0.1.1)
[X] grid의 모양이 안 이쁩니다. 정사각형 thumbnail을 넣었습니다. (v0.1.1)
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
[ ] 사진을 저장하는 데 시간이 많이 걸립니다. 아무래도 한번에 모두 넣어서 그런것 같습니다.
```


- - -

