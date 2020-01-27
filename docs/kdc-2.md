# KDC 2 서버 설정

- [KDC 2 서버 설정](#kdc-2-%ec%84%9c%eb%b2%84-%ec%84%a4%ec%a0%95)
  - [설정 파일 복사](#%ec%84%a4%ec%a0%95-%ed%8c%8c%ec%9d%bc-%eb%b3%b5%ec%82%ac)
  - [동기화](#%eb%8f%99%ea%b8%b0%ed%99%94)
    - [kpropd.acl 설정](#kpropdacl-%ec%84%a4%ec%a0%95)
    - [xinetd 설정](#xinetd-%ec%84%a4%ec%a0%95)
  - [Replica KDC로 데이터베이스 전달](#replica-kdc%eb%a1%9c-%eb%8d%b0%ec%9d%b4%ed%84%b0%eb%b2%a0%ec%9d%b4%ec%8a%a4-%ec%a0%84%eb%8b%ac)
  - [replica KDC 실행](#replica-kdc-%ec%8b%a4%ed%96%89)
  - [자동 동기화](#%ec%9e%90%eb%8f%99-%eb%8f%99%ea%b8%b0%ed%99%94)

---

## 설정 파일 복사

KDC 1 서버에서 KDC 2 서버로 파일을 복사한다.

```bash
sudo scp $USER@kdc1:/etc/krb5.conf /etc/krb5.conf
sudo scp $USER@kdc1:/var/kerberos/krb5kdc/kdc.conf /var/kerberos/krb5kdc/kdc.conf
sudo scp $USER@kdc1:/var/kerberos/krb5kdc/kadm5.acl /var/kerberos/krb5kdc/kadm5.acl
sudo scp $USER@kdc1:/var/kerberos/krb5kdc/.k5.EXAMPLE.COM /var/kerberos/krb5kdc/.k5.EXAMPLE.COM
sudo scp $USER@kdc1:/etc/krb5.keytab /etc/krb5.keytab
```

## 동기화

Master KDC는 kpropd 데몬을 이용해서 복제 KDC 서버로 데이터를 전파한다.

### kpropd.acl 설정

`/var/kerberos/krb5kdc/kpropd.acl`: [source code](/src/kpropd.acl)

### xinetd 설정

`/etc/xinetd.d/krb5_prop`: [source code](/src/krb5_prop)

xinetd를 실행한다.

```bash
sudo systemctl start xinetd
sudo systemctl enable xinetd
```

---

## Replica KDC로 데이터베이스 전달

KDC 1에서 다음 명령을 실행한다.

```bash
sudo kdb5_util dump /var/kerberos/krb5kdc/replica_datatrans
sudo kprop -f /var/kerberos/krb5kdc/replica_datatrans kdc2.example.com
```

KDC 2로 데이터 전파가 성공했다.

```bash
Database propagation to kdc2.example.com: SUCCEEDED
```

---

## replica KDC 실행

KDC 2에서 krb5kdc를 실행한다.

```bash
sudo systemctl start krb5kdc
sudo systemctl enable krb5kdc
```

---

## 자동 동기화

KDC 1에서 동기화 스크립트를 작성한다.

`/var/kerberos/krb5kdc/propagator.sh`: [source code](/src/propagator.sh)

실행 권한을 부여한다.

```bash
sudo chmod u+x /var/kerberos/krb5kdc/propagator.sh
```

crontab에 작업을 등록한다.

```bash
sudo crontab -e
```

매 분마다 `propagator.sh`가 실행되어, 복제 KDC 서버로 데이터가 전파된다.

```bash
* * * * * /var/kerberos/krb5kdc/propagator.sh
```