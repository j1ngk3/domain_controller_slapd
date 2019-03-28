ryezone_labs.domain_controller_slapd
=========

This role installs and configures slapd on Ubuntu.

- Enables TLS.
- Adds sudoRole schema.
- Configures users, groups and organizational units.

Requirements
------------

- This role does not generate SSL certificates, these must be pre-generated.

Role Variables
--------------

### Global Variables

- `ryezone_labs_domain_suffix` (string)

   Domain suffix to use when creating the domain.
   
   Defaults to `local`.

- `ryezone_labs_domain_organization` (string)

   Organization name to use when creating the domain.
   
   Defaults to `domain`.

- `ryezone_labs_domain_admin_password` (string)

   Password to use for the default admin account `cn=admin,dc={{ dc_org }},dc={{ dc_suffix }}`.
   
   Defaults to `My$uper$ecretP@$$w0rd`.

### Role Variables

- `dc_org` (string)

   Organization name to use when creating the domain.
   Sets the value of the `shared/organization` debconf question of the slapd install.
   
   Defaults to `{{ ryezone_labs_domain_organization }}`.

- `dc_suffix` (string)

   Domain suffix to use when creating the domain.
   
   Defaults to `{{ ryezone_labs_domain_suffix }}`.

- `dc_domain_admin_password` (string)

   Password to use for the default admin account `{{ dc_admin_distinguished_name }}`.

   Defaults to `{{ ryezone_labs_domain_admin_password }}`

