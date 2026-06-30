

# INSTALLATION

setenforce 0

dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm

dnf -qy module disable postgresql

dnf install -y postgresql17-server

/usr/pgsql-17/bin/postgresql-17-setup initdb

systemctl enable postgresql-17 --now

grep 'epel' /etc/yum/repos.d # disable if found

rpm -Uvh https://repo.zabbix.com/zabbix/7.4/release/rocky/9/noarch/zabbix-release-latest-7.4.el9.noarch.rpm

dnf clean all

dnf install -y zabbix-server-pgsql zabbix-web-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent2 zabbix-web-service

cd /tmp

sudo -u postgres createuser --pwprompt zabbix    # password is zabbix

sudo -u postgres createdb -O zabbix zabbix

zcat /usr/share/zabbix/sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix

tee /etc/yum.repos.d/timescale_timescaledb.repo <<EOL
[timescale_timescaledb]
name=timescale_timescaledb
baseurl=https://packagecloud.io/timescale/timescaledb/el/$(rpm -E %{rhel})/\$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/timescale/timescaledb/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
EOL

dnf install -y timescaledb-2-postgresql-17 timescaledb-2-loader-postgresql-17

timescaledb-tune --pg-config /usr/pgsql-17/bin --max-conns=125      # 'y' for everything

systemctl restart postgresql-17.service

echo "CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;" | sudo -u postgres psql zabbix

cat /usr/share/zabbix/sql-scripts/postgresql/timescaledb/schema.sql | sudo -u zabbix psql zabbix

vi /etc/zabbix/zabbix_server.conf   # set DBPasswor, StartReportWriters=1, WebServiceURL, AllowUnsupportedDBVersions

vi /etc/nginx/conf.d/zabbix.conf    # set listen to 8080 and server_name to your server name

systemctl restart zabbix-server zabbix-web-service zabbix-agent2 nginx php-fpm

systemctl enable zabbix-server zabbix-web-service zabbix-agent2 nginx php-fpm

tail -f /var/log/zabbix/zabbix_server.log       # see that zabbix has started

http://monotrious.gaz.net.nz:8080/




# COMPONENTS


Zabbix Server


Zabbix Agent


Nginx


Postgres


PHP-FPM


Zabbix Web Service