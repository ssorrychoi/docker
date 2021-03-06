# CH.2-9 원타임 비밀번호

- 한 번만 사용 가능한 비밀번호
- 생성된 비밀번호는 한 번 이용되면 더 이상 사용할 수 없다.
- *장점* : 이용자는 비밀번호를 돌려 사용할 수 없다는 것 , 보다 투명도가 높은 인증 시스템을 구축 할 수 있다.
- *단점* : 서비스 제공측(인증하는 측)에 원타임 비밀번호를 해석하기 위한 장치나 시스템을 도입해야 한다.



# CH.2-10 이중 인증

- 성질이 다른 두 종류의 정보를 조합하여 인증을 하는 인증 방식을 말한다. Ex) '비밀번호'+'인증코드'
- *장점* : 한 쪽 정보가 유출되어도 보안이 확보된다는 점.
- *단점* : 이중 인증을 구현할 시스템에 비용을 들여 구축해야 한다.
- 공격자가 한 쪽의 정보를 입수한 후에 그 정보를 사용하여 또 다른 한쪽의 정보를 입수할 수 있게 되는 경우는 이중 인증이라고 할 수 없다.



# CH.2-11 싱글 사인온

- **싱글 사인온(Single Sigh-On : SSO)** : 한 번의 사인온(인증)으로 여러 시스템을 이용할 수 있도록 하기 위한 장치.
- *장점* : 인증 정보를 관리하는 시스템을 일원화 할 수 있기 때문에 관리성이 향상된다.
- *단점* : 그 성질상 인증 정보나 인증 시스템의 보안 확보가 보통의 시스템 이상으로 중요해진다.
- 표준화를 위한 장치 => SAML, OpenID, OAuth 등이 있다.



# CH.2-12 전자서명과 그 응용 예

- **전자서명** : 전자 데이터의 작성자를 원래의 데이터에 부여하여 원래의 데이터가 변조되지 않았다는 것을 보증하기 위한 기술이다.
- 전자서명을 구현하는 세가지 알고리즘
  - **키 생성 알고리즘** : '서명하기 위한 키(비밀키)'와 '서명을 검증하기 위한 키(공개키)'를 생성하는 알고리즘이다.
  - **서명 알고리즘** : 서명자가 생성한 비밀키를 사용하여 전자 데이터에 대응하는 서명 데이터를 생성하는 알고리즘이다.
  - **검증 알고리즘** : 전자 데이터와 대응하는 서명 데이터가 주어졌을 떄 다른 사람이 서명자의 공개키를 사용하여 대응이 적절한 것인지를 확인하는 알고리즘.
- 전자서명을 구현하기 위한 기술 : 'RSA', 'ElGamal', 'DSA'등이 있다.
- 'PGP'나 'S/MIME'는 주로 전자메일에 대해 전자서명이나 공개키 암호 처리를 하는 장치이다.