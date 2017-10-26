# konrad54/docker-openldap-encrypted-backup

Licence AGPLv3

This Docker image runs both: the openldap service and a configurable cron backup.

## Prepare private & public key for encrypt and decrypt backup files

``` sh
openssl req -x509 -nodes -newkey rsa:4096 -keyout ldapbackup-secure.priv.pem -out ldapbackup-secure.pub.pem -subj '/C=a/ST=b/L=c/O=d/OU=e/CN=example.com/emailAddress=test@example.com'
docker container run --rm -v /home/user/conf/ldapbackup-secure.pub.pem:/source:ro  -v ldap_certs:/target alpine:latest cp -TR /source /target/ldapbackup-secure.pub.pem
```

## run container

``` sh
 docker run --name ldap --detach \
 -p 389:389 \
 -p 636:636 \
 -e TZ=Europe/Berlin \
 -e LDAP_ADMIN_PASSWORD=admin \
 -e LDAP_ORGANISATION=example \
 -e LDAP_DOMAIN=example.org \
 -e LDAP_TLS_KEY_FILENAME=ldap.key \
 -e LDAP_TLS_CRT_FILENAME=ldap.pem \
 -e LDAP_CRYPT_PUBLIC_KEY_FILENAME=ldapbackup-secure.pub.pem \
 -e LDAP_TLS_CA_CRT_FILENAME=ca.pem \
 -e LDAP_TLS_VERIFY_CLIENT=never \
 -e LDAP_TLS_PROTOCOL_MIN=1.2 \
 -e LDAP_TLS_CIPHER_SUITE=SECURE128:-VERS-SSL3.0:+VERS-TLS1.2 \
 -e LDAP_BACKUP_DATA_CRON_EXP="* * * * *" \
 -e LDAP_BACKUP_CONFIG_CRON_EXP="* * * * *" \
 -e LDAP_BACKUP_TTL=30 \
 -e BACKUP_FILESYSTEM_GROUPID=4000 \
 -v ldap_certs:/container/service/slapd/assets/certs \
 -v ldap_init:/container/service/slapd/assets/config/bootstrap/schema/example \
 -v ldap_data:/var/lib/ldap \
 -v ldap_conf:/etc/ldap/slapd.d \
 -v /data/openldap/backup:/data/backup \
 -v /06-load-ppolicy.ldif:/container/service/slapd/assets/config/bootstrap/ldif/06-load-ppolicy.ldif \
 -v /07-configure-ppolicy.ldif:/container/service/slapd/assets/config/bootstrap/ldif/07-configure-ppolicy.ldif \
 konrad54/openldap-encrypted-backup:1.1.9-07 \
 --copy-service --loglevel info
```


## Backup location

- Container -> /data/backup
- Docker-Host -> /data/openldap/backup

## Restore LDAP 

1. decrypt LDAP backup 

``` sh
cd /data/openldap/backup
docker container run --rm \
-e backupfile=abc-data.gz.enc \
-v /data/openldap/backup:/data/backup \
-v /directory-of-private-key/ldapbackup-secure.priv.pem:/ldapbackup-secure.priv.pem:ro \
konrad54/openssl-decrypt-file:1.0.1
```

2. start restore with encrypted backup file

``` sh
 docker exec -it ldap slapd-restore-data abc-data.gz 
```

3. delete decrypted LDAP backup

``` sh
sudo rm abc-data.gz 
```

## How to build image

``` sh
mkdir /tmp/openldap-encrypted-backup
cd /tmp/openldap-encrypted-backup
vi Dockerfile
vi slapd-backup

docker build -t konrad54/openldap-encrypted-backup:1.1.9-07 .
``` 
