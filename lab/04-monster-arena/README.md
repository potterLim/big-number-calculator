# MonsterArena 실습

이 실습에서는 여러 몬스터가 하나의 경기장에 입장하여 전투를 수행하는 턴 기반 배틀 로열 게임 Monster Battle Royale의 프로토타입을 구현한다. 각 플레이어는 자신이 선택한 몬스터 하나를 경기장에 출전시키며 경기장은 정해진 규칙에 따라 몬스터 간 전투를 관리한다.

경기장은 수용 가능한 최대 인원(capacity)을 가지며 인원이 가득 찬 경기장에는 추가로 몬스터를 입장시킬 수 없다. 경기장에 입장한 몬스터들은 매 턴마다 고정된 순서에 따라 전투를 수행하며 각 몬스터는 바로 다음에 위치한 몬스터를 공격한다. 마지막 몬스터의 공격 대상은 순환 구조에 따라 경기장의 첫 번째 몬스터가 된다.

전투 과정에서 몬스터의 체력이 0이 되면 해당 몬스터는 해당 턴의 공격 처리가 모두 끝난 뒤 경기장에서 퇴출되며 더 이상 전투에 참여하지 않는다. 이 과정을 반복하여 경기장에 단 하나의 몬스터만 남게 되면 해당 몬스터가 최종 승자가 된다.

이 실습의 목적은 객체 지향적 설계를 기반으로 몬스터, 속성 상성, 전투 규칙, 경기장 상태를 체계적으로 모델링하고 주어진 규칙에 따라 정확하게 동작하는 턴 기반 전투 시뮬레이션을 구현하는 데 있다.

## 전반적인 규칙

- 실습 명세에서 제공된 메서드 시그니처와 멤버 변수는 수정할 수 없다.
    - 구현에 필요하다면 추가적인 `private` 도움 메서드를 작성하는 것은 허용된다.
    - 구현에 필요하다면 추가적인 `private` 멤버 변수를 작성하는 것은 허용된다.

- 실습 명세에서 제공된 멤버 변수에 해당하는 정보는 클래스 외부에서 조회 가능해야 한다.

- 실습 명세에 명시되지 않은 동작은 제공된 예시 코드를 통해 추론하여 구현한다.
    - 예시 코드로도 추론이 어려운 경우 명세의 규칙을 위반하지 않는 범위 내에서 합리적으로 판단하여 구현한다.

## 1. 프로젝트를 준비한다

1. IntelliJ에서 `MonsterArena` 프로젝트를 생성한다.
2. `monsterarena` 패키지를 생성한다.
3. `monsterarena` 패키지에 `ElementType` 열거형(enum)을 정의한다.
4. `monsterarena` 패키지에 `Monster` 클래스를 정의한다.
5. `monsterarena` 패키지에 `Arena` 클래스를 정의한다.

## 2. 게임을 구현한다

### 2.1. `ElementType` 열거형을 구현한다

- 몬스터는 다음의 4가지 속성 중 하나를 가진다:
    - 불(FIRE)
    - 물(WATER)
    - 바람(WIND)
    - 땅(EARTH)

- 각 속성 간의 상성 관계는 다음과 같이 정의된다:
    - 불 속성은 바람 속성에 강하며, 물 속성과 땅 속성에 약하다.
    - 물 속성은 불 속성에 강하며, 바람 속성에 약하다.
    - 땅 속성은 불 속성에 강하며, 바람 속성에 약하다.
    - 바람 속성은 땅 속성과 물 속성에 강하며, 불 속성에 약하다.

### 2.2. `Monster` 클래스를 구현한다

- `Monster` 클래스의 생성자는 다음 순서의 인자들을 받는다:
    - `String name`
    - `ElementType elementType`
    - `int health`
    - `int attackStat`
    - `int defenseStat`

- 생성자의 `health`, `attackStat`, `defenseStat`의 인자로 음수값이 들어오지 않는다고 가정한다.

- `Monster` 클래스는 다음의 멤버 변수들을 가진다:
    - `String name`
    - `ElementType elementType`
    - `int health`
    - `int attackStat`
    - `int defenseStat`

- 위 멤버 변수들은 클래스 외부에서 직접 수정할 수 없어야 한다.

