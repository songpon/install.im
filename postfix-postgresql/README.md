ubuntu install:

    install postgresql postfix-pgsql


Postfix install:

    select  "no configuration"


postgresql configuration

pg_hba.conf

    local   mailserver      all     peer map=mailmap

pg_ident.conf

    mailmap         postfix                 mailuser
    mailmap         root                    mailuser

##Create user/role/database

    sudo su - postgresl
    psql


```sql
    CREATE USER mailuser;
    REVOKE CREATE ON SCHEMA public FROM PUBLIC;
    REVOKE USAGE ON SCHEMA public FROM PUBLIC;
    GRANT CREATE ON SCHEMA public TO postgres;
    GRANT USAGE ON SCHEMA public TO postgres;
    CREATE DATABASE mailserver WITH OWNER mailuser;
```


##Create tables
code (psql -U mailuser -d mailserver)


```sql
CREATE SEQUENCE seq_mail_domain_id START 1;
CREATE SEQUENCE seq_mail_user_id START 1;
CREATE SEQUENCE seq_mail_alias_id START 1;
 
CREATE TABLE virtual_domains (
  domain_id INT2 NOT NULL DEFAULT nextval('seq_mail_domain_id'),
  domain_name varchar(50) NOT NULL,
  PRIMARY KEY (domain_id)
);
 
 
CREATE TABLE virtual_aliases (
  alias_id INT2 NOT NULL DEFAULT nextval('seq_mail_alias_id'),
  domain_id INT2 NOT NULL,
  source varchar(100) NOT NULL,
  destination varchar(100) NOT NULL,
  PRIMARY KEY (alias_id),
  FOREIGN KEY (domain_id) REFERENCES virtual_domains(domain_id) ON DELETE CASCADE
);

/* NOT YET USE 
CREATE TABLE virtual_users (
  user_id INT2 NOT NULL DEFAULT nextval('seq_mail_user_id'),
  domain_id INT2 NOT NULL,
  password varchar(106) NOT NULL,
  email varchar(100) NOT NULL,
  PRIMARY KEY (user_id),
  FOREIGN KEY (domain_id) REFERENCES virtual_domains(domain_id) ON DELETE CASCADE
);
*/

```

## create configuration file
code (cd /etc/postfix)

main.cf

```

#Virtual domains, and aliases
virtual_mailbox_domains = pgsql:/etc/postfix/pgsql-virtual-mailbox-domains.cf
virtual_alias_maps = pgsql:/etc/postfix/pgsql-virtual-alias-maps.cf
```

pgsql-virtual-alias-maps.cf  

```
user = mailuser
password = password
hosts = 127.0.0.1
dbname = mailserver
query = SELECT destination FROM virtual_aliases WHERE source='%s'
```

pgsql-virtual-mailbox-domains.cf
```
user = mailuser
password = sqlblabla
hosts = 127.0.0.1
dbname = mailserver
query = SELECT 1 FROM virtual_domains WHERE domain_name='%s'
```
## restart service
service postgresql restart
sudo service postfix restart

## test configuration

postmap -q install.im pgsql:/etc/postfix/pgsql-virtual-mailbox-domains.cf


refernces:

[1] (https://www.digitalocean.com/community/tutorials/how-to-set-up-a-postfix-email-server-with-dovecot-dynamic-maildirs-and-lmtp)
[2] (https://www.linode.com/docs/email/postfix/email-with-postfix-dovecot-and-mysql?r=b787a92a121f3d67ac0012480780d5f233148e9a)
[3] (http://www.devissues.com/postfix-dovecot-postgresql)
