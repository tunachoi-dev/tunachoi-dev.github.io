---
layout: post
title: "[Java] 변수 값 보존 vs 변경: 파괴적 연산과 비파괴적 연산의 차이 (feat. 자릿수 쪼개기)"
date: 2026-02-05 11:29:00 +0900
categories: Java
tags:
  - Java
  - 파괴적연산
  - 비파괴적연산
---
# \[Java] 변수 값 보존 vs 변경: 파괴적 연산과 비파괴적 연산의 차이 (feat. 자릿수 쪼개기)
저는 연산자를 공부를 하고 연습 문제를 풀었습니다. 문제는 아래와 같습니다.

문제 2: 자릿수 쪼개기 (Digit Splitter)설명: 세 자릿수 정수 number가 678로 주어졌을 때,  
백의 자리, 십의 자리, 일의 자리를 각각 분리하여 출력하는 코드를 작성하세요.  
  
힌트: int 형 나눗셈의 특성(소수점 버림)을 이용하면 자릿수를 잘라낼 수 있습니다.

나머지 연산자인 %를 이용하여 백의 자리 계산 후 나머지를 십의 자리, 그 후 동일하게 일의 자리 연산을 한 후 출력을 통해 쉽게 해결할 수 있었습니다.

**Test2.java**
```java
public class Test2 {
	public static void main(String[] args) {
		int number = 678;
		
		int hundreds = number / 100;
		number %= 100;
		
		int tens = number / 10;
		number %= 10;
		
		int ones = number;
		
		System.out.println("백의 자리: " + hundreds);
		System.out.println("십의 자리: " + tens);
		System.out.println("일의 자리: " + ones);
	}
}
```

문제는 쉽게 해결! 하지만 여기서 조금 아쉽다는 의견이 나왔습니다. (Gemini님의 의견...)
그것은 바로 파괴적 연산 방식을 사용했다는 의견입니다.
그래서 저는 파괴적 연산에 대해 추가적으로 알아보기로 했습니다.

## 파괴적 연산 (Destructive Approach)
제가 처음에 작성했던 방식입니다. 변수 `number`의 값을 계속 업데이트하며 계산하는 방식입니다.

```java
public class Test2 {
	public static void main(String[] args) {
		int number = 678;  // 원본 데이터
		
		// 1. 백의 자리 추출
		int hundreds = number / 100;  // 6
		number %= 100;  // number가 78로 변경 (원본 손실)
		
		// 2. 십의 자리 추출
		int tens = number / 10;  // 7
		number %= 10;  // number가 8로 변경
		
		// 3. 일의 자리 추출
		int ones = number;  // 8
		
		System.out.println("백의 자리: " + hundreds);
		System.out.println("십의 자리: " + tens);
		System.out.println("일의 자리: " + ones);
	}
}
```

**특징**
이 방식은 `number`라는 변수 하나를 계속 사용합니다. `%=` (나머지 할당 연산자)를 사용해 변수의 상태를 영구적으로 변경합니다.

**장점:**
- 코드가 직관적이고 단계별로 진행되는 느낌을 줍니다.
- 추가적인 변수 선언을 최소화할 수 있습니다.

**단점:**
- **원본 데이터(`678`)가 소실됩니다.**
- 로직이 끝난 후 원본 숫자가 무엇이었는지 알 수 없게 됩니다.
- 디버깅 시 변수의 값이 계속 변하므로 추적하기 까다로울 수 있습니다.

## 비파괴적 연산 (Non-destructive Approach)
원본 `number`의 값은 읽기(Read)만 하고, 변경하지 않는 방식입니다. 비파괴적 연산으로 다시 코드를 작성해보겠습니다.

```java
public class Test2 {
	public static void main(String[] args) {
		int number = 678;  // 원본 데이터 유지
		
		// 필요한 값만 계산해서 가져옴 (number는 변하지 않음)
		int hundreds = number / 100;
		int tens = (number % 100) / 10;
		int ones = number % 10;
		
		System.out.println("백의 자리: " + hundreds);
		System.out.println("십의 자리: " + tens);
		System.out.println("일의 자리: " + ones);
		
		// 원본이 살아 있음
		System.out.println("입력된 숫자: " + number);  // 출력: 678
	}
}
```

**특징**
수학적 공식을 활용하여 `number`의 값을 변경하지 않고 필요한 값만 추출합니다.

**장점:**
- **데이터의 불변성(Immutability)에 가깝습니다.**
- 로직 수행 후에도 원본 데이터(`678`)를 언제든 다시 사용할 수 있습니다.
- 코드의 순서가 바뀌어도(예: 일의 자리를 먼저 구해도) 서로 영향을 주지 않아 안전합니다.

**단점:**
- 수식이 조금 더 복잡해 보일 수 있습니다.(`(number % 100) / 10` 처럼)
## 무엇이 더 좋을까?
결론부터 말하면, **대부분의 경우 '비파괴적 연산'이 더 안전하고 좋습니다.**

### 왜 비파괴적 연산을 선호할까?
소프트웨어 개발에서 **'사이드 이펙트(Side Effect, 부작용)'** 를 줄이는 것은 매우 중요합니다. 변수의 값이 코드 실행 도중에 계속 바뀐다면, 나중에 코드를 수정하거나 기능을 추가할 때 예상치 못한 버그가 발생할 확률이 높습니다.

"입력받은 `number`를 DB에 저장해줘"라는 요청이 나중에 추가되었다고 상상해 보세요. 파괴적 연산을 썼다면 `678`이 아닌 `8`을 저장하는 대참사가 일어날 수 있습니다.

### 그래도 파괴적 방식을 쓰고 싶다면?
복잡한 계산 로직에서는 파괴적 방식이 가독성이 좋을 때가 있습니다. 그럴 때는 반드시 **백업(Backup)** 을 해두는 것이 관례입니다.

```java
int number = 678;
int originalNumber = number;  // 백업

// ... 파괴적 연산 수행 ...

// 나중에 원본이 필요하면 originalNumber를 사용
System.out.println("원본: " + originalNumber);
```
## 5. 요약
|**구분**|**파괴적 연산 (Destructive)**|**비파괴적 연산 (Non-destructive)**|
|---|---|---|
|**방식**|원본 변수의 값을 직접 수정|원본을 읽기만 하고 새 값을 생성|
|**연산자**|`+=`, `-=`, `%=`, `++` 등|`+`, `-`, `%` 등|
|**원본 보존**|**X (보존 안 됨)**|**O (보존됨)**|
|**안전성**|낮음 (값 추적 어려움)|높음 (사이드 이펙트 적음)|
|**추천**|임시 변수 계산 시 제한적 사용|**기본적으로 권장됨**|

코드를 짤 때, **"이 변수의 값이 나중에도 필요할까?"** 를 항상 고민하는 습관을 들이면 더 견고한 프로그램을 만들 수 있습니다.

[^1]: 
