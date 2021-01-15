---
title: "[Vue.js] 라이프사이클 - 그런데 이제... axios를 끼얹은... 🤯"
date: 2021-01-13 00:00:00 -0400
categories: vue javascript frontend
published: false
---

#### ⚡ 이런 사람에게 도움이 될 포스트에요
* Backend통신으로 데이터를 받아와 Vue 의 자식 컴포넌트에 그려주는 과정에 어려움이 있는 사람
*

####  ⚡ 3줄 요약
* ㅇㅇ
* ㅇㅇ
<br/><br/>

> '아 왜 화면이 새로 안그려지지... 이거 백퍼 lifecycle때문인데...'
Vue Lifecycle로 문제를 겪다가 구글 검색하면 매번 마주하는 Vue Lifecycle 다이어그램.
created 후에 자식부터 mount인 것 까진 알겠는데.. 여전히 문제는 해결이 안된다..

Vue로 개발을 하면서 애먹은 부분이 이 Lifecycle인데요, 특히 컴포넌트의 계층이 많아지고 데이터를 백엔드에서 가져오는 경우에 코드가 의도한대로 동작하지 않아 애를 먹었습니다.

이 포스트에서는 백엔드에서 가져온 데이터를 child component로 넘겨주고,
또 child component에서 데이터를 로딩하여 grand child compoent 로 넘겨 주려고 할 때의 문제점과, 이에 대응한 방법을 소개합니다.

<br/><br/>

## ♻️ Vue Lifecycle

![vue lifecycle](/assets/vue%20lifecycle_qicil0ze0.png)
Vue로 개발을 할 때에, 대부분은 데이터를 백엔드와의 통신을 통해 가져오게 됩니다. 그런데 vue lifecycle 다이어그램은 부모가 모든 데이터를 가지고 있는 상황을 가정하고 있죠.
이 포스트에서는 이렇게 **부모가 통신을 통해 가져온 데이터를 활용해 child component의 데이터를 그려줄 때**의 문제점과 해결 방안을 다뤄보려 합니다.


<br/><br/>
## 📡 바보야, 문제는 axios야!
Vue Lifecycle 그림에는 당연히 parent가 모든 데이터를 가지고 있다고 가정하고 있을 것 입니다. 하지만 일반적으로는, Parent Component도 통신을 통에 데이터를 가져옵니다. 여기서 머리아픈게 시작되죠. 통신을 통해 가져온 데이터는 created, mounted, updated 중 어디에서 들어오는 것이며, child의 created, mounted, updated 중 어느 시점에서 갖고 있게 되는 것일까요?

## 🌟 데이터는 'updated'시점에서야 들어온다!
실험 결과.. 실제로 updated에 와서야 나타나는 것을 알 수 있습니다.
아무리 child component에 넘겨주어 이 값으로 새로운 값을 가져오려 해봐도 null이니 오류만 나는 것이죠. 즉, parent에서 가져온 데이터를 바탕으로 다시 child에서 데이터를 가져오고 싶다면, 그 시점은 'updated'내부가 되어야 합니다.

## 해결책 1, 2, 3
### 1. 아낌없이 주는 부모
첫번째 방법은, 부모에서 모든 데이터를 패치하고, 자식은 그리기만 하는 방법입니다.
업데이트 후 알아서
### 2. watch 👀
### 3. store 📦
