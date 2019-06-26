## SBT에 NEXUS 설정하기


### SBT란?
sbt는 자바의 maven과 비슷한 일을 하는 빌드툴입니다.

sbt는 기본적으로 여러곳의 repository에 접근해서 필요한 라이브러리들을 가지고 옵니다.

**-local :** local 레파지토리

**-maven-central** : maven에서 관리하는 라이브러리

**-typesafe-ivy** : sbt는 기본적으로 의존성 관리를 위해 ivy 사용

**-scala-plugin** : sbt에서 관리하는 scala 라이브러리

![img/sbt/8ef3ff58116711636570dd39ec6f0348](img/sbt/736dbbe70ecd6cab0731beea95ad414a.tmp)

</br>

NEXUS를 통해 해당 주소를 접근하기 위해서는 외부 레파지토리로 미리 등록해 줘야합니다.
![img/sbt/187a0a515dafcd4c72b4e47dc5082d2f](img/sbt/6c777d7e9b830b1155d1ef107da8d24c.tmp)
#참고)

<http://www.bench87.com/content/7>

<http://www.scala-sbt.org/1.x/docs/Proxy-Repositories.html>

</br>

### 1) SBT NEXUS Repository 설정
SBT 관련 NEXUS를 사용하려면 maven에서 .m2/settings.xml에 NEXUS local repository 를 설정하는것 처럼 설정 파일을 추가해 주어야 합니다.

</br>

#### 1-1) sbtconfig.txt


\${SBT_HOME}/conf/sbtconfig.txt 에 다음과 같은 옵션을 추가 합니다.
``` config
-Dsbt.override.build.repos=true
-Dsbt.repository.config="\${USERS}/.sbt/repositories"
``` 

\-Dsbt.override.build.repos 내부 레파지토리만 사용 여부

\-Dsbt.repository.config 레파지토리 관련 설정 위치. \${USERS}/.sbt 경로내에 repositories 파일이 있다면 따로 설정 필요 없음.

</br>

#### 1-2) repositories
\${USERS}/.sbt에 repositories 파일을 추가해 주고 다음과 같이 설정합니다.

``` config
[repositories]
maven-repo : http://NEXUS주소:8080/nexus/content/groups/rootRepo
sbt-plugin-repo : http://NEXUS주소:8080/nexus/content/repositories/org-scala-sbt-repo-plugin-release, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext]
typesafe-ivy-repo : http://NEXUS주소:8080/nexus/content/repositories/typesafe-ivy-release, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext]
```

**-maven-repo :** mvn-release proxy 대응

**-sbtplugin-repo :** ivy-release proxy에 http://repo.scala-sbt.org/scalasbt/sbt-plugin-releases 대응

**-typesafevy-repo** : ivy-release proxy에 http://repo.typesafe.com/typesafe/releases 대응

</br>

### 1-3) 매크로 체계
repositories에 매크로를 설정해서 관련 파일 포멧을 설정할 수 있습니다.

</br>

#### Default
저장소 뒤에 매크로가 없으면 maven 기본 형태로 라이브러리를 찾아오는것 같습니다.

</br>

ex) addSbtPlugin("org.flywaydb" % "flyway-sbt" % "4.1.2")
``` config
https://flywaydb.org/repo/ + org/flywaydb/flyway-sbt_2.10_0.13/4.1.2/flyway-sbt-4.1.2.pom
[organization] = org/flywaydb
[module](_[scalaVersion])(_[sbtVersion])/=flyway-sbt_2.10_0.13
[revision] = 4.1.2
[module](-[revision] ).[ext] = flyway-sbt-4.1.2.pom
``` 

</br>

#### ivy
ivy 관련 레파지토리 관련 매크로는 다음과 같습니다.
``` config
[organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext]
``` 

</br>

ex) addSbtPlugin("com.typesafe.play" % "sbt-plugin" % "2.5.3")  
``` config
http://dl.bintray.com/sbt/sbt-plugin-releases/ + com.typesafe.play/sbt-plugin/scala_2.10/sbt_0.13/2.5.3/ivys/ivy.xml
[organization] = com.typesafe.play
[module]=sbt-plugin (scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]=scala_2.10/sbt_0.13/2.5.3
[type]s=ivys
[artifact](-[classifier]).[ext]=ivy.xml
``` 

</br>

### 2) SBT 라이브러리 관련 동기화
![img/sbt/afa870bd637155dd150b5e102cfdb9c2](img/sbt/9f52d42df65bacea75e50195bee48574.tmp)

SBT내에 NEXUS 설정이 끝났으면 **sbt update** 명령어를 이용해서 관련 라이브러리를 동기화 시킵니다.

</br>

### 3) 구동
#### 3-1) 경로
Run \> Edit Configurations 를 실행 시킵니다.

![img/sbt/2eeac3e8b212cc2c908c9f5b33d1ccff](img/sbt/e9ee2d3242b496f9944eb2ff1c398c26.tmp)

</br>

#### 3-2) Task 등록
Tasks에 프로젝트에 맞는 기동 스크립트를 설정 후, 프로젝트 구동 시켜 주시면 됩니다.
``` config
"project http" "~re-start" 
``` 

**- project http :** project 내에 http 프로젝트

**- \~re-start : 기동 스크립트
![img/sbt/77ca6d06a75826d0d4b4183132f85aa8](img/sbt/3d34cee3277b1415d58c35c26de04a1c.tmp)

</br>

### 4) 이슈 해결
#### 4-1)  PKIX path building failed..

사내 로컬인 경우, SSL/TLS 관련 신뢰 할 수 없는 사이트일 경우 sbt update 하다가 인증서 관련 에러가 나기도 합니다.
하위 인증 관련 부분 참고하시면 됩니다.

\#참고)

<https://groups.google.com/forum/#!topic/pinpoint_user/8nyymde8XSo>
