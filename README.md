# Pasteque API

An ansible role for pasteque API | Server deployment.

## Requirements

### On the node controller

On your node controller, you must have `passlib` python library installed to be able to use the bcrypt encryption with the `password_hash` ansible filter:

```bash
pip install passlib

# or 

pip3 install passlib
```

### On the node

This role requires you have a LAMP server (with `Apache` or `NGINX`) running on your server along with `composer` and `git`.

The PHP packages required are : 
- `php-cli`
- `php-mysql`
- `php-zip`
- `php-apcu`
- `php-common`
- `php-gnupg`

You can install the required components with [geerlingguy](https://galaxy.ansible.com/geerlingguy/) roles collections :
- `geerlingguy.mysql`
- `geerlingguy.php`
- `geerlingguy.apache`
- `geerlingguy.git`
- `geerlingguy.composer`

## Role Variables

Main available variables are listed below, along with default values (see [defaults/main.yml](defaults/main.yml)`):

The URL of the API (Not needed in this role but it's a helper for other role to be configured like `apache` and its vhosts):

```yaml
pasteque_api_url: "pasteque.{{ domain | d(example.com) }}"
```

The root directory for the files:

```yaml
pasteque_api_dir: "/var/www/pasteque_api"
```

The URL of the repo (default is setted to pasteque official repo):

```yaml
pasteque_api_git_repo: "https://framagit.org/pasteque/pasteque-server.git"
```

The list of the pasteque users.

```yaml
pasteque_api_users: []
```

By default, this list is empty. If you want to add users, you'll have to use the following structure:

```yaml
pasteque_api_users:
  - name: user1
    password: password_user1
    db_name: db_user1
  - name: user2
    password: password_user2
    db_name: db_user2
```

The test user (used for creating proxies and testing your configuration)

```yaml
pasteque_api_test_user:
    db_type: mysql
    db_host: localhost
    db_port: 3306
    db_name: "pasteque_test"
    db_user: "test"
    db_password: "test"
    gpg_path: "{{ pasteque_api_dir }}/config/gpg"
    gpg_fingerprint: "1234123412341234123412341234123412341234"
    http_host: http://localhost
    http_user: pasteque_test
    http_password: pasteque_test
```

## Examples Playbooks

If you have met all the requirements, this playbook should be enough:

```yaml
- hosts: all
  become: yes
  vars:
    - pasteque_api_users:
        - name: user1
          password: password_user1
          db_name: db_user1
  roles:
     - role: masterit_dev.pasteque_api
```

For a from-scratch installation, you can begin with this complete playbook:

```yaml
- hosts: all
  become: true
  vars:
    mysql_root_password: pass
    pasteque_api_url: pasteque.example.com
    pasteque_api_dir: /var/www/pasteque_api
    pasteque_api_users:
      - name: user1
        password: password_user1
        db_name: db_user1
      - name: user2
        password: password_user2
        db_name: db_user2
    php_packages:
      - php
      - php-mysql
      - php-cli
      - php-apcu
      - php-common
      - php-zip
      - php-gnupg
    php_packages_extra:
      - libapache2-mod-php
    apache_vhosts_filename: pasteque_api.conf
    apache_vhosts:
      - servername: "{{ pasteque_api_url }}"
        documentroot: "{{ pasteque_api_dir }}/src/http/public"
  roles:
    - role: geerlingguy.mysql
    - role: geerlingguy.php
    - role: geerlingguy.apache
    - role: geerlingguy.git
    - role: geerlingguy.composer
    - role: masterit_dev.pasteque_api
```

It will be preferable to convert all the playbook variables into `group_vars` variables and use the `ansible-vault` for the users definition.

## License

[CC-BY-SA-4.0](https://creativecommons.org/licenses/by-sa/4.0/)

## Author Information

This role was created in 2021 for [Pasteque](https://www.pasteque.org) by Lucien DURAND-HARDY from [MasterIT](http://www.masterit.fr).
