---
layout: post
title:  Gist와 Secret을 이용한 Awesome Pinned Gist 적용하기
date:   2021-12-23 14:04:30 +0300
image:  02.jpg
tags:   Github
categories: github tips
---

깃허브 프로필을 정리하면서 알게 된 기능인 Gist와 Secrets에 대해 간략한 설명을 곁들여 Awesome Pinned Gists의 Gist들을 사용하는 방법을 정히해보았다. 내가 Gist와 Secrets를 사용한 건 아래 사진과 같은 [스테이트 박스](https://github.com/bokub/github-stats-box)다. 

![](https://user-images.githubusercontent.com/86394389/147191070-926d5c45-74d7-4cb5-b08d-ae3f7fa42743.png)

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


### 사용할 Gist와 토큰 만들기

Gist의 이름과 내용은 아무것이나 좋다. 아래 사진과 같이 Github Gist 파일을 하나 만든다. __중요한 것은 공개 Gist로 만드는 것이다.__ 사진에서 오른쪽 아래 초록색 버튼을 보면 화살표를 눌러 private와 public을 선택할 수 있다. 프로필에 pin하기 위해서는 public이어야 하므로 public으로 설정한다.이 Gist 파일의 확장자는 중요하지 않다.

![](https://user-images.githubusercontent.com/86394389/147466154-453718b5-4423-44dd-902e-dbf5741c7a9d.png)

그 다음으로 토큰을 만들어야 한다. 토큰은 Github에서 Setting(설정)에서 만들 수 있다. 자신의 프로필과 계정에 관한 것이 뜨는데, 왼쪽 카테고리 중 가운데 블록에 있는 __Developer Settings__ 으로 들어간다. 그럼 다음과 같은 페이지가 뜬다. 여기서 __Personal access tokens__ 를 선택한다.

![](https://user-images.githubusercontent.com/86394389/147466682-a2a5ddf1-2579-4043-8fe3-29f70847e017.png)

Generate new token 버튼을 눌러 토큰을 새로 하나 만든다.

![](https://user-images.githubusercontent.com/86394389/147466934-ac8bae98-16ee-4e4b-a70f-ff74be04a70f.png)

필요한 내용들을 설정한다.

* Note: 토큰의 이름 정도로 생각하면 된다.
* Expiration: 만료일이다. 원하는 수치로 선택한다.
* Scope: 토큰이 정보를 얻어오는 범위다. Gist에 정보를 보여주기 위해 __repo__ 와 __gist__ 에 체크해야한다.

제일 아래에 있는 Generate token 버튼을 눌러 토큰을 만들면 제일 위에 복사할 수 있는 코드가 뜬다. 이 코드는 꼭 복사해두도록 한다. 이를 붙여넣기 해야하고 페이지를 닫으면 코드가 사라져 *토큰을 재발행 해야한다*.

### 원하는 Awesome Pinned Gist의 레포지토리 Fork 하기

내가 프로필에서 바로 보여주고 싶은 건 


