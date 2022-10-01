Ansible role: SSSD
==================

Install and configure SSSD

Role Variables
--------------

```
ENTRY POINT: main - Install and configure SSSD

OPTIONS (= is mandatory):

- sssd_config
        SSSD configuration
        Top-level = name of the section
        Second-level = options and their values
        See https://linux.die.net/man/5/sssd.conf
        [Default: (null)]
        type: dict

- sssd_config_file
        Path to configuration file
        [Default: /etc/sssd/sssd.conf]
        type: str

- sssd_disabled_activatable_responder_services
        List of SSSD services that have a corresponding systemd
        service to disable
        [Default: ['nss', 'pam']]
        elements: str
        type: list

- sssd_disabled_activatable_responder_systemd_services
        List of systemd services to disable (excluding "sssd-" prefix)
        [Default: (null)]
        elements: str
        type: list

- sssd_disabled_activatable_responder_systemd_sockets
        List of systemd sockets to disable (excluding "sssd-" prefix)
        [Default: (null)]
        elements: str
        type: list

- sssd_disabled_activatable_responder_systemd_units
        List of systemd units to disable (excluding "sssd-" prefix)
        [Default: (null)]
        elements: str
        type: list

- sssd_disabled_activatable_responders
        List of SSSD services to disable as systemd activatable
        responders. In most cases, this variable and the
        sssd_disabled_activatable_responder_* variables can be left as
        their default value. The default values for the
        sssd_disabled_activatable_responder_* variables are based on
        this list.
        [Default: ['nss', 'pam', 'pam-priv']]
        elements: str
        type: list

- sssd_enabled_activatable_responder_services
        List of SSSD services that have a corresponding systemd
        service to enable
        [Default: (null)]
        elements: str
        type: list

- sssd_enabled_activatable_responder_systemd_services
        List of systemd services to enable (excluding "sssd-" prefix)
        [Default: (null)]
        elements: str
        type: list

- sssd_enabled_activatable_responder_systemd_sockets
        List of systemd sockets to enable (excluding "sssd-" prefix)
        [Default: (null)]
        elements: str
        type: list

- sssd_enabled_activatable_responder_systemd_units
        List of systemd units to enable (excluding "sssd-" prefix)
        [Default: (null)]
        elements: str
        type: list

- sssd_enabled_activatable_responders
        List of SSSD services to enable as systemd activatable
        responders. In most cases, this variable and the
        sssd_enabled_activatable_responder_* variables can be left as
        their default value. By default, this is pulled from
        sssd_config.sssd.services and any listed in
        sssd_disabled_activatable_responders are removed. The default
        values for the sssd_enabled_activatable_responder_* variables
        are based on this list.
        [Default: (null)]
        elements: str
        type: list

- sssd_manage_activatable_responders
        If true, manage systemd activatable responders. By default,
        this is set to true for distributions Ubuntu 20.04+ and Debian
        11+. See https://sssd.io/design-
        pages/systemd_activatable_responders.html.
        [Default: (null)]
        type: bool

- sssd_mkhomedir
        If true, when a user logs in for the first time create a home
        directory for that user with a copy of the files from
        /etc/skel
        [Default: False]
        type: bool

- sssd_packages
        List of packages to install
        [Default: ['sssd', 'sssd-tools', 'sssd-dbus', 'libpam-sss',
        'libnss-sss']]
        elements: str
        type: list
```

Installation
------------

This role can either be installed manually with the ansible-galaxy CLI tool:

    ansible-galaxy install git+https://github.com/wandansible/sssd,main,wandansible.sssd
     
Or, by adding the following to `requirements.yml`:

    - name: wandansible.sssd
      src: https://github.com/wandansible/sssd

Roles listed in `requirements.yml` can be installed with the following ansible-galaxy command:

    ansible-galaxy install -r requirements.yml

Example Playbook
----------------

    - hosts: all
      roles:
         - role: wandansible.sssd
           become: true
           vars:
             ldap_auth_host: "ldap.example.org"
             ldap_auth_port: "636"
             ldap_search_base: "dc=example,dc=org"
             ldap_filter_groups:
               - root
             ldap_config:
               id_provider: "ldap"
               auth_provider: "ldap"
               ldap_uri: "ldaps://{{ ldap_auth_host }}:{{ ldap_auth_port }}"
               ldap_id_use_start_tls: "true"
               ldap_search_base: "{{ ldap_search_base }}"
               ldap_tls_reqcert: "demand"
               ldap_tls_cacert: "/etc/ssl/ldap/cacert.pem"
               cache_credentials: "true"
               enumerate: "true"
               sudo_provider: "none"
             sssd_config:
               sssd:
                 config_file_version: "2"
                 domains: "LDAP"
                 services:
                   - "nss"
                   - "pam"
                   - "sudo"
                   - "ssh"
               nss:
                 filter_users: "root"
                 filter_groups: "{{ ldap_filter_groups | join(', ') }}"
               domain/LDAP: "{{ ldap_config }}"
