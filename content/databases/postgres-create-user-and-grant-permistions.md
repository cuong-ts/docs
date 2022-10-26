# Postgres create user and grant permistions

## List all user:

```bash
\du
```

## List all schema:

```bash
\dn
```

## Create user read\_only

```bash
create user ro_user with password 'pass_123';
```

## Grant permistion on schema

```bash
grant usage on schema common to ro_user;
grant select on all tables in schema common to ro_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA common GRANT SELECT ON TABLES TO ro_user;
```

##  DUMP & restore

```
pg_dump -a -h your_host -U your_user -W -Fc your_database > DATA.dump

pg_restore -a -h your_host -U your_user -W -Fc your_database < DATA.dump

```