- `health`, `attackStat`, `defenseStat`의 값은 음수가 될 수 없다.

#### 2.2.1. `takeDamage` 메서드를 구현한다

- `takeDamage` 메서드는 몬스터가 다른 몬스터로부터 공격을 받아 피해를 입을 때 사용한다.

- 이 메서드는 유일한 인자로 `int amount`를 받는다.

- 이 메서드는 `amount`만큼 몬스터의 체력(`health`)을 감소시킨다.

- `amount`는 음수가 들어오지 않는다고 가정한다.

- 이 메서드는 반환값을 가지지 않는다.

```java
Monster monster = new Monster("Monster1", ElementType.FIRE, 80, 5, 0);

monster.takeDamage(20); // monster의 health는 60이 된다.
monster.takeDamage(30); // monster의 health는 30이 된다.
```

#### 2.2.2. `attack` 메서드를 구현한다

- `attack` 메서드는 몬스터가 다른 몬스터를 공격할 때 사용한다.

- 이 메서드는 유일한 인자로 `Monster otherMonster`를 받으며, 이는 공격을 받는 몬스터를 나타낸다.

- 공격받는 몬스터가 입는 피해치는 다음 규칙에 따라 계산한다:
    - 기본 피해치는 `공격하는 몬스터의 공격력 - 공격받는 몬스터의 방어력`으로 계산한다.
    - 공격하는 몬스터가 공격받는 몬스터보다 속성 상성에서 유리한 경우, 기본 피해치를 50% 증가시킨다. (최종 피해치 = 기본 피해치 × 1.5)
    - 공격하는 몬스터가 속성 상성에서 불리한 경우, 기본 피해치를 절반으로 감소시킨다. (최종 피해치 = 기본 피해치 × 0.5)
    - 속성 상성의 우열을 판단할 수 없는 경우, 기본 피해치가 곧 최종 피해치가 된다.
    - 최종 피해치는 반드시 1 이상이어야 한다.
    - 최종 피해치에서 소수점 이하는 버린다.

- 공격받는 몬스터는 위 규칙에 따라 계산된 최종 피해치만큼의 피해를 입는다.

- 이 메서드는 반환값을 가지지 않는다.

```java
Monster monster1 = new Monster("Monster1", ElementType.FIRE, 80, 5, 0);
Monster monster2 = new Monster("Monster2", ElementType.FIRE, 80, 5, 0);
Monster monster3 = new Monster("Monster3", ElementType.WIND, 80, 7, 1);

monster1.attack(monster2); // monster2의 health는 75가 된다.
monster1.attack(monster3); // monster3의 health는 74가 된다.
monster3.attack(monster1); // monster1의 health는 77이 된다.
```

### 2.3. `Arena` 클래스를 구현한다

- `Arena` 클래스의 생성자는 다음 순서의 인자들을 받는다:
    - `String arenaName`
    - `int capacity`

- 생성자의 `capacity` 인자로 음수값이 들어오지 않는다고 가정한다.

- `Arena` 클래스는 다음의 멤버 변수들을 가진다:
    - `int capacity`
    - `String arenaName`
    - `int turns`
    - `int monsterCount`

- 위 값들은 클래스 외부에서 직접 수정할 수 없어야 한다.

- `turns`와 `monsterCount`의 값은 음수가 될 수 없다.

#### 2.3.1. `loadMonsters` 메서드를 구현한다

- `loadMonsters` 메서드는 `.csv` 파일로부터 몬스터들을 읽어 경기장에 추가할 때 사용한다.

- 이 메서드는 유일한 인자로 `String filePath`를 받는다.

- 이 메서드는 반환값을 가지지 않는다.

- `.csv` 파일의 각 줄은 몬스터 하나의 정보를 담고 있으며, 각 줄의 포맷은 다음과 같다:
    - `Name,ElementType,Health,AttackStat,DefenseStat`

```
MyMonster1,EARTH,100,20,10
```

- 위 데이터는 다음 정보를 의미한다:
    - 이름: `MyMonster1`
    - 속성: `EARTH`
    - 체력: `100`
    - 공격력: `20`
    - 방어력: `10`

