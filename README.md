# How to OpenLDAP with Kerberos V

OpenLDAP과 Kerberos V 연동

이 레포 내용은 [How to OpenLDAP](https://github.com/rurumimic/how-to-openldap)의 심화 과정이다.  
Kerberos V 없이 OpenLDAP만 사용하고 싶다면 **How to OpenLDAP**을 확인한다.

---

## Architecture

- MirrorMode
  - 장점: 고가용성. Hot-Standby 혹은 Active-Active. 
  - 단점: 로드밸런싱 기능 필요.
- Delta-syncrepl: 변경로그 기반 복제
- SASL GSSAPI
- Kerberos V

## Documentation

- [구성요소](docs/#%ea%b5%ac%ec%84%b1%ec%9a%94%ec%86%8c)
- [Servers](docs/#servers)
- [Vagrant](docs/#vagrant)
- [공통 설정](docs/#%ea%b3%b5%ed%86%b5-%ec%84%a4%ec%a0%95)
  - [Hosts 설정](docs/#hosts-%ec%84%a4%ec%a0%95)
- [인증서](docs/#%ec%9d%b8%ec%a6%9d%ec%84%9c)
- [OpenLDAP Servers](docs/#openldap-servers)
  - [LDAP 패키지 설치](docs/#ldap-%ed%8c%a8%ed%82%a4%ec%a7%80-%ec%84%a4%ec%b9%98)
  - [클라이언트 설정](docs/#%ed%81%b4%eb%9d%bc%ec%9d%b4%ec%96%b8%ed%8a%b8-%ec%84%a4%ec%a0%95)
  - [Provider 1](docs/#provider-1)
    - [관리자 비밀번호 생성](docs/#%ea%b4%80%eb%a6%ac%ec%9e%90-%eb%b9%84%eb%b0%80%eb%b2%88%ed%98%b8-%ec%83%9d%ec%84%b1)
    - [LDIF 작성](docs/#ldif-%ec%9e%91%ec%84%b1)
    - [설정 적용](docs/#%ec%84%a4%ec%a0%95-%ec%a0%81%ec%9a%a9)
  - [Provider 2](docs/#provider-2)
    - [LDIF 작성](docs/#ldif-%ec%9e%91%ec%84%b1-1)
    - [설정 적용](docs/#%ec%84%a4%ec%a0%95-%ec%a0%81%ec%9a%a9-1)
  - [Delta-Syncrepl 테스트](docs/#delta-syncrepl-%ed%85%8c%ec%8a%a4%ed%8a%b8)
  - [slapd 로그 설정](docs/#slapd-%eb%a1%9c%ea%b7%b8-%ec%84%a4%ec%a0%95)
- [Kerberos Servers](docs/#kerberos-servers)
  - [Links](docs/#links)
  - [Kerberos 패키지 설치](docs/#kerberos-%ed%8c%a8%ed%82%a4%ec%a7%80-%ec%84%a4%ec%b9%98)
  - [KDC 1](docs/#kdc-1)
  - [KDC 2](docs/#kdc-2)
- [OpenLDAP과 Kerberos 연동](docs/#openldap%ea%b3%bc-kerberos-%ec%97%b0%eb%8f%99)
- [Client](docs/#client)

---

## Links

### OpenLDAP

- [OpenLDAP](https://www.openldap.org/): 공식 홈페이지
- [Documentation](https://www.openldap.org/doc/): 문서
  - [2.4 Administrator's Guide](https://www.openldap.org/doc/admin24/): 설치, 환경설정, 사용법 등
- [Manual Pages](https://www.openldap.org/software/man.cgi): 설정 파일 옵션, 명령어 옵션 등
- [Faq-O-Matic](http://www.openldap.org/faq/data/cache/1.html): 자주 묻는 질문
- [Mirror Repository](https://github.com/openldap/openldap): GitHub 레포지터리

### Kerberos

- [MIT Kerberos](http://web.mit.edu/KERBEROS/)
- [Documentation](http://web.mit.edu/KERBEROS/krb5-latest/doc/)
  - [For users](http://web.mit.edu/KERBEROS/krb5-latest/doc/user/index.html)
  - [For administrators](http://web.mit.edu/KERBEROS/krb5-latest/doc/admin/index.html)