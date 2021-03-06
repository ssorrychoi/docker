# 1. 보안의 필요성

### 무엇을 어떻게 보호할 것인가?

- 기밀성 -> 외부에 보여주고 싶지 않다.
- 무결성 -> 수정할 수 없도록 하고싶다.
- 가용성 -> 멈추면 곤란하다.

이 세가지를 중심으로 생각해야 한다.

👉🏻 인터넷에서 보이는 서버 (DMZ) - 기업이나 단체가 관리하는 네트워크 중 외부에 대해 공개하는 것을 전제로 구성된 네트워크.

- 메일서버 
  - 인터넷 경유 메일은 수신하지 않는다.
  - 저장되어 있는 메일을 인터넷에서는 보이지 않게 하고 싶다.
  - 멈추지 않고 가동시키고 싶다.
- DNS 서버
  - 인터넷에서 엑세스 가능한 정보는 보여도 되지만, 그 외의 정보는 보이지 않게 하고 싶다.
  - 정보가 변조되지 않도록 하고 싶다.
  - 멈추지 않고 가동시키고 싶다.
- 웹 서버
  - 웹 콘텐츠는 인터넷에 보여도 괜찮지만, 그 외의 정보는 보이지 않게 하고 싶다.
  - 정보가 변조되지 않도록 하고 싶다.
  - 멈추지 않고 가동시키고 싶다.



# 2. 보안의 3대 요소

⭐️**기밀성 (Confidentiality)** : 허가받은 사람만 정보에 엑세스 할 수 있다.

⭐️**무결성 (Integrity)** : 정보가 본래 상태에서 변조되지 않았고 신뢰할 만한 상태

⭐️**가용성 (Availability)** : 정보에 엑세스할 수 있는 사람은 언제든지 그 정보에 엑세스 할 수 있다.



👉🏻기밀성 침해 사례 : 개인정보 유출

👉🏻무결성 침해 사례 : 웹 사이트 변조 사건과 같이 데이터가 본래와는 다른 상태로 바뀌어있음.

👉🏻가용성 침해 사례 : 웹 사이트에 대한 의도적인 엑세스가 집중되어 서비스 제공이 방해받은 상태



# 3. 보안의 추가 요소

⭐️**진정성**(Authenticity) : 서명이나 인증 등을 사용해 이용자가 맞다는 것과 데이터가 변조되지 않았다는 것. *(지문인증기술 , 해시)*

⭐️**책임성**(Accountability) : 언제 어떤일이 일어났는지를 적절히 추적 및 추구할 수 있다는 것과 그러한 로그가 변조되지 않고 시스템의 움직임을 추적하기 위한 장치가 갖춰져 있다는 것 *(로그 같은 증거를 사용)*

⭐️**부인방지**(Non-repudiation) : 발생 현상이나 작성된 데이터를 나중에 없었던 일로 하지 않도록 하는 것  *(타임스탬프, 서명 기술의 활용)*

⭐️**신뢰성**(Reliability) : 정보 시스템의 처리가 적절하며 모순 없이 작동할 수 있는 구성이라는 것 *(하드웨어의 다중화)*

