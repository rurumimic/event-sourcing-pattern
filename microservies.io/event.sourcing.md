# 패턴: 이벤트 소싱

- [Pattern: Event sourcing](https://microservices.io/patterns/data/event-sourcing.html)
- [Eventuate Event Sourcing examples](https://eventuate.io/exampleapps.html#eventuate-event-sourcing-examples)

## Context

데이터베이스를 업데이트하고 메시지나 이벤트를 전송할 때, 원자성을 지켜야 한다.  
하지만 분산 트랜잭션 환경에서는 쉽지 않다.

## Problem

어떻게 안정적이며 원자적인 데이터베이스 업데이트와 메시지, 이벤트 발행을 할 수 있을까?

## Forces

2PC로 해결하지 않는다.

### Two-phase commit protocol: 2단계 커밋 프로토콜

Wikipedia: [2PC](https://en.wikipedia.org/wiki/Two-phase_commit_protocol)

2PC는 Atomic commitment protocol(ACP)의 종류다.  
분산된 원자적 트랜잭션에 참여한 모든 프로세스를 조정하는 분산 알고리즘이다.  
일시적으로 시스템에 장애가 발생하더라도 작업을 완료해야 한다.
그러나 모든 장애 구성에 대해 탄력적이지는 않다.  
결과를 수정하기 위해 수동으로 개입할 때도 있다.  

## Solution

이벤트 소싱은 비즈니스 엔터티의 상태를 기억한다. 예를 들어, 주문이나 고객처럼 상태가 계속 바뀌는 이벤트를 저장한다.  
비즈니스 엔터티의 상태가 바뀔 때, 새로운 이벤트는 이벤트 목록에 추가된다.  
하나의 명령으로 이벤트를 저장하기 때문에, 이 작업은 당연하게 원자적이다.  
애플리케이션은 이벤트 기록을 사용해서 엔터티의 현재 상태를 알 수 있다.

이벤트 저장소는 이벤트를 저장하는 곳이다.  
이벤트 저장소는 이벤트를 더하고 검색하는 API를 제공한다.  
이벤트 저장소는 마치 메시지 브로커처럼 동작한다.  
서비스는 저장소의 API를 이용해서 이벤트를 구독할 수 있다.  
서비스가 이벤트를 저장할 때, 모든 구독자들에게도 이벤트가 전달된다.

애플리케이션이 엔터티의 현재 상태를 빠르게 알아내기 위해서,  
애플리케이션은 주기적으로 엔터티의 현재 상태 스냅샷을 만들어야 한다.  

## Example

이벤트 소싱과 CQRS를 사용한 [예제](https://github.com/eventuate-examples/eventuate-examples-java-customers-and-orders).

![](https://microservices.io/i/storingevents.png)

`주문 서비스`는 각 주문마다 현재 주문 단계가 무엇인지 저장하지 않고,  
이벤트 저장소에 주문 과정의 변화 과정을 모두 저장한다.  

`고객 서비스`는 주문 이벤트를 구독하고 자신의 상태를 수정할 수 있다.

## Resulting context

### 장점

- 이벤트 기반 아키텍처에서 상태 변화를 안정적으로 추적할 수 있다.
- 도메인 개체가 아닌 이벤트를 저장하기 때문에, [객체-관계 임피던스 불일치](https://en.wikipedia.org/wiki/Object%E2%80%93relational_impedance_mismatch) 문제를 피할 수 있다.
  - 객체-관계 임피던스 불일치: 객체 지향 언어의 객체와 관계형 데이터베이스는 1:1 연결이 어렵다.
- 비즈니스 엔터티의 변경 사항에 대한 100%로 신뢰할 수 있는 감사 로그를 제공한다.
- 언제든지 엔터티의 상태를 결정하는 임시 쿼리를 구현할 수 있다.
- 이벤트 소싱 기반 비즈니스 로직은 이벤트를 교환하는 느슨하게 결합된 비즈니스 엔터티로 구성된다. 모놀리식 애플리케이션에서 마이크로 서비스 아키텍처로 더욱 쉽게 마이그레이션 할 수 있다.

### 단점

- 학습하기 어렵다.
- CQRS가 필요하다.

## Related patterns

- [Saga](https://microservices.io/patterns/data/saga.html), [Domain event](https://microservices.io/patterns/data/domain-event.html)
- [CQRS](https://microservices.io/patterns/data/cqrs.html)
- [Audit logging](https://microservices.io/patterns/observability/audit-logging)

## See also

- [Eventuate](https://eventuate.io/): a platform for developing applications with Event Sourcing and CQRS
- [Articles about event sourcing and CQRS](https://eventuate.io/articles.html)
- [How Eventuate implements snapshots](https://blog.eventuate.io/2017/03/07/eventuate-local-now-supports-snapshots/)
