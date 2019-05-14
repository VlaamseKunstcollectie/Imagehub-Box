# Imagehub Box (Vagrant, Packer and Ansible)

[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](http://www.gnu.org/licenses/gpl-3.0)

This project builds and configures a virtual image containing the necessary
dependencies for developing, deploying and managing [thedatahub/datahub](github.com/thedatahub/datahub)
and [projectblacklight/blacklight](https://github.com/projectblacklight/blacklight)
on a local or remote hosts. It also includes an installation of [ResourceSpace](https://www.resourcespace.com/), a Digital Asset Manager (DAM) that allows you to upload, manage, and expose digital image files.

This is currently a development repository geared towards developing an imagehub, with the following goals:

1. getting images from a file system imported into resourcespace automatically, with correct extraction of embedded metadata
2. setting up a IIIF image API using a Cantaloupe IIIF image server that extracts images from the Resourcespace API.
3. the creation of an [Imagehub](https://github.com/VlaamseKunstcollectie/Imagehub), a platform similar to Datahub that extracts and combines metadata from:
  - the ResourceSpace API
  - The Cantaloupe IIIF image server info.json files
  - the Datahub<br/>converting that metadata into IIIF manifest.json files, presenting those using a IIIF Presentation API.
4. further developing the [Arthub](https://github.com/VlaamseKunstcollectie/Arthub-Imagehub) to show image files associated with cultural heritage records using the [Universal Viewer](http://universalviewer.io).   

Development of the Imagehub should happen in the [Imagehub](https://github.com/VlaamseKunstcollectie/Imagehub) repo, development of the Arthub should happen in the [Imagehub-Arthub repo](https://github.com/VlaamseKunstcollectie/Arthub-Imagehub). 


You'll get an [Ubuntu 16.04.5 Server (AMD 64)](old-releases.ubuntu.com/releases/trusty/)
box ready for use with [Vagrant](https://www.vagrantup.com/). This box is
provisioned with [Ansible](https://www.ansible.com/).

Provisioning from scratch can take a while, so [Packer](http://www.packer.io/)
support is included to generate a provisioned base box which you can reuse each
time you destroy an instance.

## Requirements

The following software must be installed/present on your local machine before
you can use Packer to build the Vagrant box file:

  - [Packer](http://www.packer.io/)
  - [Vagrant](http://vagrantup.com/)
  - [VirtualBox](https://www.virtualbox.org/) (if you want to build the
    VirtualBox box)
  - [VMware Fusion](http://www.vmware.com/products/fusion/) (or Workstation - if
    you want to build the VMware box)
  - [Ansible](http://docs.ansible.com/intro_installation.html)

## How to use

Then copy or rename the `imagehub.default` file which contains the Ansible 
configuration file:

```bash
$ cp ./ansible/imagehub.default ./ansible/imagehub
```

Change the `path-to-machine` string in the inventory configuration and let it
point to the directory that contains the Imagehub-Box installation i.e:

```
ansible_ssh_private_key_file='/Users/John/Machines/My-Box/.vagrant/machines/imagehub.box/virtualbox/private_key'
```

Next, make sure you have all the required software (listed above) before is installed,
then cd into the `ansible/extension/setup` directory and run:

```bash
$ sh role_update.sh
```

This script will pull down a set of external ansible roles created and
maintained by other authors. These are installed in the `ansible/roles/external`
directory. Then cd into the `packer` directory and run:

```bash
$ packer build imagehub.json
```

After a few minutes, Packer should tell you the box was generated succesfully.
You'll find the box file in the `packer/build` directory.

If you want to only build a box for one of the supported virtualization
platforms (e.g. only build the Virtualbox box), add --only=virtualbox-iso to   
the packer build command:

```bash
$ packer build --only=virtualbox-iso imagehub.json
```

## Using built boxes

### Configuration

Copy the `default.config.yml` file to `config.yml`. 

```bash
$ cp default.config.yml config.yml
```

Change the variables in the configuration file for your particular setup. You should download a copy of the [datahub](https://github.com/thedatahub/datahub), the [arthub](https://github.com/VlaamseKunstcollectie/Arthub-Imagehub), project blacklight, and [resourcespace](https://www.resourcespace.com/get) into the folder you wish to make your shared folder with vagrant. 
Make sure you point the `vagrant_synced_folders` to the directory on the host 
machine where you installed both an instance of the datahub and Project 
Blacklight. If your setup looks like this:

```bash
$ cd ~/Projects
$ ls -lah
drwxr-xr-x  12 user  staff   408B Oct 24 11:09 .
drwxr-xr-x  58 user  staff   1.9K Dec  2 16:50 ..
drwxr-xr-x  20 user  staff   680B Dec  6 14:05 datahub
drwxr-xr-x  27 user  staff   918B Oct 24 15:59 project-blacklight
drwxr-xr-x 22  user  staff   704B Apr 12 15:39 resourcespace
```

the YAML configuration should look like this:

```bash
vagrant_synced_folders:
  # The first synced folder will be used for the default Datahub installation, if
  # any of the build_* settings are 'true'. By default the folder is set to
  # the datahub folder.
  - local_path: /Users/username/Projects
    destination: /vagrant
    type: nfs
create: true
```

Note: if you change the name of the `datahub`, `project-blacklight`, and `resourcespace` folders, 
you will have to update the variables in `ansible/group_vars/all/nginx.yml` as 
well and run the `ansible-playbook` command to update the box with the new 
settings.

### Using the box

After updating the `config.yml` file, run the following command.

```bash
$ vagrant up
```

Vagrant will automatically update your `/etc/hosts` file with the correct 
entries. If this hasn't happened, append these lines to your `/etc/hosts` file:

```bash
192.168.1.152   datahub.box      # http://datahub.box
192.168.1.152   blacklight.box   # http://blacklight.box
192.168.1.152   resourcespace.box   # http://resourcespace.box
```

Access:

| URL                         |  Destination                       |
| --------------------------- |  --------------------------------- |
|  http://cantaloupe.box      |  Your cantaloupe server            |
|  http://datahub.box         |  Your datahub instance             |
|  http://resourcespace.box   |  Your resourcespace instance       |
|  http://blacklight.box      |  Your Project Blacklight instance  |
|  http://blacklight.box:3000 |  Direct access to Rails server     |
|  http://blacklight.box:8983 |  Direct access to Solr             |

Alternatively, you can just run the Ansible playbook after altering the included
inventory file. This ables developers to deploy a fully fledged environment to
a remote host.

## Contents

This box contains Ubuntu 14.04.1 Server (AMD 64) with these packages:

* Git
* PHP-FPM 7 (with mongdb extension)
* Ruby 2.5.3 via rvm (with rails and bundler)
* OpenJDK 11
* Nginx
* MongoDB 3.6
* Rails 5.0.0
* Cantaloupe 4.1
* Tomcat 8

The Datahub box provides you with a fully fledged environment in which you can
deploy an instance of [thedatahub/datahub](github.com/thedatahub/datahub) and
(optional) an instance of [projectblacklight/blacklight](https://github.com/projectblacklight/blacklight). 

## Getting ResourceSpace running

To get Resourcespace up and running, you'll need to:
- download [resourcespace](https://www.resourcespace.com/get).
- Put the download in your shared vagrant folder that you specified in the 'vagrant_synced_folders' variable in config.yml (cfr "Configuration").
- you should now be able to surf to resourcespace.box and configure your resourcespace installation. Don't forget to keep the admin user and password you create during this configuration somewhere safe! you'll have to reinstall your instance of resourcespace if you lose it. 

## Getting Cantaloupe running

- Ansible should download the Cantaloupe installation in opt/cantaloupe in your vagrant box, and should copy the war file to the tomcat8 webapps folder (var/lib/tomcat8/webapps).The -DCantaloupe config file should be automatically set to tomcat in the tomcat group_vars.
- to start tomcat and run Cantaloupe, `vagrant ssh` in to your box and check the status of your tomcat server by using `service tomcat8 status`.
- If tomcat is not running, you can start it with `service tomcat8 start`. If you encounter a 'failed to start service' error, rerun the command with `sudo`. 
- you can now check the status of tomcat again, and if it is running it should be accessible on your vagrant_ip you specified in config.yml, by default on port 8080, e.g. 192.168.2.152:8080 . You should see an 'it works!' tomcat message. To see if cantaloupe is running, you can go to 192.168.2.152:8080/cantaloupe/ , or surf to cantaloupe.box .
- You can access the tomcat manager console at 192.168.2.152:8080/manager/html, which you can access with the username and password defined in the tomcat.yml group variables (default they are 'user' and 'pass')
- to stop Cantaloupe and tomcat, as vagrant ssh you should use `service tomcat8 stop`, or sudo if that doesn't work. 
- you can edit cantaloupe properties using the cantaloupe.yml group_vars file and then reprovisioning your box, or directly while in your vagrant box by editing the cantaloupe.properties file in opt/cantaloupe/cantaloupe.properties. 
- you can use the cantaloupe admin console by surfing to 192.168.2.152:8080/cantaloupe/admin or to cantaloupe.box/cantaloupe/admin. The password is set in the cantaloupe.yml group variables, by default it is set to 'pass' (the username is always 'admin'). Cantaloupe can also be configured from that admin panel.



## Credits


## Authors

* Matthias Vandermaesen <matthias.vandermaesen@vlaamsekunstcollectie.be>
* Jolan Wuyts <jolan.wuyts@vlaamsekunstcollectie.be>

## Copyright

Copyright 2016 - PACKED vzw, Vlaamse Kunstcollectie vzw

## License

This library is free software; you can redistribute it and/or modify it under
the terms of the GPLv3.

