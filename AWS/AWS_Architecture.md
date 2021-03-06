구조와 설계
==========

## 0. 개요
- 데이터 센터 -> 가용 영역 -> 리전
  - 데이터센터가 모여 가용 영역이된다.
  - 각 리전은 두개이상의 가용 영역으로 이루어져 있다.
  - 엣지 로케이션
    - 주 목적 : 웹 서비스의 트래픽 완화와 응답속도 향상 등을 위해 설치, 보안을 강화


- 비관리형과 관리형 서비스
  - 비관리형(EC2)
    - 사용자가 확장, 내결함성 및 가용성을 관리
    - 튜닝에 자유롭다.
  - 관리형(RDS)
    - 일반적으로 확장, 내결함성 및 가용성이 서비스에 내장되어 있음
    - 튜닝에 제약사항이 크다.

## VPC
- IP주소 : CIDR 표기법을 사용(CIDR은 IP의 사용영역을 간단하게 보여줌 예:10.0.0.0 **/16**) 숫자가 올라갈수록 가용가능한 IP가 절반으로 줄어든다.

- 서브넷 : 서브넷은 CIDR범위로 이루어진 네트워크 세그먼트, 파티션을 의미
  - 서브넷을 사용하여 인터넷 접근성을 정의 : public, private
  - 권장사항 : 퍼블릭 서브넷보다 프라이빗 서브넷에 더 많은 IP를 할당한다.
  - 서브넷의 장점
    - 워크로드 배치를 간소화
    - IP낭비를 줄이고 부족한 경우를 막는다.
- VPC 트래픽 제어
  - 보안그룹을 통해 트래픽 보안
    - 모든 수신 트래픽을 필터링한다.
    - 상태 기반이라 인바운드 요청이 허용되는 경우 아웃바운드 응답이 자동으로 허용된다.
- VPC 트래픽 로깅
  - 트래픽 흐름 세부 정보를 캡처
- 여러개의 VPC 연결하는 방법
  - VPN(가상 사설 망)을 이용하여 연결 하는 방법
  - VPC 피어링 : VPC간 트래픽을 라우팅 할 수 있다.
    - IP공간은 중복 될 수 없다.
    - 프라이빗 IP주소를 사용
    - 전이적 피어링 관계는 지원X (양뱡향 통신 X ex. 노드 통신 같은 것은 안됨)
    - 게이트웨이가 필요가 없음
    - 피어링은 상호 승인이 필요
- 기본 VPC과 기본 서브넷 : 인스턴스 제작시 디폴트값을 사용하여 간편하게 만들 떄 사용, 하지만 권장하지는 않는다.


- 1. vpc 생성 (이름 및 ipv4 CIDR)
- 2. 서브넷 생성 (이름 및 vpc, 가용범위 ipv4 CIDR)
- 3. 인터넷 게이트웨이 생성
- 4. 라우팅 테이블 생성 -> 라우트(게이트웨이와 연결) -> 서브넷 어소시에이션(public 서브넷)

## 환경설정

#### 리전 선택의 기준
- 1. 데이터 주권 및 규정 준수
  - 국가 및 지역 데이터보안 법률인가?
  - 고객 데이터가 국가 외부에서 사용할 수 있는가?
  - 거버넌트 요구사항을 만족 시키는가?
- 2. 데이터에 대한 사용자 근접성
  - ex) 100밀리초 지연으로 1% 매출 감소가 발생
  - 등거리 리전시 비용 비교
- 3. 서비스 및 기능 가용성
  - 일부 서비스는 특정 리전에서만 사용 가능하다.
  - 몇가지 서비스 교차 리전을 사용할 수 있지만 지연 시간이 증가한다.
  - 서비스가 주기적으로 새 리전으로 확장된다.
- 4. 비용 효율성
  - 리전마다 서비스 비용이 다르다.

#### 가용 영역은 몇개나 사용해야 하는가?
- 권장 : 리전당 2개의 가용 영역을 시작하는게 좋다.
- 3개이상 사용시 효율성이 떨어진다.

#### VPC의 개수는 얼마가 적절한가?
- 하나의 VPC 적절한 사용사례는 제한적이며 상황에 따라 복수개의 VPC를 사용하는 것이 좋다.
- 다중 VPC는 대규모 조직에 가장 적절하다.

## 고가용성 환경 만들기
- 무엇이든 언제나 고장이 난다.
- 고가용성 횐경이란? 가동 중단시간이 최소화되도록 보장하는 것이다.
- 고가용성 요구기반
  - 복구 목표 시간(RTO)
  - 복구 목표 지점(RPO)
- 고가용성 요소
  - 내결함성 : 결함이 발생시 내성을 가지도록 해준다 -> 애플리케이션 중복성
  - 복구성 :
  - 확장성 :

## Elastic Load Balancing
- 처리해야되는 일의 밸런스를 조정하는 기능을 담당한다.
  - 인스턴스 간에 로드를 분산한다.
  - 비정상 인스턴스를 인식하고 이에 대응한다.
  - 퍼블릭 또는 내부 솔루션이 될 수 있다.
  - Amazon EC2 인스턴스에 대한 HTTP, HTTPS, SSL 및 TCP 트래픽의 라우팅과 로드 밸런싱 지원
  - 각 로드 밸런서에 퍼블릭 DNS 이름이 제공된다.

## 인프라 자동화
- 수동 프로세스 : 관리 콘솔을 통해 AWS 서비스와 리소스를 생성 및 구성하는 것
  - 문제점 : 안전성(사람의 실수에 의해)
  - 설명서가 필요하다.
- 환경 자동화의 이점 : 확장성 및 일관성, 조직의 효율성을 개선 가능
- 서버 및 기타 구성 요소를 임시 리소스라고 생각하자.
- 코드형 인트라란? 재사용, 유지관리 및 확장 및 테스트 가능한 인프라를 생성하는데 적용되는 기술
#### CloudFormation 템플릿
- 간단한 JSON 편집기 (모의 아키텍처 또는 가용되고있는 구조를 시각화 할 수 있다.)
- 코드를 이용하여 관리 가능

#### Elastic Beanstalk
- 웹 애플리케이션용 자동배포 및 조정 서비스이다.
- 자동화 가능한 기능
  - 로드 밸런싱
  - 오토 스케일링
  - 상태 모니터링
  - 애플리케이션 플랫폼 관리
  - 코드 배포

## 인프라 결합 해제
- 웹서버와 앱서버가 1:1대응 되어 있는 상태가 밀결합 되어있다고 한다. 하지만 로드 밸런서를 중간에 놓음으로서 가용성을 높힐 수 있다.

#### 느슨한 결합 전략
- 뛰어난 안전성과 효율성을 제공한다.
- 서버 지향 아키텍처(SOA) 지향

#### Amazon Simple Queue Service(SQS)
- 완전 관리형 메시지 대기열 서비스이다.
- SQS의 장점
  - 확장성
  - 안전성
  - 보안
  - 동시 읽기/쓰기
- 가시성 제한시간
  - 여러 구성 요소가 같은 메시지를 처리 하는 것을 방지한다.
  - 메시지를 수신할 때 동안 메시지가 잠금 되어 보안에 용이하다

#### 기타 결합 해제 서비스
- DynamoDB, Amazon API GateWay, Amazon Lambda

## 실습 4. 서버리스 아키텍처
