# Introduction

Your box is highly customizable and designed to be personalized by editing
your ``Cakebox.yaml`` configuration file. This way you can always
(re)create exact copies of your box including installed applications,
databases and virtual hosts.

**Notes:**

+ use spaced indentation or your box will fail to start
+ apply (and test) configuration changes by rebooting your box using
``vagrant reload --provision``

## TL;DR

For those who don't like reading... here's an example of a rich-filled
Cakebox.yaml with a lot of different provisioning variations.

```yaml
# cakebox.yaml configuration examples

vm:
  hostname: cakebox
  ip: 10.33.10.10
  memory: 2048
  cpus: 1

git:
  username: your-username@github.com
  email: your-username@users.noreply.github.com

security:
  ssh_public_key: ~/.ssh/cakebox_rsa.pub
  ssh_private_key: ~/.ssh/cakebox_rsa

  # Windows
  #ssh_public_key: /Users/your-username/.ssh/cakebox_rsa.pub
  #ssh_private_key: /Users/your-username/.ssh/cakebox_rsa

synced_folders:
  - local:  Apps
    remote: /home/vagrant/Apps

  - local: /tmp/fast/non-windows/nfs-mounted/folder
    remote: /tmp/fast
    mount_options: 'type: "nfs"'

  - local:  some.app
    remote: /home/vagrant/Apps/some.app

  - local:  c:\git\clones\other.app
    remote: /var/www/awesome.app

apps:
  - url: mycake3.app

  - url: mycake2.app
    options: --majorversion 2 --path /var/www/cake2.app

  - url: mylaravel.app
    options: --framework laravel

  - url: git-ssh-repository.app
    options: --source git@github.com:owner/repository.git

  - url: git-https-repository.app
    options: --source http://github.com/your-name/repository

  - url: mycomposer.app
    options: --source yiisoft/yii2-app-basic

sites:
    - url: app1.dev
      webroot: /home/vagrant/Apps/app1.dev

    - url: app2.dev
      webroot: /var/www/somedir

    - url: app3.dev
      webroot: /var/www/app3.dev
      options: --force

databases:
  - name: test1.db

  - name: test2.db
    options: --force

  - name: test3.db
    options: --username user123 --password pass123

software:
  - package: whois
  - package: phpmyadmin

```



## VM

Use the ``vm`` section to customize your Virtual Machine settings.

Option    | Description
:---------|:------------
hostname  | hostname used by your box
ip        | ip-address used to connect to your box
memory    | amount of memory available to your box (in MB)
cpus      | number of virtual CPUs available to your box

```yaml
vm:
  hostname: cakebox
  ip: 10.33.10.10
  memory: 2048
  cpus: 1
```

## Git

Use the ``git`` section if you want to be able to use:

+ private git repositories
+ ssh git repositories (instead of https)

Key         | Option    | Description
:-----------|:----------|:------------
git         | username  | your git(hub) username as it will be used by the vagrant user
git         | email     | your git email-address as it will be used by the vagrant user

```yaml
git:
  username: your-username@github.com
  email: your-username@users.noreply.github.com
```

## Security

The "security" section is used to specify the SSH key pair you want your box
to use for authenticating SSH logins. See
[Hardening Box Authentication](tutorials/hardening-box-authentication/)
for more information.

> **Note:** Windows users will probably find their keys in /Users/your-username/.ssh/

Option          | Description
:---------------|:-----------
ssh_public_key  | full path to your personal public key file
ssh_private_key | full path to your personal private key file

```yaml
security:
  ssh_public_key: ~/.ssh/cakebox_rsa.pub
  ssh_private_key: ~/.ssh/cakebox_rsa
```

## Synced Folders

Define "synced_folders" to create [Vagrant Synced Folders](https://docs.vagrantup.com/v2/synced-folders/index.html)
that will auto-sync data between a folder on your local machine and a folder on
your box.

**Please note:**

+ all applications generated by the ``cakebox application`` command will be placed
in subdirectories directly below /home/vagrant/Apps (unless specified otherwise)
+ local paths starting with a(ny) character will be created as subfolders in
the local Cakebox root folder
+ local paths starting with a / are treated as absolute paths

**Windows users:**

+ should avoid using driveletters (Vagrant will conveniently translate / to C:\)
+ should avoid sharing the Cakebox root folder (.) as a whole as this could
seriously impact performance

Option          | Description
:---------------|:-----------
local  | path of the local folder
remote | path of the remote folder
mount_options | Vagrant Synced Folder supported mount options (ignored on Windows).

```yaml
synced_folders:
  - local:  Apps
    remote: /home/vagrant/Apps

  - local: /tmp/fast/non-windows/nfs-mounted/folder
    remote: /tmp/fast
    mount_options: 'type: "nfs"'

  - local:  some.app
    remote: /home/vagrant/Apps/some.app

  - local:  c:\git\clones\another.app
    remote: /var/www/awesome.app
```

## Apps

Define "apps" to provision fully configured instant-up applications.

Option  | Description
:-------|:-----------
url     | fully qualified domain name used to expose the site
options | any combination of options displayed running ``cakebox application add --help``

```yaml
apps:
  - url: mycake3.app

  - url: mycake2.app
    options: --majorversion 2 --path /var/www/cake2.app

  - url: ssh-cloned.cake3.app
    options: --ssh

  - url: public.app
    options: --source http://github.com/your-name/repository

  - url: myprivate.app
    options: --source git@github.com:your-name/private-repository.git

  - url: mylaravel.app
    options: --framework laravel

  - url: mycomposer.app
    options: --source yiisoft/yii2-app-basic
```

## Sites
Define "sites" to provision Nginx virtual hosts.


Option  | Description
:-------|:------------
url     | fully qualified domain name used to expose the site
webroot | full path to site's webroot folder serving pages
options | any combination of options displayed running ``cakebox site add --help``

```yaml
sites:
  - url: app1.dev
    webroot: /home/vagrant/Apps/app1.dev

  - url: app2.dev
    webroot: /var/www/somedir

  - url: app3.dev
    webroot: /var/www/app3.dev
    options: --force
```

## Databases

Define "databases" to provision a MySQL database accompanied by a
'test_' prefixed database for test purposes.

**Please note:**

+ database names are normalized (e.g. test1.db will be named test1_db)
+ default username/password used to grant localhost access is cakebox/secret
+ use options to override default username/password
+ use --force with caution: it will DROP your existing database

Option  | Description
:-------|:-----------
name    | name of the database
options | any combination of options displayed running ``cakebox database add --help``


```yaml
databases:
  - name: test1.db

  - name: test2.db
    options: --force

  - name: test3.db
    options: --username user123 --password pass123
```

## Additional Software

Define "packages" to install addtional software from the Ubuntu Package Archive.

**Please note:**

+ all installations are executed using "DEBIAN_FRONTEND=noninteractive"

Option  | Description
:-------|:-----------
package | name of the software package as used by ``apt-get install``

```yaml
additional_software:
  - package: whois
  - package: phpmyadmin
```
