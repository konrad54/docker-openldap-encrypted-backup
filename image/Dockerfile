# Dockerfile for konrad54/openldap-encrypted-backup
FROM       osixia/openldap-backup:1.1.9

MAINTAINER Konrad Biedowicz <konrad.biedowicz@uni-greifswald.de>

COPY       slapd-backup /container/service/slapd-backup/assets/tool/slapd-backup

RUN        chmod +x /container/service/slapd-backup/assets/tool/slapd-backup