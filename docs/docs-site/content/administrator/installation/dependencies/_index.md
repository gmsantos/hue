---
title: "Dependencies"
date: 2019-03-13T18:28:09-07:00
draft: false
weight: -1
---

You will need Python, some OS packages, a Database and Node.js.

## Python

Hue employs some Python modules which use native code and requires certain development libraries be installed on your system. To install from the tarball, you'll need these library development packages and tools installed on your system:

Versions supported:
* Python 2.7
* Python 3.6+
```
# If you are using Python 3.6+, set PYTHON_VER before the build, like
export PYTHON_VER=python3.8
```

## Database

Hue is being ran the most with [MySQL InnoDB, PostgreSQL or Oracle](https://docs.cloudera.com/documentation/enterprise/latest/topics/hue_dbs_0.html).

### MySQL / MariaDB

The client lib [MySQL-python](https://github.com/cloudera/hue/tree/master/desktop/core/ext-py/MySQL-python-1.2.5) is already included so it should work out of the box.

**Note**

With the recent OSs like Ubuntu 20.04 or using Python 3, the MySQL-python lib won't compile properly and will produce an error similar to:

    _mysql.c:44:10: fatal error: my_config.h: No such file or directory
      44 | #include "my_config.h"
          |          ^~~~~~~~~~~~~

The lib folder needs to be swapped with https://pypi.org/project/mysqlclient/. Unfortunately for licensing reason (GPL vs Apache) this can't be officially done in the Hue repository.

Remove and add the compatible MySql lib with these two commands:

    git cherry-pick 7a9100d4a7f38eaef7bd4bd7c715ac1f24a969a8
    git cherry-pick e67c1105b85b815346758ef1b9cd714dd91d7ea3

    git clean -fdx

    make apps

Read [HUE-9390](https://issues.cloudera.org/browse/HUE-9390) for more details.

### PostgreSQL

You would need the OS `python-psycopg2` or Python [psycopg2](https://pypi.org/project/psycopg2/).

### Oracle

How to install the client lib.

#### Version 11

Download both instantclient-basic and instantclient-sdk of the same version (11.2.0.4.0 for this example) and on your ~/.bash_profile, add

    export ORACLE_HOME=/usr/local/share/oracle
    export VERSION=11.2.0.4.0
    export ARCH=x86_64
    export DYLD_LIBRARY_PATH=$ORACLE_HOME
    export LD_LIBRARY_PATH=$ORACLE_HOME

and then

    source ~/.bash_profile
    sudo mkdir -p $ORACLE_HOME
    sudo chmod 775 $ORACLE_HOME

then unzip the content of both downloaded zip files into the newly created $ORACLE_HOME in a way that the 'sdk' folder is at the same level with the other files and then

    ln -s libclntsh.dylib.11.1 libclntsh.dylib
    ln -s libocci.dylib.11.1 libocci.dylib

and finally

    cd sdk
    unzip ottclasses.zip

#### Version 12

Hue comes with an older Oracle client cx_Oracle-5.2.1 Python module so it will fail. We need to do the above client install and then upgrade Hue's client module to at least cx_Oracle-5.3.

Make sure you have the `python-dev` package dependencies, ensure that ORACLE_HOME and LB_LIBRARY_PATH are properly set so that pip knows which version to install, then:

    echo $ORACLE_HOME $LD_LIBRARY_PATH

    ./build/env/bin/pip install cx_Oracle

Tip: You can also wget the proper cx_Oracle file yourself: https://pypi.python.org/pypi/cx_Oracle/.

Tip: Going to where is the Oracle client, e.g. /usr/local/share/oracle then creating a symlink similar to below could even trick the cx_Oracle-5.2.1 module to work with 12.2 without doing the pip upgrade:

    ln -s libclntsh.so.12.2 libclntsh.so.11.1

There is more details on this [Apply Temporary Workaround for Oracle 12 Client](https://docs.cloudera.com/documentation/enterprise/latest/topics/hue_dbs_oracle_pkg.html#concept_qx3_hfw_4z).


#### Mac

* [Oracle Instant Client](https://www.oracle.com/database/technologies/instant-client/downloads.html)

## OS Packages

### Ubuntu

    sudo apt-get install git ant gcc g++ libffi-dev libkrb5-dev libmysqlclient-dev libsasl2-dev libsasl2-modules-gssapi-mit libsqlite3-dev libssl-dev libxml2-dev libxslt-dev make maven libldap2-dev python-dev python-setuptools libgmp3-dev

With Python 3, also:

    sudo apt-get install python3.8-dev python3-distutils


### CentOS/RHEL

    sudo yum install ant asciidoc cyrus-sasl-devel cyrus-sasl-gssapi cyrus-sasl-plain gcc gcc-c++ krb5-devel libffi-devel libxml2-devel libxslt-devel make mysql mysql-devel openldap-devel python-devel sqlite-devel gmp-devel

* mvn (from [``apache-maven``](https://gist.github.com/sebsto/19b99f1fa1f32cae5d00) package or maven3 tarball)
* libtidy (for unit tests only)
* openssl-devel (for version 7+)

### SLES 12

    zypper in ant asciidoc cyrus-sasl-devel cyrus-sasl-gssapi cyrus-sasl-plain gcc gcc-c++ krb5-devel libffi-devel libxml2-devel libxslt-devel make mysql mysql-devel openldap2-devel python-devel sqlite-devel gmp-devel

Tip: if you run into building kerberos extension issue and see message `krb5-config: command not found`, install and append it to environment variable PATH.

    rpm -ql krb5-devel | grep krb5-config ## find the path of krb5-config
    export PATH=$PATH:/usr/lib/mit/bin

### MacOS

* Xcode command line tools
* [Homebrew](https://brew.sh)

For a step-by-step guide with Python 3, Big Sur and M1-based macs see section below.

Install Dependencies via Homebrew

    brew install mysql@5.7 maven gmp openssl libffi

Install Xcode command line tools

    sudo xcode-select --install

Fix openssl errors (required for MacOS 10.11+)

    export LDFLAGS=-L/usr/local/opt/openssl/lib && export CPPFLAGS=-I/usr/local/opt/openssl/include

If `runserver` stops abruptly with a  `"zsh: abort"` message

    export DYLD_FALLBACK_LIBRARY_PATH=/usr/local/opt/openssl/lib

If you are getting `Could not find Python.h` message

    export SKIP_PYTHONDEV_CHECK=true

On macOS 10.15+, install an older version of openssl

    brew uninstall --ignore-dependencies openssl && brew install https://github.com/tebelorg/Tump/releases/download/v1.0.0/openssl.rb

Fix the possible missing Python headers message by installing the MacOS SDK headers

On macOS 10.14.x

    open /Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg

On macOS 10.15.x

    sudo ln -s /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/* /usr/local/include/

### macOS Big Sur on M1 or Intel with Python 3

This is a verified step-by-step guide on how to get up and running on a fresh installation of macOS Big Sur, tested on both M1 and Intel based MacBook Pro.

1. Clone the Hue repo

    `git clone https://github.com/cloudera/hue.git`

2. Install Brew if not already installed

   `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

3. Install postgres (unless you prefer something else)

   `brew install postgres`

4. Start Postgres

   `brew services start postgresql`

5. Create the hue database and set permissions

   ```
   createdb hue_d --lc-collate='en_US.UTF-8' -T "template0"
   psql -s hue_d
   hue_d=# create user hue_u with password 'huepassword';
   hue_d=# grant all privileges on database hue_d to hue_u;
   hue_d=# \q
   ```

6. Install gmp, openssl and libffi (Note some might be installed from postgres)

   `brew install gmp openssl libffi`

7. Install python, tested with the "macOS 64-bit installer" 3.7.9 direct download from python.org.
   - For M1 Macs 3.7.9 from python.org is the only one that works and none of the homebrew version work at the time of writing.

   - For Intel Macs, other 3.x Python versions as well as Homebrew might work.

   `https://www.python.org/ftp/python/3.7.9/python-3.7.9-macosx10.9.pkg`

8. Install mysql client

   `brew install mysql`

9. Install node and npm (unless already present)

   `brew install node`

10. In the cloned hue folder run:

    ```
    export PYTHON_VER=python3.7
    export SKIP_PYTHONDEV_CHECK=true
    export CFLAGS="-I$(xcrun --show-sdk-path)/usr/include/sasl"
    export LDFLAGS="-L/usr/local/opt/libffi/lib -L/usr/local/opt/openssl@1.1/lib"
    export CPPFLAGS="-L/usr/local/opt/libffi/lib/include -L/usr/local/opt/openssl@1.1/lib/include"

    make apps
    ```

11. Configure Hue to your liking, edit

    `desktop/conf/pseudo-distributed.ini`

    For postgres according to step 5 above set:

    ```
    [[database]]
      engine=postgresql_psycopg2
      host=localhost
      port=5432
      user=hue_u
      password=huepassword
      name=hue_d
    ```

12. If you use postgres as configured in the previous step you'll need to add the psycopg2 lib manually

    `./build/env/bin/pip install psycopg2-binary`

12. Start Hue (note that export from step 10 above needs to be active)

    `./build/env/bin/hue runserver 0.0.0.0:8888`


## NodeJs

Version 10+ is needed.

e.g. how to install on Ubuntu:

    curl -sL https://deb.nodesource.com/setup_14.x | sudo bash -
    sudo apt-get install -y nodejs

## Installing Python 2.7

### CentOS 6.8 / 6.9 OS

Check your OS Version:

    cat /etc/redhat-release

Make sure "/etc/redhat-release" contains "CentOS 6.8 or 6.9" version. These instructions are tested on CentOS 6.8 and 6.9 versions only. It may or may not work on previous CentOS 6 series OS.

    yum install -y centos-release-SCL
    yum install -y scl-utils
    yum install -y python27

### RedHat 6.8 / 6.9 OS

Check your OS Version

    cat /etc/redhat-release

Make sure `/etc/redhat-release` contains "RedHat 6.8 or 6.9" version. These instructions are tested on RedHat 6.8 and 6.9 versions only. It may or may not work on previous RedHat 6 series OS.

    wget http://mirror.infra.cloudera.com/centos/6/extras/x86_64/Packages/centos-release-scl-rh-2-3.el6.centos.noarch.rpm
    rpm -ivh centos-release-scl-rh-2-3.el6.centos.noarch.rpm
    yum install -y scl-utils
    yum install -y python27

### Oracle 6.8 / 6.9 OS

Check your OS Version

    cat /etc/redhat-release

Make sure `/etc/redhat-release` contains "Oracle 6.8 or 6.9" version. These instructions are tested on Oracle 6.8 and 6.9 versions only. It may or may not work on previous Oracle 6 series OS.

    wget -O /etc/yum.repos.d/public-yum-ol6.repo http://yum.oracle.com/public-yum-ol6.repo

Set the value of the enabled parameter for the software_collections repository to 1: for file `/etc/yum.repos.d/public-yum-ol6.repo`

    [ol6_software_collections]
    name=Software Collection Library release 3.0 packages for Oracle Linux 6 (x86_64)
    baseurl=http://yum.oracle.com/repo/OracleLinux/OL6/SoftwareCollections/x86_64/
    gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-oracle
    gpgcheck=1
    enabled=1

for more details, refer to this link: [https://docs.oracle.com/cd/E37670_01/E59096/html/section_e3v_nbl_cr.html](https://docs.oracle.com/cd/E37670_01/E59096/html/section_e3v_nbl_cr.html)

    yum install -y scl-utils
    yum install -y python27

## Java

Optional.

Java is only for the [JDBC proxy](/developer/connectors/#jdbc) connector which is not built automatically anymore.

**Install Oracle JDK**

On Ubuntu 16.04 or less only:

    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update
    sudo apt-get install oracle-java8-installer

On Centos:

* [Oracle JDK](https://www.digitalocean.com/community/tutorials/how-to-install-java-on-centos-and-fedora)

On Mac:

    brew cask install adoptopenjdk

## Supported Browsers

The two latest LTS versions of each browsers:

* Edge
* Safari
* Chrome
* Firefox
