## Ansible Role: postfix

**IMPORTANT**
_This role is a fork of the [debops.postfix](https://github.com/debops/debops/tree/master/ansible/roles/debops.postfix)
Ansible role. It's not supported by the DebOps project nor anyone else. Don't
open any issues! For the original postfix role checkout the [DebOps](https://github.com/debops/debops)
repository._

This role installs and manages [Postfix](http://postfix.org/), an SMTP server.

The `debops.postfix` role is designed to manage Postfix on different hosts in
a cluster, with different "capabilities". At the moment the role can configure
Postfix to act as:

* a null client: Postfix sends all mail to another system specified
  either via DNS MX records or an Ansible variable, no local mail is enabled
  (this is the default configuration);
* a local SMTP server: local mail is delivered to local user accounts;
* a network SMTP server: network access is enabled separately from other
  capabilities, to avoid exposing misconfigured SMTP server by mistake and
  becoming an open relay;
* an incoming MX gateway: Postfix will listen on the port 25 (default SMTP
  port) and process connections using `postscreen` daemon with automatic
  greylisting and optional RBL checking;
* an outgoing SMTP client: Postfix will relay outgoing mail messages to
  specified remote MX hosts, you can optionally enable SMTP client
  authentication, the passwords will be stored separate from the inventory in
  the `secret/` directory (see `debops.secret` role). Sender dependent
  authentication is also available.

More "capabilities" like user authentication, support for virtual mail,
spam/virus filtering and others will be implemented in the future.

This role can also be used as a dependency of other roles which then can
enable more features of the Postfix SMTP server for their own use. For
example, the `debops.mailman` role enables mail forwarding to the configured
mailing lists, and the `debops.smstools` role uses Postfix as mail-SMS gateway.


### Documentation

More information about `debops.postfix` can be found in the
[official debops.postfix documentation](http://docs.debops.org/en/latest/ansible/roles/ansible-postfix/docs/).


### Role dependencies

- `debops.ferm`
- `debops.secret`


### Example Playbook

```yaml
- name: Manage Postfix
  hosts: [ 'debops_service_postfix' ]
  become: True

  environment: '{{ inventory__environment | d({})
                   | combine(inventory__group_environment | d({}))
                   | combine(inventory__host_environment  | d({})) }}'

  roles:

    # This role along with 'debops.etc_aliases' can be used to maintain the
    # /etc/aliases database.
    #
    #- role: debops.etc_aliases/env
    #  tags: [ 'role::etc_aliases', 'role::secret' ]

    - role: debops.postfix/env
      tags: [ 'role::postfix', 'role::secret' ]

    - role: debops.secret
      tags: [ 'role::secret', 'role::postfix' ]
      secret__directories:
        - '{{ postfix__secret__directories }}'

    # Normally a 'debops.ferm' role would be here for 'debops.postfix'
    # to manage the firewall. You don't need it if you run the main
    # 'debops.postfix' playbook before yours.
    #
    #- role: debops.ferm
    #  tags: [ 'role::ferm', 'skip::ferm' ]
    #  ferm__dependent_rules:
    #    - '{{ etc_aliases__secret__directories }}'
    #    - '{{ postfix__ferm__dependent_rules }}'

    #- role: debops.etc_aliases
    #  tags: [ 'role::etc_aliases' ]

    - role: debops.postfix
      tags: [ 'role::postfix' ]
```


### Authors and license

`postfix` role was written by:
- Maciej Delmanowski | [e-mail](mailto:drybjed@gmail.com) | [Twitter](https://twitter.com/drybjed) | [GitHub](https://github.com/drybjed)

License: [GPLv3](https://tldrlegal.com/license/gnu-general-public-license-v3-%28gpl-3%29)
