Nextcloud
=========

This role installs and configures [Nextcloud](https://nextcloud.com/).
It is also capable of installing, enabling and configuring additional apps, creating initial users and groups and setting up external users from an LDAP directory.

Requirements
------------

This role requires Ansible 2.8 but has no further requirements on the controller.

Role Variables
--------------

* `nextcloud_use_pkg`  
  Whether to prefer the distribution's package of Nextcloud.
  Defaults to `true` but is set to `false` if the distribution is not known provide a package.
* `nextcloud_version`  
  What version of Nextcloud to install from <https://nextcloud.com/>.
  If left unset, the latest stable version is chosen.
  This setting is ignored when using a distribution package (cf. `nextcloud_use_pkg`).
* `nextcloud_web_root`  
  The directory where Nextcloud is installed.
  If a distribution package is used to install Nextcloud, this setting is ignored.
  Defaults to `/var/www/nextcloud/`.
* `nextcloud_data_path`  
  Path where Nextcloud's data is stored, such as users' files.
  Defaults to `/var/lib/nextcloud/data/`.
* `nextcloud_apps_path`  
  Path where apps downloaded from the app store are installed.
  Apps that come with Nextcloud or are installed via the distribution's package manager are *not* stored here.
  Defaults to `apps-appstore` in `nextcloud_web_root`.
  The apps directory may be placed outside `nextcloud_web_root`.
  Distribution packages may override this setting.
* `nextcloud_log_file`  
  Path to Nextcloud's log file.
  Defaults to `/var/log/nextcloud/nextcloud.log`.
* `nextcloud_skeleton_files`  
  Path to a directory on the Ansible controller that contains files that should be copied to each new user's file store.
  Optional.
  If not set, new users start without any files, not even Nextcloud's default files.
* `nextcloud_skeleton_path`  
  Path to a directory on the remote system where the files from `nextcloud_skeleton_files` are copied to.
  Defaults to `/var/lib/nextcloud/skeleton/`.
  To provide new users with Nextcloud's default files, set this to `{{ nextcloud_web_root }}/core/skeleton/`.
* `nextcloud_php_cli`  
  Name of the PHP command line interpreter.
  May be an absolute path or the name of a executable in `$PATH`.
  Defaults to `php`, which should rarely need changing.
* `nextcloud_php_cli_env`  
  A dictionary of environment variables and their values that should be set when running PHP cli commands.
  Optional.
* `nextcloud_user`, `nextcloud_group`  
  Name of the system user and group running Nextcloud.
  This is usually the web server's user, such as `www-data`, `www`, `apache`, `nginx` or `http`.
  Defaults to `www-data`.
* `nextcloud_database_type`  
  The type of database for Nextcloud to use.
  One of `sqlite`, `mysql` or `pgsql`.
  Defaults to `sqlite`, which is, however, not recommended except for very small instances.
* `nextcloud_database_host`  
  The host name of the database server.
  If the database runs on the same system as Nextcloud, this may also be an absolute path to the database server's UNIX socket.
  Defaults to `localhost`.
* `nextcloud_database_port`  
  The port where the database server is listening on.
  If not set, the default port for the respective database system is used.
* `nextcloud_database_user`  
  Name of the user account to connect to the database server.
  Defaults to `nextcloud`.
* `nextcloud_database_password`  
  Password for `nextcloud_database_user`.
  Optional.
* `nextcloud_database_name`  
  The name of the database for Nextcloud.
  Defaults to `nextcloud`.
* `nextcloud_database_table_prefix`  
  A prefix string to prepend to all database tables names.
  Only useful if other applications share the same database (i.e. `nextcloud_database_name`, not the database server).
  Empty by default.
* `nextcloud_enabled_apps`  
  A list of Nextcloud apps to enable.
  All apps in this list are automatically downloaded from the app store, if necessary.
  Empty by default, which disabled all but the bare minimum of apps.
  Note that even apps shipped with Nextcloud are disabled if they are not listed here.
* `nextcloud_extra_options`  
  A dictionary of additional `config.php` settings for Nextcloud.
  Values that are lists or dictionaries are automatically converted to PHP arrays.
  Optional.
* `nextcloud_app_options`  
  A dictionary of settings for specific apps.
  Keys are app names and values are in turn dictionaries of the settings for the app.
  Optional.
* `nextcloud_admin`
  Name of the user account created during initial Nextcloud setup.
  This account is granted administrative privileges.
  Defaults to `admin`.
* `nextcloud_admin_password`  
  The password for `nextcloud_admin`.
  Optional.
  If not supplied, a random password is generated and the account is disabled.
* `nextcloud_users`  
  A list of local user accounts to set up within Nextcloud.
  Each list entry is a dictionary with the following keys:
    * `name`: The user's internal name. Mandatory.  
    * `display_name`: The user's display name. Optional.  
    * `password`: The user's initial password. Mandatory.  
  Note that this can not be used to modify existing users.
  Optional.
* `nextcloud_groups`  
  A list of local groups to create within Nextcloud.
  Each list entry is a dictionary with the following keys:
    * `name`: The group's name. Mandatory.  
    * `users` A list of users to be added to the group. This may include users from an LDAP back end. Optional.  
  Note that this only adds users to the given groups.
  Memberships not listed here are left unchanged.
  Optional.
* `nextcloud_ldap_backends`  
  A list of LDAP authentication back ends to set up.
  Each list item is a dictionary of settings for the back end.
  Refer to `occ ldap:show-config` and the [documentation](https://docs.nextcloud.com/server/latest/admin_manual/configuration_user/user_auth_ldap.html) for valid options and their meaning.
  Optional.
* `nextcloud_redis_host`  
  If the `redis` PHP extension is detected, Redis is set up as a cache for transactional file locking and -- if APCu is not available -- local caching.
  This setting configures the host name of the Redis instance to connect to.
  It may also be an absolute path to a UNIX socket, if Redis runs on the same system as Nextcloud.
  If set to `false`, Redis is not used.
  Defaults to `localhost`.
* `nextcloud_redis_port`  
  The TCP port Redis runs on.
  Ignored if Redis is not used or `nextcloud_redis_host` is a path to a UNIX socket.
  Defaults to `6379`.
* `nextcloud_redis_password`  
  The password required to connect to to Redit.
  Ignored if Redis is not used.
  Optional.
* `nextcloud_redis_dbindex`  
  The index of the Redis database to use.
  Optional.
* `nextcloud_default_app`  
  The name of the app to open on login.
  If this is a list, Nextcloud picks the first enabled app.
  Defaults to `files`.
* `nextcloud_trusted_domains`  
  A list of domain names that are expected to refer to this system.
  Users can only log in to Nextcloud if they use one of these names.
  Defaults to the system's host name and fully qualified domain name as detected by Ansible.

Dependencies
------------

Nextcloud requires a web server and possibly additional software, such as a SQL database management system or in-memory cache.
As these dependencies are complex software of their own, they are not considered to be within the scope of this role.

Example Configuration
---------------------

The following is a short example for some of the configuration options this role provides:

```yaml
nextcloud_database_type: mysql
nextcloud_database_host: '/run/mysqld/mysqld.sock'
nextcloud_database_name: nextcloud
nextcloud_database_user: nextcloud
nextcloud_enabled_apps:
  - files_sharing
  - twofactor_u2f
  - twofactor_totp
  - contacts
  - calendar
nextcloud_extra_options:
  lost_password_link: disabled
  mail_smtpmode: sendmail
  mail_sendmailmode: pipe
  log_rotate_size: 1048576 # 1 MiB
  simpleSignUpLink.shown: false
  ldapUserCleanupInterval: 3600
  overwritehost: "{{ ansible_facts['fqdn'] }}"
nextcloud_app_options:
  dav:
    sendEventReminders: 'no'
    sendInvitations: 'no'
  user_ldap:
    background_sync_interval: 3600
nextcloud_ldap_backends:
  - ldapHost: localhost
    ldapPort: 389
    ldapAgentName: 'cn=nextcloud,ou=machines,dc=my,dc=domain'
    ldapAgentPassword: 'very secret'
    ldapBase: 'dc=my,dc=domain'
    ldapBaseUsers: 'ou=people,dc=my,dc=domain'
    ldapBaseGroups: 'ou=groups,dc=my,dc=domain'
    ldapUserFilter: '(&(objectClass=inetOrgPerson)(memberOf=cn=Nextcloud Users,ou=groups,dc=my,dc=domain))'
    ldapLoginFilter: '(&(objectClass=inetOrgPerson)(memberOf=cn=Nextcloud Users,ou=groups,dc=my,dc=domain)(uid=%uid))'
    ldapExpertUsernameAttr: uid
    ldapExpertUUIDUserAttr: entryUUID
    ldapExpertUUIDGroupAttr: entryUUID
nextcloud_users:
  - name: user1
    password: 'very secret'
nextcloud_groups:
  - name: admin
    users:
      - ldap_user1
```

License
-------

MIT
