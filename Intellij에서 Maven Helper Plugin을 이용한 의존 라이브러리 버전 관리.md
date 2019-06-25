## Intellij에서 Maven Helper Plugin을 이용한 의존 라이브러리 버전 관리


예전에 Visual Studio에서 라이브러리들을 관리해 주는 nuget 이란 프로그램을 사용한 적이 있었는데 매우 편리했던 기억이 있습니다.

그래서 **Intellij**에서도 좀 더 편하게 라이브러리들을 관리하는 방법이 없을까
해서 찾아보았습니다.

</br>

### Intellij Show Dependencies

Maven Projects에 다이어그램 형태(Show Dependencies)로 보는 기능이 있긴 하지만
의존하는 라이브러리가 늘어날수록 속도가 느려지고 보기도 매우 힘듭니다.
(**pass...**)

![img/mavenhelper/081446f1af5884b86f56274756ece43e](img/mavenhelper/943f93ed58e8220ffd07a8cc31d1a9b1.tmp)

### mvn dependency:tree

해당 명령어를 사용하면 의존하는 라이브러리에 대한 정보를 볼 수 있지만 TEXT
형태라서 직관적이지 않습니다. (**pass...**)

![img/mavenhelper/f15722a3cdd6a9f69440c341dc1702cc](img/mavenhelper/319042ab7996db3921b8f560d6feb399.tmp)

## Maven Helper Plugin
Intellij의 Maven 의존 라이브러리 관리 플러그인 중 가장 다운로드 수가 많고 별점도
높으며 업데이트 최근까지 진행되고 있습니다. (**good**)

몇 가지 비슷한 Plugin을 테스트해보았는데 개인적으로는 가장 좋았습니다.

![img/mavenhelper/7121202b4391b783231f05bf6472f75d](img/mavenhelper/a6d288f618883b9123e581fab9ed4c51.tmp)

<br/>

#### 1) 설 치

인터넷 망인 경우 Intellij의 **File \> Settings \> Plugins \> Browse
Repositories**에서 **Maven Helper** 검색 후 설치하시면 됩니다.

![img/mavenhelper/69083366c14fb921acd6e236259d1bdb](img/mavenhelper/03c5028b1dc579ecd092679a1b1f2ca6.tmp)

사내 망인 경우 Intellij Version에 맞는 Plugin을 다운로드한 후 **File \> Settings
\> Plugins \> Install plugin from disk**로 수동 설치하시면 됩니다.

**다운로드**) <https://plugins.jetbrains.com/plugin/7179-maven-helper/versions>

![img/mavenhelper/b0be9512565690b4f9a7897c0dada2c6](img/mavenhelper/122f879b714edcc6ff5bf3377f287117.tmp)

<br/>

#### 2) 실 행

실행은 매우 간단합니다. 해당 Plugin을 설치 후(재시작) Intellij에서 pom.xml 을
클릭하면 **Text** / **Dependency Analyzer** 두 가지 탭이 생기는데

Text는 기존 pom.xml 창과 동일하며 **Dependency Analyzer**에서 실제로 의존
라이브러리들의 버전 확인 및 관리를 할 수 있습니다.

![img/mavenhelper/81c4f3831617494c35bad39a7f2681bc](img/mavenhelper/08a150dc74b4d440387469077c0780f3.tmp)

<br/>

#### 3) 기 능

검색 및 여러 관점으로(Conflicts, List, Tree) 의존하고 있는 라이브러리들을 확인할
수 있으며 충돌 나는 라이브러리 대해서 직접 **Exclude** 시킬 수 있습니다.

(참고로 우측 창에 현재 사용하고 있는 버전은 하얀색으로 나오고 사용되지 않고 있는
버전은 빨간색으로 나옵니다.)

![img/mavenhelper/5c9903e505143ec15cc811e1dc1a7e27](img/mavenhelper/275c16dc90c4f8125843eae5300e0d41.tmp)

**Exclude**를 사용하면 실제 pom.xml에 자동으로 exclusion 태그가 추가돼서 해당
라이브러리 제외됩니다.

**pom.xml**
``` xml
<dependency>
   <groupId>net.logstash.logback</groupId>
   <artifactId>logstash-logback-encoder</artifactId>
   <version>4.6</version>
   <exclusions>
      <exclusion>
         <artifactId>logback-core</artifactId>
         <groupId>ch.qos.logback</groupId>
      </exclusion>
   </exclusions>
</dependency>
```

한 가지 단점으로는 Plugin 화면이 자동으로 **Refresh**가 되지 않기 때문에
수동으로 처리해야 됩니다.

![img/mavenhelper/71f7a0c0b5cf9ca33bc339af5e2fcb54](img/mavenhelper/b56fd7667d81c2626ecfdf65737f925b.tmp)

<br/>

#### 4) 사용사례

logback에 kafka appender를 추가 후, 기동을 하다가 다음과
같이 **NoClassDefFoundError**  에러가 발생했습니다.

![img/mavenhelper/f4c868a7b5e82c3f132e0909b6a41214](img/mavenhelper/180ed38c8a0c837ee936966386cfec87.tmp)

Maven Helper Plugin을 통해 충돌된 라이브러리의 패키지를 검색해보았습니다.

![img/mavenhelper/e7537943adb765b2f2f539a42e89e7d5](img/mavenhelper/410624f0a3fd0b4732859ed774623e50.tmp)

해당 라이브러리를 의존하고 있는 곳은 3곳이었고 pom.xml에 kafka appender와 같이
추가한 logstash-logback-encoder 라이브러리에서

**logback-core 1.1.3**을 참조하고 있었기 때문에 기존에 사용하고 있던 1.1.7
버전에서 1.1.3으로 변경되면서 충돌이 발생했습니다.
우선순위 참고) <https://blog.sapzil.org/2018/01/21/taming-maven-transitive-dependencies>

</br>

충돌이 나는 **logback-core 1.1.3** 버전에 대해서 **Exclude** 시켰고 이후
정상적으로 기동되었습니다.

![img/mavenhelper/5cc038992683a451386036495b8095ab](img/mavenhelper/2eb8cf2941cb4e41cf92c1c41223e556.tmp)

사용 사례와 같이 Maven Helper Plugin를 사용하면 의존 라이브러리 버전 관리를
손쉽게 할 수 있습니다.

저처럼 라이브러리 충돌 때문에 고생하시는 분들이 있을까 해서 정리해 보았습니다.

혹시 더 좋은 방법이 있으면 공유 부탁드립니다.

참고) <https://plugins.jetbrains.com/plugin/7179-maven-helper>
