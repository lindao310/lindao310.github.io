### Logstash

- 先安装好java

 ```
 yum install java-1.8.0-openjdk
 export JAVA_HOME=/usr/java
 
 ```
 
- 我的系统是centos， installed from package repositories using [yum](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html#package-repositories)
 
- Download and install the public signing key:

 `rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch`

- Add the following in your /etc/yum.repos.d/ directory in a file with a .repo suffix, for example logstash.repo

 ```
[logstash-5.x]
name=Elastic repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
 ```
- And your repository is ready for use. You can install it with:

 `sudo yum install logstash`