# ExpressionEvaluator 실습

산술식은 단순해 보이지만 실제로는 사람이 여러 판단과 처리를 거쳐 계산을 수행한다. 연산자 우선순위를 판단하고 중간 결과를 기억하며 계산 순서를 조정하는 일련의 과정은 대부분 무의식적으로 이루어진다.

이 실습에서는 문자열로 주어진 산술식을 분석하여 정수 결과를 계산하는 계산기 `ExpressionEvaluator`를 구현한다.

실제 산술식 계산은 훨씬 복잡한 규칙과 다양한 경우를 포함하지만 이 실습에서는 `+`, `-`, `*`, `/`만을 사용하는 단순한 산술식으로 범위를 한정한다. 이를 통해 사람이 무의식적으로 수행하는 계산 과정을 하나하나 절차적인 단계로 분해하여 구현하는 데 집중한다.  
문자열로 주어진 산술식을 단계적으로 해석하고 연산자 우선순위를 고려하여 정확한 결과를 계산하며 잘못된 입력을 안정적으로 처리하는 로직을 작성한다.

이 실습의 목적은 복잡한 사고 과정을 작은 판단과 처리 단위로 나누어 절차적으로 구현하는 데 있다.

## 전반적인 규칙

- 숫자는 10진수 정수 형태로만 표현되며 선행하는 0을 포함할 수 없다.

- 지원하는 연산자는 다음과 같다:
    - 이항 연산자:`+`, `-`, `*`, `/`
    - 단항 부호: 음수를 나타내는 부호(`-`)

- 연산자 우선순위는 일반적인 산술 규칙을 따른다.

- 실습 명세에서 제공된 함수 시그니처는 수정할 수 없으나 필요에 따라 추가적인 `private` 도움 함수를 작성하는 것은 허용된다.

## 1. 프로젝트를 준비한다

1. IntelliJ에서 `ExpressionEvaluator` 프로젝트를 생성한다.
2. `expressionevaluator` 패키지를 생성한다.
3. `expressionevaluator` 패키지에 `ExpressionEvaluator` 클래스를 정의한다.
4. `expressionevaluator` 패키지에 `Main` 클래스를 정의한다.

## 2. `ExpressionEvaluator` 클래스를 구현한다

###  `evaluateOrNull()` 정적 메서드를 구현한다

- 이 메서드는 문자열로 주어진 산술식을 입력으로 받아 계산 결과를 반환한다.

- 이 메서드는 유일한 인자로 `String expression`을 받는다.
    - `expression`에는 공백이 포함될 수 있으며 공백 문자의 종류와 개수는 일정하지 않을 수 있다.
    - 공백은 계산 과정에서 의미 없는 문자로 취급하며 산술식의 올바름을 판단할 때도 무시한다.

- 산술식은 이 실습에서 정의한 규칙에 부합하며 수학적으로 올바른 형태만을 올바른 산술식으로 간주한다.

- 이 실습에서 `-` 기호는 다음과 같이 해석한다.
    - 이항 연산자 `-`
    - 음수를 나타내는 부호(`-`)
        - 음수를 나타내는 부호(`-`)는 반드시 숫자와 붙어 있다고 간주한다.

- `/` 연산은 Java의 `int` 정수 나눗셈 규칙을 따른다. (소수점 이하 버림)

- 산술식을 연산하는 과정에서 수행되는 모든 이항 연산의 결과는 `int` 범위 내에 있다고 간주한다.

- 반환 규칙은 다음과 같다:
    - 산술식이 올바르지 않은 경우 `null`을 반환한다.
    - 올바른 산술식인 경우 계산 결과를 공백이 없는 10진수 정수 문자열로 반환한다.
        - Java의 표준 변환을 이용해 `int` 값을 문자열로 변환해 반환한다.

