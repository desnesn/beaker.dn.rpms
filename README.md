# **DN's downstream of Beaker RPMS**

Repo to track DN's downstream rebuilds of beaker packages.

Original src rpms:
* https://beaker-project.org/yum/server/RedHatEnterpriseLinux7/

## 1.0 BUILD ENV SETUP

~~~
# RHEL-7.9 x86_64

sudo yum -y install rpm-build
sudo yum -y install python-mock python2-devel python2-gevent112 python-docutils python-sphinx python-sphinxcontrib-httpdomain pkgconfig\(bash-completion\)
sudo yum -y install nodejs-devel nodejs-packaging nodejs-grunt-cli
sudo yum -y install python-greenlet libev
~~~

## 3.0 NODEJS-LESS RPMS BUILD
~~~
echo "%_topdir ~/beaker.dn.rpms/nodejs-less" > ~/.rpmmacros

git clone git@github.com:desnesn/beaker.dn.rpms.git ~/beaker.dn.rpms.git
cd ~/beaker.dn.rpms/nodejs-less

rpmbuild -ba SPECS/nodejs-less.spec
~~~

##### 3.1 NODEJS-LESS REPO COMMIT
~~~
git add nodejs-less/SOURCES/*patch nodejs-less/SPECS/nodejs-less.spec nodejs-less/SRPMS/*

git commit -s -m "dn: nodejs-less: build v1.7.0-2.<X>.dn"

git tag -a -m "dn: nodejs-less: tagging nodejs-less-1.7.0-2.<X>.dn.el7" nodejs-less-1.7.0-2.<X>.dn.el7
~~~
