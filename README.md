# zabbix
Repositório de Templates e Scripts Zabbix.

###   TODOS COMANDOS DEVEM SER EXECUTADOS COM USUÁRIO ROOT  ###

#######   Instalação Postgres   ######### https://www.postgresql.org/download/linux/ubuntu/

echo "deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main" > /etc/apt/sources.list.d/pgdg.list
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add -
apt-get update
apt-get install postgresql-contrib-9.5

cat <<EOF > /etc/postgresql/9.5/main/pg_hba.conf
local   all             postgres                                trust
local   all             all                                     trust
host    all             all             127.0.0.1/32            trust
host    all             all             ::1/128                 trust
EOF


service postgresql reload
psql -U postgres

######## Deve conseguir acessar o console do postgres

#####################################################################

#######  Instalação Zabbix por Pacotes * https://www.zabbix.com/documentation/3.0/pt/manual/installation/install_from_packages

cd /tmp
wget http://repo.zabbix.com/zabbix/3.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.0-1+xenial_all.deb
dpkg -i zabbix-release_3.0-1+xenial_all.deb
apt-get update
apt-get install zabbix-sender zabbix-server-pgsql zabbix-proxy-sqlite3 zabbix-get zabbix-frontend-php zabbix-agent bc
apt-get install libapache2-mod-php
apt-get install php-xmlwriter php-xmlreader php-bcmath php-mbstring php-pgsql


#### ajustar php ini
vi /etc/php/7.0/apache2/php.ini 

post_max_size = 16M
max_execution_time = 300
max_input_time = 300
date.timezone = America/Sao_Paulo


service apache2 restart

## Criar usuário e base no postgres

psql -U postgres -c "CREATE USER zabbix WITH PASSWORD 'zabbix';"
psql -U postgres -c "CREATE DATABASE zabbix OWNER zabbix;"


zcat /usr/share/doc/zabbix-server-pgsql/create.sql.gz | psql -U zabbix zabbix

service zabbix-server start
service zabbix-agent start

#Acessar
http://localhost/zabbix

Admin
zabbix


### Instalar ferramentes de SNMP
apt-get install php-snmp snmp snmp-mibs-downloader snmpd snmptt



## UserParameter
UserParameter=custom.count.files[*],echo "`ls -a $1 | wc -l` - 2 " | bc
