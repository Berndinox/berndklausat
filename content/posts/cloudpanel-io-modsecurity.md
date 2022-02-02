---
title: "Adding ModSecurity to CloudPanel.io"
date: 2020-10-16T13:55:49+01:00
draft: false
tags: [Server,CloudPanel,ModSecurity,Security]
---
## CloudPanel.io
CloudPanel beschreibt sich selbst wie folgt:

*CloudPanel is a modern server control panel with lightweight components for PHP applications with specific features for all major clouds. It has been developed with more than ten years of Magento hosting experience in the AWS cloud. With CloudPanel, you can run your favorite PHP application in any cloud or dedicated server within minutes.*

**Homepage:** https://www.cloudpanel.io

Im unterschied zur Konkurrenz wie Beispielsweise CyberPanel, Plesk oder VestaCP konzentriet sich CloudPanel auf das wesentliche. Die Oberfläche ist dermaßen aufgeräumt, fast schon klinisch rein. So mag ich das!

Damit Ihr einen eindruck davon bekommt, hier die Willkommens-Seite:  

![CloudPanel Interface](/images/cloudpanel-dashboard.png)

Der Gesamteindruck ist wirklich hervorragend. Alles smooth, keine Popus oder zusätzliche Fenster, welchen einem anspringen und PhpMyAdmin ist mit Auto-Logon versehen.

Ein Feature vermisse ich jedoch: **ModSecurity**.

Zwar ist ein IP/Bot Block-Mechanismus bereits implementiert, mit den Features von ModSecurity kann das jedoch nicht ganz mithalten.

## ModSecurity
ModSecurity ist eine vollwertige WAF, welche sich als Modul in Nginx integrieren lässt. Ursprünglich für Apache geschrieben, über die Jahre wurde das Projekt jedoch ständig erweitert. Heute gibt es für fast jeden bekannten WebServer ein derartiges Modul.

ModSecurity’s Intro startet wie folgt:

*ModSecurity is a toolkit for real-time web application monitoring, logging, and access control. I like to think about it as an enabler: there are no hard rules telling you what to do; instead, it is up to you to choose your own path through the available features. That’s why the title of this section asks what ModSecurity can do, not what it does.*

![ModSecurity Logo](/images/modsecurity-logo.png)

## CoreRuleSet
Dabei handelt es sich um eine Erweiterung für ModSecurity welche eine fertige Regelsammlung enthält. Damit kann eine Vielzahl an unterschiedlichen Attacken ohne “großen Aufwand” abgefangen werden. Das Regelset lässt sich dabei Modulr erweitern und verfeiner.

Weitere Informationen zum Projekt: https://coreruleset.org

## Installation
### CloudPanel

Verbindet euch via SSH auf eure Debian 10 VM (Ubuntu und CentOS werden nicht untersützt, CentOS support soll jedoch folgen). Installation von CloudPanel:

{{< highlight bash>}}
apt update && apt -y upgrade && apt -y install curl wget sudo
curl -sSL https://installer.cloudpanel.io/ce/v1/install.sh | sudo bash
{{< / highlight >}}

Je nach Cloud-Provider bietet CloudPanel unterschiedliche Installer bzw. fertige Images: https://www.cloudpanel.io/docs/cloudpanel-ce/getting-started

Im aktuellen Fall wird das Paket auf einem “nicht unterstützten” Cloud-Provider installiert.

### Requirements
{{< highlight bash>}}
apt-get install libtool autoconf build-essential libpcre3-dev zlib1g-dev libssl-dev libxml2-dev libgeoip-dev liblmdb-dev libyajl-dev libcurl4-openssl-dev libpcre++-dev pkgconf libxslt1-dev libgd-dev automake
{{< / highlight >}}

### Download ModSecurity
{{< highlight bash>}}
cd /opt/
# The correct command for downloading the latest Libmodsecurity source code
git clone --depth 100 -b v3/master --single-branch https://github.com/SpiderLabs/ModSecurity
cd ModSecurity
git submodule init
git submodule update
{{< / highlight >}}

### Compile ModSecurity
{{< highlight bash>}}
# Generate configure file
sh build.sh
# Pre compilation step. Checks for dependencies
./configure
# Compiles the source code
make
# grab a Coffee! That's going to take its time.
# Installs the Libmodsecurity to **/usr/local/modsecurity/lib/libmodsecurity.so**
make install
{{< / highlight >}}

### Compile the nginx connector
{{< highlight bash>}}
cd /opt/
# Fetch the Nginx source code. See the directory: **http://nginx.org/download/** for all versions of Nginx
wget http://nginx.org/download/nginx-1.18.0.tar.gz
# Extract the downloaded source code
gunzip nginx-1.18.0.tar.gz
tar -xvf nginx-1.18.0.tar
rm nginx-1.18.0.tar
# Fetch the source code for ModSecurity-nginx connector
git clone https://github.com/SpiderLabs/ModSecurity-nginx
# Change directory to the Nginx source code
cd nginx-1.18.0/
# Compile the Nginx 
./configure --with-cc-opt='-g -O2 -fdebug-prefix-map=/home/clp/packaging/nginx/tmp/nginx-1.18.0=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -fPIC' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --with-http_addition_module --with-http_geoip_module=dynamic --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_xslt_module=dynamic --with-stream=dynamic --with-stream_ssl_module --with-stream_ssl_preread_module --with-mail=dynamic --with-mail_ssl_module  --add-dynamic-module=/opt/ModSecurity-nginx
# Generate the module
make modules
# Copy the module to the Nginx module directory
cp objs/ngx_http_modsecurity_module.so /usr/share/nginx/modules
{{< / highlight >}}

