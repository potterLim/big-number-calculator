# Java 실습

이 Java 실습들은 Java 문법에 대한 기초 숙련부터 구조적 사고, 파일 처리, 클래스 설계, 그리고 종합 구현 능력까지 단계적으로 훈련할 수 있도록 설계되었다.

각 실습은 이전 단계에서 익힌 개념을 자연스럽게 확장하며 단순한 문법 암기나 API 사용을 넘어 문제를 분석하고 해결 과정을 체계적으로 나누는 사고 방식을 기르는 데 목적이 있다.

최종 목표는 Java 문법을 아는 것에 그치지 않고 Java로 문제를 설계하고 구현할 수 있는 개발자로 성장하는 것이다.

## 실습 환경(권장)

이 실습은 다음 환경을 기준으로 설명을 제공한다.
- IDE: [IntelliJ IDEA](https://www.jetbrains.com/ko-kr/idea/)
- JDK: [Java 17(LTS) Temurin OpenJDK](https://adoptium.net/temurin/releases?version=17)
- Language Level: Java 17
- 외부 라이브러리: 사용하지 않음 (Java 표준 라이브러리만 사용)

이미 익숙하게 사용 중인 IDE나 설치된 JDK가 있다면 그대로 사용해도 무방하다.  
다만 IDE나 JDK를 새로 설치해야 하거나 아직 특정 환경에 익숙하지 않은 경우에는
본 문서에서 안내하는 환경으로 설정하여 실습을 진행하는 것을 권장한다.

특정 IDE나 특정 JDK 배포판, 특정 Java 버전에 종속되는 내용은 포함하지 않으며
너무 오래된 버전이 아니라면 표준 Java 환경에서 동일하게 실습을 진행할 수 있다.


## Java 실습 목록

| 번호 | 제목                  | 경로                                                      |
| :--  | :-------------------- | :-------------------------------------------------------- |
|  01  | Big Number Calculator | [`01-big-number-calculator`](./01-big-number-calculator/) |
|  02  | Expression Evaluator  | [`02-expression-evaluator`](./02-expression-evaluator/)   |
|  03  | TODO Analyzer         | [`03-todo-analyzer`](./03-todo-analyzer/)                 |
|  04  | Monster Arena         | [`04-monster-arena`](./04-monster-arena/)                 |
|  05  | Data Structures       | [`05-data-structures`](./05-data-structures/)             |

각 실습 디렉터리에는 과제 설명(`README.md`), 기본 코드 구조, 실행 및 테스트 방법이 포함되어 있다.

## 실습 흐름과 학습 목표

실습은 절차적 구현 → 구조적 사고 → 파일 처리 → 클래스 설계 → 종합 구현의 흐름으로 진행된다.

초반에는 문자열, 배열, 조건문, 반복문을 직접 다루며 Java 문법과 절차적 구현 방식에 익숙해진다.  
이후 파일 입출력과 문자열 파싱을 통해 입력 데이터가 파일 시스템으로 확장될 때의 처리 흐름을 경험한다.

다음 단계에서는 문제를 작은 단위로 나누고 상태를 분리하는 구조적 사고를 훈련하며 Monster Arena 실습을 통해 여러 클래스가 협력하는 객체지향적 구조를 실제 코드로 구현한다.

마지막 실습에서는 지금까지 익힌 모든 요소를 종합하여 비교적 복잡한 구현을 완성한다.  
이 단계에는 자료구조 구현 요소가 포함되지만, 핵심은 클래스를 활용해 구조적으로 설계하고 구현하는 종합 훈련에 있다.

## 코딩 스탠다드와 가독성

모든 실습은 제공된 [Java 코딩 표준](../java-coding-standard.md)을 기준으로 작성한다.

이 실습들은 단순히 동작하는 코드를 작성하는 데서 그치지 않고 가독성과 일관성을 고려한 코드 작성 습관을 함께 훈련하는 것을 목표로 한다.  
변수명과 메서드 이름, 코드 구조와 흐름을 신경 쓰며 읽는 사람이 이해하기 쉬운 코드를 작성하는 경험을 쌓는 것이 중요하다.

이를 통해 실습 과정 전반에서 단순히 동작하는 코드를 넘어 의도와 구조가 드러나는 코드를 작성하는 감각을 익히기를 기대한다.

## 실습 프로젝트 초기 설정

### 1. IntelliJ에서 새 프로젝트 만들기

#### 1.1. 새 프로젝트 생성

1. IntelliJ 실행
2. New Project 선택
3. 왼쪽 목록에서 Java 선택
4. 아래 항목 설정

|       항목      |        설정 값       |
|       :---      |         :---         |
|       Name      |      `java-labs`     |
|     Location    | 원하는 작업 디렉터리 |
|   Build system  |       `Gradle`       |
|        JDK      |       `JDK 17`       |
|    Gradle DSL   |       `Groovy`       |
| Add sample code |       체크 해제      |

5. Advanced Settings 설정
    - Gradle distribution: `Wrapper`
    - GroupId: `com.example` 또는 개인 식별용 네임스페이스 (`com.yourname` 등)
        - 실습에서 안내되는 모든 문서는 `com.example`라고 가정
    - 나머지는 기본값 유지

6. Create 버튼을 클릭하여 프로젝트를 생성한다.

#### 1.2. 생성 후 디렉터리 정리

프로젝트 생성 직후 구조:

```
java-labs/
 ├─ .gradle/
 ├─ .idea/
 ├─ gradle/
 ├─ src/
 ├─ .gitignore
 ├─ build.gradle
 ├─ gradlew
 ├─ gradlew.bat
 ├─ settings.gradle
```

기본 생성된 src 디렉터리 삭제 후 구조:

```
java-labs/
 ├─ .gradle/
 ├─ .idea/
 ├─ gradle/
 ├─ .gitignore
 ├─ build.gradle
 ├─ gradlew
 ├─ gradlew.bat
 ├─ settings.gradle
```

### 2. 디렉터리 생성 및 Gradle 설정

#### 2.1. 디렉터리 생성

`java-labs` 루트 디렉터리에서 아래 디렉터리를 생성:
- `01-big-number-calculator`
- `02-expression-evaluator`
- `03-todo-analyzer`
- `04-monster-arena`
- `05-data-structures`

생성 후 구조:

```
java-labs/
 ├─ .gradle/
├─ .idea/
 ├─ 01-big-number-calculator/
 ├─ 02-expression-evaluator/
 ├─ 03-todo-analyzer/
 ├─ 04-monster-arena/
 ├─ 05-data-structures/
 ├─ gradle/
 ├─ .gitignore
 ├─ build.gradle
 ├─ gradlew
 ├─ gradlew.bat
 └─ settings.gradle
```

#### 2.2. settings.gradle 수정

`java-labs/settings.gradle`을 열고 아래처럼 작성:

```
rootProject.name = 'java-labs'

include '01-big-number-calculator'
include '02-expression-evaluator'
include '03-todo-analyzer'
include '04-monster-arena'
include '05-data-structures'
```

#### 2.3. 루트 build.gradle 공통 설정 작성

`java-labs/build.gradle`을 아래처럼 설정:

```
group = 'com.example'
version = '1.0-SNAPSHOT'

allprojects {
    repositories {
        mavenCentral()
    }
}

subprojects {
    apply plugin: 'java'

    java {
        toolchain {
            languageVersion = JavaLanguageVersion.of(17)
        }
    }
}
```

#### 2.4. 각 디렉터리에 build.gradle 생성
각 디렉터리에 `build.gradle` 파일을 하나씩 생성한다.

- `01-big-number-calculator/build.gradle`
- `02-expression-evaluator/build.gradle`
- `03-todo-analyzer/build.gradle`
- `04-monster-arena/build.gradle`
- `05-data-structures`

이 단계에서는 파일 내용은 비워 둔다.

#### 2.5. Gradle Reload

IntelliJ 오른쪽 Gradle 창에서 다음 버튼 클릭:

- `Sync All Gradle Projects` (새로고침 아이콘)

정상이라면 Gradle 창에 모든 실습 모듈이 표시된다.
