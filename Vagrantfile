# -*- mode: ruby -*-
# vi: set ft=ruby :

$kibana_td_agent_conf = <<CONFIG
<source>
  type tail
  format apache
  path /var/log/httpd/access_log
  pos_file /var/log/td-agent/kibana-apache-access.pos
  tag kibana.apache.access
</source>

<match kibana.apache.access>
  type elasticsearch
  include_tag_key true
  tag_key @log_name
  host localhost
  port 9200
  logstash_format true
  flush_interval 5s
</match>
CONFIG

$kibana_script = <<SCRIPT
rm -f /etc/sysconfig/clock
echo "ZONE=\"Asia/Hong_Kong\"" > /etc/sysconfig/clock
rm -f /etc/localtime
ln -s /usr/share/zoneinfo/Asia/Hong_Kong /etc/localtime
rpm --import http://packages.treasuredata.com/GPG-KEY-td-agent

rm -f /etc/sysconfig/iptables
cat >/etc/sysconfig/iptables <<'EOF';
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 9200 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 9300 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
EOF

cat >/etc/yum.repos.d/td.repo <<'EOF';
[treasuredata]
name=TreasureData
baseurl=http://packages.treasuredata.com/2/redhat/\$releasever/\$basearch
gpgcheck=1
gpgkey=http://packages.treasuredata.com/GPG-KEY-td-agent
EOF

yum -y update
yum -y install wget vim policycoreutils-python td-agent httpd java-1.7.0-openjdk libcurl-devel make
wget -bqc https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.4.2.noarch.rpm
wget -bqc https://download.elasticsearch.org/kibana/kibana/kibana-3.1.0.tar.gz
chmod 755 /var/log/httpd -R
while true; do echo "0f2513b0a060973497210ce150656df6290a3163  elasticsearch-1.4.2.noarch.rpm" | sha1sum -c - && break; sleep 10; done
rpm -Uvh elasticsearch-1.4.2.noarch.rpm
chkconfig elasticsearch on
/usr/share/elasticsearch/bin/plugin -s -i elasticsearch/marvel/latest
cat >/etc/elasticsearch/elasticsearch.yml <<'EOF';
http.cors.enabled: true
http.cors.allow-origin: /http(s)?:\/\/localhost(:[0-9]+)?/
EOF
while true; do echo "effc20c83c0cb8d5e844d2634bd1854a1858bc43  kibana-3.1.0.tar.gz" | sha1sum -c - && break; sleep 10; done
tar zxvf kibana-3.1.0.tar.gz
mv kibana-3.1.0 /var/www/html/kibana
sed -i 's|elasticsearch: "http://"+window.location.hostname+":9200",|elasticsearch: "http://localhost:9200",|' /var/www/html/kibana/config.js
chown apache:apache -R /var/www/html
chcon -R -t httpd_sys_content_t /var/www/html
/usr/sbin/td-agent-gem install --no-ri --no-rdoc fluent-plugin-elasticsearch
echo "#{$kibana_td_agent_conf}" >> /etc/td-agent/td-agent.conf
#semanage port -a -t http_port_t -p tcp 9200
setenforce 0
chkconfig httpd on
chkconfig elasticsearch on
chkconfig td-agent on
service iptables restart
service httpd restart
service elasticsearch start
service td-agent restart
SCRIPT

$httpd_td_agent_conf = <<CONFIG
<source>
  type tail
  format apache
  path /var/log/httpd/access_log
  pos_file /var/log/td-agent/kibana-apache-access.pos
  tag kibana.apache.access
</source>
<match kibana.apache.access>
  type elasticsearch
  include_tag_key true
  tag_key @log_name
  host 192.168.30.100
  port 9200
  logstash_format true
  flush_interval 5s
</match>
CONFIG

$httpd_script = <<SCRIPT
rm -f /etc/sysconfig/clock
echo "ZONE=\"Asia/Hong_Kong\"" > /etc/sysconfig/clock
rm -f /etc/localtime
ln -s /usr/share/zoneinfo/Asia/Hong_Kong /etc/localtime
rpm --import http://packages.treasuredata.com/GPG-KEY-td-agent

rm -f /etc/sysconfig/iptables
cat >/etc/sysconfig/iptables <<'EOF';
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 9200 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 9300 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
EOF

cat >/etc/yum.repos.d/td.repo <<'EOF';
[treasuredata]
name=TreasureData
baseurl=http://packages.treasuredata.com/2/redhat/\$releasever/\$basearch
gpgcheck=1
gpgkey=http://packages.treasuredata.com/GPG-KEY-td-agent
EOF

yum -y update
yum -y install wget vim policycoreutils-python td-agent httpd java-1.7.0-openjdk libcurl-devel make
chmod 755 /var/log/httpd -R
chown apache:apache -R /var/www/html
chcon -R -t httpd_sys_content_t /var/www/html
/usr/sbin/td-agent-gem install --no-ri --no-rdoc fluent-plugin-elasticsearch
echo "#{$httpd_td_agent_conf}" >> /etc/td-agent/td-agent.conf
#semanage port -a -t http_port_t -p tcp 9200
setenforce 0
chkconfig httpd on
chkconfig td-agent on
service iptables restart
service httpd restart
service td-agent restart
SCRIPT

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.define "kibana" do |kibana|
    kibana.vm.hostname = "5min-kibana-centos65"
    kibana.vm.box = "Centos6.5"

    kibana.vm.provision "shell", inline: $kibana_script

    kibana.vm.network :private_network, :ip => "192.168.30.100"
    kibana.vm.network :forwarded_port, guest: 80, host: 8080
    kibana.vm.network :forwarded_port, guest: 9200, host: 9200
    kibana.vm.network :forwarded_port, guest: 9300, host: 9300

    kibana.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "1024"]
    end
  end
  config.vm.define "httpd" do |httpd|
    httpd.vm.hostname = "httpd"
    httpd.vm.box = "Centos6.5"

    httpd.vm.provision "shell", inline: $httpd_script

    httpd.vm.network :private_network, :ip => "192.168.30.101"
    httpd.vm.network :forwarded_port, guest:80, host:8000

    httpd.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "1024"]
    end
  end
end