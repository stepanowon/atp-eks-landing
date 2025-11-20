# EKS: Cluster AutoScaler vs Karpenter 비교

## 1. Cluster AutoScaler (CA)

### 📌 특징
- Kubernetes 공식 오토스케일러
- Auto Scaling Group(ASG) 기반으로 동작
- Pod이 스케줄링 실패하면 ASG Desired Capacity 증가
- 노드 축소 가능하지만 보수적
- EKS뿐 아니라 Kubernetes 어디든 사용 가능

### 📌 처리 흐름
  * 새로운 Pod가 추가되는 상황 발생
  * Scheduler에 의해 기존에 존재하는 Worker node에 새로운 Pod를 할당 시도
  * 기존 Worker node에 리소스 부족 때문에 Scheduling하지 못하고 Pod는 Pending 상태가 됨
  * Cluster Autoscaler는 Pending 상태의 Pod를 감지
  * 감지결과를 바탕으로 ASG의 Desired Capacity 값 증가시킴
  * ASG의 증가된 desired 값에 의해 새로운 Worker node가 생성
  * Scheduler에 의해 새로 생성된 Worker node에 Pending 상태인 Pod가 배포

---

### 👍 장점
#### 1) 안정성 높음
- 오래 사용되어 검증된 방식

#### 2) ASG 기반 조직에 적합
- 기존 AWS 인프라 구성과 자연스럽게 연동

#### 3) 세부 제어 가능
- ASG와 관련된 낮은 수준 제어 가능

---

### 👎 단점
#### 1) 느린 확장 속도
- ASG 변경 후 EC2가 뜰 때까지 시간이 걸림

#### 2) 노드 활용 효율 떨어질 수 있음
- 고정 타입의 노드에서 다양한 Pod 처리 시 비효율 발생

#### 3) 운영 복잡
- ASG, Launch Template 관리 필요

---

## 2. Karpenter

### 📌 특징
- AWS가 제작한 고급 노드 프로비저너(2021~)
- ASG 없이 EC2 직접 생성
- Pending Pod 리소스 요청 기반으로 최적 인스턴스 자동 선택

### 📌 처리 흐름
  * 새로운 Pod가 추가되는 상황 발생
  * Scheduler에 의해 기존에 존재하는 Worker node에 새로운 Pod를 할당 시도
  * 기존 Worker node에 리소스 부족 때문에 Scheduling하지 못하고 Pod는 Pending 상태가 됨
  * Karpenter는 Pending 상태의 Pod를 감지하고 Pod의 상태과 갯수를 파악하고 생성할 Worker Node의 적정 용량을 계산함.
  * 계산된 적정 용량을 수용할 수 있는 Worker Node의 인스턴스 타입을 결정하고 Worker Node를 생성함.
  * Scheduler에 의해 새로 생성된 Worker node에 Pending 상태인 Pod가 배포된다.
  * 
---

### 👍 장점
#### 1) 매우 빠른 확장
- Pod Pending 발생 → 즉시 EC2 생성

#### 2) 비용 최적화 강력
- Spot 자동 선택
- 다양한 인스턴스 타입 중 가장 효율적 인스턴스 선택

#### 3) 정책 기반 유연 운영
- GPU, 특정 AZ, 특정 AMI 등 다양한 조건 적용 가능

#### 4) ASG가 없어 단순함
- 노드 그룹 관리 부담 줄어듦

---

### 👎 단점
#### 1) 비교적 신기술
- CA에 비해 성숙도 낮음

#### 2) 설치·구성 난이도 높음
- IAM, Provisioner 설정, 권한 설정 등 초기 러닝커브 존재


---

## 3. 기능 비교

| 항목 | Cluster AutoScaler | Karpenter |
|---|---|---|
| 아키텍처 | ASG 기반 | EC2 직접 생성 |
| 확장 속도 | 느림 | 매우 빠름 |
| 비용 최적화 | 제한적 | 자동화된 비용 최적화 지원 |
| 설정 난이도 | 낮음 | 다소 높음 |
| 안정성 | 매우 높음 | 빠르게 발전 중 |
| 관리 포인트 | ASG 중심 | Provisioner 중심 |
| Pod 기반 배치 효율 | 일반적 | 매우 효율적 |
| AWS 종속성 | 낮음 | 높음 |


## 4. Karpenter를 이용한 ondemand + spot 인스턴스 배포
https://malwareanalysis.tistory.com/740


