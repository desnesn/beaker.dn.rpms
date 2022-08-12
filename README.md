# **DN's downstream of Beaker RPMS**

Repo to track DN's downstream rebuilds of beaker rpms.

Fedora COPR repository with beaker.dn binaries:
* https://copr.fedorainfracloud.org/coprs/desnesn/beaker.dn/

Quick enable:
~~~
dnf copr enable desnesn/beaker.dn
~~~

Beaker DN's code repository:
* https://github.com/desnesn/beaker.dn

Original src rpms:
* https://beaker-project.org/yum/server/RedHatEnterpriseLinux7/

The Beaker dn branch is a fork on top of upstream Beaker that add these main features:

~~~
* Enables SLES support
* Enables Ubuntu support
* Provides a few tweaks on Fedora repos
* Provides a very ugly hack to enable CentosStream8/9 from a remote repository (temporary).
~~~

Even though beaker.dn works fairly really well, as has been **extremely** helpful to our daily development by providing automatic installations and reservations, **there isn't no guarantees whatsoever of this tweaked usage**; since it didn't go through as much testing as the real upstream Beaker -> Enjoy and use it at your own risk!

Furthermore, we have mostly used only for ppc64le guests on this; hence, feel free to contribute back to this fork with patches changing the autoyasts (SLES) and (autoinstall) Ubuntu to support other arches, as well as new harness packages to support newer versions of these distros.

## 1) SLES SETUP INSTRUCTIONS

#### 1.1) SLES - SERVER SETUP

The following changes need to be performed on the config file /etc/httpd/conf.d/beaker-server.conf :
~~~
...
RewriteCond %{REQUEST_URI} !^/sles_images/.* [NC]
RewriteCond %{REQUEST_URI} !^/harness/.* [NC]
...
~~~

Moreover, we created the first versions of the ported SLES harness packages. The Beaker sys. admin needs to fetch the SUSELinuxEnterprise15_SP2.tar.gz:

~~~
$ mkdir -p /var/www/beaker/harness/SUSELinuxEnterprise15/
$ tar -xzf ./harness/rpms/SUSELinuxEnterprise15_SP2.tar.gz -C /var/www/beaker/harness/SUSELinuxEnterprise15/
~~~

And as of this moment, we are also using the same packages on SP3, SP4, thus just add links:

~~~
$ ln -s /var/www/beaker/harness/SUSELinuxEnterprise15/ /var/www/beaker/harness/SUSELinuxEnterprise15/SUSELinuxEnterprise15\ SP2
$ ln -s /var/www/beaker/harness/SUSELinuxEnterprise15/ /var/www/beaker/harness/SUSELinuxEnterprise15/SUSELinuxEnterprise15\ SP3
$ ln -s /var/www/beaker/harness/SUSELinuxEnterprise15/ /var/www/beaker/harness/SUSELinuxEnterprise15/SUSELinuxEnterprise15\ SP4
~~~

#### 1.2) SLES - CONTROLLER SETUP

SLES distros were setup on the folder /var/www/sles_images/ at the controller, thus, on the config file /etc/httpd/conf.d/beaker-lab-controller.conf :

~~~
...
# Local SLES distros
Alias /sles_images /data/www/sles_images
<Directory "/data/www/sles_images">
...
~~~

#### 1.3) SLES - DISTRO IMPORT
 
Beaker is able to import sles distro normally (just not the date) due to SLES' .treeinfo, thus, this is enough at this moment:

~~~
# Import SLES-15 SP2 on ppc64le
NAME="SLE-15-SP2-Full-GM"
RELEASE="SLE-15-SP2-Full-GM"
ARCH="ppc64le"

# wget your sles iso

mkdir -p /var/www/sles_images/$RELEASE/
mv SLE-15-SP2-Full-ppc64le-GM-Media1.iso /var/www/sles_images/$RELEASE/
sles_iso="$(ls /var/www/sles_images/$RELEASE/*iso)"

mkdir -p /tmp/$RELEASE
mount -t iso9660 -o loop $sles_iso /tmp/$RELEASE
rsync -a /tmp/$RELEASE/ /var/www/sles_images/$RELEASE
umount /tmp/$RELEASE ; rm -rf /tmp/$RELEASE

