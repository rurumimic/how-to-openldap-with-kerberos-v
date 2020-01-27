# Client

- [Client](#client)
  - [패키지 설치](#%ed%8c%a8%ed%82%a4%ec%a7%80-%ec%84%a4%ec%b9%98)
  - [KRB 클라이언트 호스트 등록](#krb-%ed%81%b4%eb%9d%bc%ec%9d%b4%ec%96%b8%ed%8a%b8-%ed%98%b8%ec%8a%a4%ed%8a%b8-%eb%93%b1%eb%a1%9d)
  - [매커니즘 확인](#%eb%a7%a4%ec%bb%a4%eb%8b%88%ec%a6%98-%ed%99%95%ec%9d%b8)
  - [LDAP 접속 설정](#ldap-%ec%a0%91%ec%86%8d-%ec%84%a4%ec%a0%95)
  - [KRB 접속 설정](#krb-%ec%a0%91%ec%86%8d-%ec%84%a4%ec%a0%95)
  - [KRB 티켓 발급](#krb-%ed%8b%b0%ec%bc%93-%eb%b0%9c%ea%b8%89)
  - [신원 확인](#%ec%8b%a0%ec%9b%90-%ed%99%95%ec%9d%b8)
  - [GSSAPI 접속](#gssapi-%ec%a0%91%ec%86%8d)

---

## 패키지 설치

`client` 호스트에 필요한 패키지를 설치한다.

- OpenLDAP 라이브러리, 클라이언트
- Kerberos 라이브러리, 클라이언트
- SASL 라이브러리

```bash
yum install -y openldap openldap-clients
yum install -y krb5-workstation krb5-libs libkadm5
yum install -y cyrus-sasl cyrus-sasl-gssapi cyrus-sasl-ldap cyrus-sasl-md5 cyrus-sasl-plain
```

---

## KRB 클라이언트 호스트 등록

```bash
sudo kadmin -p admin/admin
```

## 매커니즘 확인

GSSAPI 메커니즘을 사용할 수 있는지 확인한다.

```bash
pluginviewer | grep -i gssapi
```

## LDAP 접속 설정

`/etc/openldap/ldap.conf`: [source code](/src/ldap.conf)

## KRB 접속 설정

Client 호스트에도 KDC 1, KDC 2와 같은 설정을 사용한다.

`/etc/krb5.conf`: [source code](/src/krb5.conf)

## KRB 티켓 발급

클라이언트에서 티켓을 받는다.

```bash
kinit host/client.example.com
```

`host/client.example.com@EXAMPLE.COM`의 비밀번호를 입력한다.

티켓을 확인한다.

```bash
klist
```

## 신원 확인

LDAP 사용자의 신원을 확인한다.

```bash
ldapwhoami -Z
```

```bash
SASL/GSSAPI authentication started
SASL username: host/client.example.com@EXAMPLE.COM
SASL SSF: 256
SASL data security layer installed.
dn:cn=host/client.example.com@example.com,ou=people,dc=example,dc=com
```

## GSSAPI 접속

클라이언트에서 GSSAPI를 사용해서 LDAP 서버에 접근한다.

먼저 매커니즘을 확인한다.

```bash
ldapsearch -LLL -Y GSSAPI -H ldap://ldap1.example.com -s base -b "" supportedSASLMechanisms
# supportedSASLMechanisms: GSSAPI
```

전체 엔트리 정보를 검색한다.

```bash
ldapsearch -LLL -Y GSSAPI -b "dc=example,dc=com" -Z;
```