# OpenLDAP과 Kerberos 연동

- [OpenLDAP과 Kerberos 연동](#openldap%ea%b3%bc-kerberos-%ec%97%b0%eb%8f%99)
  - [프로토콜 확인](#%ed%94%84%eb%a1%9c%ed%86%a0%ec%bd%9c-%ed%99%95%ec%9d%b8)
    - [LDAPI 프로토콜 확인](#ldapi-%ed%94%84%eb%a1%9c%ed%86%a0%ec%bd%9c-%ed%99%95%ec%9d%b8)
    - [LDAP 프로토콜 확인](#ldap-%ed%94%84%eb%a1%9c%ed%86%a0%ec%bd%9c-%ed%99%95%ec%9d%b8)
  - [GSSAPI 접근 실패 확인](#gssapi-%ec%a0%91%ea%b7%bc-%ec%8b%a4%ed%8c%a8-%ed%99%95%ec%9d%b8)
  - [매커니즘 선택](#%eb%a7%a4%ec%bb%a4%eb%8b%88%ec%a6%98-%ec%84%a0%ed%83%9d)
  - [KRB 사용자 등록](#krb-%ec%82%ac%ec%9a%a9%ec%9e%90-%eb%93%b1%eb%a1%9d)
  - [keytab 파일 복사](#keytab-%ed%8c%8c%ec%9d%bc-%eb%b3%b5%ec%82%ac)
  - [keytab 등록](#keytab-%eb%93%b1%eb%a1%9d)
  - [slapd 재실행](#slapd-%ec%9e%ac%ec%8b%a4%ed%96%89)
  - [매커니즘 확인](#%eb%a7%a4%ec%bb%a4%eb%8b%88%ec%a6%98-%ed%99%95%ec%9d%b8)
    - [LDAPI 프로토콜](#ldapi-%ed%94%84%eb%a1%9c%ed%86%a0%ec%bd%9c)
    - [LDAP 프로토콜](#ldap-%ed%94%84%eb%a1%9c%ed%86%a0%ec%bd%9c)

---

## 프로토콜 확인

LDAP 1, LDAP 2 호스트에서 GSSAPI 메커니즘을 사용할 수 있는지 확인한다.

```bash
pluginviewer | grep -i gssapi
```

### LDAPI 프로토콜 확인

```bash
ldapsearch -LLL -x -H ldapi:/// -s base -b "" supportedSASLMechanisms
```

```bash
dn:
supportedSASLMechanisms: GSS-SPNEGO
supportedSASLMechanisms: GSSAPI
supportedSASLMechanisms: DIGEST-MD5
supportedSASLMechanisms: EXTERNAL
supportedSASLMechanisms: CRAM-MD5
supportedSASLMechanisms: LOGIN
supportedSASLMechanisms: PLAIN
```

### LDAP 프로토콜 확인

```bash
ldapsearch -LLL -x -H ldap://ldap1.example.com -s base -b "" supportedSASLMechanisms
```

```bash
dn:
supportedSASLMechanisms: GSS-SPNEGO
supportedSASLMechanisms: GSSAPI
supportedSASLMechanisms: DIGEST-MD5
supportedSASLMechanisms: CRAM-MD5
```

---

## GSSAPI 접근 실패 확인

아직 Kerberos 티켓이 없어 LDAP 서버에 접근할 수가 없다.

```bash
ldapsearch -LLL -Y GSSAPI -H ldap://ldap1.example.com -s base -b "" supportedSASLMechanisms
```

```bash
SASL/GSSAPI authentication started
ldap_sasl_interactive_bind_s: Local error (-2)
        additional info: SASL(-1): generic failure: GSSAPI Error: Unspecified GSS failure.  Minor code may provide more information (No Kerberos credentials available (default cache: KEYRING:persistent:1000))
```

---

## 매커니즘 선택

`/etc/sasl2/slapd.conf`를 생성한다.

```bash
mech_list: GSSAPI EXTERNAL
```

---

## KRB 사용자 등록

KDC 1 서버에서 LDAP 1, LDAP 2 호스트를 등록하여 keytab 파일을 생성한다.

```bash
sudo kadmin -p admin/admin
```

`admin/admin@EXAMPLE.COM`의 비밀번호를 입력한다.

```bash
kadmin: addprinc -randkey host/ldap1.example.com
kadmin: addprinc -randkey host/ldap2.example.com
kadmin: addprinc -randkey ldap/ldap1.example.com
kadmin: addprinc -randkey ldap/ldap2.example.com
kadmin: ktadd -k /etc/ldap.keytab host/ldap1.example.com
kadmin: ktadd -k /etc/ldap.keytab host/ldap2.example.com
kadmin: ktadd -k /etc/ldap.keytab ldap/ldap1.example.com
kadmin: ktadd -k /etc/ldap.keytab ldap/ldap2.example.com
kadmin: quit
```

## keytab 파일 복사

생성한 keytab 파일을 `ldap1`, `ldap2` 서버로 복사한다.

```bash
sudo scp /etc/ldap.keytab $USER@ldap1:/etc/openldap/ldap.keytab
sudo scp /etc/ldap.keytab $USER@ldap2:/etc/openldap/ldap.keytab
```

keytab 파일 권한을 변경한다.

```bash
sudo chmod 660 /etc/openldap/ldap.keytab
sudo chgrp ldap /etc/openldap/ldap.keytab
```

## keytab 등록

LDAP 1, LDAP 2의 slapd 설정에 keytab을 등록한다. 파일의 마지막 줄 주석을 해제한다.

`/etc/sysconfig/slapd`

```bash
KRB5_KTNAME="FILE:/etc/openldap/ldap.keytab"
```

## slapd 재실행

LDAP 1, LDAP 2의 slapd를 모두 재시작한다.

```bash
sudo systemctl restart slapd
```

---

## 매커니즘 확인

LDAP 1, LDAP 2 모두 매커니즘 목록을 확인한다.

### LDAPI 프로토콜

```bash
ldapsearch -LLL -x -H ldapi:/// -s base -b "" supportedSASLMechanisms
```

```bash
dn:
supportedSASLMechanisms: GSSAPI
supportedSASLMechanisms: EXTERNAL
```

### LDAP 프로토콜

```bash
ldapsearch -LLL -x -H ldap://ldap1.example.com -s base -b "" supportedSASLMechanisms
ldapsearch -LLL -x -H ldap://ldap2.example.com -s base -b "" supportedSASLMechanisms
```

```bash
dn:
supportedSASLMechanisms: GSSAPI
```
