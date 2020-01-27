# OpenLDAP 서버 이중화 구축 및 TLS 보안 통신 방법

- [OpenLDAP 서버 이중화 구축 및 TLS 보안 통신 방법](#openldap-%ec%84%9c%eb%b2%84-%ec%9d%b4%ec%a4%91%ed%99%94-%ea%b5%ac%ec%b6%95-%eb%b0%8f-tls-%eb%b3%b4%ec%95%88-%ed%86%b5%ec%8b%a0-%eb%b0%a9%eb%b2%95)
  - [구성요소](#%ea%b5%ac%ec%84%b1%ec%9a%94%ec%86%8c)
  - [Servers](#servers)
  - [Vagrant](#vagrant)
  - [공통 설정](#%ea%b3%b5%ed%86%b5-%ec%84%a4%ec%a0%95)
    - [Hosts 설정](#hosts-%ec%84%a4%ec%a0%95)
  - [OpenLDAP Servers](#openldap-servers)
    - [LDAP 패키지 설치](#ldap-%ed%8c%a8%ed%82%a4%ec%a7%80-%ec%84%a4%ec%b9%98)
    - [공통 설정](#%ea%b3%b5%ed%86%b5-%ec%84%a4%ec%a0%95-1)
    - [인증서](#%ec%9d%b8%ec%a6%9d%ec%84%9c)
    - [클라이언트 설정](#%ed%81%b4%eb%9d%bc%ec%9d%b4%ec%96%b8%ed%8a%b8-%ec%84%a4%ec%a0%95)
    - [Provider 1](#provider-1)
      - [관리자 비밀번호 생성](#%ea%b4%80%eb%a6%ac%ec%9e%90-%eb%b9%84%eb%b0%80%eb%b2%88%ed%98%b8-%ec%83%9d%ec%84%b1)
      - [LDIF 작성](#ldif-%ec%9e%91%ec%84%b1)
      - [설정 적용](#%ec%84%a4%ec%a0%95-%ec%a0%81%ec%9a%a9)
    - [Provider 2](#provider-2)
      - [LDIF 작성](#ldif-%ec%9e%91%ec%84%b1-1)
      - [설정 적용](#%ec%84%a4%ec%a0%95-%ec%a0%81%ec%9a%a9-1)
    - [Delta-Syncrepl 테스트](#delta-syncrepl-%ed%85%8c%ec%8a%a4%ed%8a%b8)
  - [Kerberos Servers](#kerberos-servers)
    - [Links](#links)
    - [Kerberos 패키지 설치](#kerberos-%ed%8c%a8%ed%82%a4%ec%a7%80-%ec%84%a4%ec%b9%98)
    - [KDC 1](#kdc-1)
    - [KDC 2](#kdc-2)
  - [OpenLDAP과 Kerberos 연동](#openldap%ea%b3%bc-kerberos-%ec%97%b0%eb%8f%99)
  - [Client](#client)

---

## 구성요소

- MirrorMode
- Delta-syncrepl
- SASL GSSAPI
- Kerberos V

---

## Servers

공통 도메인(REALM)은 `EXAMPLE.COM`으로 설정한다.

| Role | Domain | IP |
|---|---|---|
| Provider | ldap1.example.com | 192.168.9.101 |
| Provider | ldap2.example.com | 192.168.9.102 |
| KDC | kdc1.example.com | 192.168.9.103 |
| KDC | kdc2.example.com | 192.168.9.104 |
| Client | client.example.com | 192.168.9.105 |

---

## Vagrant

[Vagrantfile](/src/Vagrantfile)

**서버 시작/종료**

```bash
vagrant up
vagrant halt
```

**서버 접속**

```bash
vagrant ssh ldap1 # ldap 1
vagrant ssh ldap2 # ldap 2
vagrant ssh kdc1 # kdc 1
vagrant ssh kdc2 # kdc 2
vagrant ssh client # client
```

---

## 공통 설정

### Hosts 설정

각 서버 `/etc/hosts`마다 전체 주소 도메인 네임(**FQDN**)을 등록한다. FQDN이 LDAP 설정값과 일치하지 않으면 LDAP 서버를 실행할 수 없다.

예를 들어, ldap1 서버에서 `/etc/hosts`를 고친다.

```bash
127.0.0.1        ldap1.example.com        ldap1
192.168.9.101    ldap1.example.com        ldap1
192.168.9.102    ldap2.example.com        ldap2
192.168.9.103    kdc1.example.com         kdc1
192.168.9.104    kdc2.example.com         kdc2
192.168.9.105    client.example.com       client
```

호스트 네임을 확인한다.

```bash
# 호스트 네임 확인 명령
hostname # ldap1
hostname -f # ldap1.example.com
# 호스트 네임 변경 명령
hostnamectl set-hostname ldap1
```

나머지 서버들도 `/etc/hosts`를 설정한다.

---

## OpenLDAP Servers

### LDAP 패키지 설치

LDAP 1, LDAP 2 호스트에 필요한 패키지를 설치한다.

- OpenLDAP 라이브러리, 서버, 클라이언트
- SASL 라이브러리

```bash
yum install -y openldap openldap-servers openldap-clients
yum install -y cyrus-sasl cyrus-sasl-gssapi cyrus-sasl-ldap
```

### 공통 설정

- [기존 설정 제거](https://github.com/rurumimic/how-to-openldap/blob/master/docs/common.md/#%ea%b8%b0%ec%a1%b4-%ec%84%a4%ec%a0%95-%ec%a0%9c%ea%b1%b0)
- [데이터베이스 디렉터리 생성](https://github.com/rurumimic/how-to-openldap/blob/master/docs/common.md/#%eb%8d%b0%ec%9d%b4%ed%84%b0%eb%b2%a0%ec%9d%b4%ec%8a%a4-%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%83%9d%ec%84%b1)
- [로그 설정](https://github.com/rurumimic/how-to-openldap/blob/master/docs/common.md/#%eb%a1%9c%ea%b7%b8-%ec%84%a4%ec%a0%95)
  - [rsyslog 설정](https://github.com/rurumimic/how-to-openldap/blob/master/docs/common.md/#rsyslog-%ec%84%a4%ec%a0%95)
  - [logrotate 설정](https://github.com/rurumimic/how-to-openldap/blob/master/docs/common.md/#logrotate-%ec%84%a4%ec%a0%95)

### 인증서

- [RootCA 인증서](https://github.com/rurumimic/how-to-openldap/blob/master/docs/certificates.md/#rootca-%ec%9d%b8%ec%a6%9d%ec%84%9c)
- [LDAP Provider 인증서](https://github.com/rurumimic/how-to-openldap/blob/master/docs/certificates.md/#ldap-provider-%ec%9d%b8%ec%a6%9d%ec%84%9c)
- [Replicator 인증서](https://github.com/rurumimic/how-to-openldap/blob/master/docs/certificates.md/#replicator-%ec%9d%b8%ec%a6%9d%ec%84%9c)
- [결과 파일](https://github.com/rurumimic/how-to-openldap/blob/master/docs/certificates.md/#%ea%b2%b0%ea%b3%bc-%ed%8c%8c%ec%9d%bc)
- [인증서 전달](https://github.com/rurumimic/how-to-openldap/blob/master/docs/certificates.md/#%ec%9d%b8%ec%a6%9d%ec%84%9c-%ec%a0%84%eb%8b%ac)

### 클라이언트 설정

[클라이언트 설정 방법: 3가지](https://github.com/rurumimic/how-to-openldap/blob/master/docs/client.md)

### Provider 1

#### 관리자 비밀번호 생성

Python으로 LDAP 관리자 비밀번호를 생성한다.

```bash
python -c 'import sys, crypt; print("{CRYPT}" + crypt.crypt(sys.argv[1], crypt.mksalt(crypt.METHOD_SHA256)))' "password"
```

비밀번호를 기록한다.

`{CRYPT}$5$MTyIW7Nq/1hpAbav$dzp/GM6XreRoU0m7t4iphosNt7ltyOj6Uktg3W4DbbC`

#### LDIF 작성

1. 서버 설정 LDIF: [slapd1.ldif](/src/slapd1.ldif)
   - 관리자 비밀번호를 *olcRootPW*의 값으로 넣는다.
2. 기본 디렉터리 정보 설정 LDIF: [directories.ldif](/src/directories.ldif)

#### 설정 적용

- [설정 적용](https://github.com/rurumimic/how-to-openldap/blob/master/docs/provider-1.md/#%ec%84%a4%ec%a0%95-%ec%a0%81%ec%9a%a9)
- [서버 실행](https://github.com/rurumimic/how-to-openldap/blob/master/docs/provider-1.md/#%ec%84%9c%eb%b2%84-%ec%8b%a4%ed%96%89)
- [디렉터리 정보 초기화](https://github.com/rurumimic/how-to-openldap/blob/master/docs/provider-1.md/#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ec%b4%88%ea%b8%b0%ed%99%94)
  - [디렉터리 정보 확인](https://github.com/rurumimic/how-to-openldap/blob/master/docs/provider-1.md/#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ed%99%95%ec%9d%b8)

### Provider 2

#### LDIF 작성

서버 설정 LDIF: [slapd2.ldif](/src/slapd2.ldif)

#### 설정 적용

- [설정 적용](https://github.com/rurumimic/how-to-openldap/blob/master/docs/provider-2.md/#%ec%84%a4%ec%a0%95-%ec%a0%81%ec%9a%a9)
- [서버 실행](https://github.com/rurumimic/how-to-openldap/blob/master/docs/provider-2.md/#%ec%84%9c%eb%b2%84-%ec%8b%a4%ed%96%89)
- [디렉터리 정보 확인](https://github.com/rurumimic/how-to-openldap/blob/master/docs/provider-2.md/#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ed%99%95%ec%9d%b8)

### Delta-Syncrepl 테스트

- [디렉터리 정보 추가](https://github.com/rurumimic/how-to-openldap/blob/master/docs/test-syncrepl.md/#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ec%b6%94%ea%b0%80)
- [추가 정보 확인](https://github.com/rurumimic/how-to-openldap/blob/master/docs/test-syncrepl.md/#%ec%b6%94%ea%b0%80-%ec%a0%95%eb%b3%b4-%ed%99%95%ec%9d%b8)

---

## Kerberos Servers

### Links

- [Installing KDCs](http://web.mit.edu/KERBEROS/krb5-latest/doc/admin/install_kdc.html)

### Kerberos 패키지 설치

`kdc1`, `kdc2` 호스트에 필요한 패키지를 설치한다.

- Kerberos 라이브러리, 서버, 클라이언트
- xinetd: kerberos 동기화 용도

```bash
yum install -y krb5-server krb5-workstation krb5-libs libkadm5 words
yum install -y xinetd
```

### KDC 1

- [KDC 설정 파일](kdc-1.md/#kdc-%ec%84%a4%ec%a0%95-%ed%8c%8c%ec%9d%bc)
  - [krb5.conf](kdc-1.md/#krb5conf)
  - [kdc.conf](kdc-1.md/#kdcconf)
- [데이터베이스 생성](kdc-1.md/#%eb%8d%b0%ec%9d%b4%ed%84%b0%eb%b2%a0%ec%9d%b4%ec%8a%a4-%ec%83%9d%ec%84%b1)
- [ACL 파일에 관리자 추가](kdc-1.md/#acl-%ed%8c%8c%ec%9d%bc%ec%97%90-%ea%b4%80%eb%a6%ac%ec%9e%90-%ec%b6%94%ea%b0%80)
- [Kerberos 데이터베이스에 관리자 추가](kdc-1.md/#kerberos-%eb%8d%b0%ec%9d%b4%ed%84%b0%eb%b2%a0%ec%9d%b4%ec%8a%a4%ec%97%90-%ea%b4%80%eb%a6%ac%ec%9e%90-%ec%b6%94%ea%b0%80)
- [Master KDC kerberos 데몬 실행](kdc-1.md/#master-kdc-kerberos-%eb%8d%b0%eb%aa%ac-%ec%8b%a4%ed%96%89)
- [관리자 티켓 생성](kdc-1.md/#%ea%b4%80%eb%a6%ac%ec%9e%90-%ed%8b%b0%ec%bc%93-%ec%83%9d%ec%84%b1)
- [호스트 keytabs 생성](kdc-1.md/#%ed%98%b8%ec%8a%a4%ed%8a%b8-keytabs-%ec%83%9d%ec%84%b1)

### KDC 2

- [설정 파일 복사](kdc-2.md/#%ec%84%a4%ec%a0%95-%ed%8c%8c%ec%9d%bc-%eb%b3%b5%ec%82%ac)
- [동기화](kdc-2.md/#%eb%8f%99%ea%b8%b0%ed%99%94)
  - [kpropd.acl 설정](kdc-2.md/#kpropdacl-%ec%84%a4%ec%a0%95)
  - [xinetd 설정](kdc-2.md/#xinetd-%ec%84%a4%ec%a0%95)
- [Replica KDC로 데이터베이스 전달](kdc-2.md/#replica-kdc%eb%a1%9c-%eb%8d%b0%ec%9d%b4%ed%84%b0%eb%b2%a0%ec%9d%b4%ec%8a%a4-%ec%a0%84%eb%8b%ac)
- [replica KDC 실행](kdc-2.md/#replica-kdc-%ec%8b%a4%ed%96%89)
- [자동 동기화](kdc-2.md/#%ec%9e%90%eb%8f%99-%eb%8f%99%ea%b8%b0%ed%99%94)

---

## OpenLDAP과 Kerberos 연동

- [프로토콜 확인](coupling.md/#%ed%94%84%eb%a1%9c%ed%86%a0%ec%bd%9c-%ed%99%95%ec%9d%b8)
  - [LDAPI 프로토콜 확인](coupling.md/#ldapi-%ed%94%84%eb%a1%9c%ed%86%a0%ec%bd%9c-%ed%99%95%ec%9d%b8)
  - [LDAP 프로토콜 확인](coupling.md/#ldap-%ed%94%84%eb%a1%9c%ed%86%a0%ec%bd%9c-%ed%99%95%ec%9d%b8)
- [GSSAPI 접근 실패 확인](coupling.md/#gssapi-%ec%a0%91%ea%b7%bc-%ec%8b%a4%ed%8c%a8-%ed%99%95%ec%9d%b8)
- [매커니즘 선택](coupling.md/#%eb%a7%a4%ec%bb%a4%eb%8b%88%ec%a6%98-%ec%84%a0%ed%83%9d)
- [KRB 사용자 등록](coupling.md/#krb-%ec%82%ac%ec%9a%a9%ec%9e%90-%eb%93%b1%eb%a1%9d)
- [keytab 파일 복사](coupling.md/#keytab-%ed%8c%8c%ec%9d%bc-%eb%b3%b5%ec%82%ac)
- [keytab 등록](coupling.md/#keytab-%eb%93%b1%eb%a1%9d)
- [slapd 재실행](coupling.md/#slapd-%ec%9e%ac%ec%8b%a4%ed%96%89)
- [접속 확인](coupling.md/#%ec%a0%91%ec%86%8d-%ed%99%95%ec%9d%b8)
  - [매커니즘 확인](coupling.md/#%eb%a7%a4%ec%bb%a4%eb%8b%88%ec%a6%98-%ed%99%95%ec%9d%b8)
  - [GSSAPI 확인](coupling.md/#gssapi-%ed%99%95%ec%9d%b8)

---

## Client

- [패키지 설치](client.md/#%ed%8c%a8%ed%82%a4%ec%a7%80-%ec%84%a4%ec%b9%98)
- [KRB 클라이언트 호스트 등록](client.md/#krb-%ed%81%b4%eb%9d%bc%ec%9d%b4%ec%96%b8%ed%8a%b8-%ed%98%b8%ec%8a%a4%ed%8a%b8-%eb%93%b1%eb%a1%9d)
- [매커니즘 확인](client.md/#%eb%a7%a4%ec%bb%a4%eb%8b%88%ec%a6%98-%ed%99%95%ec%9d%b8)
- [LDAP 접속 설정](client.md/#ldap-%ec%a0%91%ec%86%8d-%ec%84%a4%ec%a0%95)
- [KRB 접속 설정](client.md/#krb-%ec%a0%91%ec%86%8d-%ec%84%a4%ec%a0%95)
- [KRB 티켓 발급](client.md/#krb-%ed%8b%b0%ec%bc%93-%eb%b0%9c%ea%b8%89)
- [신원 확인](client.md/#%ec%8b%a0%ec%9b%90-%ed%99%95%ec%9d%b8)
- [GSSAPI 접속](client.md/#gssapi-%ec%a0%91%ec%86%8d)



