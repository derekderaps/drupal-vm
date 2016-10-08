# A fork of [Drupal VM](http://www.drupalvm.com/) with some pre-configured sites

## Usage

### First-run

1.  If you haven't already, download and install [Vagrant](https://www.vagrantup.com/) and [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
1.  Install [Vagrant Host
    Manager](https://github.com/devopsgroup-io/vagrant-hostmanager) to
    automatically keep your `/etc/hosts` file up-to-date. Note that this step
    seems a little wonky on linux. It's okay to skip; it just means manually
    updating your `/etc/hosts` file later.

    ```
    vagrant plugin install vagrant-hostmanager
    ```
1.  Clone this fork of Drupal VM anywhere on your computer. It will
    automatically track the `forked` branch, which represents the latest
    approved Drupal VM release with all our changes added in.

    ```
    git clone git@github.com:derekderaps/drupal-vm.git
    ```
1.  From the command line, `cd` into the `drupal-vm` repo directory you just
    cloned.

    ```
    cd drupal-vm
    ```
1.  Create a `config.yml` file and adjust your VM's memory allocation to ¼ your
    system total and ½ your virtual CPUs. Virtual CPUs are typically double the
    number of physical cores (on OS X, use `sysctl -n hw.ncpu` to find out).

    ```
    ---
    vagrant_memory: 4096
    vagrant_cpus: 4
    ```
1.  Run `vagrant up` and wait for it to finish, usually 15-30 minutes from the
    Ubuntu base box download step.
1.  Visit http://dashboard.dvm to see all the sites, drush aliases, and tools
    available to you.
1.  If you couldn't install Vagrant Host Manager in step two above, copy the
    `/etc/hosts` file entries from the dashboard and paste them into your
    `/etc/hosts` file.

### Site setup

If you see a database connection error when trying to visit your site, it probably means that `settings.php` has not yet been set up for Drupal VM. For a long-term solution, submit an issue or PR to your project repo to have the Drupal VM connection info added to `sites/default/settings.php`. But to get going immediately, follow these steps:

1.  Edit your global `.gitignore` file (probably `~/.gitignore`) and add lines for,

    ```
    sites/*.dvm
    docroot/sites/*.dvm
    ```
1.  Create a `<site>.dvm` sites directory.
1.  Add these lines to `settings.php` (update the database name):

    ```
    <?php
    include DRUPAL_ROOT . '/sites/default/settings.php';
    $databases['default']['default'] = array(
      'driver' => 'mysql',
      'database' => '<site>_drupalvm',
      'username' => 'drupal',
      'password' => 'drupal',
      'host' => 'localhost',
      'prefix' => '',
    );
    ```

Once `settings.php` is configured for the site, you can import a database dump
to it. Thanks to Drupal VM, [Adminer](https://www.adminer.org/) is installed at
http://adminer.dvm. Log in with user `root` and password `root`. Or use `drush`
to import a dump:

```
drush @drupalvm.<site>.dvm sqlc < /path/to/dump.sql
```

That's it, folks! Follow the links on http://dashboard.dvm to visit each of your sites.

### Adding new sites

When we add new sites or make other changes to the VM configuration, you
typically don't need to `vagrant destroy` it and start over; a simple
re-provisioning should do fine (and is much quicker):

`vagrant reload --provision`

If you want more sites added to this config, submit an issue or PR to this
repo.  It typically requires adding entries to these three YAML arrays in
`default.config.yml`:

1.  `vagrant_synced_folders`
1.  `nginx_hosts`
1.  `mysql_databases`

## FAQ

1.  _Why append `_drupalvm` to database names?_

    As a flag to provide peace-of-mind that "Yes, this is my local" when running `sql-drop` and other scary operations. E.g.,

    > Are you sure you want to drop database `mysite_drupalvm`?

1.  _Can I add some local sites without committing them to this repo?_

    Yes, but I'm not satisifed with the solution. You have to copy
    `default.config.yml` to `config.yml` and update the configuration specified
    above in "Adding new sites." However, by doing so you lose some changes
    (like new sites) that get made to `default.config.yml`. I'm working on
    this. It probably just needs the call to `merge()` in `Vagrantfile`
    replaced with a [recursive
    implementation](https://github.com/danielsdeleo/deep_merge).

## Upstream Drupal VM README continues below

![Drupal VM Logo](https://raw.githubusercontent.com/geerlingguy/drupal-vm/master/docs/images/drupal-vm-logo.png)

[![Build Status](https://travis-ci.org/geerlingguy/drupal-vm.svg?branch=master)](https://travis-ci.org/geerlingguy/drupal-vm) [![Documentation Status](https://readthedocs.org/projects/drupal-vm/badge/?version=latest)](http://docs.drupalvm.com) [![Packagist](https://img.shields.io/packagist/v/geerlingguy/drupal-vm.svg)](https://packagist.org/packages/geerlingguy/drupal-vm)

[Drupal VM](https://www.drupalvm.com/) is A VM for local Drupal development, built with Vagrant + Ansible.

This project aims to make spinning up a simple local Drupal test/development environment incredibly quick and easy, and to introduce new developers to the wonderful world of Drupal development on local virtual machines (instead of crufty old MAMP/WAMP-based development).

It will install the following on an Ubuntu 16.04 (by default) linux VM:

  - Apache 2.4.x (or Nginx)
  - PHP 7.0.x (configurable)
  - MySQL 5.7.x (or MariaDB, or PostgreSQL)
  - Drush (configurable)
  - Drupal 7.x, or 8.x.x (configurable)
  - Optional:
    - Drupal Console
    - Varnish 4.x (configurable)
    - Apache Solr 4.10.x (configurable)
    - Elasticsearch
    - Node.js 0.12 (configurable)
    - Selenium, for testing your sites via Behat
    - Ruby
    - Memcached
    - Redis
    - SQLite
    - XHProf, for profiling your code
    - Blackfire, for profiling your code
    - XDebug, for debugging your code
    - Adminer, for accessing databases directly
    - Pimp my Log, for easy viewing of log files
    - MailHog, for catching and debugging email

It should take 5-10 minutes to build or rebuild the VM from scratch on a decent broadband connection.

Please read through the rest of this README and the [Drupal VM documentation](http://docs.drupalvm.com/) for help getting Drupal VM configured and integrated with your development workflow.

## Documentation

Full Drupal VM documentation is available at http://docs.drupalvm.com/

## Customizing the VM

There are a couple places where you can customize the VM for your needs:

  - `config.yml`: Override any of the default VM configuration from `default.config.yml`; customize almost any aspect of any software installed in the VM (more about [overriding configurations](http://docs.drupalvm.com/en/latest/other/overriding-configurations/).
  - `drupal.composer.json` or `drupal.make.yml`: Contains configuration for the Drupal core version, modules, and patches that will be downloaded on Drupal's initial installation (you can build using Composer, Drush make, or your own codebase).

If you want to switch from Drupal 8 (default) to Drupal 7 on the initial install, do the following:

  1. Switch to using a [Drush Make file](http://docs.drupalvm.com/en/latest/deployment/drush-make/).
  1. Update the Drupal `version` and `core` inside your `drupal.make.yml` file.
  2. Set `drupal_major_version: 7` inside `config.yml`.

## Quick Start Guide

This Quick Start Guide will help you quickly build a Drupal 8 site on the Drupal VM using Composer with `drupal-project`. You can also use Drupal VM with [Composer](http://docs.drupalvm.com/en/latest/deployment/composer/), a [Drush Make file](http://docs.drupalvm.com/en/latest/deployment/drush-make/), with a [Local Drupal codebase](http://docs.drupalvm.com/en/latest/deployment/local-codebase/), or even a [Drupal multisite installation](http://docs.drupalvm.com/en/latest/deployment/multisite/).

If you want to install a Drupal 8 site locally with minimal fuss, just:

  1. Install [Vagrant](https://www.vagrantup.com/downloads.html) and [VirtualBox](https://www.virtualbox.org/wiki/Downloads).
  2. Download or clone this project to your workstation.
  3. `cd` into this project directory and run `vagrant up`.

But Drupal VM allows you to build your site exactly how you like, using whatever tools you need, with almost infinite flexibility and customization!

### 1 - Install Vagrant and VirtualBox

Download and install [Vagrant](https://www.vagrantup.com/downloads.html) and [VirtualBox](https://www.virtualbox.org/wiki/Downloads).

You can also use an alternative provider like Parallels or VMware. (Parallels Desktop 11+ requires the "Pro" or "Business" edition and the [Parallels Provider](http://parallels.github.io/vagrant-parallels/), and VMware requires the paid [Vagrant VMware integration plugin](http://www.vagrantup.com/vmware)).

Notes:

  - **For faster provisioning** (macOS/Linux only): *[Install Ansible](http://docs.ansible.com/intro_installation.html) on your host machine, so Drupal VM can run the provisioning steps locally instead of inside the VM.*
  - **NFS on Linux**: *If NFS is not already installed on your host, you will need to install it to use the default NFS synced folder configuration. See guides for [Debian/Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-14-04), [Arch](https://wiki.archlinux.org/index.php/NFS#Installation), and [RHEL/CentOS](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-centos-6).*
  - **Versions**: *Make sure you're running the latest releases of Vagrant, VirtualBox, and Ansible—as of February 2016, Drupal VM recommends: Vagrant 1.8.5, VirtualBox 5.1.x, and Ansible 2.1.x.*

### 2 - Build the Virtual Machine

  1. Download this project and put it wherever you want.
  2. (Optional) Copy `default.config.yml` to `config.yml` and modify it to your liking.
  3. Create a local directory where Drupal will be installed and configure the path to that directory in `config.yml` (`local_path`, inside `vagrant_synced_folders`).
  4. Open Terminal, `cd` to this directory (containing the `Vagrantfile` and this README file).
  5. Type in `vagrant up`, and let Vagrant do its magic.

Once the process is complete, you will have a Drupal codebase available inside the `drupal/` directory of the project.

Note: *If there are any errors during the course of running `vagrant up`, and it drops you back to your command prompt, just run `vagrant provision` to continue building the VM from where you left off. If there are still errors after doing this a few times, post an issue to this project's issue queue on GitHub with the error.*

### 3 - Configure your host machine to access the VM.

  1. [Edit your hosts file](http://www.rackspace.com/knowledge_center/article/how-do-i-modify-my-hosts-file), adding the line `192.168.88.88  drupalvm.dev` so you can connect to the VM.
    - You can have Vagrant automatically configure your hosts file if you install the `hostsupdater` plugin (`vagrant plugin install vagrant-hostsupdater`). All hosts defined in `apache_vhosts` or `nginx_hosts` will be automatically managed. `vagrant-hostmanager` is also supported.
    - The `auto_network` plugin (`vagrant plugin install vagrant-auto_network`) can help with IP address management if you set `vagrant_ip` to `0.0.0.0` inside `config.yml`.
  2. Open your browser and access [http://drupalvm.dev/](http://drupalvm.dev/). The default login for the admin account is `admin` for both the username and password.

## Extra software/utilities

By default, this VM includes the extras listed in the `config.yml` option `installed_extras`:

    installed_extras:
      - adminer
      # - blackfire
      - drupalconsole
      - mailhog
      # - memcached
      # - newrelic
      # - nodejs
      - pimpmylog
      # - redis
      # - ruby
      # - selenium
      # - solr
      - varnish
      # - xdebug
      # - xhprof

If you don't want or need one or more of these extras, just delete them or comment them from the list. This is helpful if you want to reduce PHP memory usage or otherwise conserve system resources.

## Using Drupal VM

Drupal VM is built to integrate with every developer's workflow. Many guides for using Drupal VM for common development tasks are available on the [Drupal VM documentation site](http://docs.drupalvm.com).

## Updating Drupal VM

Drupal VM follows semantic versioning, which means your configuration should continue working (potentially with very minor modifications) throughout a major release cycle. Here is the process to follow when updating Drupal VM between minor releases:

  1. Read through the [release notes](https://github.com/geerlingguy/drupal-vm/releases) and add/modify `config.yml` variables mentioned therein.
  2. Do a diff of your `config.yml` with the updated `default.config.yml` (e.g. `curl https://raw.githubusercontent.com/geerlingguy/drupal-vm/master/default.config.yml | git diff --no-index config.yml -`).
  3. Run `vagrant provision` to provision the VM, incorporating all the latest changes.

For major version upgrades (e.g. 2.x.x to 3.x.x), it may be simpler to destroy the VM (`vagrant destroy`) then build a fresh new VM (`vagrant up`) using the new version of Drupal VM.

## System Requirements

Drupal VM runs on almost any modern computer that can run VirtualBox and Vagrant, however for the best out-of-the-box experience, it's recommended you have a computer with at least:

  - Intel Core processor with VT-x enabled
  - At least 4 GB RAM (higher is better)
  - An SSD (for greater speed with synced folders)

## Other Notes

  - To shut down the virtual machine, enter `vagrant halt` in the Terminal in the same folder that has the `Vagrantfile`. To destroy it completely (if you want to save a little disk space, or want to rebuild it from scratch with `vagrant up` again), type in `vagrant destroy`.
  - To log into the virtual machine, enter `vagrant ssh`. You can also get the machine's SSH connection details with `vagrant ssh-config`.
  - When you rebuild the VM (e.g. `vagrant destroy` and then another `vagrant up`), make sure you clear out the contents of the `drupal` folder on your host machine, or Drupal will return some errors when the VM is rebuilt (it won't reinstall Drupal cleanly).
  - You can change the installed version of Drupal or drush, or any other configuration options, by editing the variables within `config.yml`.
  - Find out more about local development with Vagrant + VirtualBox + Ansible in this presentation: [Local Development Environments - Vagrant, VirtualBox and Ansible](http://www.slideshare.net/geerlingguy/local-development-on-virtual-machines-vagrant-virtualbox-and-ansible).
  - Learn about how Ansible can accelerate your ability to innovate and manage your infrastructure by reading [Ansible for DevOps](http://www.ansiblefordevops.com/).

## License

This project is licensed under the MIT open source license.

## About the Author

[Jeff Geerling](http://www.jeffgeerling.com/) created Drupal VM in 2014 for a more efficient Drupal site and core/contrib development workflow. This project is featured as an example in [Ansible for DevOps](http://www.ansiblefordevops.com/).
