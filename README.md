# Event Sourcing Pattern

## Docs

Microsoft Docs: [Event Sourcing Pattern](https://docs.microsoft.com/ko-kr/azure/architecture/patterns/event-sourcing)

---

## 이벤트 소싱 패턴

- 기존: 도메인에 데이터의 현재 상태만 저장
- 이벤트 소싱: 데이터를 이용한 **기록**을 **추가 전용 저장소**(append-only store)에 저장
- 추가 전용 저장소: 기록 보관소 역할
  - 이 기록으로 도메인 개체를 구체화할 수 있다.
- **장점**:
  1. 복잡했던 도메인 작업이 간단해진다.
     - 이유: 동기화 불필요. 데이터 모델 != 비즈니스 도메인
     - 결과: 확장성, 응답성 ↑
  1. 일관적인 트랜잭션 데이터: 데이터 변화 과정에 모순이 없다.
  1. 전체 감사 추적(full audit trails)과 기록이 남는다.
     - 데이터를 다시 보정할 수 있다.

### Context와 Problem

#### 기존 CRUD 데이터 프로세스

1. 현재 상태 변경 요구
1. 데이터 잠금 필요
1. 트랜잭션 사용
1. 새 값으로 수정

#### 기존 CRUD 제한 사항

- 성능, 응답 속도 저하, 확장성 제한: 데이터 저장소에서 직접 업데이트 작업을 수행해야 하기 때문.
- 업데이트 충돌: 많은 동시 사용자 → 단일 데이터 항목 공동 작업 → 충돌
- 기록 유실: 로그를 기록하는 추가적인 감사 메커니즘(auditing mechanism)이 없을 때

#### 참고

[CRUD, only when you can afford it](https://docs.microsoft.com/ko-kr/archive/blogs/maarten_mullender/crud-only-when-you-can-afford-it-revisited)

### 해결책

1. 이벤트 생성: 애플리케이션 코드 작업
   - 데이터에 대한 변경 집합
1. 이벤트 저장: 이벤트 저장소에 기록한다
1. 이벤트 게시: 구체화된 뷰(Materialized View) 업데이트 
1. 이벤트 재생: 엔티티의 현재 상태 구체화
   - 구체화된 도메인 개체가 필요할 때 이벤트 재생이 일어난다. 
   - 요청을 처리하거나 예약된 작업을 수행하면서, 프레젠테이션 레이어가 사용하고 있는 구체화된 뷰에 저장할 수 있다.

![](https://docs.microsoft.com/ko-kr/azure/architecture/patterns/_images/event-sourcing-overview.png)

#### 장점

- 이벤트는 불변이다. 
  - 이벤트를 처리하는 태스크를 분리할 수 있다: 백그라운드에서 작업이 가능하다.
  - 애플리케이션의 **성능과 확장성 ↑**
    - **트랜잭션 충돌이 일어나지 않는다.**
    - 이벤트를 추가 전용 작업으로 저장하기 때문이다.
- 이벤트는 단순한 개체일뿐이다.
  - 작업에 대한 설명만 기록한다.
  - 데이터 저장소를 직접 수정하지 않는다.
  - 나중에 적절한 때가 되면 처리하기 위해 기록될 뿐이다.
  - **구현과 관리가 간편해진다.**
- [개체 관계형 임피던스 불일치](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch) 해결:
  - RDBMS와 애플리케이션은 구조가 다르다.
- 모니터링, 디버깅, 테스트 지원
  - 변경 내용이 기록되기 때문이다.
  - 이벤트 목록으로 기타 유용한 비즈니스 정보도 얻을 수 있다.
- 이벤트와 태스크의 분리
  - 유연성, 확장성 ↑
  - **하지만, 이벤트 소싱의 이벤트는 매우 낮은 레벨이기 때문에 이벤트를 조합할 필요도 있다.**

이벤트 소싱은 일반적으로 CQRS 패턴과 결합된다.

### 이슈와 고려 사항

참고: [데이터 일관성에 대해서](https://docs.microsoft.com/en-us/previous-versions/msp-n-p/dn589800(v=pandp.10)?redirectedfrom=MSDN)

- 시스템 일치 시점:
  1. 구체화된 뷰를 만들었을 때
  1. 데이터 프로젝션을 생성했을 때
- 요청을 처리하는 사이에 새 이벤트가 추가될 수도 있다.
- 이벤트 데이터는 업데이트 하지 않는다.
  - 업데이트 대신 변경을 위한 이벤트를 추가해야한다.
- 마이그레이션 과정 중에 지속형 이벤트 형식(format of the persisted events) 변경이 필요할 때, 이미 존재하는 이벤트를 새 버전 형식에 맞추기 어려울 것이다.
  - 모든 이벤트를 순차적으로 변경하거나, 새로운 형식을 사용하는 새 이벤트를 추가해야 한다.
  - 각 버전마다 버전 스탬프를 사용해서 이전과 새로운 이벤트 형식을 유지 관리하는 것이 좋다.


