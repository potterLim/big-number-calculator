# BigNumberCalculator 실습 

`int`(32비트 정수형)와 `long`(64비트 정수형)은 매우 큰 정수 값을 표현하고 연산하기에 적합하지 않다. 덧셈이나 뺄셈과 같은 기본 연산에서도 오버플로 또는 언더플로가 발생할 수 있으며 특히 값의 범위가 `2^63 - 1`을 초과하는 경우에는 표준 정수형으로는 표현 자체가 불가능하다.  
`double`과 같은 부동소수점 자료형을 사용할 수도 있으나 이는 근본적으로 오차를 포함할 수 있으므로 정확한 정수 연산이 요구되는 상황에는 적합하지 않다.

이 실습에서는 이러한 한계를 극복하기 위해 매우 큰 정수를 문자열 기반으로 표현하고 처리하는 계산기 BigNumberCalculator를 구현한다.  
이 실습의 목적은 언어에서 제공하는 내장 큰 정수 자료형에 의존하지 않고 문자열 기반 표현을 통해 매우 큰 정수의 덧셈과 뺄셈을 정확하게 수행할 수 있는 연산 로직을 직접 구현하는 데 있다.

## 전반적인 규칙

- `java.math.BigInteger`를 포함한 모든 Big Integer 관련 라이브러리는 허용되지 않는다.

- `double`, `float`, `BigDecimal` 등 부동소수점 및 고정소수점 자료형은 허용되지 않는다.

- 정수 연산을 위해 입력 전체를 `int` 또는 `long`으로 변환하는 방식은 허용되지 않는다.

- 입력되는 정수 문자열은 다음과 같은 형태를 가질 수 있다.
    - 선행 0을 포함할 수 있다. (예: `"000123"`)
    - 음수를 나타내기 위해 `-` 기호를 포함할 수 있다. (예: `"-00000000007"`)

- 모든 연산의 최종 결과는 정규화된 10진수 정수 문자열로 반환해야 한다.
    - 선행 0이 없어야 한다.
    - `-0`은 허용되지 않으며, 이 경우 반드시 `"0"`으로 반환해야 한다.
    - 결과는 수학적으로 정확한 값이어야 한다.

- 실습 명세에서 제공된 함수 시그니처는 수정할 수 없으나 필요에 따라 추가적인 `private` 도움 함수를 작성하는 것은 허용된다.

- 필수 요구 사항은 아니지만 `String`을 이용한 반복적인 수정 작업은 불필요한 연산 비용을 유발할 수 있으므로 상황에 따라 보다 효율적인 구현 방법을 고려하는 것이 바람직하다.

## 1. 프로젝트를 준비한다

1. IntelliJ에서 `BigNumberCalculator` 프로젝트를 생성한다.
2. `bignumbercalculator` 패키지를 생성한다.
3. `bignumbercalculator` 패키지에 `BigNumberCalculator` 클래스를 정의한다.
3. `bignumbercalculator` 패키지에 `Main` 클래스를 정의한다.

```java
package bignumbercalculator;

public class BigNumberCalculator {
    public static String addOrNull(String num1, String num2) {
        return null;
    }

    public static String subtractOrNull(String num1, String num2) {
        return null;
    }

    private static boolean isValidSignedInteger(String num) {
        return false;
    }

    private static String normalizeSignedIntegerOrNull(String num) {
        return null;
    }
}
```

## 2. `BigNumberCalculator` 클래스를 구현한다

### 2.1. `isValidSignedInteger()` 정적 함수를 구현한다

- 이 함수는 필수 구현 함수가 아니지만 입력 문자열을 검증하고 전처리하기 위한 도움 함수(Helper Function)로 사용하면 구현을 훨씬 안전하고 단순하게 만들 수 있다.

- 이 함수는 다음의 인자를 받는다:
    - `String num`: 10진수 정수 문자열

- `num`이 올바른 10진수 정수 문자열 형식이면 `true`를 반환하고, 그렇지 않으면 `false`를 반환한다.