- 이 메서드는 `.csv` 파일에서 몬스터들을 순서대로 읽어 경기장에 추가한다.

- 이 메서드를 호출하면 기존의 경기장 상태를 모두 초기화한 후 파일의 내용으로 몬스터를 다시 로드한다.

- 경기장이 수용 인원(`capacity`)에 도달한 경우 남은 몬스터는 추가하지 않는다.

```java
/*
monsters.csv
MyMonster1,EARTH,100,20,10
MyMonster2,WATER,40,2,1
MyMonster3,WIND,44,15,10
MyMonster4,FIRE,250,50,10
MyMonster5,FIRE,200,10,10
MyMonster6,WIND,51,3,3
*/

Arena arena = new Arena("5 Free For All", 5);
arena.loadMonsters("C://Some/Path/To/CSV/monsters.csv");

// MyMonster1부터 MyMonster5까지 총 5마리가 경기장에 추가된다.
```

- `.csv` 파일의 데이터는 언제나 올바르다고 가정한다. (누락된 값이 없고, 정수 값이 올바르며, 속성 값이 유효함)

#### 2.3.2. `goToNextTurn` 메서드를 구현한다

- `goToNextTurn` 메서드는 경기장에 있는 몬스터들이 서로를 공격하도록 한 턴을 진행할 때 사용한다.

- 경기장에 몬스터가 1마리 이하인 경우 몬스터의 공격은 발생하지 않는다.

- 몬스터가 2마리 이상인 경우 각 몬스터는 바로 다음에 위치한 몬스터를 공격한다.

- 마지막 몬스터의 공격 대상은 경기장의 첫 번째 몬스터이다.


```text
경기장에 출전한 몬스터 목록: [monster1, monster2, monster3, monster4, monster5, monster6]

monster1은 monster2를 공격한다.
monster2는 monster3을 공격한다.
monster3은 monster4를 공격한다.
monster4는 monster5를 공격한다.
monster5는 monster6을 공격한다.
monster6은 monster1을 공격한다.
```

- 한 턴의 모든 공격 처리가 끝난 뒤 체력이 0이 된 몬스터는 경기장에서 퇴출된다.

- 몬스터가 퇴출될 때마다 `monsterCount`를 1만큼 감소시켜야 한다.

- 몬스터들의 공격 처리가 모두 끝났다면 `turns`를 1만큼 증가시켜야 한다.

```java
/*
monsters.csv
MyMonster1,EARTH,100,20,10
MyMonster2,WATER,40,2,1
MyMonster3,WIND,44,15,10
MyMonster4,FIRE,50,50,10
MyMonster5,FIRE,200,10,10
MyMonster6,WIND,51,3,3
*/

Arena arena = new Arena("Fun Arena", 6);
arena.loadMonsters("C://Some/Path/To/CSV/monsters.csv");

arena.goToNextTurn();
arena.goToNextTurn();
arena.goToNextTurn();
arena.goToNextTurn();
```

위와 같은 세팅에서 `goToNextTurn`을 호출할 때마다 각 몬스터의 체력(`health`)이 변하는 모습은 다음과 같다:

| 이름\턴  |  0  |  1  |  2  |     3    |   4  |
| :--------: | :-: | :-: | :-: | :------: | :--: |
| MyMonster1 | 100 |  99 |  98 |    97    |  96  |
| MyMonster2 |  40 |  21 |   2 | 0 (퇴출) | N/A  |
| MyMonster3 |  44 |  43 |  42 |     42   |  37  |
| MyMonster4 |  50 |  48 |  46 |     44   |  42  |
| MyMonster5 | 200 | 160 | 120 |     80   |  40  |
| MyMonster6 |  51 |  41 |  31 |     21   |  11  |

#### 2.3.3. `getHealthiestOrNull` 메서드를 구현한다

- `getHealthiestOrNull` 메서드는 경기장에서 체력이 가장 높은 몬스터를 반환할 때 사용한다.

- 이 메서드는 인자를 받지 않는다.

- 경기장에 몬스터가 없는 경우 `null`을 반환한다.