- `dc_olcLogLevel` (string)

   Sets the [OpenLDAP logging level](http://www.zytrax.com/books/ldap/ch6/#loglevel).  This value is a space delimited string.

   Defaults to `stats`

### Role Variables - TLS configuration

- `dc_certificate_directory` (string)

   Sets the source directory for the pre-generated TLS certificates.

   Defaults to `/opt/ansible/certificates`

- `dc_tls_ca_certificate_file` (string)

   Sets the name of the TLS CA certificate file.

   Defaults to `{{ dc_tld }}.crt`.
   Default assumes a self-signed certificate.

- `dc_tls_certificate_file` (string)

   Sets the name of the TLS certificate file.

   Defaults to `{{ dc_tld }}.crt`.
   Default assumes a self-signed certificate.

- `dc_tls_key_file`

   Sets the name of the TLS key file.

   Defaults to `{{ dc_tld }}.pem`.

### Role Variables - Organizational Units

- `dc_organizational_units` (list of hashes)

   This list defines the top level organizational units of the domain.

   - `name` (string)
   
      This is the common name of the organizational unit.

   - `domain_dn` (string)

      This is the distinguished name of the domain.  A helper variable, `dc_distinguished_name`, is provided to reduce code duplication.

   Defaults to:
   ```yaml
   dc_organizational_units:
     - name: users
       domain_dn: "{{ dc_distinguished_name }}"
     - name: groups
       domain_dn: "{{ dc_distinguished_name }}"
     - name: sudoers
       domain_dn: "{{ dc_distinguished_name }}"
   ```

### Role Variables - Posix Groups

- `dc_posix_groups` (list of hashes)

   This list defines the posix user groups in the domain.

   - `name` (string)

      This is the common name of the group.

   - `ou_dn` (string)

      This is the distinguished name of the organizational unit under which the posix group should be created.
      A helper variable, `dc_distinguished_name`, is provided to reduce code duplication.

   - `gid` (int)

      This is the group id that will be assigned.  Care should be taken to ensure that this is unique and does not conflict with pre-existing groups on your servers.

   Defaults to:

   ```yaml
   dc_posix_groups:
     - name: ldapUsers
       ou_dn: "ou=groups,{{ dc_distinguished_name }}"
       gid: 4000
   ```

### Role Variables - Posix Users

- `dc_posix_users` (list of hashes)

   This list defines the posix users in the domain.
   These users are capable of authenticating to machines in the domain.

   - `username` (string)

      This is the common name of the user.
      It is also the username the user will use to authenticate to systems.

   - `ou_dn` (string)

      This is the distinguished name of the organizational unit under which the posix user should be created.
      A helper variable, `dc_distinguished_name`, is provided to reduce code duplication.

   - `surname` (string)

      This is the last name of the user.

   - `givenName` (string)

      This is the first name of the user.

   - `password` (string)

      This is the user's password.
      It can be provided in both plain text and a hash.
      [This blog post](https://www.redpill-linpro.com/techblog/2016/08/16/ldap-password-hash.html) has a good explanation of the hashing process.

   - `uid` (int)

      This is the user id for the user.
      Care should be taken to ensure it is unique and does not conflict with accounts that might already exist on the machine.

   - `gid` (int)

      This defines the default group for the user.

   - `shell` (string)

      This configures the default shell for the user.

   This defaults to:

   ```yaml
   dc_posix_users:
     - username: jdoe
       ou_dn: "ou=users,{{ dc_distinguished_name }}"
       surname: Doe
       givenName: John
       password: Password1
       uid: 5000
       gid: 4000
       shell: /bin/bash
   ```

### Role Variables - inetOrgPersons

`dc_inet_org_persons` (list of hashes)

   This list creates domain users that can log into browser based systems, but cannot authenticate to the host systems.

   - `username` (string)

      This is the common name of the user.
      It is also the username the user will use to authenticate to systems.

   - `ou_dn` (string)

      This is the distinguished name of the organizational unit under which the inetOrgPerson user should be created.
      A helper variable, `dc_distinguished_name`, is provided to reduce code duplication.

   - `surname` (string)

      This is the last name of the user.

   - `givenName` (string)

      This is the first name of the user.

   - `password` (string)

      This is the user's password.
      It can be provided in both plain text and a hash.
      [This blog post](https://www.redpill-linpro.com/techblog/2016/08/16/ldap-password-hash.html) has a good explanation of the hashing process.

   This defaults to:

   ```yaml
   dc_inet_org_persons:
     - username: joeSchmoe
       ou_dn: "ou=users,{{ dc_distinguished_name }}"
       surname: Schmoe
       givenName: Joe
       password: '{SSHA}mm7AhcbWv7h8V8oZffzrRwdMFIiMZ9dF'
   ```

### Role Variables - Sudoers

- `dc_sudoers` (list of hashes)

   This list defines who is granted sudo access on the domain.

   - `cn` (string)

      This is the common name of the sudoRole.

   - `ou_dn` (string)

      This is the distinguished name of the organizational unit under which the sudoRole entry should be created.
      A helper variable, `dc_distinguished_name`, is provided to reduce code duplication.

   - `sudoHost` (string)

      This is the set of hosts on which the user is authorized to use sudo privileges.

   - `sudoUser` (string)

      This is the user allowed to use sudo.

   - `sudoCommand` (string)

      This is the set of commands the user is authorized to perform.

   This defaults to:
   
   ```yaml
   dc_sudoers:
     - cn: role_ldapUsers
       ou_dn: "ou=sudoers,{{ dc_distinguished_name }}"
       attributes:
         sudoHost: ALL
         sudoUser: '%ldapUsers'
         sudoCommand: ALL
   ```

### Role Variables - Helper Variables

- `dc_tld` (string)

   Name of the domain to be created.
   Sets the value of the `slapd/domain` debconf question of the slapd install.

   Defaults to `{{ dc_org }}.{{ dc_suffix }}`.

- `dc_hostname` (string)

   Hostname of the domain controller.

   Defaults to `{{ ansible_hostname }}`

- `dc_domain_controller_uri` (string)

   Fully qualified hostname of the domain controller.

   Defaults to `{{ dc_hostname }}.{{ dc_tld }}`

- `dc_distinguished_name` (string)

   Distinguished name of the domain to be created.

   Defaults to `dc={{ dc_org }},dc={{ dc_suffix }}`.

- `dc_admin_distinguished_name` (string)

   Distinguished name of the default domain admin account.

   Defaults to `cn=admin,{{ dc_distinguished_name }}`

- `dc_packages` (list of string)

   List of packages to be installed as part of the role.

   Defaults to:

   - ldap-utils
   - slapd
   - python-pip
   - libldap2-dev
   - python2.7-dev
   - libsasl2-dev

### Role Variables - debconf Settings

- `dc_purge_database` (boolean)

   Sets the answer to the `slapd/purge_database` debconf question of the slapd install.

   Defaults to `false`.

- `dc_backend` (string)

   Sets the answer to the `slapd/backend` debconf question of the slapd install.

   Defaults to `MDB`.

- `dc_no_configuration` (boolean)

   Sets the answer to the `slapd/no_configuration` debconf question of the slapd install.

   Defaults to `false`.

- `dc_policy_schema_needs_update` (string)

   Sets the answer to the `slapd/ppolicy_schema_needs_update` debconf question of the slapd install.

   Defaults to `abort installation`.

- `dc_dump_database` (string)

   Sets the answer to the `slapd/dump_database` debconf question of the slapd install.

   Defaults to `when needed`.

- `dc_database_dump_directory` (string)

   Sets the answer to the `slapd/dump_database_destdir` debconf question of the slapd install.

   Defaults to `/var/backups/slapd-VERSION`.

- `dc_invalid_config` (boolean)

   Sets the answer to the `slapd/invalid_config` debconf question of the slapd install.

   Defaults to `true`.

- `dc_move_old_database` (boolean)

   Sets the answer to the `slapd/move_old_database` debconf question of the slapd install.

   Defaults to `true`.

- `dc_default_admin_password` (string)

   Sets the answer to the following debconf questions of the slapd install:

   - `slapd/internal/adminpw`
   - `slapd/password1`
   - `slapd/password2`
   - `slapd/internal/generated_adminpw`

   Defaults to `Password1234%`.  
   
   Note: This value is overwritten by the value of `dc_domain_admin_password` after the installation of slapd is complete.

License
-------

BSD
