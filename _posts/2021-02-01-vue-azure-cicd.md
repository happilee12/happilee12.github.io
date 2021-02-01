---
title: "Azure App Service에 Vue프로젝트 CICD 설정"
date: 2021-02-01 00:00:00 -0400
categories: azure app-service vue cicd
---

### 🧑‍🏫 이런 분들이 읽으면 도움이 되는 글입니다

* **프론트엔드 프로젝트(Vue.js)**를 **Azure App service**에 배포해보려고 하시는 분
* <u>VS Code를 사용하지 않고</u> App service를 배포하고 싶으신 분
* <u>Github이나 Azure pipeline연동 없이</u> App service를 배포하고 싶으신 분



###  📝 요약하자면 이런 내용입니다

* Deployment Center를 local git로 설정하고, 프로젝트의 remote git에 추가하면 된다
* startup command는 `pm2 serve /home/site/wwwroot/build --no-daemon`  로 설정해야 한다



------------



이번 포스트에서는 **Azure App Service**의 **Deployment Center** 콘솔에서 **local git**과 **kudu** 통해  **Vue.js** 프로젝트의 **CI/CD(Continuous Deploymenet)** 설정을 해보겠습니다.

<br>

Azure App Service 디플로이에 대해서는 MS에서 여러 도큐먼트가 제공되고 있습니다.

하지만 매번 찾아보려고 하면 상황에 딱 맞는 도큐먼트를 찾기는 쉽지 않죠.

이번에 Azure App Service 디플로이를 위해 찾아본 도큐먼트는 주로 VS Code를 활용하거나 Azure Devops를 통한 방법을 다루고 있었는데,  현재 팀에서는 intelij를 통해 코드를 작성하고 bitbucket을 통해 버전관리, jenkins를 통해 CICD를 구성하고 있기 때문에 도큐먼트대로 따라하기엔 제약이 있었습니다. (매번 이런 식인것 같아요... 도큐먼트는 많은데 막상 내 상황에 필요한 건 없는)



<img src="/assets/2021-02-01-vue-azure-cicd/image-20210129173403765.png" alt="image-20210129173403765" style="zoom:33%;" />

이렇게 힘으로라도 도큐먼트에 꾸겨 맞출 수 있으면 좋으련만.. 삽질 시간만 늘어가네요.. ㅠㅠ

혹시 저와 비슷한 환경에서 App Service 를 배포하시려는 분이 있다면 도움이 될까 하여 포스트를 작성해 봅니다.



참고로, Azure Pipeline이나 github, bitbucket을 연동하실 수 있는 상황이면 아래 (1)을,
Node앱을 배포하고 VS Code를 사용할 수 있으면 (2) 를 따라하시면 됩니다.

