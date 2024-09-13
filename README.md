# 내 어플리케이션 특성에 적합한 ec2 인스턴스는 무엇일까?

## 개요 
실제 신한투자증권의 운영 환경에서의 대규모 데이터 처리 작업이나, 웹 서버 운영 시 인스턴스를 선택할 때 보다 기술적으로 결정을 정확하게 내리고자 진행했습니다.

## ⚒️ OS
- **운영 체제**: Ubuntu

## ⚙️ 아키텍처
- OS와 스토리지는 모든 인스턴스에서 동일하게 설정

## 사용 인스턴스 

### 1. t3.small (x86, 범용 인스턴스)
- **아키텍처**: x86 (Intel/AMD 기반)
- **용도**: 가벼운 웹 애플리케이션, 개발 및 테스트 환경 등
- **장점**: 저렴한 요금으로 기본적인 범용 작업을 수행할 수 있으며, 일시적인 성능 향상이 필요한 경우 유용합니다.

### 2. t4g.small (ARM, 범용 인스턴스)
- **아키텍처**: ARM (AWS Graviton2 프로세서)
- **용도**: 웹 서버, 테스트 환경 등
- **장점**: ARM 기반으로 에너지 효율성이 높으며, 가격 대비 성능이 우수합니다. t3.small보다 높은 성능을 제공하면서도 비용이 더 낮습니다.

### 3. t4g.large (ARM, 범용 인스턴스)
- **아키텍처**: ARM (AWS Graviton2 프로세서)
- **용도**: 중간 규모의 웹 애플리케이션, 데이터 처리 등
- **장점**: 더 많은 메모리와 높은 성능을 제공하며, 비용 효율성이 뛰어납니다. 메모리와 CPU 성능이 향상되어 보다 복잡한 작업을 처리할 수 있습니다.

### 4. t3.large (x86, 범용 인스턴스)
- **아키텍처**: x86 (Intel/AMD 기반)
- **용도**: 중간 규모의 웹 애플리케이션, 데이터베이스 서버 등
- **장점**: 높은 메모리 용량과 안정적인 성능을 제공하며, 다양한 범용 작업에 적합합니다. 비용이 t4g.large보다 약간 더 높습니다.

### 5. m5.large (x86, 범용 인스턴스)
- **아키텍처**: x86 (Intel/AMD 기반)
- **용도**: CPU 집약적인 작업, 데이터 분석, 머신러닝 등
- **장점**: t3.large와 비슷한 메모리 용량을 가지지만, 더 높은 CPU 성능과 네트워크 성능을 제공하여 성능이 중요한 작업에 적합합니다. 요금이 가장 비쌉니다.

## 분석 주제
- **x86,ARM 아키텍처 차이에 따른 성능 비교**

- **인스턴스 크기 차이에 따른 성능 비교**

- **범용 인스턴스 종류 별 비교**

## 테스트 항목

1. **크롤링 작업 처리 속도**
   - 네이버 증권에서 다량 현재 주가 크롤링  

2. **메모리 할당 속도**
   - Numpy 배열에 10억 개의 float 값을 할당 

3. **디스크 I/O 처리 속도**
   - 대량의 문자열을 파일로 저장 (대략 2억 개)

4. **API 요청 처리 속도**
   - 다수의 사용자 깃허브 프로필 조회 API 요청

  
 
## 🔎 아키텍처 차이에 따른 성능 차이
 ## t3.small ❓ t4g.small 
 | 특징       | t3.small | t4g.small  |
|------------|------------------------------|--------------------------------|
| 아키텍처    | x86 (Intel/AMD 기반)          | ARM (AWS Graviton2 프로세서)   |
| 메모리     | 2 GiB                        | 2 GiB                          |
| vCPU       | 2 코어                       | 2 코어                         |
| 요금       | $0.026 / 시간                | $0.0496 / 시간                 |
| 용도       | 범용 인스턴스  | 범용 인스턴스|
| 네트워크 성능 | 최대 5 Gbps (EBS 최적화 포함)   | 최대 5 Gbps (EBS 최적화 포함)   |

<img src="https://github.com/user-attachments/assets/5efbb9cd-8bac-4c94-b0d1-3590b1d2823d" alt="Benchmark" width="800px" />

## 📈결과 분석

ARM 기반 아키텍처를 사용하는 t4g와 인텔의 x86 아키텍처를 사용하는 t3 인스턴스를 비교한 결과, t4g가 t3보다 모든 테스트에서 더 높은 성능을 보였습니다. 두 인스턴스 모두 2개의 vCPU와 2 GiB의 메모리, 5기가비트 네트워크 성능을 갖추고 있지만, 성능 차이는 아키텍처의 차이에서 기인할 가능성이 높다고 생각했습니다. 

