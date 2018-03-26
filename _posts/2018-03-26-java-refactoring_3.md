---
layout: post
title: "[리팩토링] 객체 간의 기능 이동"
tags: [refactoring]
---

<!-- TOC -->

- [메서드 이동 (Move Method)](#메서드-이동-move-method)
- [필드 이동 (Move Field)](#필드-이동-move-field)
- [클래스 추출 (Extract Class)](#클래스-추출-extract-class)
- [클래스 내용 직접 삽입 (Inline Class)](#클래스-내용-직접-삽입-inline-class)
- [대리 객체 은폐 (Hide Delegate)](#대리-객체-은폐-hide-delegate)
- [과잉 중개 메서드 제거 (Remove Middle Man)](#과잉-중개-메서드-제거-remove-middle-man)

<!-- /TOC -->
### 메서드 이동 (Move Method)
- 요약
    + 메서드가 자신이 속한 클래스보다 다른 클래스의 기능을 더 많이 이용할 땐, 그 메서드가 제일 많이 이용하는 클래스 안에서 비슷한 내용의 새 메서드를 작성하자. 기존 메서드는 간단한 대리 메서드로 전환하든지 아예 삭제하자.
- 동기
    + 클래스에 기능이 많거나 다른 클래스와 과하게 연동되어 의존성이 지나칠 때.

### 필드 이동 (Move Field)
- 요약
    + 어떤 필드가 자신이 속한 클래스보다 다른 클래스에서 더 많이 사용될 때는 대상 클래스 안에 새 필드를 선언하고 그 필드 참조 부분을 전부 새 필드 참조로 수정하자.
- 동기
    + 어떤 필드가 다른 클래스의 메서드를 더 많이 참조하는 경우 그 필드를 옮기는 것을 생각하자.
    + 인터페이스에 따라 메서드를 옮기거나, 메서드의 현재 위치가 적절한 경우 필드를 옮긴다.
- 예제
    ```java
    /* 필드 캡슐화 */
    /* _interestRate 필드를 AccountType 클래스로 옮기려 할 때 */
    class Account {
        private AccountType _type;
        private double _interestRate;

        double interestForAmount_days(double amount, int days) {
            return _interestRate * amount * days / 365;
        }
    }

    class AccountType {
        private double _interestRate;

        void setInterestRate(double arg) {
            _interestRate = arg;
        }

        double getInterestRate() {
            return _interestRate;
        }
    }

    /* 필드 자체 캡슐화 */
    /* 많은 메서드가 _interestRate 할 때는 다음과 같이 필드 자체 캡슐화를 실시한다 */
     class Account {
        private AccountType _type;
        private double _interestRate;

        double interestForAmount_days(double amount, int days) {
            return getInterestRate() * amount * days / 365;
        }

        void setInterestRate(double arg) {
            _interestRate = arg;
        }

        double getInterestRate() {
            return _interestRate;
        }

        /* 추후에 AccountType으로 옮기는 경우에 새 객체 참조로만 변경하면 된다 */
        /*
        void setInterestRate(double arg) {
            type._interestRate = arg;
        }

        double getInterestRate() {
            return type._interestRate;
        }
        */
    }

    ```
### 클래스 추출 (Extract Class)
- 요약
    + 두 개의 클래스가 해야 할 일을 하나의 클래스가 하고 있는 경우, 새로운 클래스를 만들어서 관련 있는 필드와 메소드를 예전 클래스에서 새로운 클래스로 옮겨라
- 동기
    + 데이터의 부분 집합과 메소드의 부분 집합이 같이 몰려다니는 것은 별도의 클래스로 분리할 수 있다는 좋은 신호이다.
    + 보통 같이 변하거나 특별히 서로에게 의존적인 데이터의 부분 집합 또한 별도의 클래스로 분리할 수 있다는 좋은 신호이다.
- 예제
    ```java
    // 간단한 Person 클래스를 가지고 시작한다.

    class Person {
        public String getName() {
            return _name;
        }
        public String getTelephoneNumber() {
            return ("(" + _officeAreaCode + ")" + _officeNumber);
        }
        String getOfficeAreaCode() {
            return _officeAreaCode;
        }
        void setOfficeAreaCode(String arg) {
            _officeAreaCode = arg;
        }
        String getOfficeNumber() {
            return _officeNumber;
        }
        void setOfficeNumber(String arg) {
            _officeNumber = arg;
        }
        private String _name;
        private String _officeAreaCode;
        private String _officeNumber;
    }
    
    //이 경우에 전화번호와 관련된 동작을 별도의 클래스로 분리할 수 있다. TelephoneNumber클래스를 정의한다.
    class TelephoneNumber {
    }
    
    // 그리고 Person 클래스에서 TelephoneNumber 클래스에 대한 링크를 만든다.
    class Person {
        private TelephoneNumber _officeTelephone = new TelephoneNumber();
    }
    
    // 이제 필드중의 하나에 Move Field를 사용한다.
    class TelephoneNumber{
        String getAreaCode() {
            return _areaCode;
        }
        void setAreaCode(String arg) {
            _areaCode = arg;
        }
        private String _areaCode;
    }

    class Person {
        public String getTelephoneNumber() {
            return ("(" + getOfficeAreaCode() + ")" + _officeNumber);
        }
        String getOfficeAreaCode() {
            return _officeTelephone.getAreaCode();
        }
        void setOfficeAreaCode(String arg) {
            _officeTelephone.setAreaCode(arg);
        }
    }
    
    // 그런 다음 다른 필드로 옮길 수 있고 Move Method를 사용해서 메소드를 Telephone Number 클래스로 옮길 수 있다
    class Person {
        public String getName() {
            return _name;
        }
        public String getTelephoneNumber() {
            return _officeTelephone.getTelephoneNumber();
        }
        TelephoneNumber getOfficeTelephone() {
            return _officeTelephone;
        }
        private String _name;
        private TelephoneNumber _officeTelephone = new TelephoneNumber();
    }

    class TelephoneNumber {
        public String getTelephoneNumber() {
            return ("(" + _areaCode + ") " + _number);
        }
        String getAreaCode() {
            return _areaCode;
        }
        void setAreaCode(String arg) {
            _areaCode = arg;
        }
        String getNumber() {
            return _number;
        }
        void setNumber(String arg) {
            _number = arg;
        }
        private String _number;
        private String _areaCode;
    }
    ```

### 클래스 내용 직접 삽입 (Inline Class) 
- 요약
    + 클래스가 하는 일이 많지 않은 경우에는 그 클래스에 있는 모든 변수와 메소드를 다른 클래스로 옮기고 그 클래스를 제거하라.
- 동기
    + Inline Class는 Cxtract Class의 반대이다. 
    + 클래스가 더 이상 제 몫을 하지 못하고 더 이상 존재할 필요가 없다면 Inline Class를 사용한다.
- 예제
    ```java
    class Person {
        public String getName() {
            return _name;
        }
        public String getTelephoneNumber() {
            return _officeTelephone.getTelephoneNumber();
        }
        TelephoneNumber getOfficeTelephone() {
            return _officeTelephone;
        }
        private String _name;
        private TelephoneNumber _officeTelephone = new TelephoneNumber();
    }

    class TelephoneNumber {
        public String getTelephoneNumber() {
            return ("(" + _areaCode + ") " + _number);
        }
        String getAreaCode() {
            return _areaCode;
        }
        void setAreaCode(String arg) {
            _areaCode = arg;
        }
        String getNumber() {
            return _number;
        }
        void setNumber(String arg) {
            _number = arg;
        }
        private String _number;
        private String _areaCode;
    }
    
    // Person 클래스에다가 TelephoneNumber 클래스에 있는 눈에 보이는(TelephoneNumber 밖에서 볼 수 있는) 모든 메소드를 정의한다.
    class Person {
        String getAreaCode() {
            return _officeTelephone.getAreaCode();
        }
        void setAreaCode(String arg) {
            _officeTelephone.SetAreaCode(arg);
        }
        String getNumber() {
            return _officeTelephone.getNumber();
        }
        void setNumber(String arg) {
            _officeTelephone.setNumber(arg);
        }
    }
    
    // TelephoneNumber 클래스의 클라이언트를 찾아서 Person 클래스의 인터페이스를 사용하도록 바꾼다
    Person martin = new Person();
    martin.getOfficeTelephone().setAreaCode("781");
    // 는 다음과 같이 된다.
    Person martin = new Person();
    martin.setAreaCode("781");

    // 이제 TelephoneNumber 클래스에 아무것도 남지 않을 때까지 Move Method와 Move Field를 사용할 수 있다.
    ```

### 대리 객체 은폐 (Hide Delegate) 
- 요약
    + 클라이언트가 객체의 위임 클래스를 직접 호출하고 있는 경우, 서버에 메서드를 만들어서 대리객체를 숨겨라.
- 동기
    + 클라이언트가 서버 객체의 필드 중 하나에 정의된 메서드를 호출할 때, 클라이언트는 이 대리 객체에 관해 알아야 한다. 대리 객체가 변경될 때 클라이언트도 변경해야 할 가능성이 있기 때문이다. 이와 같은 경우에 서버 객체에 간단한 위임 메서드를 두어 종속성을 제거할 수 있다.
- 예제
    ```java
    /* 변경 전 */
    class Person {
        Department _department;
        public Department getDepartment() {
            return _department;
        }
        public void setDepartment(Department arg) {
            _department = arg;
        }
    }

    class Department {
        private String _chargeCode;
        private Person _manager;
        public Department (Person manager) {
            _manager = manager;
        }
        public Person getManager() {
            return _manager;
        }
    }

    manager = john.getDepartment().getManager();

    /* 변경 후 */
    class Person {
        public Person getManager() {
            return _department.getManager();
        }
    }
   
    manager = john.getManager();
    ```

### 과잉 중개 메서드 제거 (Remove Middle Man)
- 요약
    + 클래스에 자잘한 위임이 너무 많을 땐 대리 객체를 클라이언트가 직접 호출하게 하자.
- 동기
    + **대리 객체 은폐**를 사용하면 클라이언트가 대리 개체의 새 기능을 사용할 때마다 서버에 간단한 위임 메서드를 추가해야 한다. 이때는 클라이언트가 대리 객체를 직접 호출하게 해야 한다.
    + 시스템이 변할수록 얼마나 숨겨야 하는지에 대한 원칙 또한 변경된다. 
- 예제
    ```java
    /* 변경 전 */
    class Person {
        Department _department;
        public Person getManager() {
            return _department.getManager();
        }
    }

    class Department {
        private Person _manager;
        public Department (Person manager) {
            _manager = manager;
        }
    }

    manager = john.getManager();

    /* 변경 후 */
    class Person {
        public Department getDepartment() {
            return _department;
        }
    }

    manager = john.getDepartment().getManager();
    ```