```java
/*
monsters.csv
MonsterA,FIRE,30,5,1
MonsterB,WATER,50,4,2
MonsterC,EARTH,40,6,3
*/

Arena arena = new Arena("Test Arena", 3);
arena.loadMonsters("monsters.csv");

Monster healthiestMonster = arena.getHealthiestOrNull();
// healthiestMonster는 MonsterB를 가리킨다.
```

## 3. 본인 컴퓨터에서 테스트

### 3.1 테스트 데이터 다운로드 및 위치

테스트를 위해 [datas.zip](./datas.zip) 파일을 다운로드한 후 프로젝트 루트 디렉터리에서 압축을 해제하여 `datas/` 디렉터리를 생성해야 한다.

압축 해제 후 프로젝트 구조는 다음과 같아야 한다:

```
MonsterArena/
├─ src/
│  └─ monsterarena/
│     ├─ Main.java
│     ├─ Arena.java
│     ├─ Monster.java
│     └─ ElementType.java
├─ datas/
│  ├─ monsters_case01_example_full.txt
│  ├─ monsters_case02_empty.txt
│  ├─ monsters_case03_single.txt
│  ├─ monsters_case04_two.txt
│  ├─ monsters_case05_six_capacity_stress.txt
│  ├─ monsters_case06_tie_healthiest.txt
│  ├─ monsters_case07_ring3.txt
│  ├─ monsters_case08_remove_timing.txt
└─ MonsterArena.iml
```

### 3.2. 테스트용 `Main.java`

- 프로젝트의 Main.java 파일을 아래의 예처럼 수정한다.