> Azure에서 제공되는 Azure App Service Deploy관련 Documnets
>
> * [(1) Azure App Service에 지속적인 배포](https://docs.microsoft.com/ko-kr/azure/app-service/deploy-continuous-deployment)
>
>   : App Service build service나 Azure pipeline을 활용한 배포
>
> * [(2) Create a Node.js web app in Azure](https://docs.microsoft.com/en-us/azure/app-service/quickstart-nodejs?pivots=platform-linux)
>
>   : VS Code를 통해 Node.js 서비스 배포



## 1. Vue project생성

Vue 프로젝트를 하나 생성해주겠습니다.

Vue Create을 통해 생성하였고, 설정은 Vue3 Default로하였습니다.



* vue create로 프로젝트 생성 :    `$ vue create testapp`

* 설정 선택 중 Default (Vue 3 Preview) 선택 ![image-20210201094619248](/assets/2021-02-01-vue-azure-cicd/image-20210201094619248.png)

* 해당 프로젝트로 들어가서 (`cd testapp`) 로컬에서 프로젝트를 띄워보면(`npm run serve`)  **localhost:8080**에 vue 기본 화면이 띄워져있는걸 확인할 수 있습니다

  ![image-20210201094957018](/assets/2021-02-01-vue-azure-cicd/image-20210201094957018.png)





이제 이 프로젝트를 그대로 auzre app service에 띄워보겠습니다.



## 2. App Service 리소스 생성



<img src="https://ms-azuretools.gallerycdn.vsassets.io/extensions/ms-azuretools/vscode-azureappservice/0.20.0/1604973785944/Microsoft.VisualStudio.Services.Icons.Default" alt="image" style="zoom:50%;" />


해당 프로젝트를 띄울 azure app service를 만들어줍니다. Vue 프로젝트를 띄울 앱서비스이기 때문에 Runtime stack은 Node로, OS는 Linux로 설정해주세요. 저는 아래와 같이 만들었으니 참고해주세요

```
Name : 202101testapp
Publish : Code
Runtime stack : Node 12 LTS
Operating System : Linux
App service plan : B1
Application Insights : Not enabled
```



## 3. local git 설정을 위한 KUDU정보 복사

app service의 왼쪽 패널 중 Deployment Center를 들어가보면 어떤 방식으로 디플로이 할지 선택할 수 있습니다.



#### 01. Source Control - local git

![image-20210129172808204](/assets/2021-02-01-vue-azure-cicd/image-20210129172808204.png)

app service의 왼쪽 패널 중 Deployment Center를 들어가보면 어떤 방식으로 디플로이 할지 선택할 수 있습니다.

여기서 CICD의 네번째 메뉴인 **local git을 선택**해 줍니다.

local git을 사용하면 다른 서비스의 계정을 생성하거나 연동할 필요 없이, 그냥 생성된 remote git으로 push만 해주면 되는 방식이라 가장 제약 없이 사용할 수 있습니다. (저희 회사의 경우, 회사 내에서는 bitbucket기반의 프로그램을 사용하고 있음에도 사내 보안 규정 때문인지 azure와 bitbucket연동은 막혀있어서 활용하지 못하고 있습니다 ㅠㅠ)



#### 02. Build Provider - App Service build service

![image-20210129173113685](/assets/2021-02-01-vue-azure-cicd/image-20210129173113685.png)

Build Provider로는 App Service build service를 선택해줍니다. 이는 **kudu** 기반으로 동작하는데, kudu는 App service kudu engine을 통해 별도의 설정 없이 자동으로 코드를 빌드할 수 있는 서비스입니다. [kudu 프로젝트 더보기](https://github.com/projectkudu/kudu/wiki)



![image-20210129174404362](/assets/2021-02-01-vue-azure-cicd/image-20210129174404362.png)

Deploy Center쪽 설정은 끝입니다.



#### 03. remote git 정보 복사

다시 Deployment Center 메뉴에 들어가 보면, source: local git 으로 설정된 것을 확인할 수 있습니다.

![image-20210129174628805](/assets/2021-02-01-vue-azure-cicd/image-20210129174628805.png)

**Git Clone Uri**는 우리가 코드를 푸시해야 할 remote git 주소입니다.





<img src="/assets/2021-02-01-vue-azure-cicd/image-20210129174834674.png" alt="image-20210129174834674" style="zoom:50%;" />

그리고 열쇄 모양의 **Deployment Credentials** 버튼을 클릭해보면 저 remote git 주소로 push하기 위한 계정 정보를 볼 수 있습니다.

패널에 보여지는 Git Clone URL, Username, Password는 로컬에서 remote git을 설정할 때 필요합니다.

Git Clone URL : 푸시할 remote url

Username/password:  remote url에 대한 계정. 첫 푸시일 한번만 설정해주면 됨.



![image-20210129174747760](/assets/2021-02-01-vue-azure-cicd/image-20210129174747760.png)



remote git과 username, password를 local project에 설정해주고, **해당 remote git의 master브랜치로 푸시**하면 app service로 디플로이 되는 방식입니다.





## 4. 로컬 프로젝트에 azure remote git 연결

1에서만든 vue 프로젝트에 deployment center의 remote git 정보를 연결해 줍니다.

```
$ git remote add azure [remote git url] // 새로운 remote git url을 azure라는 이름으로 추가
$ git push azure master // azure의 master브렌치로 푸시
// (초기1회) username과 password 설정
Username for 'https://202101testapp.scm.azurewebsites.net:443': [Username]
Password for 'https://$202101testapp@202101testapp.scm.azurewebsites.net:443': [Password]
```



![image-20210201092738987](/assets/2021-02-01-vue-azure-cicd/image-20210201092738987.png)
(디플로이 진행 화면)


디플로이가 완료된 후 아까 Deployment Center를 들어가보면 디플로이 상태가 업데이트 되어있습니다.



![image-20210201093315043](/assets/2021-02-01-vue-azure-cicd/image-20210201093315043.png)

STATUS에 Success라고 잘 떠있죠? ✌️





## 05. App Service Startup Command 설정

디플로이 후 App Service에서 실행할 명령어(Startup Comand)를 설정해줍니다.

 Configuration > General settings > StartupCommand 쪽에 아래 command를 입력해줍니다.

 `pm2 serve /home/site/wwwroot/dist --no-daemon` 라고 입력해줍니다.

참고) https://azureossd.github.io/2020/04/30/run-production-build-on-app-service-linux/

![image-20210201101410838](/assets/2021-02-01-vue-azure-cicd/image-20210201101410838.png)



설정 후 app service를 재시작한 후 App Service URL에 접속해보면 Vue 프로젝트가 정상적으로 디플로이 된 것을 확인할 수 있습니다.  🎉🥳



![image-20210201101730659](/assets/2021-02-01-vue-azure-cicd/image-20210201101730659.png)





## 06. 새로운 commit에 대한 deploy 확인

새로 커밋했을 때 app service에 잘 반영되는지도 확인해보겠습니다.

로컬 프로젝트에서 App.vue의 msg를

"Welcome to Your Vue.js App 🥳 🥳 🥳"

 로 바꿔주고 커밋해보았습니다.

```
$ git add .
$ git commit -m "changed msg"
[master c5f7327] changed msg
 1 file changed, 1 insertion(+), 1 deletion(-)
$ git push azure master
Enumerating objects: 7, done. (...)
```
