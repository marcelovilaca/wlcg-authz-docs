# OIDC Agent 

## **Installation instructions**

### **Quick CENTOS7 installation recipe**

Current releases are available at [GitHub](https://github.com/indigo-dc/oidc-agent/releases)

This recipe shows how to quickly install ```oidc-agent``` on CENTOS 7.

```
$ VERSION=$(curl --silent "https://api.github.com/repos/indigo-dc/oidc-agent/releases/latest" | grep -Po '"tag_name": "v\K.*?(?=")') '"tag_name": "v\K.*?(?=")'

$ yum -y install epel-release

$ yum -y install https://github.com/indigo-dc/oidc-agent/releases/download/v$VERSION/oidc-agent-$VERSION-1.el7.x86_64.rpm
```

### **Bootstrapping oidc-agent**

The first thing to do is to start oidc-agent. This can be done issuing the following command:

```
$ eval $(oidc-agent)
Agent pid 62088
```


### **Install From Source** 

#### **Basic Dependencies** 

To be able to build oidc-agent, you need at least the following dependencies installed on your system

* gcc
* make
* ​[libcurl](https://curl.haxx.se/libcurl/) (libcurl4-openssl-dev)  
* ​[libsodium (>= 1.0.14)](https://download.libsodium.org/doc/) (libsodium-dev)
* ​[libmicrohttpd](https://www.gnu.org/software/libmicrohttpd/) (libmicrohttpd-dev)
* libseccomp (libseccomp-dev)
* libsecret (libsecret-1-dev)

We note that libsodium-dev might not be available by default on all systems with the required version of at least 1.0.14. It might be included in backports or has to build from source.
```
$ sudo yum install libcurl-devel libsodium-devel libsodium-static libmicrohttpd-devel libseccomp-devel libsecret-devel
```
#### **Additional Build Dependencies** 

oidc-agent can be installed easiest from package. So even when building from source it is recommended to build the package and install it.

Building the rpm package might have additional dependencies.

* help2man
* check
* debhelper
* pkg-config
* perl
* sed
* fakeroot
* devscripts 

#### **Optional Runtime Dependencies** 

Some features require additional dependencies.

* ssh-askpass (required for password prompting by the agent)

* qrencode    (required for generating an optional QR-Code when using the device flow)

#### **Download oidc-agent** 

After installing the necessary dependencies, one has to obtain a copy of the source. Possible ways are:

* clone the git repository

* download a [release version​](https://github.com/indigo-dc/oidc-agent/releases)

* download the source from [GitHub](https://github.com/indigo-dc/oidc-agent)

##### **Using git** 

```
$ git clone https://github.com/indigo-dc/oidc-agent

$ cd oidc-agent
```

##### **Using curl** 

```
$ curl -L  https://github.com/indigo-dc/oidc-agent/archive/master.tar.gz -o /tmp/oidc-agent-master.tar.gz

$ tar xzf /tmp/oidc-agent-master.tar.gz

$ cd oidc-agent
```

#### **Build and install oidc-agent** 

##### **Building a package** 

To build an rpm package and install it run the following commands inside the oidc-agent source directory:

```
$ make rpm

$ sudo yum install ../oidc-agent-<version>-1.x86_64.rpm
```

##### **Building Binaries** 

The binaries can be build with ```make```. To build and install run:

```
$ make

$ sudo make install_lib

$ sudo make install

$ sudo make post_install
```

This will:

* build the binaries
* create man pages
* install the binaries
* install the man pages
* install configuration files
* install bash completion
* install a custome scheme handler
* enable a linker to use the newly installed libraries
* update the desktop database to enable the custome scheme handler

If you want to install any of these files to another location you can pass a different path to make. E.g. ```sudo make install BIN_PATH=/home/user``` will install the binaries in ```/home/user/bin``` instead of ```/usr/bin```.

One could also run only make and manually copy the necessary files to another location and / or add the binaries' location to PATH. However, this is not recommend, because some files have to be placed at specific locations or additional configuration is needed. But it is also possible to install only a subset of all files, by calling the different ```make install_``` rules. Available targets are:

```
$ sudo make install_bin

$ sudo make install_man

$ sudo make install_conf

$ sudo make install_bash

$ sudo make install_priv

$ sudo make install_scheme_handler

$ sudo make install_xsession_script

$ sudo make install_lib

$ sudo make install_lib-dev
```


* ***To install OIDC Agent in a Debian/Ubuntu distro, please reffer to oidc-agent [instalation documentation](https://indigo-dc.gitbook.io/oidc-agent/installation/install)***

## **How to register a client** 

In order to obtain a token out of IAM, a user needs a client registered. oidc-agent can automate this step and store client credentials securely on the user machine.

A new client can be registered using the ```oidc-gen``` command, as follows:

```
$ oidc-gen -w device wlcg
```
The ```-w device``` instructs ```oidc-agent``` to use the device code flow for the authentication, which is the recommended way with IAM.

oidc-agent will display a list of different providers that can be used for registration:

```
[1] https://iam-test.indigo-datacloud.eu/
[2] https://iam.deep-hybrid-datacloud.eu/
...
[16] https://wlcg.cloud.cnaf.infn.it/
Issuer [https://wlcg.cloud.cnaf.infn.it/]:_
```

Select one of the registered providers, or type a custom issuer (for IAM, the last character of the issuer string is always a /, e.g. https://wlcg.cloud.cnaf.infn.it/).

Then oidc-agent asks for the scopes, typing max (without quotes) allows to get all the allowed scopes or one or more can be defined using spaces between them.

oidc-agent will register a new client and store the client credentials and a refresh token locally in encrypted form (the agent will ask for a password from the user).

## **How to get a token from oidc-agent** 

Tokens can be obtained using the ```oidc-token``` command, as follows:

```
oidc-token wlcg
```

This will request a token with all the scopes requested at client registration time. 

### **Limiting issued scopes** 

To limit the scopes included in the token, the -s flag can be used, as follows:

```
oidc-token -s storage.read:/ wlcg
```

In this example the scopes is being limited to ```storage.read:/```

### **Limiting token audience** 

The token audience can be limited using the --aud flag,

```
oidc-token --aud example.audience -s storage.read:/ wlcg
```

In this example the audience is being defined as ```example.audience```

* ***For more usage options please refer to ```oidc-agent --help``` or to [oidc-agent usage documentation](https://indigo-dc.gitbook.io/oidc-agent/user)***