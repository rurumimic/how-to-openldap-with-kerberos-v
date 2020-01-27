# KDC 1 서버 설정

- [KDC 1 서버 설정](#kdc-1-%ec%84%9c%eb%b2%84-%ec%84%a4%ec%a0%95)
  - [KDC 설정 파일](#kdc-%ec%84%a4%ec%a0%95-%ed%8c%8c%ec%9d%bc)
    - [krb5.conf](#krb5conf)
    - [kdc.conf](#kdcconf)
  - [데이터베이스 생성](#%eb%8d%b0%ec%9d%b4%ed%84%b0%eb%b2%a0%ec%9d%b4%ec%8a%a4-%ec%83%9d%ec%84%b1)
  - [ACL 파일에 관리자 추가](#acl-%ed%8c%8c%ec%9d%bc%ec%97%90-%ea%b4%80%eb%a6%ac%ec%9e%90-%ec%b6%94%ea%b0%80)
  - [Kerberos 데이터베이스에 관리자 추가](#kerberos-%eb%8d%b0%ec%9d%b4%ed%84%b0%eb%b2%a0%ec%9d%b4%ec%8a%a4%ec%97%90-%ea%b4%80%eb%a6%ac%ec%9e%90-%ec%b6%94%ea%b0%80)
  - [Master KDC kerberos 데몬 실행](#master-kdc-kerberos-%eb%8d%b0%eb%aa%ac-%ec%8b%a4%ed%96%89)
  - [관리자 티켓 생성](#%ea%b4%80%eb%a6%ac%ec%9e%90-%ed%8b%b0%ec%bc%93-%ec%83%9d%ec%84%b1)
  - [호스트 keytabs 생성](#%ed%98%b8%ec%8a%a4%ed%8a%b8-keytabs-%ec%83%9d%ec%84%b1)

---

## KDC 설정 파일

### krb5.conf

`/etc/krb5.conf`: [source code](/src/krb5.conf)

### kdc.conf

`/var/kerberos/krb5kdc/kdc.conf`: [source code](/src/kdc.conf)

---

## 데이터베이스 생성

krb5 사용자를 관리하는 데이터베이스를 생성한다.

```bash
sudo kdb5_util create -r EXAMPLE.COM -s
```

`K/M@EXAMPLE.COM`의 비밀번호를 설정한다.

## ACL 파일에 관리자 추가

`/var/kerberos/krb5kdc/kadm5.acl`: [source code](/src/kadm5.acl)

krb5 관리자 목록에 `*/admin@EXAMPLE.COM`를 추가한다.

## Kerberos 데이터베이스에 관리자 추가

```bash
sudo kadmin.local
```

`admin/admin@EXAMPLE.COM`를 추가하고 비밀번호를 설정한다.

```bash
kadmin.local: addprinc admin/admin@EXAMPLE.COM
```

kadmin.local을 종료한다.

```bash
kadmin.local: quit
```

---

## Master KDC kerberos 데몬 실행

krb5kdc와 kadmin를 실행한다.

```bash
sudo systemctl start krb5kdc
sudo systemctl start kadmin
sudo systemctl enable krb5kdc
sudo systemctl enable kadmin
```

---

## 관리자 티켓 생성

`admin/admin@EXAMPLE.COM`의 티켓을 생성한다.

```bash
kinit admin/admin@EXAMPLE.COM
```

`admin/admin@EXAMPLE.COM`의 비밀번호를 입력한다.

## 호스트 keytabs 생성

다음 명령들을 실행하면 `/etc/krb5.keytab`라는 파일이 생성된다. 이 파일을 kdc2 서버와 공유해야 한다.

```bash
sudo kadmin -p admin/admin
```

`admin/admin@EXAMPLE.COM`의 비밀번호를 입력한다.

```bash
kadmin: addprinc -randkey host/kdc1.example.com
kadmin: addprinc -randkey host/kdc2.example.com
kadmin: ktadd host/kdc1.example.com
kadmin: ktadd host/kdc2.example.com
kadmin: quit
```