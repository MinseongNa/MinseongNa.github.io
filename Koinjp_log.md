# 2019/06/29
도커로 젠킨스를 설치해보았다.  

$ docker pull jenkins  
$ docker run -d -p 8080:8080 -v /jenkins:/var/jenkins_home --name jenkins -u root jenkins  
-d : 데몬 상태로 실행한다는 뜻이다. 이 옵션을 주지 않으면, 실행되는 로그를 바로 보여준다.  
-p : 컨테이너 내부의 포트를 외부로 내보낼 포트로 연결시켜준다.  
-v : 호스트에 볼륨을 지정해 주는 것이다. 굳이 하지 않아도 되지만, 만약 해당 컨테이너가 삭제되면 내부에 작성했던 스크립트 등의 데이터가 다 없어지기 때문에 볼륨을 지정해 외부에 백업하는 용도로 볼륨을 잡았다.  
--name : 해당 컨테이너의 이름을 지정해준다.  
-u : root 사용자로 실행되게 하기 위해 지정해 줬다.  
https://www.leafcats.com/215  

docker run -d -p 8080:8080 -v /jenkins:/var/jenkins_home --name jenkins -u root jenkins

버전을 지정하지 않아서 가장 마지막 버전으로 설치되었는데, 플러그인의 버전문제로 대부분의 플러그인이 설치되지 않는 문제가 발생했다.  
그 플러그인을 설치하려면 젠킨스버전 2.1xx 이상이 필요한데 설치된 버전은 2.60.3이였다.  
플러그인 가장 밑에 자동 버전업 링크가 있어서 버전업해줬다. 2.176.1이 되었다.  

버전업을 하니 에러메시지만 표시하면서 메뉴가 제대로 표시되지 않는 문제가 발생하였다.
새버전에서는 config.xml의 문서정의가 XML1.1이라 발생한 문제이다. 단순히 config.xml의 버전을 1.0으로 바꾸는 것으로 해결.  

설정을 하다가 보안부분에서 세이브를 했더니 로그인화면으로 이동하고 암호는 모르는 상황이 발생했다.  
젠킨스 도커이미지의 /var/jenkins_home/config.xml에서 <useSecurity>태그를 false로 하면 진입 가능.  

도커 내부에 vi가 없어서 새로 설치해주었다.  
docker -it ${CONTAINER_ID} /bin/bash
apt-get update  
apt-get install vim  

에러메시지는 거의 잡았는데 하나가 남았다.  
Builds in Jenkins run as the virtual SYSTEM user with full permissions by default.  
This can be a problem if some users have restricted or no access to some jobs, but can configure others.  
If that is the case, it is recommended to install a plugin implementing build authentication, and to override this default.  
No implementation of access control for builds is present. It is recommended that you install Authorize Project Plugin or another plugin implementing the QueueItemAuthenticator extension point.  
권한관련 문제인 듯 하다.
아래는 No implementation으로 시작하는 줄에서 표시된 링크.  
https://plugins.jenkins.io/authorize-project  
https://jenkins.io/doc/developer/extensions/jenkins-core/#queueitemauthenticator  
