Ansible Role: Drupal
=========
[![Build Status](https://travis-ci.org/bmeme/ansible-role-drupal.svg?branch=master)](https://travis-ci.org/bmeme/ansible-role-drupal)

Create, Install and Build a [Drupal](http://www.drupal.org) project.

This is the first published Ansible Role produced by [Bmeme](https://www.bmeme.com), actually used in our Drupal development process.
It was really inspired by other roles we usually use/used and that are really great:
- [geerlingguy.drupal](https://github.com/geerlingguy/ansible-role-drupal)
- [ansistrano.deploy](https://github.com/ansistrano/deploy) / [ansistrano.rollback](https://github.com/ansistrano/rollback)

Requirements
------------
To correctly use this role you will need a classic LAMP/LEMP stack:
- a standard Webserver (recommended Apache or Nginx)
- MySQL/PostgreSQL database
- PHP interpreter (7.x or latest)
- Composer (recommended `geerlingguy.composer`)

**Drush** is a needed dependency that this role will provide in your Drupal project.

Dependencies
--------------
- `community.mysql` collection
- `community.general` collection

Installation
--------------
This is an Ansible role distributed using Ansible Galaxy.
In order to install this role you can use the following command.

`$ ansible-galaxy install bmeme.drupal`

Update
------
If you want to update the role, you need to pass --force parameter when installing. Please, check the following command:

`$ ansible-galaxy install --force bmeme.drupal`

Pre-release Version
-------------------
This role has a pre-release version (called *0.5-beta1*) that use a different workflow and different variables schema.
This version is **completely deprecated** and used in Bmeme only for backward compatibility, __do not use it__!

Role Variables
--------------
Available Variables:

| Variable Name  | Description  | Default  |
|----------------|--------------|----------|
| `drupal_project_dir` | Absolute path in which the Drupal project is or have to be installed| `/var/www/html`|
| `drupal_project_owner` | The system user that has the project ownership | `www-data` |
| `drupal_project_group` | The system group that has the project ownership | `www-data` |
| `drupal_project_composer_project` | The Drupal composer project you want to use to install it freshly | `drupal/recommended-project:^9` |
| `drupal_project_web_root` | Directory in which is stored Drupal core. It may depend on the Drupal distribution. | `web` |
| `drupal_site_name` | Drupal site name. Used by drush during Drupal installation. | `Your new Drupal instance` |
| `drupal_site_mail` | Drupal site mail. Used by drush during Drupal installation. | `info@example.com` |
| `drupal_account_mail` | Drupal account mail. Used by drush during Drupal installation. | same as `drupal_site_mail` |
| `drupal_account_name` | Drupal account name. Used by drush during Drupal installation. | `admin` |
| `drupal_account_pass` | Drupal account pass. Used by drush during Drupal installation. | `admin` |
| `drupal_db_schema` | Database schema to be used by Drupal. Available options: `mysql` or `pgsql` | `mysql` |
| `drupal_db_name` | Database name. Used by drush during Drupal installation. | `drupal` |
| `drupal_db_user` | Database user. Used by drush during Drupal installation. | `drupal` |
| `drupal_db_pass` | Database user password. Used by drush during Drupal installation. | `drupal` |
| `drupal_db_host` | Database host address. Used by drush during Drupal installation. | `127.0.0.1` |
| `drupal_db_port` | Database host port. Used by drush during Drupal installation. | `3306` |
| `drupal_profile` | Drupal installation profile. See [this bug](https://www.drupal.org/project/drupal/issues/2982052) | `minimal` |
| `drupal_composer_nodev` | Composer "nodev" option. Boolean  | `false` |
| `drupal_composer_prefer_dist` | Composer "prefer-dist" option. Boolean  | `false` |
| `drupal_composer_scaffold_nonamespace` | Use composer without namespace to install Drupal Scaffold. Boolean  | `false` |

Workflow
--------
This role executes three main tasks:
- **create**: Create a Drupal fresh project using composer: you can customise composer project using `drupal_project_composer_project` variable.
This role automatically recognizes when a Drupal project already exists and skips this set of tasks
- **install**: Install Drupal instance using drush. At the end of the process, Drupal configurations are automatically exported to the `{{ drupal_project_dir }}/config/sync` directory.
This role automatically recognized when Drupal installation already exists and skip this set of tasks
- **build**: Build the Drupal instance via drush by installing from existing exported configurations.
It automatically recognizes the configurations and executes the build set of tasks

Starting from scratch, it performs only the **create** and **install** tasks.

Hooks
--------
You can add your custom tasks using the hooks available:
- `drupal_before_create`: executed before the **create** set of tasks. If a Drupal project is already present, this hook will be skipped.
- `drupal_after_create`: executed after the **create** set of tasks. If a Drupal project is already present, this hook will be skipped.
- `drupal_before_install`: executed before the **install** set of tasks. If Drupal configurations are already available, this hook will be skipped.
- `drupal_after_install`: executed after the **install** set of tasks. If Drupal configurations are already available, this hook will be skipped.
- `drupal_before_build`: executed before the **build** set of tasks. If Drupal configurations are not available, this hook will be skipped.
- `drupal_after_build`: executed after the **build** set of tasks. If Drupal configurations are not available, this hook will be skipped.

Bmeme way of work
-----------------
Bmeme uses this role to automate the creation / installation / build operations of a Drupal project during
its development process. We use docker as development environment, both locally and in remote test instances.
The role is performed within our docker images `php` available [here](https://hub.docker.com/r/bmeme/php).
Same images are also used by the role *molecule* test.

Dependencies
------------
N/A

Example Playbook
----------------

    - hosts: webserver
      vars_files:
        - vars/main.yml
      roles:
        - geerlingguy.apache
        - geerlingguy.mysql
        - geerlingguy.php-versions
        - geerlingguy.php
        - geerlingguy.php-mysql
        - geerlingguy.composer
        - bmeme.drupal

Inside `vars/main.yml`

    drupal_project_dir: /path/to/my/project
    drupal_site_name: My Awesome Drupal instance
    drupal_after_build: "tasks/my-after-build-tasks.yml"

If you want to use this role to install a specific Drupal distribution (for example: Lightning):

    - hosts: webserver
      vars_files:
        - vars/main.yml
      roles:
        - geerlingguy.apache
        - geerlingguy.mysql
        - geerlingguy.php-versions
        - geerlingguy.php
        - geerlingguy.php-mysql
        - geerlingguy.composer
        - bmeme.drupal
          drupal_project_composer_project: "acquia/lightning-project:8.8.1"

License
-------

MIT/BSD

Author Information
------------------
This role was created in 2020 by [Bmeme](https://www.bmeme.com).