- 올바른 10진수 정수 문자열의 조건은 다음과 같다.
    - 문자열은 `null`이 아니어야 하며 길이가 1 이상이어야 한다.
    - 맨 앞에 `-` 기호가 올 수 있다. 단, `-`만 단독으로 존재하는 것은 허용하지 않는다.
    - `-`를 제외한 모든 문자는 반드시 `'0'`부터 `'9'` 사이의 숫자여야 한다.
    - 공백, `+`, 소수점, 지수 표기, 기타 문자가 포함되면 올바르지 않은 입력으로 간주한다.
    - 선행 0(예: `"000123"`, `"-0007"`)은 허용한다.

```java
BigNumberCalculator.isValidSignedInteger(null);            // false
BigNumberCalculator.isValidSignedInteger("");              // false
BigNumberCalculator.isValidSignedInteger("-");             // false
BigNumberCalculator.isValidSignedInteger("   ");           // false

BigNumberCalculator.isValidSignedInteger("159");           // true
BigNumberCalculator.isValidSignedInteger("-00000000007");  // true
BigNumberCalculator.isValidSignedInteger("000123");        // true
BigNumberCalculator.isValidSignedInteger("0");             // true
BigNumberCalculator.isValidSignedInteger("-0");            // true
BigNumberCalculator.isValidSignedInteger("00000");         // true

BigNumberCalculator.isValidSignedInteger("+12");           // false
BigNumberCalculator.isValidSignedInteger("12.3");          // false
BigNumberCalculator.isValidSignedInteger("1e5");           // false
BigNumberCalculator.isValidSignedInteger("12a3");          // false
BigNumberCalculator.isValidSignedInteger("0xFF");          // false
BigNumberCalculator.isValidSignedInteger("0b1010");        // false
```

### 2.2. `normalizeSignedIntegerOrNull()` 정적 함수를 구현한다

- 이 함수는 필수 구현 함수가 아니지만 입력 문자열을 검증하고 전처리하기 위한 도움 함수(Helper Function)로 사용하면 구현을 훨씬 안전하고 단순하게 만들 수 있다.


- 이 함수는 다음의 인자를 받는다:
    - `String num`: 10진수 정수 문자열

- 반환 규칙은 다음과 같다:
    - `num`이 올바른 10진수 정수 문자열이면 정규화된 문자열을 반환한다.
    - `num`이 올바르지 않은 포맷이면 `null`을 반환한다.

- 정규화는 다음 규칙을 따른다:
    - 선행 0을 모두 제거한다.  
      _예: `"000123" → "123"`, `"-0000007" → "-7"`_

    - 값이 0이 아닌 경우 음수 부호 `-`는 유지한다.  
      _예: `"-00120" → "-120"`_

    - 결과가 0인 경우(모든 숫자가 0인 경우)에는 반드시 `"0"`을 반환한다.  
      _예: `"0" → "0"`, `"0000" → "0"`, `"-0" → "0"`, `"-0000" → "0"`_

```java
BigNumberCalculator.normalizeSignedIntegerOrNull(null);            // null
BigNumberCalculator.normalizeSignedIntegerOrNull("");              // null
BigNumberCalculator.normalizeSignedIntegerOrNull("-");             // null
BigNumberCalculator.normalizeSignedIntegerOrNull("   ");           // null

BigNumberCalculator.normalizeSignedIntegerOrNull("0");             // "0"
BigNumberCalculator.normalizeSignedIntegerOrNull("0000");          // "0"
BigNumberCalculator.normalizeSignedIntegerOrNull("-0");            // "0"
BigNumberCalculator.normalizeSignedIntegerOrNull("-0000");         // "0"

BigNumberCalculator.normalizeSignedIntegerOrNull("000123");        // "123"
BigNumberCalculator.normalizeSignedIntegerOrNull("-00000000007");  // "-7"
BigNumberCalculator.normalizeSignedIntegerOrNull("159");           // "159"
BigNumberCalculator.normalizeSignedIntegerOrNull("-00120");        // "-120"

BigNumberCalculator.normalizeSignedIntegerOrNull("+12");           // null
BigNumberCalculator.normalizeSignedIntegerOrNull("12.3");          // null
BigNumberCalculator.normalizeSignedIntegerOrNull("1e5");           // null
BigNumberCalculator.normalizeSignedIntegerOrNull("12a3");          // null
BigNumberCalculator.normalizeSignedIntegerOrNull("0xFF");          // null
BigNumberCalculator.normalizeSignedIntegerOrNull("0b1010");        // null
```