FAMILY=$(grep family /var/www/sles_images/$RELEASE/.treeinfo | cut -c10- | sed 's/ //g')
VERSION=$(grep version /var/www/sles_images/$RELEASE/.treeinfo | tail -n1 | cut -c 11-)
MAJOR=$(echo $VERSION | awk -F' ' '{print $1}')

beaker-import -n $NAME http://$(hostname --fqdn)/sles_images/$RELEASE/
~~~

If your SLES distro is non GA, change the README file (or else, the user has to press ENTER for the non GA disclaimer during install):

~~~
if [[ -f /data/www/sles_images/$RELEASE/README.BETA ]]; then
	mv /data/www/sles_images/$RELEASE/README.BETA /data/www/sles_images/$RELEASE/NON-README.BETA
fi
~~~

## 2) UBUNTU SETUP INSTRUCTIONS

#### 2.1) UBUNTU - SERVER SETUP

The following changes need to be performed on the config file /etc/httpd/conf.d/beaker-server.conf :
~~~
...
RewriteCond %{REQUEST_URI} !^/ubuntu_images/.* [NC]
RewriteCond %{REQUEST_URI} !^/autoinstall/.* [NC]
...
~~~

Also, for ubuntu install on ppc64le baremetals systems (which are booted with petiboot), the system also needs to have the company's CA crt during the install, in order to validate the URL of fetched files (such as vmlinux and initrd), thus, also on /etc/httpd/conf.d/beaker-server.conf:
~~~
...
RewriteCond %{REQUEST_URI} !^/certs/.* [NC]
...
~~~

And of course, scp your company CA to /var/www/certs/.


Moreover, we created the first versions of the ported Ubuntu 20 harness packages. The Beaker sys. admin needs to fetch the Ubuntu-Server20.tar.gz:

~~~
mkdir -p /var/www/beaker/harness/Ubuntu-Server20/
tar -xzf ./harness/debs/Ubuntu-Server20.tar.gz -C /var/www/beaker/harness/Ubuntu-Server20/
~~~

And as of this moment, we are also using the same packages on 22.04 thus just add links:

~~~
$ ln -s /var/www/beaker/harness/Ubuntu-Server20/ /var/www/beaker/harness/Ubuntu-Server22
~~~

#### 2.2) UBUNTU - CONTROLLER SETUP

Ubuntu distros were setup on the folder /var/www/ubuntu_images/ at the controller, thus, on the config file /etc/httpd/conf.d/beaker-lab-controller.conf :

~~~
...
# Local Ubuntu distros
Alias /ubuntu_images /data/www/ubuntu_images
<Directory "/data/www/ubuntu_images">
...
~~~

#### 2.3) UBUNTU - DISTRO IMPORT
 
Beaker is not able to import Ubuntu distro directly, thus we need to use a naked install:

~~~
UBU_URL="https://cdimage.ubuntu.com/releases"
#        https://cdimage.ubuntu.com/releases/20.04.4/release/ubuntu-20.04.4-live-server-ppc64el.iso
#        https://cdimage.ubuntu.com/releases/22.04/beta/ubuntu-22.04-beta-live-server-ppc64el.iso

...

# Import UBUNTU-22.04 on ppc64le
NAME="UBUNTU"
VERSION="22.04"
ARCH="ppc64le"
BASE_URL="$UBU_URL/$VERSION/release"

mkdir -p /data/www/ubuntu_images/$VERSION/
pushd /data/www/ubuntu_images/$VERSION/

if [[ ($URL =~ "beta") ]]; then
	iso_name="ubuntu-$VERSION-beta-live-server-$(echo $ARCH | sed 's/le/el/').iso"
else
	iso_name="ubuntu-$VERSION-live-server-$(echo $ARCH | sed 's/le/el/').iso"
fi

wget $BASE_URL/$iso_name

