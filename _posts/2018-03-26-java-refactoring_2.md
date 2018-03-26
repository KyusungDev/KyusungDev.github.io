---
layout: post
title: "[리팩토링] 메서드 정리 #2"
tags: [refactoring]
---

<!-- TOC -->

- [임시변수 내용 직접 삽입 (Inline Temp)](#임시변수-내용-직접-삽입-inline-temp)
- [임시변수를 메서드 호출로 전환 (Replace Temp with Query)](#임시변수를-메서드-호출로-전환-replace-temp-with-query)
- [직관적 임시변수 사용 (Introduce Explaining Variable)](#직관적-임시변수-사용-introduce-explaining-variable)
- [임시변수 분리 (Split Temporary Variable)](#임시변수-분리-split-temporary-variable)
- [매개변수로의 값 대입 제거 (Split Temporary Variable)](#매개변수로의-값-대입-제거-split-temporary-variable)
- [메서드를 메서드 객체로 전환 (Replace Method with Method Object)](#메서드를-메서드-객체로-전환-replace-method-with-method-object)
- [알고리즘 전환 (Substitue Algorithm)](#알고리즘-전환-substitue-algorithm)
- [기타](#기타)

<!-- /TOC -->

### 임시변수 내용 직접 삽입 (Inline Temp)
- 요약
    + 간단한 수식을 대입받는 임시변수로 인해 다른 리팩토링 기법 적용이 힘들 땐, 그임시변수를 참조하는부분을 전부 수식으로 치환하자.
- 동기
    + **임시변수를 메서드 호출** 실시중에 병용하게 됨.
    + 메서드 호출의 결과값이 임시변수에 대입될 때만 순수하게 **임시변수 내용 직접 삽입**만 사용함.
    + **메서드 추출**등 다른 리팩토링에 방해가 되면, **임시변수 내용을 직접 삽입**을 적용.
- 예제
    ```java
    /* 변경 전 */
    double basePrice = anOrder.basePrice();
    return (basePrice> 1000);

    /* 변경 후 */
    return (anOrder.basePrice() > 1000);
    ```

### 임시변수를 메서드 호출로 전환 (Replace Temp with Query)
- 요약
    + 수식의 결과를 저장하는 임시변수가 있을 땐, 그 수식을 빼내어 메서드로 만든 후, 임시변수 참조 부분을 전부 수식으로 교체하자. 새로 만든 메서드는 다른 메서드에서도 호출 가능하다.
- 동기
    + 임시변수는 일시적이고 국소적 범위로 사용이 제한됨.
    + 임시변수는 자신이 속한 메서드 안에서만 사용가능하므로 접근하려다보면 코드가 길어짐.
    + 임시변수를 메서드 호출로 변경하면 클래스안 모든곳에서 접근이 가능함.
    + 지역변수가 많을 수록 메서드 추출이 어려움.
    + 대부분의 경우 **메서드 추출** 적용 전에 반드시 적용해야 함.
- 예제
    ```java
    /* 변경 전 */
    double basePrice = _quantity * _itemPrice;
    if (basePrice > 1000) 
        return basePrice * 0.95;
    else
        return basePrice * 0.98;

    /* 변경 후 */
    if (basePrice() > 1000) 
        return basePrice * 0.95;
    else
        return basePrice * 0.98;

    double basePrice() {
        return _quantity * _itemPrice;
    }
    ```

### 직관적 임시변수 사용 (Introduce Explaining Variable)
- 요약
    + 사용된 수식이복잡할 땐, 수식의 결과나 수식의 일부분을 용도에 부합하는 직관적 이름의 임시변수에 대입하자.
- 동기
    + 조건문에서 각 조건절을 직관적인 이름의 임시변수를 사용해 조건의 의미를 설명하려할 때 사용.
    + 긴 알고리즘에서 계산의 각 단계를 설명할 수 있을 때 사용.
    + 먼저 **메서드 추출** 기법을 사용하려 노력하고, 지역변수로 적용이 힘들 때 사용.
- 예제
    ```java
    /* 변경 전 */
    double price() {
        return _quantity * _itemPrice - Math.max(0, _quantity - 500) * _itemPrice * 0.05 + Math.min(_quantity * _itemPrice * 0.1, 100.0);
    }

    /* 변경 후 - 직관적 임시변수 사용 */
    double price() {
        final double basePrice = _quantity * _itemPrice;
        final double quantityDiscount = Math.max(0, _quantity - 500) * _itemPrice * 0.05;
        final double shipping = Math.min(basePrice * 0.1, 100.0);
        return basePrise - quantityDiscount + shipping;
    }

    /* 변경 후 - 메서드 추출 */
    double pirce() {
        return basePrice() - quantityDiscount() + shipping();
    }

    private double quantityDiscount() {
        return Math.max(0, _quantity - 500) * _itemPrice * 0.05;
    }

    private double shipping() {
        return Math.min(basePrise() * 0.1, 100.0);
    }

    private double basePrise() {
        return _quantity * _itemPrise;
    }
    ```
    
### 임시변수 분리 (Split Temporary Variable)
- 요약
    + 루프 변수나 값 누적용 임시변수가 아닌 임시변수에 여러 번 값이 대입될 땐, 각 대입마다 다른 임시변수를 사용하자.
- 예제 
    ```java
    /* 변경 전 */
    double temp = 2 * (_height + _width);
    System.out.println(temp);
    temp = _height *_width;
    System.out.println(temp);

    /* 변경 후 */
    final double perimeter = 2 * (_height + _width);
    System.out.println(perimeter);
    final double area = _height *_width;
    System.out.println(area);
    ```

### 매개변수로의 값 대입 제거 (Split Temporary Variable)
- 요약
    + 매개변수로 값을 대입하는 코드가 있을 땐, 매개변수 대신 임시변수를 사용하게 수정하자.
- 예제
    ```java
    int discount (int inputVal, int quantity, int yearToDate) {
        if (inputVal > 50) inputVal -= 2;
    }

    int discount (int inputVal, int quantity, int yearToDate) {
        int result = inputVal;
        if (inputVal > 50) result -= 2;
    }
    ```

### 메서드를 메서드 객체로 전환 (Replace Method with Method Object)
- 요약
    + 지역변수 때문에 메서드 추출을 적용할 수 없는 긴 메서드가 있을 땐, 그 메서드 자체를 객체로전환해서 모든 지역변수를 객체의 필드로 만들자. 그런 다음 그 메서드를 객체 안의 여러 메서드로 쪼개면 된다.
- 동기
    + 장황한 메서드에서 각 부분을 간결한 메서드로 빼내면 코드가 이해하기 쉬워짐.
    + 지역변수가 많은 경우, ** 임시변수를 메서드 호출로 전환**으로도 메서드 분해가 어려울 때.
    + 모든 지역변수를 메서드 객체의 속성으로 변환하고, 그 객체의 메서드 추출을 적용해서 메서드를 쪼개자.
- 예제
    ```java
    /* 변경 전 */
    class Account {
        int gamma (int inputVal, int quantity, int yearToDate) {
            int importantValue1 = (inputVal * quantity) + delta();
            int importantValue2 = (inputVal * yearToDate) + 100;
            if ((yearToDate - importantValue1) > 100) 
                importantValue2 -= 20;
            int importantValue3 = importantValue2 * 7;
            // 기타 작업
            return importantValue3 - 2 * importantValue1;
        }
    }

    /* 변경 후 */
    class Gamma {
        private final Account _account;
        private int inputVal;
        private int quantity;
        private int yearToDate;
        private int importantValue1;
        private int importantValue2;
        private int importantValue3;
        ...

        // Constructor
        Gamma (Account source, int inputValArg, int quantityArg, int yearToDateArg) {
            _account = source;
            inputVal = inputValArg;
            quantity = quantityArg;
            yearToDate = yearToDateArg;
        }

        int compute() {
            importantValue1 = (inputVal * quantity) + _account.delta();
            importantValue2 = (inputVal * yearToDate) + 100;
            importantThing();
            int importantValue3 = importantValue2 * 7;
            // 기타 작업
            return importantValue3 - 2 * importantValue1;
        }

        void importantThing() {
            if ((yearToDate - importantValue1) > 100)
                importantValue2 -= 20;
        }
    }
    ```

### 알고리즘 전환 (Substitue Algorithm)
- 요약
    + 알고리즘을 더 분명한 것으로 교체해야 할 땐, 해당 메서드의 내용을 새 알고리즘으로 바꾸자.
- 동기
    + 문제에 관해 파악하다가 더 간단한 방법이 있음을 알게 됐을 때.
- 예제
    ```java
    /* 변경 전 */
    String foundPerson(String[] people) {
        for (int i = 0; i < people.length; i++) {
            if (people[i].euqals("Don")) {
                return "Don";
            }
            if (people[i].euqals("John")) {
                return "John";
            }
            if (people[i].euqals("Kent")) {
                return "Kent";
            }
        }
    }

    /* 변경 후 */
    String foundPerson(String[] people) {
        List candidates = Array.asList(new String[] { "Don", "John", "Kent" });
        for (int i = 0; i < people.length; i++) {
            if (candidates.contains(people[i])) 
                return people[i];
        }
        return "";
    }
    ```

### 기타 
- [equals와 == 의 차이점](https://goo.gl/YdhXzG)
- [String = ""와 new Stirng("") 차이점](http://ict-nroo.tistory.com/18)
- [19+ JavaScript Shorthand Coding Techniques](https://goo.gl/RZ3x5q)
