[![CI](https://github.com/de-it-krachten/ansible-role-postfix/workflows/CI/badge.svg?event=push)](https://github.com/de-it-krachten/ansible-role-postfix/actions?query=workflow%3ACI)


# ansible-role-postfix

Install/manage Postfix


## Platforms

Supported platforms

- Red Hat Enterprise Linux 7<sup>1</sup>
- Red Hat Enterprise Linux 8<sup>1</sup>
- Red Hat Enterprise Linux 9<sup>1</sup>
- CentOS 7
- RockyLinux 8
- RockyLinux 9
- OracleLinux 8
- AlmaLinux 8
- AlmaLinux 9
- Debian 10 (Buster)
- Debian 11 (Bullseye)
- Ubuntu 18.04 LTS
- Ubuntu 20.04 LTS
- Ubuntu 22.04 LTS
- Fedora 35
- Fedora 36

Note:
<sup>1</sup> : no automated testing is performed on these platforms

## Role Variables
### defaults/main.yml
<pre><code>
# Activate ipv6
postfix_ipv6:                       true

# list of all domains hosted
postfix_domains:                    "{{ [ postfix_domain ] }}"

# Should we configure support for amavisd
postfix_amavisd:                    false

# smtp relay host to use
postfix_relay_use:                  false
postfix_relay_host:                 mail.example.com
postfix_relay_port:                 587
postfix_relay_user:                 relay_user
postfix_relay_pwd:                  relay_password


postfix_settings:
  myhostname:                       '{{ postfix_fqdn }}'
  mydomain:                         '{{ postfix_domain }}'
  myorigin:                         '$mydomain'
  home_mailbox:                     'mail/'
  mynetworks:                       "{{ '127.0.0.0/8' if not postfix_ipv6|bool else '127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128' }}"
  inet_interfaces:                  'all'
  inet_protocols:                   'ipv4'
  mydestination:                    'localhost.$mydomain, localhost, $myhostname, $mydomain'
  mailbox_command:                  '{{ postfix_dovecot_deliver }}'
  smtpd_sasl_type:                  'dovecot'
  smtpd_sasl_path:                  'private/auth'
  smtpd_sasl_local_domain:          ''
  smtpd_sasl_security_options:      'noanonymous'
  broken_sasl_auth_clients:         'yes'
  smtpd_sasl_auth_enable:           'yes'
  smtpd_tls_loglevel:               '1'
  smtpd_tls_key_file:               '{{ postfix_ssl_key }}'
  smtpd_tls_cert_file:              '{{ postfix_ssl_chain }}'
  smtpd_tls_received_header:        'yes'
  smtpd_tls_session_cache_timeout:  '3600s'
  tls_random_source:                'dev:/dev/urandom'
  virtual_alias_maps:               'hash:/etc/postfix/virtual'
  virtual_alias_domains:            "{{ postfix_domains | difference([postfix_domain]) | join(' ') }}"
  message_size_limit:               '52428800'
  mailbox_size_limit:               '0'
  header_checks:                    'regexp:/etc/postfix/header_checks'
  # amavis ==
  # content_filter:                   'smtp-amavis:[127.0.0.1]:10024'

  # smtp | outgoing
  relayhost: >-
    {{ (postfix_relay_host + ':' + postfix_relay_port|string) if postfix_relay_use|bool }}
  smtp_sasl_auth_enable:            'yes'
  smtp_sasl_security_options:       'noanonymous'
  smtp_sasl_password_maps:          'hash:/etc/postfix/sasl_passwd'
  smtp_use_tls:                     'yes'
  smtp_tls_security_level:          'encrypt'
  smtp_tls_note_starttls_offer:     'yes'
  smtp_tls_mandatory_protocols:     '!SSLv2,!SSLv3,!TLSv1,!TLSv1.1'
  smtp_tls_protocols:               '!SSLv2,!SSLv3,!TLSv1,!TLSv1.1'

  # smtpd | incoming
  smtpd_tls_mandatory_ciphers:      'high'
  smtpd_tls_mandatory_exclude_ciphers:
    - aNULL
    - eNULL
    - EXPORT
    - SSLv2
    - LOW
    - DES
    - RC4
    - MD5
    - PSK
    - 3DES
    - IDEA
    - EXP
    - PSK
    - aECDH
    - EDH-DSS-DES-CBC3-SHA
    - EDH-RSA-DES-CBC3-SHA
    - KRB5-DES
    - CBC3-SHA
  smtpd_tls_security_level:         'may'
  smtpd_tls_mandatory_protocols:    '!SSLv2,!SSLv3,!TLSv1,!TLSv1.1'
  smtpd_tls_protocols:              '!SSLv2,!SSLv3,!TLSv1,!TLSv1.1'

  # DH1024 ==
  smtpd_tls_dh1024_param_file:      '/etc/postfix/dh2048.pem'

  # SPAM ==
  smtpd_helo_required: 'yes'
#  smtpd_helo_restrictions:
#    - permit_mynetworks
#    - permit_sasl_authenticated
#    - check_helo_access hash:/etc/postfix/helo_access
#    - reject_invalid_helo_hostname
#    - reject_non_fqdn_helo_hostname
#    - reject_unknown_helo_hostname
#  smtpd_sender_restrictions:
#    - permit_mynetworks
#    - permit_sasl_authenticated
#    - check_client_access hash:/etc/postfix/rbl_override
#    - check_sender_access pcre:/etc/postfix/reject_domains
#    - reject_unknown_sender_domain
#    - reject_unknown_reverse_client_hostname
#    - reject_unknown_client_hostname
  smtpd_recipient_restrictions:
    - permit_mynetworks
    - permit_sasl_authenticated
    # - check_policy_service unix:postgrey/socket
    - reject_unauth_destination
    - reject_rhsbl_helo dbl.spamhaus.org
    - reject_rhsbl_reverse_client dbl.spamhaus.org
    - reject_rhsbl_sender dbl.spamhaus.org
    - reject_rbl_client zen.spamhaus.org

#  # dovecot / lmtp
#  mailbox_transport: lmtp:unix:private/dovecot-lmtp
#  smtputf8_enable: 'no'

  # Hardening
  disable_vrfy_command: 'yes'

# setting for rspamd
postfix_rspamd:
  smtpd_milters:                    'inet:localhost:11332'
  milter_default_action:            accept
  milter_protocol:                  6

# lists of accepted/rejected top-level domains
postfix_accept_tld: []
postfix_reject_tld: []

postfix_header_checks:
  - '10\.'
  - '172\.'
  - '192\.168\.'

postfix_templates:
  # virtual
  - src: "{{ postfix_virtual | default('virtual.j2') }}"
    dest: /etc/postfix/virtual
  # white lists
  - src: rbl_override.j2
    dest: /etc/postfix/rbl_override
  - src: helo_access.j2
    dest: /etc/postfix/helo_access
  # black lists
  - src: reject_domains.j2
    dest: /etc/postfix/reject_domains
  # sasl
  - src: sasl_passwd.j2
    dest: /etc/postfix/sasl_passwd
    owner: root
    group: root
    mode: '0600'
  # remove internal headers
  - src: header_checks.j2
    dest: /etc/postfix/header_checks

# Firewall ports to open for incoming traffic
postfix_fw_ports:
  - { port: 25, proto: tcp }
  - { port: 465, proto: tcp }
</pre></code>

### vars/family-RedHat.yml
<pre><code>
# list of postfix packages
postfix_packages:
  - postfix
  - postfix-perl-scripts
  - mailx
  - cyrus-sasl
  - cyrus-sasl-plain

# postfix service
postfix_service: postfix

# dovecot delivery command
postfix_dovecot_deliver: /usr/libexec/dovecot/deliver

# postfix log file location
postfix_maillog: /var/log/maillog
</pre></code>

### vars/family-Debian.yml
<pre><code>
# list of postfix packages
postfix_packages:
  - postfix
#  - postfix-perl-scripts
  - mailutils
#  - cyrus-sasl
#  - cyrus-sasl-plain
  - pflogsumm

# postfix service
postfix_service: postfix

# dovecot delivery command
postfix_dovecot_deliver: /usr/lib/dovecot/deliver

# postfix log file location
postfix_maillog: /var/log/mail.log
</pre></code>

### vars/family-RedHat-9.yml
<pre><code>
# list of postfix packages
postfix_packages:
  - postfix
  - postfix-perl-scripts
  - s-nail
  - cyrus-sasl
  - cyrus-sasl-plain

# postfix service
postfix_service: postfix

# dovecot delivery command
postfix_dovecot_deliver: /usr/libexec/dovecot/deliver

# postfix log file location
postfix_maillog: /var/log/maillog
</pre></code>



## Example Playbook
### molecule/default/converge.yml
<pre><code>
- name: sample playbook for role 'postfix'
  hosts: all
  become: "{{ molecule['converge']['become'] | default('yes') }}"
  vars:
    postfix_ipv6: False
    postfix_domain: example.com
    postfix_fqdn: host.example.com
    postfix_ssl_key: "{{ openssl_server_key }}"
    postfix_ssl_chain: "{{ openssl_server_crt }}"
  pre_tasks:
    - name: Create 'remote_tmp'
      file:
        path: /root/.ansible/tmp
        state: directory
        mode: "0700"
  roles:
    - cron
    - openssl
  tasks:
    - name: Include role 'postfix'
      include_role:
        name: postfix
</pre></code>