x86보다 왜 ARM이 성능이 좋을까? 생각해보았습니다.
- ARM 프로세서는 작업을 여러 코어에 분산시켜 동시에 처리할 수 있는 스케줄링 기술을 갖추고 있습니다. 
- ARM 프로세서는 네트워크 성능을 최적화하기 위해 설계된 네트워크 스택을 사용합니다. 이는 높은 네트워크 대역폭을 지원하며, 많은 데이터 전송이 필요한 작업에서 성능을 극대화합니다.
- ARM 프로세서는 RISC(Reduced Instruction Set Computer) 아키텍처를 사용합니다. RISC 아키텍처는 간단하고 효율적인 명령어 집합을 제공하여, 각 명령어가 수행하는 작업이 단순하고 빠릅니다. 

따라서, ARM 기반 프로세서는 멀티코어 성능을 극대화하고 높은 네트워크 대역폭을 지원하여, 크롤링 작업에서 매우 중요한 요소인 속도와 효율성을 높이는 데 유리하다는 점을 알게 되었습니다.

## 🔎 인스턴스 크기에 따른 성능 차이 비교 
 ##  t4g.small ❓ t4g.large 
| 특징         | t4g.small                        | t4g.large                        |
|--------------|----------------------------------|----------------------------------|
| 아키텍처      | ARM (AWS Graviton2 프로세서)     | ARM (AWS Graviton2 프로세서)     |
| 메모리       | 2 GiB                            | 8 GiB                            |
| vCPU         | 2 코어                           | 4 코어                           |
| 요금         | $0.0496 / 시간                   | $0.0992 / 시간                   |
| 용도         | 범용 인스턴스                    | 범용 인스턴스                    |
| 네트워크 성능 | 최대 5 Gbps (EBS 최적화 포함)   | 최대 5 Gbps (EBS 최적화 포함)   |

<img src="https://github.com/user-attachments/assets/ba807302-bf57-4c66-830a-d2bd085dfd48" alt="Benchmark" width="800px" />


## 📈결과 분석
- 인스턴스의 크기가 클수록 메모리 처리 속도, 파일 쓰기 속도에서 큰 성능 차이를 보였습니다. 하지만 크롤링 작업에서는 메모리 크기가 크게 영향을 미치지 않았습니다. 이는 **크롤링 작업**이 주로 **네트워크 대역폭**과 **CPU 성능**에 의존하기 때문입니다.
- 메모리가 클수록, 많은 사이트를 크롤링할 때 **수집한 데이터를 메모리에 저장**할 수 있어 성능이 향상됩니다. 반면 메모리가 작을 경우, **메모리 부족**으로 인해 **디스크 스왑**이 발생해 성능 저하가 생길 수 있습니다.


## 🔎 범용 인스턴스 T,M 비교 
 ##  t3.large  ❓  m5.large
| 특징         | t3.large                         | m5.large                         |
|--------------|----------------------------------|----------------------------------|
| 아키텍처      | x86 (Intel/AMD 기반)              | x86 (Intel/AMD 기반)              |
| 메모리       | 8 GiB                            | 8 GiB                            |
| vCPU         | 4 코어                           | 2 코어                           |
| 요금         | $0.1 / 시간                      | $0.118 / 시간                    |
| 용도         | 범용 인스턴스                    | 범용 인스턴스                    |
| 네트워크 성능 | 최대 5 Gbps (EBS 최적화 포함)   | 최대 10 Gbps (EBS 최적화 포함)  |

<img src="https://github.com/user-attachments/assets/332eef30-261a-4f5e-9ae7-f7416c6e291d" alt="Benchmark" width="800px" />


## 📈결과 분석
- 네트워크 대역폭이 t3.large는 5 Gbps이고 m5.large는 10 Gbps인 영향으로 크롤링 속도에서 차이를 보이며, 균형 잡힌 cpu 성능을 제공하는 M이  모든 테스트에서  더 나은 성능을 보였습니다
- 하지만 chatgpt에 따르면 T 시리즈는 비용 효율성이 뛰어나고, 버스트 가능한 성능을 제공하여 일정하지 않은 CPU 사용량을 가진 워크로드에 적합하다고 합니다. 웹 서버나 API 서버와 같은 가벼운 작업에 유리하며, 필요할 때 일시적으로 성능을 높여 트래픽 변동에 대응할 수 있다고합니다.

## 최종 결론 
- 크롤링 작업: t4g.small 또는 t4g.large (ARM 기반, 멀티코어 성능과 네트워크 대역폭이 중요한 경우) 
- 메모리 및 디스크 처리 작업: t4g.large (더 많은 메모리와 CPU 자원이 필요한 경우)
- 범용 인스턴스: m5.large (네트워크 대역폭이 필요한 API 작업이 필요한 경우)

