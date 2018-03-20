---
layout: post
title: "Refactoring - 메소드 정리 기법 #1"
tags: [refactoring, extract method]
---

### 리팩토링 시점
프로그램에 기능을 추가해야 하는데 코드 구조가 조잡해서 그 기능을 추가하기 힘들다면, 우선 리팩토링을 실시해서 기능을 추가하기 쉽게 만든 후 그 기능을 추가하자.

### 리팩토링 시작
리팩토링하기 전에 반드시 신뢰도 높은 테스트 스위트가 준비됐는지 확인하자, 이 테스트들은 반드시 자체검사가 되게 작성한다.

### 메서드 정리 - 메서드 추출 기법(Extract Method)
- 동기 - 메서드가 너무 길거나 코드에 주석을 달아야만의도를 이해할 수 있을 때
- 방법
    + 목적에 부합하는 새 메서드 생성, 원리가 아닌 기능을 나타내는 이름으로
    + 기존 메서드의 모든 지역변수 참조를 찾고, 새 메서드의 지역변수나 매개변수로 사용
    + 추출 코드에 의해 변경되는 지역변수 파악, 추출 코드를 메서드 호출처럼 취급  가능한지, 그 결과를 관련 변수에 대입 가능한지 파악
    + 둘 이상의 지역변수가 변경될 때 => **임시변수 분리 기법**
    + 임시변수 제거 => **메서드 호출로 전환 기법**
    + 빼낸 코드의 지역변수를 대상 메서드에 매개변수로 전달
    + 컴파일
    + 원본 메서드를 새로 생성한 메서드 호출로 수정
    + 컴파일, 테스트
- 예제
    ```java
    /* 지역변수 사용 안함 **************************/
    /* 변경 전 */
    void printOwing() {
        Enumeration e = _orders.elements();
        double outstanding = 0.0;
        
        System.out.println("****************************");
        System.out.println("********* 고객 외상 *********");
        System.out.println("****************************");
        
        while (e.hasMoreElement()) {
            Order each = (Order) e.nextElement();
            outstanding += each.getAmount();
        }

        System.out.println("고객명:" + _name);
        System.out.println("외상액:" + outstanding)
    }

    /* 변경 후 */
    void printOwing() {
        Enumeration e = _orders.elements();
        double outstanding = 0.0;
        
        printBanner();

        while (e.hasMoreElement()) {
            Order each = (Order) e.nextElement();
            outstanding += each.getAmount();
        }

        System.out.println("고객명:" + _name);
        System.out.println("외상액:" + outstanding)
    }

    void printBanner() {
        System.out.println("****************************");
        System.out.println("********* 고객 외상 *********");
        System.out.println("****************************");
    }


    /* 지역변수 사용해야 하는 경우 **************************/
    /* 변경 전 */
    void printOwing() {
        Enumeration e = _orders.elements();
        double outstanding = 0.0;
        
        printBanner();

        while (e.hasMoreElement()) {
            Order each = (Order) e.nextElement();
            outstanding += each.getAmount();
        }

        System.out.println("고객명:" + _name);
        System.out.println("외상액:" + outstanding)
    }

    /* 변경 후 */
    void printOwing() {
        Enumeration e = _orders.elements();
        double outstanding = 0.0;
        
        printBanner();

        while (e.hasMoreElement()) {
            Order each = (Order) e.nextElement();
            outstanding += each.getAmount();
        }

        printDetails(outstanding);
    }


    void printDetails(outstanding) {     
        System.out.println("고객명:" + _name);
        System.out.println("외상액:" + outstanding)
    }

    /* 지역변수를 다시 대입하는 경우 **************************/
    /* 매개변수로의 값 대입 제거 기법 */
    /* 변경 전 */
    void printOwing() {
        Enumeration e = _orders.elements();
        double outstanding = 0.0;
        
        printBanner();
        
        while (e.hasMoreElement()) {
            Order each = (Order) e.nextElement();
            outstanding += each.getAmount();
        }

        printDetails(outstanding);
    }

    /* 변경 후 */
    void printOwing() {
        printBanner();
        double outstanding = getOutstanding(); // 계산부분 추출
        printDetails(outstanding);
    }

    void getOutstanding() {
        Enumeration e = _orders.elements();
        double result = 0.0; // 변수명 변경
        
        while (e.hasMoreElement()) {
            Order each = (Order) e.nextElement();
            result += each.getAmount();
        }
        
        return result;
    }

    /* 복잡한 계산에 대비하여 이전 값을 매개변수로 전달 */
    void printOwing(double previousAmount) {
        printBanner();
        double outstanding = getOutstanding(previousAmount * 1.2);
        printDetails(outstanding);
    }

    void getOutstanding(double initialValue) {
        double result = initialValue; // 초기화
        Enumeration e = _orders.elements();
        while (e.hasMoreElement()) {
            Order each = (Order) e.nextElement();
            result += each.getAmount();
        }
    
        return result;
    }
    ```
- 다른 상황에서?
    + 변수를 두개 이상 반환하는 경우
        + 다른 부분의 코드를 빼내서 메서드를 작성하자.
    + 임시변수가 너무 많은 경우
        + **임시변수를 메서드 호출로 전환 기법** 적용
    + 메서드 추출이 힘든 경우
        + **메서드 객체로 전환 기법**




