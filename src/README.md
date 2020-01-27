# Source

## Vagrant

[Vagrantfile](Vagrantfile): Vagrant 설정

## LDAP

- [slapd1.ldif](slapd1.ldif): LDAP 1 slapd 설정
- [slapd2.ldif](slapd2.ldif): LDAP 2 slapd 설정
- [directories.ldif](directories.ldif): 디렉터리 정보
- [ldap.conf](ldap.conf): Client LDAP 설정

## Kerberos

- [krb5.conf](krb5.conf): Kerberos 서버 정보
- [kdc.conf](kdc.conf): KDC 서버 설정
- [kadm5.acl](kadm5.acl): 접근 권한 설정
- [kpropd.acl](kpropd.acl): 데이터베이스 전파 설정
- [krb5_prop](krb5_prop): xinetd 설정
- [propagator.sh](propagator.sh): 동기화 쉘 스크립트