```java
ExpressionEvaluator.evaluateOrNull("");                   // null
ExpressionEvaluator.evaluateOrNull("   ");                // null

ExpressionEvaluator.evaluateOrNull("7");                  // "7"
ExpressionEvaluator.evaluateOrNull("-7");                 // "-7"
ExpressionEvaluator.evaluateOrNull("- 7");                // null

ExpressionEvaluator.evaluateOrNull("1 + 2");              // "3"
ExpressionEvaluator.evaluateOrNull("  10  +   20 ");      // "30"

ExpressionEvaluator.evaluateOrNull("2 + 3 * 4");          // "14"
ExpressionEvaluator.evaluateOrNull("20 / 3");             // "6"

ExpressionEvaluator.evaluateOrNull("1 +");                // null
ExpressionEvaluator.evaluateOrNull("2 * * 3");            // null
```

## 3. 본인 컴퓨터에서 테스트

- 프로젝트의 `Main` 클래스를 아래의 예처럼 수정한다.

```java
package expressionevaluator;

public class Main {
    private static int totalChecks = 0;
    private static int passedChecks = 0;

    private static int caseIndex = 1;

    private static final char[] OPERATORS = new char[]{ '+', '-', '*', '/' };

    public static void main(String[] args) {
        boolean bAllPassed = true;

        System.out.println("ExpressionEvaluator Test Runner");
        System.out.println("========================================");

        bAllPassed &= testEmptyAndWhitespaceOnly();
        bAllPassed &= testSingleNumberForms_NoLeadingZeros();
        bAllPassed &= testWhitespaceIsIgnoredInGeneral();
        bAllPassed &= testBasicBinaryOperators();
        bAllPassed &= testUnaryMinusAttachmentRules();
        bAllPassed &= testOperatorPrecedenceAndAssociativity();
        bAllPassed &= testDivisionRules();
        bAllPassed &= testDivisionByZero();
        bAllPassed &= testConcatenationByWhitespaceBetweenDigits();
        bAllPassed &= testInvalidCharacters();

        bAllPassed &= testAdjacentOperatorPairsExhaustive_NoSpaces();
        bAllPassed &= testAdjacentOperatorPairsExhaustive_SpacedUnaryAllowed();
        bAllPassed &= testAdjacentOperatorPairsExhaustive_SpacedUnaryBroken();

        bAllPassed &= testInvalidOperatorPlacement();
        bAllPassed &= testInvalidLeadingZerosEverywhere();

        bAllPassed &= testStressLongValidExpressions();
        bAllPassed &= testStressLongInvalidExpressions();

        System.out.println();
        System.out.println("========================================");
        System.out.printf("CHECKS: %d passed / %d total%n", passedChecks, totalChecks);
        if (bAllPassed) {
            System.out.println("ALL TESTS PASSED");
        } else {
            System.out.println("SOME TESTS FAILED");
        }
        System.out.println("========================================");
    }

    private static boolean testEmptyAndWhitespaceOnly() {
        boolean bPassed = true;

        printCase("empty / whitespace-only");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull(""), "empty string => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("   "), "spaces only => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("\t\t"), "tabs only => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("\n\r"), "newlines only => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull(" \t \n \r "), "mixed whitespace only => null");

        return bPassed;
    }

    private static boolean testSingleNumberForms_NoLeadingZeros() {
        boolean bPassed = true;

        printCase("single numbers (no leading zeros)");

        bPassed &= checkEquals("0", ExpressionEvaluator.evaluateOrNull("0"), "0 => \"0\"");
        bPassed &= checkEquals("7", ExpressionEvaluator.evaluateOrNull("7"), "7 => \"7\"");
        bPassed &= checkEquals("123", ExpressionEvaluator.evaluateOrNull("123"), "123 => \"123\"");

        bPassed &= checkEquals("-7", ExpressionEvaluator.evaluateOrNull("-7"), "-7 => \"-7\"");
        bPassed &= checkEquals("-123", ExpressionEvaluator.evaluateOrNull("-123"), "-123 => \"-123\"");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("-0"), "-0 => null");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("007"), "007 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("008"), "008 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("-007"), "-007 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("-008"), "-008 => null");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("0000"), "0000 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("-0000"), "-0000 => null");

        return bPassed;
    }

    private static boolean testWhitespaceIsIgnoredInGeneral() {
        boolean bPassed = true;

        printCase("whitespace ignored (general)");

        bPassed &= checkEquals("3", ExpressionEvaluator.evaluateOrNull("1 + 2"), "simple spaced addition");
        bPassed &= checkEquals("3", ExpressionEvaluator.evaluateOrNull(" \n 1 \t + \r 2 \n "), "mixed whitespace around tokens");
        bPassed &= checkEquals("30", ExpressionEvaluator.evaluateOrNull("  10  +   20 "), "arbitrary spaces around tokens");

        bPassed &= checkEquals("14", ExpressionEvaluator.evaluateOrNull("2 + 3 * 4"), "precedence with spaces");
        bPassed &= checkEquals("14", ExpressionEvaluator.evaluateOrNull(" 2   +   3\t*\n4 "), "precedence with mixed whitespace");

        return bPassed;
    }

    private static boolean testBasicBinaryOperators() {
        boolean bPassed = true;

        printCase("basic binary operators");

        bPassed &= checkEquals("3", ExpressionEvaluator.evaluateOrNull("1+2"), "1+2 => 3");
        bPassed &= checkEquals("-1", ExpressionEvaluator.evaluateOrNull("1-2"), "1-2 => -1");
        bPassed &= checkEquals("6", ExpressionEvaluator.evaluateOrNull("2*3"), "2*3 => 6");
        bPassed &= checkEquals("6", ExpressionEvaluator.evaluateOrNull("18/3"), "18/3 => 6");

        bPassed &= checkEquals("3", ExpressionEvaluator.evaluateOrNull("1 + 2"), "1 + 2 => 3");
        bPassed &= checkEquals("-1", ExpressionEvaluator.evaluateOrNull("1 - 2"), "1 - 2 => -1");
        bPassed &= checkEquals("6", ExpressionEvaluator.evaluateOrNull("2 * 3"), "2 * 3 => 6");
        bPassed &= checkEquals("6", ExpressionEvaluator.evaluateOrNull("18 / 3"), "18 / 3 => 6");

        bPassed &= checkEquals("0", ExpressionEvaluator.evaluateOrNull("0+0"), "0+0 => 0");
        bPassed &= checkEquals("5", ExpressionEvaluator.evaluateOrNull("0+5"), "0+5 => 5");
        bPassed &= checkEquals("5", ExpressionEvaluator.evaluateOrNull("0 + 5"), "0 + 5 => 5");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("-0+5"), "-0+5 => null (-0 invalid)");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("-0 + 5"), "-0 + 5 => null (-0 invalid)");

        return bPassed;
    }

    private static boolean testUnaryMinusAttachmentRules() {
        boolean bPassed = true;

        printCase("unary minus attachment rules");

        bPassed &= checkEquals("-7", ExpressionEvaluator.evaluateOrNull("-7"), "-7 => ok");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("- 7"), "- 7 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull(" - 7 "), " - 7  => null");

        bPassed &= checkEquals("-6", ExpressionEvaluator.evaluateOrNull("2*-3"), "2*-3 => -6");
        bPassed &= checkEquals("-6", ExpressionEvaluator.evaluateOrNull("2 * -3"), "2 * -3 => -6");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("2 * - 3"), "2 * - 3 => null");

        bPassed &= checkEquals("5", ExpressionEvaluator.evaluateOrNull("2--3"), "2--3 => 5");
        bPassed &= checkEquals("5", ExpressionEvaluator.evaluateOrNull("2 - -3"), "2 - -3 => 5");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("2 - - 3"), "2 - - 3 => null");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("--7"), "--7 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("---7"), "---7 => null");

        bPassed &= checkEquals("-1", ExpressionEvaluator.evaluateOrNull("1+-2"), "1+-2 => -1");
        bPassed &= checkEquals("-1", ExpressionEvaluator.evaluateOrNull("1 + -2"), "1 + -2 => -1");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1 + - 2"), "1 + - 2 => null");

        return bPassed;
    }

    private static boolean testOperatorPrecedenceAndAssociativity() {
        boolean bPassed = true;

        printCase("precedence and left-to-right");

        bPassed &= checkEquals("14", ExpressionEvaluator.evaluateOrNull("2+3*4"), "2+3*4 => 14");
        bPassed &= checkEquals("10", ExpressionEvaluator.evaluateOrNull("2*3+4"), "2*3+4 => 10");
        bPassed &= checkEquals("18", ExpressionEvaluator.evaluateOrNull("30-6*2"), "30-6*2 => 18");

        bPassed &= checkEquals("10", ExpressionEvaluator.evaluateOrNull("20/2/1*1"), "((20/2)/1)*1 => 10");
        bPassed &= checkEquals("5", ExpressionEvaluator.evaluateOrNull("20/2/2"), "(20/2)/2 => 5");
        bPassed &= checkEquals("15", ExpressionEvaluator.evaluateOrNull("30/2*1"), "(30/2)*1 => 15");

        bPassed &= checkEquals("15", ExpressionEvaluator.evaluateOrNull("3*4+5*2-7"), "3*4+5*2-7 => 15");
        bPassed &= checkEquals("9", ExpressionEvaluator.evaluateOrNull("1+2+3+3"), "1+2+3+3 => 9");

        return bPassed;
    }

    private static boolean testDivisionRules() {
        boolean bPassed = true;

        printCase("division rules (Java int division)");

        bPassed &= checkEquals("6", ExpressionEvaluator.evaluateOrNull("20/3"), "20/3 => 6");
        bPassed &= checkEquals("2", ExpressionEvaluator.evaluateOrNull("7/3"), "7/3 => 2");
        bPassed &= checkEquals("0", ExpressionEvaluator.evaluateOrNull("2/3"), "2/3 => 0");

        bPassed &= checkEquals("-6", ExpressionEvaluator.evaluateOrNull("-20/3"), "-20/3 => -6");
        bPassed &= checkEquals("-6", ExpressionEvaluator.evaluateOrNull("20/-3"), "20/-3 => -6");
        bPassed &= checkEquals("6", ExpressionEvaluator.evaluateOrNull("-20/-3"), "-20/-3 => 6");

        bPassed &= checkEquals("0", ExpressionEvaluator.evaluateOrNull("2/-3"), "2/-3 => 0");
        bPassed &= checkEquals("0", ExpressionEvaluator.evaluateOrNull("-2/3"), "-2/3 => 0");

        bPassed &= checkEquals("-1", ExpressionEvaluator.evaluateOrNull("1/-1"), "1/-1 => -1");
        bPassed &= checkEquals("-1", ExpressionEvaluator.evaluateOrNull("1 / -1"), "1 / -1 => -1");

        return bPassed;
    }

    private static boolean testDivisionByZero() {
        boolean bPassed = true;

        printCase("division by zero (/0)");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1/0"), "1/0 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("0/0"), "0/0 => null");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("10/0+3"), "10/0+3 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("10+3/0"), "10+3/0 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("10/0*3"), "10/0*3 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("10*3/0"), "10*3/0 => null");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("  1 / 0  "), "\"  1 / 0  \" => null");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("2/-0"), "2/-0 => null");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("2/ -0"), "2/ -0 => null (broken unary '-')");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1/-0"), "1/-0 => null (-0 invalid)");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1/ -0"), "1/ -0 => null (broken unary '-')");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1 0/0"), "\"10/0\" => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1 0 / 0"), "\"10 / 0\" => null");

        return bPassed;
    }


    private static boolean testConcatenationByWhitespaceBetweenDigits() {
        boolean bPassed = true;

        printCase("digit concatenation via whitespace");

        bPassed &= checkEquals("123", ExpressionEvaluator.evaluateOrNull("1 2 3"), "1 2 3 => 123");
        bPassed &= checkEquals("1000", ExpressionEvaluator.evaluateOrNull("1 0 0 0"), "1 0 0 0 => 1000");
        bPassed &= checkEquals("46", ExpressionEvaluator.evaluateOrNull("4 0 + 6"), "40 + 6 => 46");
        bPassed &= checkEquals("24", ExpressionEvaluator.evaluateOrNull("1 + 2 3"), "1 + 23 => 24");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("0 0 7"), "0 0 7 => 007 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("-0 0 7"), "-0 0 7 => -007 => null");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("0 0 0"), "0 0 0 => 000 => null");

        return bPassed;
    }

    private static boolean testInvalidCharacters() {
        boolean bPassed = true;

        printCase("invalid characters");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1a+2"), "alphabet => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1.2+3"), "dot => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1e3+2"), "exponent-like => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("(1+2)"), "parentheses => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1,000+2"), "comma => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1+2=3"), "equals => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1^2"), "caret => null");

        return bPassed;
    }

    private static boolean testAdjacentOperatorPairsExhaustive_NoSpaces() {
        boolean bPassed = true;

        printCase("adjacent operator pairs (no spaces): 1<op1><op2>2");

        for (int i = 0; i < OPERATORS.length; ++i) {
            for (int j = 0; j < OPERATORS.length; ++j) {
                char op1 = OPERATORS[i];
                char op2 = OPERATORS[j];

                String expression = "1" + op1 + op2 + "2";
                String expectedOrNull = expectedForAdjacentPair(op1, op2);

                bPassed &= checkEquals(expectedOrNull, ExpressionEvaluator.evaluateOrNull(expression),
                        "pair \"" + op1 + op2 + "\" in \"" + expression + "\"");
            }
        }

        return bPassed;
    }

    private static boolean testAdjacentOperatorPairsExhaustive_SpacedUnaryAllowed() {
        boolean bPassed = true;

        printCase("adjacent operator pairs (spaces, unary allowed): 1 <op1> <op2>2");

        for (int i = 0; i < OPERATORS.length; ++i) {
            for (int j = 0; j < OPERATORS.length; ++j) {
                char op1 = OPERATORS[i];
                char op2 = OPERATORS[j];

                String expression;
                if (op2 == '-') {
                    expression = "1 " + op1 + " " + op2 + "2";
                } else {
                    expression = "1 " + op1 + " " + op2 + " 2";
                }

                String expectedOrNull = expectedForAdjacentPair(op1, op2);

                bPassed &= checkEquals(expectedOrNull, ExpressionEvaluator.evaluateOrNull(expression),
                        "pair \"" + op1 + " " + op2 + "\" in \"" + expression + "\"");
            }
        }

        return bPassed;
    }

    private static boolean testAdjacentOperatorPairsExhaustive_SpacedUnaryBroken() {
        boolean bPassed = true;

        printCase("adjacent operator pairs (spaces, unary broken): 1 <op1> <op2> 2 (always null)");

        for (int i = 0; i < OPERATORS.length; ++i) {
            for (int j = 0; j < OPERATORS.length; ++j) {
                char op1 = OPERATORS[i];
                char op2 = OPERATORS[j];

                String expression = "1 " + op1 + " " + op2 + " 2";
                bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull(expression),
                        "pair \"" + op1 + " " + op2 + "\" in \"" + expression + "\" => null");
            }
        }

        return bPassed;
    }

    private static String expectedForAdjacentPair(char op1, char op2) {
        if (op2 != '-') {
            return null;
        }

        if (op1 == '+') {
            return "-1";
        }

        if (op1 == '-') {
            return "3";
        }

        if (op1 == '*') {
            return "-2";
        }

        if (op1 == '/') {
            return "0";
        }

        assert (false);
        return null;
    }

    private static boolean testInvalidOperatorPlacement() {
        boolean bPassed = true;

        printCase("invalid operator placement");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("+3"), "leading '+' => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1+"), "trailing operator => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("*1"), "leading '*' => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("/1"), "leading '/' => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1*"), "trailing '*' => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1/"), "trailing '/' => null");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1++2"), "++ => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("2**3"), "** => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("2//3"), "// => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("2*/3"), "*/ => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("2/*3"), "/* => null");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("2+- 3"), "2 + - 3 (broken unary) => null");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("0 + - 0"), "0 + - 0 (broken unary) => null");

        return bPassed;
    }

    private static boolean testInvalidLeadingZerosEverywhere() {
        boolean bPassed = true;

        printCase("leading zeros invalid everywhere");

        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("007"), "007 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("-007"), "-007 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1+007"), "1+007 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("1 + 0 0 7"), "1 + 007 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("10*0002"), "10*0002 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("100/0005"), "100/0005 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("2--007"), "2--007 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("0 0"), "00 => null");
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull("-0 0"), "-00 => null");

        return bPassed;
    }

    private static boolean testStressLongValidExpressions() {
        boolean bPassed = true;

        printCase("stress: long valid expressions (50~200 chars)");

        String e1 = "1+2*3-4/2+5*6-7+8/3+9*2-10/5+11-12/4+13*1-14/7+15";
        bPassed &= checkEquals("80", ExpressionEvaluator.evaluateOrNull(e1), "long valid #1");

        String e2 = "  1 + 2*3 - 4/2 + 5*6 - 7 + 8/3 + 9*2 - 10/5 + 11 - 12/4 + 13*1 - 14/7 + 15  ";
        bPassed &= checkEquals("80", ExpressionEvaluator.evaluateOrNull(e2), "long valid #2 (with spaces)");

        String e3 = "1 0+2 0*3-4 0/2+5*6-7+8/3+9*2-1 0/5+11-12/4+13*1-14/7+15";
        bPassed &= checkEquals("125", ExpressionEvaluator.evaluateOrNull(e3), "long valid #3 (digit-concat, no leading zeros)");

        String e4 = "2--3*4+5*-6+7--8/2+9*-1";
        bPassed &= checkEquals("-14", ExpressionEvaluator.evaluateOrNull(e4), "long valid #4 (uses -- and *-)");

        String e5 = "1+2+3+4+5+6+7+8+9+10+11+12+13+14+15+16+17+18+19+20";
        bPassed &= checkEquals("210", ExpressionEvaluator.evaluateOrNull(e5), "long valid #5 (many +)");

        return bPassed;
    }

    private static boolean testStressLongInvalidExpressions() {
        boolean bPassed = true;

        printCase("stress: long invalid expressions (50~200 chars)");

        String e1 = "1+2*3-4/2+5*6-7+8/3+9*2-10/5+11-12/4+13*1-14/7+015";
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull(e1), "long invalid #1 (leading zeros at end)");

        String e2 = "1+2*3-4/2+5*6-7+8/3+9*2-10/5+11-12/4+13*1-14/7+1+";
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull(e2), "long invalid #2 (trailing operator)");

        String e3 = "1+2*3-4/2+5*6-7+8/3+9*2-10/5+11-12/4+13*1-14/7+1++2";
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull(e3), "long invalid #3 (++)");

        String e4 = "1+2*3-4/2+5*6-7+8/3+9*2-10/5+11-12/4+13*1-14/7+1* - 2";
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull(e4), "long invalid #4 (broken unary '-')");

        String e5 = "1+2*3-4/2+5*6-7+8/3+9*2-10/5+11-12/4+13*(1)-14/7+1";
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull(e5), "long invalid #5 (invalid char: parentheses)");

        String e6 = "2--3*4+5*--6+7--8/2+9*-1";
        bPassed &= checkEquals(null, ExpressionEvaluator.evaluateOrNull(e6), "long invalid #6 (--6 invalid)");

        return bPassed;
    }

    private static boolean checkEquals(String expectedOrNull, String actualOrNull, String message) {
        ++totalChecks;

        boolean bOk = true;

        if (expectedOrNull == null) {
            if (actualOrNull != null) {
                bOk = false;
            }
        } else {
            if (actualOrNull == null) {
                bOk = false;
            } else if (!expectedOrNull.equals(actualOrNull)) {
                bOk = false;
            }
        }

        if (bOk) {
            ++passedChecks;
            System.out.println("[PASS] " + message);
            return true;
        }

        System.out.println("[FAIL] " + message);
        System.out.println("       expected: " + expectedOrNull);
        System.out.println("       actual  : " + actualOrNull);
        return false;
    }

    private static void printCase(String title) {
        System.out.println();
        System.out.println("----------------------------------------");
        System.out.printf("CASE %02d: %s%n", caseIndex++, title);
        System.out.println("----------------------------------------");
    }
}
```

- `FAIL`이 하나라도 있으면 실습 명세를 다시 꼼꼼히 검토한 후 수정한다.