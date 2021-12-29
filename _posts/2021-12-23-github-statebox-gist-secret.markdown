---
layout: post
title:  Gist와 Secret을 이용한 Awesome Pinned Gist 적용하기
date:   2021-12-23 14:04:30 +0300
image:  02.jpg
tags:   Github
categories: github tips
---

깃허브 프로필을 정리하면서 알게 된 기능인 Gist와 Secrets에 대해 간략한 설명을 곁들여 Awesome Pinned Gists의 Gist들을 사용하는 방법을 정히해보았다. 내가 Gist와 Secrets를 사용한 건 아래 사진과 같은 [스테이트 박스](https://github.com/bokub/github-stats-box)다. 이 글은 초보자 수준이거나 Git에 아직 익숙하지 않은 사람들에게 초점을 맞췄다.

***

![](https://user-images.githubusercontent.com/86394389/147643945-0b1ae3d2-df16-4614-9ed6-c53071d24c93.png)

***

처음에 이걸 Fork 해서 사용하는데 README를 읽으면서 따라해도 오류가 났다. 그래서 코드를 보고 눈치껏 이것저것 건드려보면서 익혔고 __해냈다!__ 이 글은 나와같은 눈치코치를 거치지 않고 스테이트 박스를 포함한 [Awesome Pinned Gists](https://github.com/matchai/awesome-pinned-gists)를 자신의 깃허브에 적용하는 방법이다.

***

## 깃허브의 Gist와 Secret

### Gist

[Github Docs: Gist](https://docs.github.com/en/rest/reference/gists)에는 다음과 같이 요약되어 있다.

###### The Gists API enables the authorized user to list, create, update and delete the public gists on GitHub.

Gist API는 인증된 깃허브 사용자에게 공개 gist들을 정렬, 생성, 업데이트, 삭제할 수 있다는 뜻이다. 가볍게 짧은 코드나 목록 등을 공유하는데 사용하기 좋다. 문서 내의 설명을 읽어보면 주의할 사항이 몇 가지 있다. Awesome Pinned Gists를 적용할 때는 '이런 게 있구나.' 하고 넘어가도 되는 주의사항들이다.

* Gist를 생성하려면 깃허브 로그인이 필요하다. 사용자가 아니면서 업데이트하고자 한다면 gist의 OAuth scope와 token이 필요하다.
* 각 gist 파일마다 1 Mbyte까지 사용할 수 있다.

### Secrets

Secret은 다음과 같이 요약되어 있다. 

###### Encrypted secrets allow you to store sensitive information in your organization, repository, or repository environments.

암호화된 secrets는 당신의 조직, 레포지토리, 레포지토리 환경의 민감한 정보들을 저장하는데 사용된다는 뜻이다. Secret도 이 정도의 내용만 알면 적용하는데 문제 없다. Secrets에 대한 자세한 설명은 [Github Docs: Encrypted secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)에서 확인할 수 있다. Secret 작명법부터 한계점까지 다룬다. 

***

## Awesome Pinned Gist 적용하기

### 전반적인 적용 순서

1. 사용할 Gist와 토큰 만들기
2. 원하는 Awesome Pinned Gist의 레포지토리 Fork하기
3. Secret과 run.yml 설정하기
4. Action 설정과 확인


### 1. 사용할 Gist와 토큰 만들기

Gist의 이름과 내용은 아무것이나 좋다. 아래 사진과 같이 Github Gist 파일을 하나 만든다. __중요한 것은 공개 Gist로 만드는 것이다.__ 사진에서 오른쪽 아래 초록색 버튼을 보면 화살표를 눌러 private와 public을 선택할 수 있다. 프로필에 pin하기 위해서는 public이어야 하므로 public으로 설정한다.이 Gist 파일의 확장자는 중요하지 않다.

![](https://user-images.githubusercontent.com/86394389/147466154-453718b5-4423-44dd-902e-dbf5741c7a9d.png)

그 다음으로 토큰을 만들어야 한다. 토큰은 Github에서 Setting(설정)에서 만들 수 있다. 자신의 프로필과 계정에 관한 것이 뜨는데, 왼쪽 카테고리 중 가운데 블록에 있는 __Developer Settings__ 으로 들어간다. 그럼 다음과 같은 페이지가 뜬다. 여기서 __Personal access tokens__ 를 선택한다.

![](https://user-images.githubusercontent.com/86394389/147466682-a2a5ddf1-2579-4043-8fe3-29f70847e017.png)

__Generate new token__ 버튼을 눌러 토큰을 새로 하나 만든다.

![](https://user-images.githubusercontent.com/86394389/147466934-ac8bae98-16ee-4e4b-a70f-ff74be04a70f.png)

필요한 내용들을 설정한다.

* Note: 토큰의 이름 정도로 생각하면 된다.
* Expiration: 만료일이다. 원하는 수치로 선택한다.
* Scope: 토큰이 정보를 얻어오는 범위다. Gist에 정보를 보여주기 위해 __repo__ 와 __gist__ 에 체크해야한다.

제일 아래에 있는 __Generate token__ 버튼을 눌러 토큰을 만들면 제일 위에 복사할 수 있는 코드가 뜬다. 이 코드는 꼭 복사해두도록 한다. 이를 붙여넣기 해야하고 페이지를 닫으면 코드가 사라져 *토큰을 재발행 해야한다*. 이 __1.__ 의 내용이 각각의 Awesome Pinned Gist에서 __Prep work__ 라고 설명해둔 두 가지를 완료한 것이다.

### 2. 원하는 Awesome Pinned Gist의 레포지토리 Fork 하기

내가 프로필에서 바로 보여주고 싶은 Awesome Pinned Gist의 레포지토리를 선택하여 들어간다. 예시의 레포지토리는 [Stats Box](https://github.com/bokub/github-stats-box)다. 그 다음 레포지토리의 오른쪽 위에 보면, Watch, Fork, Star 메뉴가 보이는데, Fork를 선택한다. 레포지토리를 Fork한다는 것은 *지금 이 상태의 레포지토리를 나의 저장소에 똑같이 배껴오겠다*는 의미라고 생각하면 된다. Fork 시 팝업으로 선택지가 두 개 나오는 경우가 있는데, 이때는 기여 용도가 아니라 개인 사용 목적으로 선택하면 된다. Fork를 마치면 나의 레포지토리 목록에서 볼 수 있다. 아래 사진과 같이 원래의 것과 같은 이름의 레포이며 어디서 fork되었는지 명시되어있다.

![](https://user-images.githubusercontent.com/86394389/147640952-6a09f3b8-7f43-4039-a986-ee90558d336d.png)

### 3. Secret과 run.yml 설정하기

1.에서 만들어둔 토큰과 Gist를 사용할 것이다. 여기부터는 Awesome Pinned Gist들이 설명하는 Project setup의 내용과 약간 다른데, 순서만 다른 것이고 동작에는 문제가 되지 않는다. 레포지토리 이름 바로 아래 여러가지 메뉴들이 있다.

![](https://user-images.githubusercontent.com/86394389/147641764-29fbb21d-e2a5-4c78-b10a-6a06fefcc375.png)

이 중에 __Setting__ 을 눌러 Secret을 설정할 것이다. 그 뒤, 왼쪽 메뉴에서 __Secrets__ 를 선택한다. Actions와 Dependabot이 있는데 기본 선택되어있는대로 Actions에서 작업할 것이다. 화면은 아래 사진에 나와있는 것처럼 구성되어 있다. 

![](https://user-images.githubusercontent.com/86394389/147642168-747dc827-d572-4de0-b12b-281288ecaefc.png)

오른쪽 위의 __New repository secret__ 버튼을 누르면 새로운 Secret을 추가할 수 있다. 그러면 전환된 화면에서 두 가지 내용을 입력해야한다.

* Name: Secret의 이름이다. 이 이름을 사용해 민감한 정보에 접근한다.
* Value: Secret의 값이다.

그런데, 어떤 이름에 어떤 값을 입력해야할까? 내가 처음 Fork 하고 문제였던 부분이다. 결론부터 말하면 이는 __run.yml__ 파일에서 확인해볼 수 있다. 이 파일의 코드 중 달러 표기 뒤 중괄호로 묶여있는 곳에 Secret 이름이 들어가면 그게 Secrets에서 설정한 값에 접근하는 방법이다. 그러므로 이런 형태를 가진 값을 Secrets에서 설정해두면 된다.

다시 차근차근 하나씩 알아보겠다. 아래는 Stats box의 Peoject setup 설명 중 일부이다.

![](https://user-images.githubusercontent.com/86394389/147642817-e778011c-e0c3-4697-9cf0-0f78af059210.png)

여기서 설명한대로 GH_TOKEN이란 이름의 Secrets를 추가하고 복사해둔 토큰을 Value에 넣고 저장했다. 그리고 run.yml을 바꾸려 보니 GIST_ID의 내용이 완전히 달라 적용하기 곤란했다. 그래서 이것 저것 바꿔보고 gist 이름도 넣어보고 코드랑 눈치 싸움을 좀 했다. '역시 형태가 같으니 GIST_ID도 Secrets에 넣어보자!'라는 생각이 정답이었다. 그래서 Secrets에는 아래의 내용대로 값을 넣어주면 된다. (GIST_ID의 경우 run.yml 코드에 바로 적어주는 경우도 있다.)

| Name | Value | Description |
| ------- | ------- | ------- |
| GH_TOKEN | 복사해둔 token 값 | 나의 Github 계정에서 제공할 정보를 알려준다. |
| GIST_ID | gist 링크 | 연결할 gist의 url 중 자신의 계정 이름 뒤 부분을 복사해 붙여넣기 한다. |

그 다음으로는 run.yml의 내용을 바꿀 것이다. github-stats-box는 ALL_COMMITS와 K_FORMAT이라는 속성을 제공한다. ALL_COMMITS가 true면 *전체 commit 갯수*를, false면 *전년도 커밋 갯수*을 제공한다. K_FORMATS가 true이면, 숫자 값이 클 때 'k'와 같은 *SI 단위계 접두어*를 붙이고 false면 *숫자 그대로* 나타난다. 일단, 레포의 유일한 폴더인 __.github/workflows__ 폴더에 들어가 __run.yml__ 파일을 눌러 확인한다. clone해서 편집기로 작업해도 되지만 나는 깃허브 사이트에서 바로 편집했다. 코드의 오른쪽 위 연필 모양의 아이콘을 누르면 편집할 수 있다. 

![](https://user-images.githubusercontent.com/86394389/147645151-b243c2fb-b97d-41e6-b681-df08c598b8a3.png)

코드의 가장 아래쪽을 보면 지금까지 설정할 수 있던 것들을 한 번에 볼 수 있다. 취향껏 이 내용들을 변경하고 저장한다. (이미 Action을 켠 상태면 Start commit 버튼을 누르면 된다.)

### 4. Action 설정과 확인

아까 Settings를 눌렀던 메뉴에 보면 Actions라는 항목이 있다. 클릭하면 enable할지 선택하는 버튼이 나온다. 선택하면 현재 수행중인 Actions가 나온다. 확인해보고 수행중인 Action이 없거나 Gist에 적용이 안되었다면 workflow를 선택하여 직접 Run workflow할 수 있다. 기본적으로 이미 1개의 workflow가 있다. 이름도 친절하게 Update Github Stats Gist다.

![](https://user-images.githubusercontent.com/86394389/147645850-806ca2cb-27eb-4f6d-81aa-7ddc1757ec00.png)

위의 사진처럼 workflow를 선택하면 수동으로 run할 수 있는 버튼이 나온다. 만약 업데이트가 되지 않았다면 이 버튼을 눌러 run한다.

![](https://user-images.githubusercontent.com/86394389/147645945-40dbc748-85d2-4db9-8197-0e4aed4111fb.png)

확인은 Gist를 새로고침해보거나 Pin하면 된다. Pin 하는 것은 프로필에 Pinned 항목 오른쪽에서 Customize your pins라는 글자를 누르면된다. *레포지토리가 아니라 Gist를 pin하도록 한다*. 

## 완성!

아래 사진은 프로필에 pin한 모습이다. 나는 코드에 이상한 점을 잘 모르겠으면 내가 fork한 레포의 원래 레포를 들어가서 코드를 비교해본다. 비교해보고 차이점을 확인하면서 진행하면 다른 Awesome Pinned Gist를 사용할 때도 도움이 될 것이다.

![](https://user-images.githubusercontent.com/86394389/147191070-926d5c45-74d7-4cb5-b08d-ae3f7fa42743.png)