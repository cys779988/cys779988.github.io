---
title:  "SOLID 원칙"
excerpt: DesignPattern
categories:
  - DesignPattern
---

## SRP(Single Responsibility Principle), 단일 책임 원칙
- 객체는 단 하나의 책임만 가져야 한다는 설계원칙
- SRP에 따른 설계를 하면 응집도는 높게, 결합도는 낮게 설계
- 여러 객체들이 하나의 책임만 갖도록 잘 분배하여 시스템에 변화가 생기더라도 그 영향을 최소화

<img src="https://cys779988.github.io/assets/img/DesignPattern(1).PNG">

## OCP(Open-Closed Principle), 개방-폐쇄 원칙
- 기존 코드를 변경하지 않으면서(closed), 기능을 추가할 수 있도록(open) 설계 되어야 한다는 설계원칙
- 확장에 대해서는 개방적이고, 수정에 대해서는 폐쇄적
- 캡슐화를 통해 여러 객체에서 사용하는 같은 기능을 인터페이스에 정의

<img src="https://cys779988.github.io/assets/img/DesignPattern(2).PNG">

## LSP(Liskov Substitution Principle), 리스코프 치환 원칙
- 자식 클래스는 최소한 자신의 부모 클래스에서 가능한 행위는 수행할 수 있어야 한다는 설계원칙
- 즉, 자식 클래스는 언제나 부모 클래스의 역할을 대체할 수 있어야 한다는 것, 부모 클래스와 자식 클래스의 행위가 일관됨을 의미
- 자식 클래스가 부모 클래스를 대체하기 위해서는 부모의 기능에 대해 오버라이드 되지 않도록 함
- 즉, 자식 클래스는 부모 클래스의 책임을 무시하거나 재정의하지 않고 확장만 수행하도록 함

## ISP(Interface Segregation Principle), 인터페이스 분리 원칙
- 자신이 사용하지 않는 인터페이스는 구현하지 말아야 한다는 설계원칙
- 하나의 거대한 인터페이스 보다는 여러 개의 구체적인 인터페이스가 낫다는 것을 의미
- SRP는 객체의 단일 책임을 뜻한다면, ISP는 인터페이스의 단일 책임

<img src="https://cys779988.github.io/assets/img/DesignPattern(3).PNG">

- Phone 인터페이스에 call(), sms(), alarm(), calculator() 함수를 모두 정의하는 것보다 Call, Sms, Alarm, Calculator 인터페이스를 각각 정의하여 OldPhone과 SmartPhone에서 4개의 인터페이스를 구현하도록 설계
- 각 인터페이스의 메서드들이 서로 영향을 미치지 않게 됨
- 자신이 사용하지 않는 메서드에 대해서 영향력을 줄어듦

## DIP(Dependency Inversion Principle), 의존 역전 원칙
- 객체들의 서로 정보를 주고 받을 때 의존 관계가 형성되는데, 이 때 객체들은 나름대로의 원칙을 갖고 정보를 주고 받아야 한다는 설계원칙
- 나름대로의 원칙이란, 추상성이 낮은 클래스보다 추상성이 높은 클래스와 의존 관계를 맺어야 한다는 것을 의미
- 일반적으로 인터페이스를 활용하면 이 원칙을 준수할 수 있음(캡슐화)

<img src="https://cys779988.github.io/assets/img/DesignPattern(4).PNG">

- Client 객체는 Cat, Dog, Bird의 crying() 메서드에 직접 접근하지 않고, Animal 인터페이스의 crying() 메서드를 호출함으로써 DIP를 만족시킴
