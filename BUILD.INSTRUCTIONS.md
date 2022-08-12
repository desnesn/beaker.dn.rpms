# **DN's downstream of Beaker RPMS**

Repo to track DN's downstream rebuilds of beaker packages.

Beaker DN's code repository:
* https://github.com/desnesn/beaker.dn

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

## 2.0 BEAKER RPMS BUILD
~~~
echo "%_topdir ~/beaker.dn.rpms/beaker" > ~/.rpmmacros

git clone git@github.com:desnesn/beaker.dn.rpms.git ~/beaker.dn.rpms.git
cd ~/beaker.dn.rpms/beaker

rpmbuild -ba SPECS/beaker.spec
~~~

##### 2.1 BEAKER REPO COMMIT
~~~
git add beaker/SOURCES/*patch beaker/SPECS/beaker.spec beaker/SRPMS/*

git commit -s -m "beaker: dn: build beaker-28.2-1.<X>.dn.el7"

git tag -a beaker-28.2-1.<X>.dn.el7 -m "beaker: dn: tagging beaker-28.2-1.<X>.dn.el7" <commit>
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

git commit -s -m "dn: nodejs-less: nodejs-less-1.7.0-<X>.el7"

git tag -a nodejs-less-1.7.0-<X>.el7 -m "beaker: dn: tagging nodejs-less-1.7.0-<X>.el7" <commit>
~~~
