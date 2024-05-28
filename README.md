- My public GitLab project/repository :  [https://gitlab.com/arrowlevent/maven-app1]

```bash
git clone https://gitlab.com/arrowlevent/maven-app1
cd maven-app1
tree
- maven-app1 :
    - src
      - main
        - java
          - com
            - example
              - app
                - App.java
      - test
        - java
          - com
            - example
              - app
                - AppTest.java
    - .gitlab-cicd.yml
    - README.md
    - pom.xml
```

# Create .gitlab-ci.yml in the GitLab :

- ***`***Maven`*** : A Build Lifecycle is Made Up of Phases. Each of these build lifecycles is defined by a different list of build phases, wherein a build phase represents a stage in the lifecycle. For example, the default lifecycle comprises of the following phases (for a complete list of the lifecycle phases, refer to the Lifecycle Reference):
[https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html]

``validate`` - validate the project is correct and all necessary information is available
``compile`` - compile the source code of the project
``test`` - test the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed
``package`` - take the compiled code and package it in its distributable format, such as a JAR.
``verify`` - run any checks on results of integration tests to ensure quality criteria are met
``install`` - install the package into the local repository, for use as a dependency in other projects locally
``deploy``- done in the build environment, copies the final package to the remote repository for sharing with other developers and projects.
These lifecycle phases (plus the other lifecycle phases not shown here) are executed sequentially to complete the default lifecycle. Given the lifecycle phases above, this means that when the default lifecycle is used, Maven will first validate the project, then will try to compile the sources, run those against the tests, package the binaries (e.g. jar), run integration tests against that package, verify the integration tests, install the verified package to the local repository, then deploy the installed package to a remote repository.

- `variables` : `MAVEN_OPTS` adında bir değişken belirleyeceğiz, bu, `.m2/repository` olacak yerel depo olacaktır. `.m2 klasörü`, herhangi bir maven komutunu çalıştırdığınızda maven tarafından oluşturulur. `.m2 dizini` Varsayılan olarak, maven yerel deposu `USER_HOME` şeklindedir. 
[https://docs.gitlab.com/ee/ci/migration/examples/jenkins-maven.html#convert-jenkins-configuration-to-gitlab-cicd]
[https://gitlab.com/gitlab-examples/maven/simple-maven-example]
[https://gitlab.com/gitlab-examples/maven/simple-maven-app]
[https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates/Maven.gitlab-ci.yml]

  - `MAVEN_OPTS` are Maven environment variables needed whenever Maven is executed
  - `-Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository` sets the location of the local Maven repository to the GitLab project directory on the runner, so the job can access and modify the repository.
  - `-Dhttps.protocols=TLSv1.2` sets the TLS protocol to version 1.2 for any HTTP requests in the pipeline.
  - `MAVEN_CLI_OPTS` are specific arguments to be added to mvn commands: `-DskipTests` skips the test stage in the Maven build lifecycle.

- `image maven:latest`  :  Burada GitLab `runner` için bir `docker image` seçiyoruz. Bu uygulama için `maven` imajını kullanacağız çünkü bu bir `Java` projesi ve maven konteynerini kullanacağız. Bir imaj seçmez isek `default imaj` olarak GitLab runner tarafından `ruby:3.1` kullanılır. Ayrıca, pipeline’daki tüm joblarda aynı imajı kullanmak için imajın konumu en üst seviyede olmalıdır.

- `stage` :  4 job'un stage isimleri sıralı şekilde

- `cache` : Maven'in `Build Life` döngüsündeki komutları çalıştırdığında `.jar` dosyası gibi dosyaları kopyalayacağı yoldur.  [https://docs.gitlab.com/ee/ci/yaml/index.html#cache]
  

- `build_job` : Bu case de ilk işin adıdır. Echo komutu ile ekrana `Maven compile started` yazdırır. İkinci satırdaki script bölümü GitLab'a compile yapmasını söyler. Maven, bunların `mvn compile` komutu ile `src` dizinindeki `source code` alıp `byte-code`'a çevirir ve `target` klasörü oluşturur ve onu `class` uzantılı bir dosya olarak `target klasöre kaydeder.

- `test_job`: `mvn test` komutu ile Maven `unit tests` çalıştırır ve çıktıları `src/test` klasörüne kaydeder. Varsayılan Yaşam döngüsünün 15. adımıdır.

- `deploy_job`: Maven, `mvn deploy` komutu ile `jar` uzantılı `artifact` dosyasını oluşturur ve `.m2 klasörüne` kaydeder. Ayrıca maven komutlarının çalışırken görmek zorunda olduğu ana dizindeki `pom.xml` dosyasını da ayarlarsak, bu komutla `Nexus` gibi uzak bir artifact deposuna da kaydedilir. Bu Varsayılan Yaşam döngüsünün 23. adımı ve son adımıdır.

- `tags`: `docker`  In this example, only `runners` with `docker` `tag` can run the job. Tüm joblar bu tag'e sahip runner içerisinde koşacak. Eğer hiç bir job içerisinde tag kullanmazsan da GitLab belirtilen image'i kullanarak bir runner kullanır.
  - `image: maven:latest` ifadesi zaten kullanılacak Docker imajını belirlediği için, tags kısmı belirli bir runner'ı işaretlemek için gereksiz olacaktır. Ancak, genel olarak projenizde birden fazla runner varsa veya farklı runner'ları farklı işlerde kullanmak istiyorsanız, tags kullanarak bu işleri belirli runner'lar üzerinde çalıştırabilirsiniz.
  [https://docs.gitlab.com/ee/ci/runners/configure_runners.html#use-tags-to-control-which-jobs-a-runner-can-run]
  [https://docs.gitlab.com/ee/ci/yaml/index.html#tags]


# `$CI_RUNNER_TAGS` : ["gce", "east-c", "linux", "ruby", "mysql", "postgres", "mongo", "git-annex", "shared", "docker", "saas-linux-small-amd64"]  BU TAG'ler KULLANILIYOR

```yaml (.gitlab-ci.yml)
variables:
  # MAVEN_OPTS: -Dmaven.repo.local=.m2/repository
  MAVEN_OPTS: >-
    -Dhttps.protocols=TLSv1.2
    -Dmaven.repo.local=$CI_PROJECT_DIR/.m2/repository
  MAVEN_CLI_OPTS: >-
    -DskipTests

image: maven:latest

stages:
    - build
    - test
    - package
    - install
    - deploy

cache:
  paths:
    - .m2/repository
    - target

build_job:
  stage: build
  tags:
    - docker 
  script: 
    - echo "Maven compile started"
    - "mvn compile"

test_job:
  stage: test
  tags:
    - docker 
  script: 
    - echo "Maven test started"
    - "mvn test"

package_job:
  stage: package
  tags:
    - docker 
  script: 
    - echo "Maven packaging started"
    - "mvn package"

install_job:
  stage: install
  tags:
    - docker 
  script: 
    - echo "Maven install started"
    - "mvn install"
    - echo "$CI_RUNNER_ID"
    - echo "$CI_RUNNER_TAGS"

Deploy_job:
  stage: deploy
  tags:
    - docker 
  script: 
    - echo "Maven deploy started"
```

# Create pom.xml :

```xml  (pom.xml)
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example.app</groupId>
  <artifactId>simple-maven-app</artifactId>
  <packaging>jar</packaging>
  <version>1.0</version>
  <name>simple-maven-app</name>
  <url>http://maven.apache.org</url>
  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>
  <repositories>
    <repository>
      <id>gitlab-maven</id>
      <url>https://gitlab.com/api/v4/projects/3467553/packages/maven</url>
    </repository>
  </repositories>
  <dependencies>
    <dependency>
      <groupId>com.example.dep</groupId>
      <artifactId>simple-maven-dep</artifactId>
      <version>1.0</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

# Create App.java 

```java (App.java)
package com.example.app;

import com.example.dep.Dep;

public class App 
{
    public static void main( String[] args )
    {
        Dep.hello( "GitLab" );
    }
}
```

# Create AppTest.java


```java (AppTest.java)
package com.example.app;

import junit.framework.Test;
import junit.framework.TestCase;
import junit.framework.TestSuite;

/**
 * Unit test for simple App.
 */
public class AppTest 
    extends TestCase
{
    /**
     * Create the test case
     *
     * @param testName name of the test case
     */
    public AppTest( String testName )
    {
        super( testName );
    }

    /**
     * @return the suite of tests being tested
     */
    public static Test suite()
    {
        return new TestSuite( AppTest.class );
    }

    /**
     * Rigourous Test :-)
     */
    public void testApp()
    {
        assertTrue( true );
    }
}
```

- Click `Commit changes`

- Pipeline aşamalarını/joblarını buradan gözlemleyebilirsin`Build` -- `Pipelines`

# Resources :

- https://gitlab.com/arrowlevent/maven-app1
- https://gitlab.com/gitlab-examples/maven/simple-maven-example
- https://gitlab.com/gitlab-examples/maven/simple-maven-app
- https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates/Maven.gitlab-ci.yml
- https://docs.gitlab.com/ee/ci/migration/examples/jenkins-maven.html#convert-jenkins-configuration-to-gitlab-cicd
- https://docs.gitlab.com/ee/ci/yaml/index.html#cache
- https://docs.gitlab.com/ee/ci/runners/configure_runners.html#use-tags-to-control-which-jobs-a-runner-can-run
- https://docs.gitlab.com/ee/ci/yaml/index.html#tags