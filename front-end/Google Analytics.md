## Environment Preparation
### Python 2.7
Default python for CentOS 6 is python 2.6
Default python for CentOS 7 is python 2.7
Update CentOS 6 to python2.7 failed

### Node.js (v6.0.0+)
#### Installation:
##### 1. src: make install
##### 2. epel + yum install nodejs
> yum install epel-release
yum repolist

Possible error:
Cannot Retrieve Metalink for Repository: EPEL

Solution:
update ca-certificates package, this helps anything that uses SSL certificates.
> yum --disablerepo=epel -y update ca-certificates

##### 3. Official way
> curl -sL https://rpm.nodesource.com/setup_6.x | sudo -E bash -
yum install -y nodejs

[This failed on Centos 7, root cause unknown]

### Google Cloud SDK (v132.0.0+)
#### 1. Requiring dev_appserver.py in your PATH
> ./google-cloud-sdk/bin

#### 2. Install and Init
> ./google-cloud-sdk/install.sh
./google-cloud-sdk/bin/gcloud init

This may require proxy if network is blocked.
If proxy is unavailable, it will give you a link to get verification code directly from browser(which also require VPN).

## Building
### Clone
> git clone https://github.com/googleanalytics/ga-dev-tools.git
cd ga-dev-tools

## Install dependencies
> pip install -r requirements.txt -t python_modules
npm install

Error: src/MD2.c:31:20: fatal error: Python.h: No such file or directory      #include "Python.h"

Solution:
> yum install python-devel

## Run
> npm start

Report a error, perhaps due to missing images...?


Server ip: 10.200.157.84