```java
package monsterarena;

public class Main {
    private static int totalChecks = 0;
    private static int passedChecks = 0;
    private static int roundPrintCounter = 0;

    private static final String FILE_CASE01_EXAMPLE_FULL = "datas/monsters_case01_example_full.txt";
    private static final String FILE_CASE02_EMPTY = "datas/monsters_case02_empty.txt";
    private static final String FILE_CASE03_SINGLE = "datas/monsters_case03_single.txt";
    private static final String FILE_CASE04_TWO = "datas/monsters_case04_two.txt";
    private static final String FILE_CASE05_SIX_CAPACITY_STRESS = "datas/monsters_case05_six_capacity_stress.txt";
    private static final String FILE_CASE06_TIE_HEALTHIEST = "datas/monsters_case06_tie_healthiest.txt";
    private static final String FILE_CASE07_RING3 = "datas/monsters_case07_ring3.txt";
    private static final String FILE_CASE08_REMOVE_TIMING = "datas/monsters_case08_remove_timing.txt";

    public static void main(String[] args) {
        boolean bAllPassed = true;

        System.out.println("Monster Arena Test Runner");
        System.out.println("----------------------------------------");

        bAllPassed &= testMissingFile();
        bAllPassed &= testEmptyFile();
        bAllPassed &= testSingleMonster();
        bAllPassed &= testTwoMonstersBasicProgression();
        bAllPassed &= testCapacityLimit();
        bAllPassed &= testReloadResetsRound();
        bAllPassed &= testExampleScenarioStrict();
        bAllPassed &= testHealthiestTieReturnsFirst();
        bAllPassed &= testRing3SymmetryStrict();
        bAllPassed &= testRemoveTimingNoCrashConsistentState();

        System.out.println("\n========================================");
        System.out.printf("CHECKS: %d passed / %d total\n", passedChecks, totalChecks);
        if (bAllPassed) {
            System.out.println("ALL TESTS PASSED");
        } else {
            System.out.println("SOME TESTS FAILED");
        }
        System.out.println("========================================");
    }

    private static boolean testMissingFile() {
        boolean bPassed = true;

        printCase("CASE: missing file");

        Arena arena = new Arena("MissingFileArena", 8);

        bPassed &= check(arena.getArenaName().equals("MissingFileArena"), "arenaName is preserved");
        bPassed &= check(arena.getCapacity() == 8, "capacity is preserved");

        arena.loadMonsters("this_file_should_not_exist_12345.txt");

        bPassed &= check(arena.getMonsterCount() == 0, "monsterCount == 0 when file missing");
        bPassed &= check(arena.getTurns() == 0, "turns == 0 when file missing");
        bPassed &= check(arena.getHealthiestOrNull() == null, "healthiest == null when empty");

        int beforeTurns = arena.getTurns();
        arena.goToNextTurn();
        bPassed &= check(arena.getTurns() == beforeTurns, "goToNextTurn() does nothing when <= 1 monster");

        return bPassed;
    }

    private static boolean testEmptyFile() {
        boolean bPassed = true;

        printCase("CASE: empty file");

        Arena arena = new Arena("EmptyFileArena", 8);
        arena.loadMonsters(FILE_CASE02_EMPTY);

        bPassed &= check(arena.getMonsterCount() == 0, "monsterCount == 0 for empty file");
        bPassed &= check(arena.getTurns() == 0, "turns == 0 after load empty (reset expected)");
        bPassed &= check(arena.getHealthiestOrNull() == null, "healthiest == null for empty arena");

        int beforeTurns = arena.getTurns();
        arena.goToNextTurn();
        bPassed &= check(arena.getTurns() == beforeTurns, "turns unchanged when monsterCount <= 1");

        return bPassed;
    }

    private static boolean testSingleMonster() {
        boolean bPassed = true;

        printCase("CASE: single monster");

        Arena arena = new Arena("SingleMonsterArena", 8);
        arena.loadMonsters(FILE_CASE03_SINGLE);

        bPassed &= check(arena.getMonsterCount() == 1, "monsterCount == 1");
        bPassed &= check(arena.getTurns() == 0, "turns == 0 after load (reset expected)");
        bPassed &= check(arena.getHealthiestOrNull() != null, "healthiest not null");
        bPassed &= check(arena.getHealthiestOrNull().getName().equals("Solo"), "healthiest name == Solo");

        int beforeTurns = arena.getTurns();
        int beforeHealth = arena.getHealthiestOrNull().getHealth();

        arena.goToNextTurn();

        bPassed &= check(arena.getTurns() == beforeTurns, "turns unchanged when monsterCount <= 1");
        bPassed &= check(arena.getMonsterCount() == 1, "monsterCount unchanged when monsterCount <= 1");
        bPassed &= check(arena.getHealthiestOrNull() != null, "healthiest still not null");
        bPassed &= check(arena.getHealthiestOrNull().getHealth() == beforeHealth, "HP unchanged when monsterCount <= 1");

        return bPassed;
    }

    private static boolean testTwoMonstersBasicProgression() {
        boolean bPassed = true;

        printCase("CASE: two monsters (basic progression)");

        Arena arena = new Arena("TwoMonstersArena", 8);
        arena.loadMonsters(FILE_CASE04_TWO);

        bPassed &= check(arena.getMonsterCount() == 2, "monsterCount == 2");
        bPassed &= check(arena.getTurns() == 0, "turns == 0 after load (reset expected)");
        bPassed &= check(arena.getHealthiestOrNull() != null, "healthiest not null initially");

        printArenaSnapshot(arena);

        bPassed &= checkTurnRule(arena, "after one turn");
        bPassed &= check(arena.getMonsterCount() >= 0 && arena.getMonsterCount() <= 2, "monsterCount remains valid range [0..2]");
        bPassed &= check(arena.getMonsterCount() == 0 || arena.getHealthiestOrNull() != null, "if non-empty, healthiest must not be null");

        int beforeCount = arena.getMonsterCount();
        int beforeTurns = arena.getTurns();

        arena.goToNextTurn();

        if (beforeCount <= 1) {
            bPassed &= check(arena.getTurns() == beforeTurns, "if <= 1 before call, turns must not increase");
        } else {
            bPassed &= check(arena.getTurns() == beforeTurns + 1, "if >= 2 before call, turns must increase by 1");
        }

        return bPassed;
    }

    private static boolean testCapacityLimit() {
        boolean bPassed = true;

        printCase("CASE: capacity limit");

        Arena arena = new Arena("CapacityArena", 3);
        bPassed &= check(arena.getCapacity() == 3, "capacity == 3");

        arena.loadMonsters(FILE_CASE05_SIX_CAPACITY_STRESS);

        bPassed &= check(arena.getMonsterCount() == 3, "monsterCount capped to capacity");
        bPassed &= check(arena.getTurns() == 0, "turns == 0 after load (reset expected)");
        bPassed &= check(arena.getMonsterCount() == 0 || arena.getHealthiestOrNull() != null, "healthiest not null when monsterCount > 0");

        return bPassed;
    }

    private static boolean testReloadResetsRound() {
        boolean bPassed = true;

        printCase("CASE: reload resets round (turns reset expected)");

        Arena arena = new Arena("ReloadArena", 8);

        arena.loadMonsters(FILE_CASE05_SIX_CAPACITY_STRESS);
        bPassed &= check(arena.getMonsterCount() == 6, "first load => 6 monsters");
        bPassed &= check(arena.getTurns() == 0, "first load => turns reset to 0");

        bPassed &= checkTurnRule(arena, "after one turn");
        bPassed &= check(arena.getMonsterCount() > 0, "after one turn => still non-empty");

        arena.loadMonsters(FILE_CASE03_SINGLE);
        bPassed &= check(arena.getMonsterCount() == 1, "reload => monsterCount == 1");
        bPassed &= check(arena.getTurns() == 0, "reload => turns reset to 0");
        bPassed &= check(arena.getHealthiestOrNull() != null, "reload => healthiest not null");
        bPassed &= check(arena.getHealthiestOrNull().getName().equals("Solo"), "reload => healthiest == Solo");

        int beforeTurns = arena.getTurns();
        arena.goToNextTurn();
        bPassed &= check(arena.getTurns() == beforeTurns, "one monster => no turn increment");

        return bPassed;
    }

    private static boolean testExampleScenarioStrict() {
        boolean bPassed = true;

        printCase("CASE: example scenario (STRICT HP checks)");

        Arena arena = new Arena("ExampleArena", 8);
        arena.loadMonsters(FILE_CASE01_EXAMPLE_FULL);

        bPassed &= check(arena.getMonsterCount() == 6, "initial monster count == 6");
        bPassed &= check(arena.getTurns() == 0, "turns == 0 after load (reset expected)");
        bPassed &= check(arena.getHealthiestOrNull() != null, "healthiest not null initially");
        bPassed &= check(arena.getHealthiestOrNull().getName().equals("MyMonster1"), "initial healthiest == MyMonster1");
        bPassed &= check(arena.getHealthiestOrNull().getHealth() == 100, "initial healthiest HP == 100");

        printArenaSnapshot(arena);

        bPassed &= doExampleTurnAndCheck(arena, 1, 6, "MyMonster1", 99);
        bPassed &= doExampleTurnAndCheck(arena, 2, null, "MyMonster1", 98);
        bPassed &= doExampleTurnAndCheck(arena, 3, 4, "MyMonster1", 97);
        bPassed &= doExampleTurnAndCheck(arena, 4, 3, "MyMonster1", 97);
        bPassed &= doExampleTurnAndCheck(arena, 5, null, "MyMonster1", 77);
        bPassed &= doExampleTurnAndCheck(arena, 6, null, "MyMonster1", 57);
        bPassed &= doExampleTurnAndCheck(arena, 7, null, "MyMonster1", 37);
        bPassed &= doExampleTurnAndCheck(arena, 8, null, "MyMonster4", 34);
        bPassed &= doExampleTurnAndCheck(arena, 9, 2, "MyMonster4", 32);
        bPassed &= doExampleFinalTurnAndCheck(arena);

        int beforeTurns = arena.getTurns();
        int beforeCount = arena.getMonsterCount();
        int beforeWinnerHp = arena.getHealthiestOrNull().getHealth();

        arena.goToNextTurn();

        bPassed &= check(arena.getMonsterCount() == beforeCount, "after end: monsterCount still == 1");
        bPassed &= check(arena.getTurns() == beforeTurns, "after end: turns unchanged");
        bPassed &= check(arena.getHealthiestOrNull() != null, "after end: healthiest not null");
        bPassed &= check(arena.getHealthiestOrNull().getHealth() == beforeWinnerHp, "after end: winner HP unchanged");

        return bPassed;
    }

    private static boolean testHealthiestTieReturnsFirst() {
        boolean bPassed = true;

        printCase("CASE: healthiest tie returns first loaded");

        Arena arena = new Arena("TieArena", 8);
        arena.loadMonsters(FILE_CASE06_TIE_HEALTHIEST);

        bPassed &= check(arena.getMonsterCount() == 3, "monsterCount == 3");
        bPassed &= check(arena.getTurns() == 0, "turns == 0 after load (reset expected)");
        bPassed &= check(arena.getHealthiestOrNull() != null, "healthiest not null");
        bPassed &= check(arena.getHealthiestOrNull().getName().equals("A"), "tie => returns first (A)");
        bPassed &= check(arena.getHealthiestOrNull().getHealth() == 50, "tie => HP == 50");

        return bPassed;
    }

    private static boolean testRing3SymmetryStrict() {
        boolean bPassed = true;

        printCase("CASE: ring3 symmetry (HP strict for symmetry)");

        Arena arena = new Arena("Ring3Arena", 3);
        arena.loadMonsters(FILE_CASE07_RING3);

        bPassed &= check(arena.getMonsterCount() == 3, "monsterCount == 3");
        bPassed &= check(arena.getTurns() == 0, "turns == 0 after load (reset expected)");
        bPassed &= check(arena.getHealthiestOrNull() != null, "healthiest not null");
        bPassed &= check(arena.getHealthiestOrNull().getName().equals("A"), "all equal => healthiest is first (A)");
        bPassed &= check(arena.getHealthiestOrNull().getHealth() == 100, "initial A HP == 100");

        int previousHp = arena.getHealthiestOrNull().getHealth();

        for (int t = 1; t <= 3; ++t) {
            int beforeCount = arena.getMonsterCount();
            int beforeTurns = arena.getTurns();

            arena.goToNextTurn();

            if (beforeCount <= 1) {
                bPassed &= check(arena.getTurns() == beforeTurns, "turns unchanged when <= 1 before call");
                break;
            }

            bPassed &= check(arena.getTurns() == beforeTurns + 1, "turns == " + t);
            bPassed &= check(arena.getMonsterCount() == 0 || arena.getMonsterCount() == 3,
                    "monsterCount is 3 until simultaneous wipe, or 0 after wipe");
            bPassed &= check(arena.getMonsterCount() == 0 || arena.getHealthiestOrNull() != null,
                    "non-empty => healthiest not null");

            if (arena.getMonsterCount() > 0) {
                bPassed &= check(arena.getHealthiestOrNull().getName().equals("A"), "symmetry => healthiest remains first (A)");
                int currentHp = arena.getHealthiestOrNull().getHealth();
                bPassed &= check(currentHp < previousHp, "HP must decrease over turns");
                previousHp = currentHp;
            }
        }

        return bPassed;
    }

    private static boolean testRemoveTimingNoCrashConsistentState() {
        boolean bPassed = true;

        printCase("CASE: remove timing (no crash, consistent state)");

        Arena arena = new Arena("RemoveTimingArena", 3);
        arena.loadMonsters(FILE_CASE08_REMOVE_TIMING);

        bPassed &= check(arena.getMonsterCount() == 3, "monsterCount == 3");
        bPassed &= check(arena.getTurns() == 0, "turns == 0 after load (reset expected)");
        bPassed &= check(arena.getHealthiestOrNull() != null, "healthiest not null initially");

        bPassed &= checkTurnRule(arena, "after one turn");
        bPassed &= check(arena.getMonsterCount() >= 0 && arena.getMonsterCount() <= 3, "monsterCount stays valid [0..3]");
        bPassed &= check(arena.getMonsterCount() == 0 || arena.getHealthiestOrNull() != null, "if non-empty, healthiest must not be null");

        int beforeCount = arena.getMonsterCount();
        int beforeTurns = arena.getTurns();

        arena.goToNextTurn();

        if (beforeCount <= 1) {
            bPassed &= check(arena.getTurns() == beforeTurns, "if <= 1 before call, turns must not increase");
        } else {
            bPassed &= check(arena.getTurns() == beforeTurns + 1, "if >= 2 before call, turns must increase by 1");
        }

        arena.loadMonsters(FILE_CASE03_SINGLE);
        bPassed &= check(arena.getTurns() == 0, "reload => turns reset to 0 (new round)");
        bPassed &= check(arena.getMonsterCount() == 1, "reload => monsterCount == 1");
        bPassed &= check(arena.getHealthiestOrNull() != null, "reload => healthiest not null");

        return bPassed;
    }

    private static boolean doExampleTurnAndCheck(Arena arena, int expectedTurns, Integer expectedCountOrNull, String expectedHealthiestName, int expectedHealthiestHp) {
        boolean bPassed = true;

        int beforeCount = arena.getMonsterCount();
        int beforeTurns = arena.getTurns();

        arena.goToNextTurn();

        if (beforeCount <= 1) {
            bPassed &= check(arena.getTurns() == beforeTurns, "turn " + expectedTurns + " skipped because <= 1 before call");
            return bPassed;
        }

        if (expectedCountOrNull != null) {
            bPassed &= check(arena.getMonsterCount() == expectedCountOrNull, "turn " + expectedTurns + " monster count == " + expectedCountOrNull);
        }

        bPassed &= check(arena.getTurns() == expectedTurns, "turn " + expectedTurns + " turns == " + expectedTurns);
        bPassed &= check(arena.getMonsterCount() == 0 || arena.getHealthiestOrNull() != null, "turn " + expectedTurns + " healthiest not null");

        if (arena.getMonsterCount() > 0) {
            bPassed &= check(arena.getHealthiestOrNull().getName().equals(expectedHealthiestName), "turn " + expectedTurns + " healthiest name == " + expectedHealthiestName);
            bPassed &= check(arena.getHealthiestOrNull().getHealth() == expectedHealthiestHp, "turn " + expectedTurns + " healthiest HP == " + expectedHealthiestHp);
        }

        return bPassed;
    }

    private static boolean doExampleFinalTurnAndCheck(Arena arena) {
        boolean bPassed = true;

        int beforeCount = arena.getMonsterCount();
        int beforeTurns = arena.getTurns();

        arena.goToNextTurn();

        if (beforeCount <= 1) {
            bPassed &= check(arena.getTurns() == beforeTurns, "final turn skipped because <= 1 before call");
            bPassed &= check(arena.getMonsterCount() == beforeCount, "final monsterCount unchanged");
            return bPassed;
        }

        bPassed &= check(arena.getMonsterCount() == 1, "final monster count == 1");
        bPassed &= check(arena.getTurns() == 10, "final turns == 10");
        bPassed &= check(arena.getHealthiestOrNull() != null, "final healthiest not null");
        bPassed &= check(arena.getHealthiestOrNull().getName().equals("MyMonster4"), "final winner == MyMonster4");
        bPassed &= check(arena.getHealthiestOrNull().getHealth() == 30, "final winner HP == 30");

        return bPassed;
    }

    private static boolean checkTurnRule(Arena arena, String label) {
        int beforeCount = arena.getMonsterCount();
        int beforeTurns = arena.getTurns();

        arena.goToNextTurn();

        if (beforeCount <= 1) {
            return check(arena.getTurns() == beforeTurns, label + " => turns unchanged (<= 1 before call)");
        }

        return check(arena.getTurns() == beforeTurns + 1, label + " => turns increased by 1 (>= 2 before call)");
    }

    private static boolean check(boolean condition, String message) {
        ++totalChecks;

        if (condition) {
            ++passedChecks;
            System.out.println("[PASS] " + message);
            return true;
        }

        System.out.println("[FAIL] " + message);
        return false;
    }

    private static void printCase(String title) {
        System.out.println("\n----------------------------------------");
        System.out.println(title);
        System.out.println("----------------------------------------");
    }

    private static void printArenaSnapshot(Arena arena) {
        System.out.println();
        System.out.println(arena.getArenaName());
        System.out.println("[Snapshot " + (roundPrintCounter++) + "]");
        System.out.println("turns=" + arena.getTurns() + ", monsterCount=" + arena.getMonsterCount());

        Monster healthiest = arena.getHealthiestOrNull();
        if (healthiest == null) {
            System.out.println("healthiest=null");
        } else {
            System.out.println("healthiest=" + healthiest.getName() + ", hp=" + healthiest.getHealth());
        }
    }
}
```

- `FAIL`이 하나라도 있으면 실습 명세를 다시 꼼꼼히 검토한 후 수정한다.