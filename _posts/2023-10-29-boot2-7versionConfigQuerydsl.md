---
title: "[JPA] SpringBoot 2.7 버전 QueryDSL 설정"
categories:
    - spring
---

### 개발 환경
---

- intelliJ Ultimate
- SpringBoot 2.7.17
- Gradle

<br>

### build.gradle
---

```java
buildscript {
	ext {
		queryDslVersion = "5.0.0"
	}
}

plugins {
	id 'java'
	id 'org.springframework.boot' version '2.7.17'
	id 'io.spring.dependency-management' version '1.0.15.RELEASE'
	id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
}

group = 'com'
version = '0.0.1-SNAPSHOT'

java {
	sourceCompatibility = '11'
}

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	compileOnly 'org.projectlombok:lombok'
	runtimeOnly 'com.h2database:h2'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'

	// queryDsl
	implementation "com.querydsl:querydsl-jpa:${queryDslVersion}"
	implementation "com.querydsl:querydsl-apt:${queryDslVersion}"
}

tasks.named('test') {
	useJUnitPlatform()
}

def querydslDir = "$buildDir/generated/querydsl"

querydsl {
	jpa = true
	querydslSourcesDir = querydslDir
}

sourceSets {
	main.java.srcDir querydslDir
}

compileQuerydsl {
	options.annotationProcessorPath = configurations.querydsl
}

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
	querydsl.extendsFrom compileClasspath
}

```

<br>

### **여기서 주의할 점**
---

쌍따옴표로 감싸진 부분은 쌍따옴표 그대로 작성해야 한다. 만약 다음과 같이 작성하면 아래 처럼 Build에 실패함.
```java
// queryDsl
	implementation 'com.querydsl:querydsl-jpa:${queryDslVersion}'
	implementation 'com.querydsl:querydsl-apt:${queryDslVersion}'
```

```
PS C:\Users\wkdrn\..dev\querydsl> ./gradlew clean compileQuerydsl

Welcome to Gradle 8.3!

Here are the highlights of this release:
 - Faster Java compilation
 - Reduced memory usage
 - Support for running on Java 20

For more details see https://docs.gradle.org/8.3/release-notes.html

Starting a Gradle Daemon, 1 incompatible Daemon could not be reused, use --status for details
> Task :compileQuerydsl FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':compileQuerydsl'.
> Could not resolve all files for configuration ':querydsl'.
   > Could not find com.querydsl:querydsl-apt:${queryDslVersion}.
     Required by:
         project :
   > Could not find com.querydsl:querydsl-jpa:${queryDslVersion}.
     Required by:
         project :
   > Could not find com.querydsl:querydsl-apt:${queryDslVersion}.
     Required by:
         project :

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.
> Get more help at https://help.gradle.org.

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.   

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.3/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

BUILD FAILED in 7s
4 actionable tasks: 3 executed, 1 up-to-date
```

<br>

- **정상적으로 수행 되었을 때**
```java
PS C:\Users\wkdrn\..dev\querydsl> ./gradlew clean compileQuerydsl

Deprecated Gradle features were used in this build, making it incompatible with Gradle 9.0.

You can use '--warning-mode all' to show the individual deprecation warnings and determine if they come from your own scripts or plugins.

For more on this, please refer to https://docs.gradle.org/8.3/userguide/command_line_interface.html#sec:command_line_warnings in the Gradle documentation.

BUILD SUCCESSFUL in 4s
4 actionable tasks: 4 executed

```


<br>

### 정리

- QueryDSL 라이브러리  
```java
implementation "com.querydsl:querydsl-jpa:${queryDslVersion}"
```

- QueryDSL 관련 코드 생성
```java
annotationProcessor "com.querydsl:querydsl-apt:${queryDslVersion}"
```

- QueryDSL 생성된 클래스 파일을 저장할 디렉토리 경로 설정
```java
def querydslDir = "$buildDir/generated/querydsl"
```

- QueryDSL 플러그인을 구성하는 블록
```java
querydsl {
	jpa = true  // JPA 엔티티 클래스를 기반으로 QueryDSL 클래스 생성
	querydslSourcesDir = querydslDir  // 생성된 QueryDSL 클래스 파일들을 'querydslDir' 디렉토리에 저장
}
```

- QueryDSL이 생성한 클래스 파일들을 프로젝트의 소스 세트에 추가하여 컴파일할 수 있도록 지정
```java
sourceSets {
	main.java.srcDir querydslDir
}
```

- QueryDSL 컴파일 작업을 구성하는 블록
```java
compileQuerydsl {
	options.annotationProcessorPath = configurations.querydsl
} // QueryDSL의 annotation processor를 clathpath에 추가하여 컴파일 시 annotation processing을 수행
```

- Gradle의 configurations를 구성하는 블록
```java
configurations {
	compileOnly {
		extendsFrom annotationProcessor // 'compileOnly' 구성을 'annotationProcessor' 구성을 확장하도록 지정 -> 컴파일 시 annotationProcessor를 사용할 수 있도록 설정
	}
	querydsl.extendsFrom compileClasspath // QueryDSL을 컴파일할 때 'complieClasspath' 구성을 확장하여 필요한 의존성을 사용할 수 있도록 설정
}
```