### 2.3. `addOrNull()` 정적 함수를 구현한다

- 이 함수는 두 개의 10진수 정수 문자열 `num1`, `num2`를 입력으로 받아 덧셈 결과를 10진수 정수 문자열로 반환한다.

- 이 함수는 다음의 인자를 받는다:
    - `String num1`: 첫 번째 10진수 정수 문자열
    - `String num2`: 두 번째 10진수 정수 문자열

- 반환 규칙은 다음과 같다:
    - `num1` 또는 `num2`가 올바른 10진수 정수 문자열이 아니면 `null`을 반환한다.
    - 두 입력이 올바르다면 `num1 + num2`의 결과를 정규화된 10진수 정수 문자열로 반환한다.

```java
BigNumberCalculator.addOrNull("123", "456");            // "579"
BigNumberCalculator.addOrNull("-123", "456");           // "333"
BigNumberCalculator.addOrNull("123", "-456");           // "-333"
BigNumberCalculator.addOrNull("-123", "-456");          // "-579"

BigNumberCalculator.addOrNull("0000123", "000");        // "123"
BigNumberCalculator.addOrNull("-0000", "0");            // "0"
BigNumberCalculator.addOrNull("-0007", "0007");         // "0"

BigNumberCalculator.addOrNull(null, "1");               // null
BigNumberCalculator.addOrNull("", "1");                 // null
BigNumberCalculator.addOrNull("-", "1");                // null
BigNumberCalculator.addOrNull("12.3", "1");             // null
BigNumberCalculator.addOrNull("0xFF", "1");             // null
```

### 2.4. `subtractOrNull()` 정적 함수를 구현한다

- 이 함수는 두 개의 10진수 정수 문자열 `num1`, `num2`를 입력으로 받아 뺄셈 결과(`num1 - num2`)를 10진수 정수 문자열로 반환한다.

- 이 함수는 다음의 인자를 받는다:
    - `String num1`: 첫 번째 10진수 정수 문자열
    - `String num2`: 두 번째 10진수 정수 문자열

- 반환 규칙은 다음과 같다:
    - `num1` 또는 `num2`가 올바른 10진수 정수 문자열이 아니면 `null`을 반환한다.
    - 두 입력이 올바르다면 `num1 - num2`의 결과를 정규화된 10진수 정수 문자열로 반환한다.

```java
BigNumberCalculator.subtractOrNull("456", "123");       // "333"
BigNumberCalculator.subtractOrNull("123", "456");       // "-333"
BigNumberCalculator.subtractOrNull("-456", "-123");     // "-333"
BigNumberCalculator.subtractOrNull("-123", "-456");     // "333"

BigNumberCalculator.subtractOrNull("0000123", "000");   // "123"
BigNumberCalculator.subtractOrNull("0", "-0");          // "0"
BigNumberCalculator.subtractOrNull("-0007", "-0007");   // "0"

BigNumberCalculator.subtractOrNull(null, "1");          // null
BigNumberCalculator.subtractOrNull("", "1");            // null
BigNumberCalculator.subtractOrNull("-", "1");           // null
BigNumberCalculator.subtractOrNull("12.3", "1");        // null
BigNumberCalculator.subtractOrNull("0b1010", "1");      // null
```