### Download CoreRuleSet
{{< highlight bash>}}
# Create the modsec folder in the Nginx configuration folder. It will contain the ModSec rules
mkdir /etc/cloudpanel/services/clp-nginx/modsec
# Change directory
cd /etc/cloudpanel/services/clp-nginx/modsec
# Download the rules
git clone https://github.com/SpiderLabs/owasp-modsecurity-crs.git
# Rename the default ModSecurity rules configuration file
mv /etc/cloudpanel/services/clp-nginx/modsec/owasp-modsecurity-crs/crs-setup.conf.example /etc/cloudpanel/services/clp-nginx/modsec/owasp-modsecurity-crs/crs-setup.conf
{{< / highlight >}}

### Configure the CoreRuleSet & ModSecurity
{{< highlight bash>}}
# Copy the default ModSecurity configuration file to the modesec folder
cp /opt/ModSecurity/modsecurity.conf-recommended /etc/cloudpanel/services/clp-nginx/modsec/modsecurity.conf
# Create a configuration file that will be loaded by Nginx. This file will load the ModSec rules configuration file and the ModSec configuration file
nano /etc/cloudpanel/services/clp-nginx/modsec/main.conf
# Add following lines to the file
Include /etc/cloudpanel/services/clp-nginx/modsec/modsecurity.conf
Include /etc/cloudpanel/services/clp-nginx/modsec/owasp-modsecurity-crs/crs-setup.conf
SecRuleRemoveById 933100
SecRuleRemoveById 933120
Include /etc/cloudpanel/services/clp-nginx/modsec/owasp-modsecurity-crs/rules/*.conf
{{< / highlight >}}

### Configure Nginx
{{< highlight bash>}}
nano /etc/cloudpanel/services/clp-nginx/modules-enabled/50-mod-http-modsecurity.conf
# Add the following lin
load_module modules/ngx_http_modsecurity_module.so;
# Edit next file
nano /etc/cloudpanel/services/clp-nginx/nginx.conf
# Add the following line to the file, after: worker_rlimit_nofile 8192 - should be Line 5
include /etc/cloudpanel/services/clp-nginx/modules-enabled/*.conf;
## Enable for the CloudPanel
nano /etc/cloudpanel/services/clp-nginx/sites-enabled/cloudpanel.conf
# Add following lines to the file, after:   server_name _ - should be Line 3&4
modsecurity on;
modsecurity_rules_file /etc/cloudpanel/services/clp-nginx/modsec/main.conf;
{{< / highlight >}}

### Enable & Verify
{{< highlight bash>}}
chown -R clp-admin /etc/cloudpanel/services/clp-nginx/modsec/
chgrp -R clp /etc/cloudpanel/services/clp-nginx/modsec/
chown clp-admin /etc/cloudpanel/services/clp-nginx/nginx.conf
chgrp clp /etc/cloudpanel/services/clp-nginx/nginx.conf
# Restart Nginx
systemctl restart clp-nginx
# If you see an error that mentions missing **unicode.mapping** file, then copy the file to the mod sec folder
cp /opt/ModSecurity/unicode.mapping /etc/cloudpanel/services/clp-nginx/modsec/
# Check Process with
systemctl status clp-nginx
# If the process did not start check:
cat /var/log/syslog
{{< / highlight >}}

### Enable CoreRuleSet
Per Default CoreRuleSet is enable in DetectOnly Mode. Fully enable it:
{{< highlight bash>}}
nano /etc/cloudpanel/services/clp-nginx/modsec/modsecurity.conf
## Change DetectionOnly to On
#SecRuleEngine DetectionOnly
SecRuleEngine On
{{< / highlight >}}

### Test ModSecurity
Fire off your Shellshock:
{{< highlight bash>}}
curl https://YOURIP:8443/login --insecure -H "custom:() { ignored; }; echo Content-Type: text/html; echo ; /bin/cat /etc/passwd"
{{< / highlight >}}

Should response:
{{< highlight bash>}}
<head><title>403 Forbidden</title></head>
{{< / highlight >}}

### Finish
Super, wir haben nun ModSecurity und CoreRuleSet auf unserem Server installiert und den vorhandenen Stack damit erweitert. Zugegeben, die notwendige Teile selbst zu compilieren ist nicht trivial, aufgrund der erhöhten Sicherheit zahlt es sich aber allemal aus!

## Attentione
Um “False Positives” vorzubeugen solltet ihr auf das Control-Panel über einen Hostname zugreifen und nicht via IP-Addresse.

## Quellen
https://pakjiddat.netlify.app/posts/integrating-modsecurity-with-nginx-on-debian-9
https://samhobbs.co.uk/2016/03/getting-started-apache-modsecurity-debian-and-ubuntu
https://www.cloudpanel.io/docs