mkdir -p /tmp/$VERSION
mount -t iso9660 -o loop /data/www/ubuntu_images/$VERSION/*iso /tmp/$VERSION

# Copy everything
rsync -a /tmp/$VERSION/ /data/www/ubuntu_images/$VERSION

# Copy minimun necessary
# mkdir -p {casper,.disk}
# cp -f /tmp/$VERSION/casper/vmlinu* casper/
# cp -f /tmp/$VERSION/casper/initrd* casper/
# cp -f /tmp/$VERSION/.disk/info .disk/

umount /tmp/$VERSION ; rm -rf /tmp/$VERSION

FAMILY="Ubuntu-Server"
DATE=$(awk -F'[()]' '{print $2}' /data/www/ubuntu_images/$VERSION/.disk/info)
RELEASE=$(awk -F' ' '{print $3}' /data/www/ubuntu_images/$VERSION/.disk/info)
MAJOR="$(awk -F' ' '{print $2}' /data/www/ubuntu_images/$VERSION/.disk/info | awk -F'.' '{print $1}')"

VMLINUX=$(find /data/www/ubuntu_images/$VERSION/ -name vmlinu* -printf '%P\n')
INITRD=$(find /data/www/ubuntu_images/$VERSION/ -name initrd* -printf '%P\n')

beaker-import --family=$FAMILY --version=$VERSION --name $NAME-$VERSION-$DATE-$RELEASE --arch=$ARCH --kernel=./$VMLINUX --initrd=./$INITRD http://$(hostname --fqdn)/ubuntu_images/$VERSION/
~~~

#### 2.4) UBUNTU SYSTEM SETUP
On the system, at the install options tab, edit the kernel option and add the following options:
~~~
hostname=<FQDN> console=hvc0 ip=dhcp
~~~

## 3) FEDORA

Here, we use Fedora repos only to install, thus all installs with Fedora will have the upstream fedora repos set.

## 4) CENTOS

#### 4.1) CENTOS - SERVER SETUP

Since Red Hat didn't provide CentoStream support on upstream Beaker yet, we did an ugly hack that will be reverted when the official support comes out.

Moreover, since we only care about the latest available version of CentoStream8/CentoStream9, we import directly from the following repos:

* URL="http://mirror.centos.org/centos/8-stream"
* URL="http://mirror.stream.centos.org/9-stream"

***If you try to import a centostream distro from another repo that contains *centos* in its URL, this will probably break!***

See:
* https://github.com/desnesn/beaker.dn/commit/0e7dd615193270290cb247250934c76604454701

Also, be sure to use the same harness packages of RHEL8/RHEL9:

~~~
ln -s /var/www/beaker/harness/RedHatEnterpriseLinux8 /var/www/beaker/harness/CentOSStream8
ln -s /var/www/beaker/harness/RedHatEnterpriseLinux9 /var/www/beaker/harness/CentOSStream9
~~~

#### 4.2) BEAKER - CONTROLLER SETUP

CentoStream8:

~~~
beaker-import --ignore-missing-tree-compose -n CENTOS-8-STREAM $URL/BaseOS/ppc64le/os
beaker-import --ignore-missing-tree-compose -n CENTOS-8-STREAM $URL/BaseOS/x86_64/os
~~~

CentoStream9:

~~~
beaker-import --ignore-missing-tree-compose -n CENTOS-9-STREAM $URL/BaseOS/ppc64le/os
beaker-import --ignore-missing-tree-compose -n CENTOS-9-STREAM $URL/BaseOS/x86_64/os
~~~

## Thanks \o/

Hiper huge thanks for patches, debugging, tips and suggestions:
~~~
Andrew Lemay
Brian Jacobs
Debora Babb
Diego Pereira Domingos
Frank Novak
Gery Schneider
Gustavo Luiz Duarte
Jeff Bastian
Jeffrey Worthey
Klaus Heinrich Kiwi
Leonardo Augusto Guimaraes Garcia
Luciano Chaves
Matheus Castanho
Michael Ellerman
Murilo Opsfelder Araujo
Murilo Fossa Vicentini
Nathan Lynch
Nick Child
Paul A. Clarke
Paulo Henrique Nobrega Tavares
Renata Andrade Matos Ravanelli
Rod Groce
Tulio Magno Quites Machado Filho
~~~