## 3. 본인 컴퓨터에서 테스트
- 프로젝트의 `Main` 클래스를 아래의 예처럼 수정한다.
```java
package bignumbercalculator;

public class Main {
    public static void main(String[] args) {
        boolean bPassed;

        // =========================================================
        // 2.3 addOrNull() — 입력 형식 오류(Invalid Input) 검증
        // =========================================================

        bPassed = true;

        // null / empty / "-" / whitespace
        bPassed &= (BigNumberCalculator.addOrNull(null, "1") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", null) == null);
        bPassed &= (BigNumberCalculator.addOrNull("", "1") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", "") == null);
        bPassed &= (BigNumberCalculator.addOrNull("-", "1") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", "-") == null);
        bPassed &= (BigNumberCalculator.addOrNull("   ", "1") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", "   ") == null);
        bPassed &= (BigNumberCalculator.addOrNull("\t", "1") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", "\n") == null);

        // leading/trailing spaces
        bPassed &= (BigNumberCalculator.addOrNull("1 ", "2") == null);
        bPassed &= (BigNumberCalculator.addOrNull(" 1", "2") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", "2 ") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", " 2") == null);

        // plus sign / decimal point / exponent / commas
        bPassed &= (BigNumberCalculator.addOrNull("+1", "2") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", "+2") == null);
        bPassed &= (BigNumberCalculator.addOrNull("+", "2") == null);
        bPassed &= (BigNumberCalculator.addOrNull("12.3", "1") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", "12.3") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1e5", "1") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", "1e5") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1,000", "1") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", "1,000") == null);

        // non-decimal prefixes / letters / mixed / misplaced '-'
        bPassed &= (BigNumberCalculator.addOrNull("0xFF", "1") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", "0xFF") == null);
        bPassed &= (BigNumberCalculator.addOrNull("0b1010", "1") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", "0b1010") == null);
        bPassed &= (BigNumberCalculator.addOrNull("12a3", "1") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", "12a3") == null);
        bPassed &= (BigNumberCalculator.addOrNull("--1", "1") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1-2", "3") == null);
        bPassed &= (BigNumberCalculator.addOrNull("-01-2", "3") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", "--2") == null);

        // unicode digits / full-width digits (must be invalid)
        bPassed &= (BigNumberCalculator.addOrNull("１２３", "1") == null);
        bPassed &= (BigNumberCalculator.addOrNull("١٢٣", "1") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", "１２３") == null);
        bPassed &= (BigNumberCalculator.addOrNull("1", "١٢٣") == null);

        if (bPassed) {
            System.out.println("2.3 addOrNull() — 입력 형식 오류 검증 PASS");
        } else {
            System.out.println("2.3 addOrNull() — 입력 형식 오류 검증 FAIL");
        }


        // =========================================================
        // 2.4 subtractOrNull() — 입력 형식 오류(Invalid Input) 검증
        // =========================================================

        bPassed = true;

        bPassed &= (BigNumberCalculator.subtractOrNull(null, "1") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", null) == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("", "1") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", "") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("-", "1") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", "-") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("   ", "1") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", "   ") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("\t", "1") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", "\n") == null);

        bPassed &= (BigNumberCalculator.subtractOrNull("1 ", "2") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull(" 1", "2") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", "2 ") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", " 2") == null);

        bPassed &= (BigNumberCalculator.subtractOrNull("+1", "2") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", "+2") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("+", "2") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("12.3", "1") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", "12.3") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1e5", "1") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", "1e5") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1,000", "1") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", "1,000") == null);

        bPassed &= (BigNumberCalculator.subtractOrNull("0xFF", "1") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", "0xFF") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("0b1010", "1") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", "0b1010") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("12a3", "1") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", "12a3") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("--1", "1") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1-2", "3") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("-01-2", "3") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", "--2") == null);

        // unicode digits / full-width digits (must be invalid)
        bPassed &= (BigNumberCalculator.subtractOrNull("１２３", "1") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("١٢٣", "1") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", "１２３") == null);
        bPassed &= (BigNumberCalculator.subtractOrNull("1", "١٢٣") == null);

        if (bPassed) {
            System.out.println("2.4 subtractOrNull() — 입력 형식 오류 검증 PASS");
        } else {
            System.out.println("2.4 subtractOrNull() — 입력 형식 오류 검증 FAIL");
        }


        // =========================================================
        // 2.3 addOrNull() — 기본 동작 검증
        // =========================================================

        bPassed = true;

        bPassed &= "0".equals(BigNumberCalculator.addOrNull("0", "0"));
        bPassed &= "2".equals(BigNumberCalculator.addOrNull("1", "1"));
        bPassed &= "579".equals(BigNumberCalculator.addOrNull("123", "456"));
        bPassed &= "100".equals(BigNumberCalculator.addOrNull("99", "1"));
        bPassed &= "1000".equals(BigNumberCalculator.addOrNull("999", "1"));
        bPassed &= "10000".equals(BigNumberCalculator.addOrNull("9999", "1"));
        bPassed &= "1000000000000000001".equals(BigNumberCalculator.addOrNull("1", "1000000000000000000"));
        bPassed &= "1000000000000000001".equals(BigNumberCalculator.addOrNull("1000000000000000000", "1"));

        // very different lengths
        bPassed &= "1000000000000000000000001".equals(BigNumberCalculator.addOrNull("1", "1000000000000000000000000"));
        bPassed &= "1000000000000000000000001".equals(BigNumberCalculator.addOrNull("1000000000000000000000000", "1"));

        if (bPassed) {
            System.out.println("2.3 addOrNull() — 기본 동작 검증 PASS");
        } else {
            System.out.println("2.3 addOrNull() — 기본 동작 검증 FAIL");
        }


        // =========================================================
        // 2.4 subtractOrNull() — 기본 동작 검증
        // =========================================================

        bPassed = true;

        bPassed &= "0".equals(BigNumberCalculator.subtractOrNull("0", "0"));
        bPassed &= "0".equals(BigNumberCalculator.subtractOrNull("1", "1"));
        bPassed &= "333".equals(BigNumberCalculator.subtractOrNull("456", "123"));
        bPassed &= "-333".equals(BigNumberCalculator.subtractOrNull("123", "456"));
        bPassed &= "99".equals(BigNumberCalculator.subtractOrNull("100", "1"));
        bPassed &= "999".equals(BigNumberCalculator.subtractOrNull("1000", "1"));
        bPassed &= "9999".equals(BigNumberCalculator.subtractOrNull("10000", "1"));
        bPassed &= "999999999999999999".equals(BigNumberCalculator.subtractOrNull("1000000000000000000", "1"));
        bPassed &= "-999999999999999999".equals(BigNumberCalculator.subtractOrNull("1", "1000000000000000000"));

        // smallest sign-flip
        bPassed &= "-1".equals(BigNumberCalculator.subtractOrNull("1", "2"));
        bPassed &= "1".equals(BigNumberCalculator.subtractOrNull("-1", "-2"));

        // very different lengths
        bPassed &= "999999999999999999999999".equals(BigNumberCalculator.subtractOrNull("1000000000000000000000000", "1"));
        bPassed &= "-999999999999999999999999".equals(BigNumberCalculator.subtractOrNull("1", "1000000000000000000000000"));

        if (bPassed) {
            System.out.println("2.4 subtractOrNull() — 기본 동작 검증 PASS");
        } else {
            System.out.println("2.4 subtractOrNull() — 기본 동작 검증 FAIL");
        }


        // =========================================================
        // 2.3 addOrNull() — 부호 조합 검증
        // =========================================================

        bPassed = true;

        bPassed &= "333".equals(BigNumberCalculator.addOrNull("-123", "456"));
        bPassed &= "-333".equals(BigNumberCalculator.addOrNull("123", "-456"));
        bPassed &= "-579".equals(BigNumberCalculator.addOrNull("-123", "-456"));
        bPassed &= "0".equals(BigNumberCalculator.addOrNull("1", "-1"));
        bPassed &= "0".equals(BigNumberCalculator.addOrNull("-1", "1"));
        bPassed &= "-2".equals(BigNumberCalculator.addOrNull("-1", "-1"));
        bPassed &= "2".equals(BigNumberCalculator.addOrNull("1", "1"));
        bPassed &= "-999999999999999999998".equals(BigNumberCalculator.addOrNull("-999999999999999999999", "1"));
        bPassed &= "999999999999999999998".equals(BigNumberCalculator.addOrNull("999999999999999999999", "-1"));

        // zero interactions
        bPassed &= "-1".equals(BigNumberCalculator.addOrNull("0", "-1"));
        bPassed &= "1".equals(BigNumberCalculator.addOrNull("0", "1"));
        bPassed &= "-1".equals(BigNumberCalculator.addOrNull("-1", "0"));
        bPassed &= "1".equals(BigNumberCalculator.addOrNull("1", "0"));

        if (bPassed) {
            System.out.println("2.3 addOrNull() — 부호 조합 검증 PASS");
        } else {
            System.out.println("2.3 addOrNull() — 부호 조합 검증 FAIL");
        }


        // =========================================================
        // 2.4 subtractOrNull() — 부호 조합 검증
        // =========================================================

        bPassed = true;

        bPassed &= "2".equals(BigNumberCalculator.subtractOrNull("1", "-1"));
        bPassed &= "-2".equals(BigNumberCalculator.subtractOrNull("-1", "1"));
        bPassed &= "0".equals(BigNumberCalculator.subtractOrNull("-1", "-1"));
        bPassed &= "-333".equals(BigNumberCalculator.subtractOrNull("-456", "-123"));
        bPassed &= "333".equals(BigNumberCalculator.subtractOrNull("-123", "-456"));
        bPassed &= "-579".equals(BigNumberCalculator.subtractOrNull("-123", "456"));
        bPassed &= "579".equals(BigNumberCalculator.subtractOrNull("123", "-456"));
        bPassed &= "1000000000000000000000".equals(BigNumberCalculator.subtractOrNull("999999999999999999999", "-1"));
        bPassed &= "-1000000000000000000000".equals(BigNumberCalculator.subtractOrNull("-999999999999999999999", "1"));

        // zero interactions
        bPassed &= "1".equals(BigNumberCalculator.subtractOrNull("0", "-1"));
        bPassed &= "-1".equals(BigNumberCalculator.subtractOrNull("0", "1"));
        bPassed &= "-1".equals(BigNumberCalculator.subtractOrNull("-1", "0"));
        bPassed &= "1".equals(BigNumberCalculator.subtractOrNull("1", "0"));

        if (bPassed) {
            System.out.println("2.4 subtractOrNull() — 부호 조합 검증 PASS");
        } else {
            System.out.println("2.4 subtractOrNull() — 부호 조합 검증 FAIL");
        }


        // =========================================================
        // 2.3 addOrNull() — 정규화(선행 0, -0) 포함 검증
        // =========================================================

        bPassed = true;

        bPassed &= "123".equals(BigNumberCalculator.addOrNull("0000123", "000"));
        bPassed &= "0".equals(BigNumberCalculator.addOrNull("-0000", "0"));
        bPassed &= "0".equals(BigNumberCalculator.addOrNull("0000", "0000"));
        bPassed &= "0".equals(BigNumberCalculator.addOrNull("-0", "0"));
        bPassed &= "0".equals(BigNumberCalculator.addOrNull("0", "-0"));
        bPassed &= "0".equals(BigNumberCalculator.addOrNull("-0007", "0007"));
        bPassed &= "-7".equals(BigNumberCalculator.addOrNull("-0007", "0000"));
        bPassed &= "7".equals(BigNumberCalculator.addOrNull("0000", "0007"));
        bPassed &= "0".equals(BigNumberCalculator.addOrNull("-0", "-0"));
        bPassed &= "-1".equals(BigNumberCalculator.addOrNull("0", "-0001"));
        bPassed &= "1".equals(BigNumberCalculator.addOrNull("-0000", "0001"));

        // zero by cancellation with leading zeros
        bPassed &= "0".equals(BigNumberCalculator.addOrNull("0000500", "-0000500"));
        bPassed &= "0".equals(BigNumberCalculator.addOrNull("-0000001", "0000001"));

        if (bPassed) {
            System.out.println("2.3 addOrNull() — 정규화 포함 검증 PASS");
        } else {
            System.out.println("2.3 addOrNull() — 정규화 포함 검증 FAIL");
        }


        // =========================================================
        // 2.4 subtractOrNull() — 정규화(선행 0, -0) 포함 검증
        // =========================================================

        bPassed = true;

        bPassed &= "123".equals(BigNumberCalculator.subtractOrNull("0000123", "000"));
        bPassed &= "0".equals(BigNumberCalculator.subtractOrNull("0", "-0"));
        bPassed &= "0".equals(BigNumberCalculator.subtractOrNull("-0", "0"));
        bPassed &= "0".equals(BigNumberCalculator.subtractOrNull("-0007", "-0007"));
        bPassed &= "-7".equals(BigNumberCalculator.subtractOrNull("-0007", "0000"));
        bPassed &= "7".equals(BigNumberCalculator.subtractOrNull("0007", "0000"));
        bPassed &= "-7".equals(BigNumberCalculator.subtractOrNull("0000", "0007"));
        bPassed &= "7".equals(BigNumberCalculator.subtractOrNull("0000", "-0007"));
        bPassed &= "0".equals(BigNumberCalculator.subtractOrNull("-0", "-0"));
        bPassed &= "-1".equals(BigNumberCalculator.subtractOrNull("0", "0001"));
        bPassed &= "1".equals(BigNumberCalculator.subtractOrNull("0001", "-0"));

        // zero by self-subtraction with leading zeros
        bPassed &= "0".equals(BigNumberCalculator.subtractOrNull("0000500", "0000500"));
        bPassed &= "0".equals(BigNumberCalculator.subtractOrNull("-0000500", "-0000500"));
        bPassed &= "0".equals(BigNumberCalculator.subtractOrNull("-0000001", "-0000001"));

        if (bPassed) {
            System.out.println("2.4 subtractOrNull() — 정규화 포함 검증 PASS");
        } else {
            System.out.println("2.4 subtractOrNull() — 정규화 포함 검증 FAIL");
        }


        // =========================================================
        // 2.3 addOrNull() — 상위 자릿수까지 영향을 미치는 덧셈 검증
        // =========================================================

        bPassed = true;

        bPassed &= "1000000".equals(BigNumberCalculator.addOrNull("999999", "1"));
        bPassed &= "1000000000".equals(BigNumberCalculator.addOrNull("999999999", "1"));
        bPassed &= "100000000000000000000".equals(BigNumberCalculator.addOrNull("99999999999999999999", "1"));
        bPassed &= "1000000000000000000000000000000".equals(BigNumberCalculator.addOrNull("999999999999999999999999999999", "1"));
        bPassed &= "100000000000000000000000000000000000000000000000000".equals(BigNumberCalculator.addOrNull("99999999999999999999999999999999999999999999999999", "1"));
        bPassed &= "100000000000000000000000000000000000000000000000001".equals(BigNumberCalculator.addOrNull("99999999999999999999999999999999999999999999999999", "2"));

        // carry chain with negative boundary
        bPassed &= "-1000000".equals(BigNumberCalculator.addOrNull("-999999", "-1"));
        bPassed &= "-1000000000".equals(BigNumberCalculator.addOrNull("-999999999", "-1"));

        if (bPassed) {
            System.out.println("2.3 addOrNull() — 상위 자릿수 영향 검증 PASS");
        } else {
            System.out.println("2.3 addOrNull() — 상위 자릿수 영향 검증 FAIL");
        }


        // =========================================================
        // 2.4 subtractOrNull() — 상위 자릿수까지 영향을 미치는 뺄셈 검증
        // =========================================================

        bPassed = true;

        bPassed &= "999999".equals(BigNumberCalculator.subtractOrNull("1000000", "1"));
        bPassed &= "999999999".equals(BigNumberCalculator.subtractOrNull("1000000000", "1"));
        bPassed &= "99999999999999999999".equals(BigNumberCalculator.subtractOrNull("100000000000000000000", "1"));
        bPassed &= "999999999999999999999999999999".equals(BigNumberCalculator.subtractOrNull("1000000000000000000000000000000", "1"));
        bPassed &= "99999999999999999999999999999999999999999999999999".equals(BigNumberCalculator.subtractOrNull("100000000000000000000000000000000000000000000000000", "1"));
        bPassed &= "99999999999999999999999999999999999999999999999998".equals(BigNumberCalculator.subtractOrNull("100000000000000000000000000000000000000000000000000", "2"));

        // borrow chain producing negative result
        bPassed &= "-999999".equals(BigNumberCalculator.subtractOrNull("0", "999999"));
        bPassed &= "-999999999".equals(BigNumberCalculator.subtractOrNull("0", "999999999"));

        if (bPassed) {
            System.out.println("2.4 subtractOrNull() — 상위 자릿수 영향 검증 PASS");
        } else {
            System.out.println("2.4 subtractOrNull() — 상위 자릿수 영향 검증 FAIL");
        }


        // =========================================================
        // 2.3 addOrNull() — 큰 수(Big) 검증 (자리수 매우 큼)
        // =========================================================

        bPassed = true;

        bPassed &= "126590769985351980636868573108".equals(BigNumberCalculator.addOrNull("126585123123216548452353151521", "5646862135432184515421587"));
        bPassed &= "0".equals(BigNumberCalculator.addOrNull("999999999999999999999999999999999999999", "-999999999999999999999999999999999999999"));
        bPassed &= "1000000000000000000000000000000000000000".equals(BigNumberCalculator.addOrNull("999999999999999999999999999999999999999", "1"));
        bPassed &= "999999999999999999999999999999999999998".equals(BigNumberCalculator.addOrNull("999999999999999999999999999999999999999", "-1"));
        bPassed &= "-999999999999999999999999999999999999998".equals(BigNumberCalculator.addOrNull("-999999999999999999999999999999999999999", "1"));
        bPassed &= "-1000000000000000000000000000000000000000".equals(BigNumberCalculator.addOrNull("-999999999999999999999999999999999999999", "-1"));

        // large cancellation to zero
        bPassed &= "0".equals(BigNumberCalculator.addOrNull("500000000000000000000000000000000000000", "-500000000000000000000000000000000000000"));

        if (bPassed) {
            System.out.println("2.3 addOrNull() — 큰 수 검증 PASS");
        } else {
            System.out.println("2.3 addOrNull() — 큰 수 검증 FAIL");
        }


        // =========================================================
        // 2.4 subtractOrNull() — 큰 수(Big) 검증 (자리수 매우 큼)
        // =========================================================

        bPassed = true;

        bPassed &= "-1467132473826363976665053196".equals(BigNumberCalculator.subtractOrNull("-889874837998729348827376462", "577257635827634627837676734"));
        bPassed &= "0".equals(BigNumberCalculator.subtractOrNull("999999999999999999999999999999999999999", "999999999999999999999999999999999999999"));
        bPassed &= "-1000000000000000000000000000000000000000".equals(BigNumberCalculator.subtractOrNull("0", "1000000000000000000000000000000000000000"));
        bPassed &= "1000000000000000000000000000000000000000".equals(BigNumberCalculator.subtractOrNull("1000000000000000000000000000000000000000", "0"));
        bPassed &= "1000000000000000000000000000000000000000".equals(BigNumberCalculator.subtractOrNull("999999999999999999999999999999999999999", "-1"));
        bPassed &= "-1000000000000000000000000000000000000000".equals(BigNumberCalculator.subtractOrNull("-999999999999999999999999999999999999999", "1"));

        // large self-subtraction to zero
        bPassed &= "0".equals(BigNumberCalculator.subtractOrNull("500000000000000000000000000000000000000", "500000000000000000000000000000000000000"));
        bPassed &= "0".equals(BigNumberCalculator.subtractOrNull("-500000000000000000000000000000000000000", "-500000000000000000000000000000000000000"));

        if (bPassed) {
            System.out.println("2.4 subtractOrNull() — 큰 수 검증 PASS");
        } else {
            System.out.println("2.4 subtractOrNull() — 큰 수 검증 FAIL");
        }


        // =========================================================
        // 2.3/2.4 — 교차 검증(덧셈/뺄셈 결과 일관성) 검증
        // =========================================================

        bPassed = true;

        bPassed &= "-333".equals(BigNumberCalculator.addOrNull("123", "-456"));
        bPassed &= "-333".equals(BigNumberCalculator.subtractOrNull("123", "456"));
        bPassed &= "579".equals(BigNumberCalculator.addOrNull("123", "456"));
        bPassed &= "579".equals(BigNumberCalculator.subtractOrNull("123", "-456"));
        bPassed &= "0".equals(BigNumberCalculator.addOrNull("999999999999999999999", "-999999999999999999999"));
        bPassed &= "0".equals(BigNumberCalculator.subtractOrNull("999999999999999999999", "999999999999999999999"));

        // identity checks: a + b - b == a, a - b + b == a
        bPassed &= "123456789".equals(BigNumberCalculator.subtractOrNull(BigNumberCalculator.addOrNull("123456789", "987654321"), "987654321"));
        bPassed &= "-123456789".equals(BigNumberCalculator.addOrNull(BigNumberCalculator.subtractOrNull("-123456789", "987654321"), "987654321"));

        if (bPassed) {
            System.out.println("2.3/2.4 — 교차 검증 PASS");
        } else {
            System.out.println("2.3/2.4 — 교차 검증 FAIL");
        }
    }
}
```
- `FAIL`이 하나라도 있으면 실습 명세를 다시 꼼꼼히 검토한 후 수정한다.
