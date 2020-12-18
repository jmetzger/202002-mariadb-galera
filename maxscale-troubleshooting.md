# MaxScale Troubleshooting 

## Ref:

 * https://mariadb.com/kb/en/maxscale-troubleshooting/

## Access denied errors for user root!

```
If you want to connect as root, you need to set enable_root_user=1 in the service declaration within your MaxScale.cnf.
```

## Access denied for user@localhost

```
If you have added your users with wildcard host in MariaDB, you may try the localhost_match_wildcard_host=1 option in the service declaration within your maxscale.cnf. Also read the MaxScale documentation regarding permissions.
```

## Access denied on databases/tables containing underscores

```
There seems to be a bug for databases containing underscores. Connect as root and use "SHOW GRANTS FOR user".

GRANT SELECT ON `my\_database`.* TO 'user'@'%' <-- bad

GRANT SELECT ON `my_database`.* TO 'user'@'%' <-- good

If you got a grant containing a escaped underscore, you can add the `strip_db_esc=true` parameter to the service to automatically strip escape characters or just replace the grant with a unescaped one.

If you have a lot of grants you want to update, you can use the pt-show-grants utility from percona toolkit like that:

pt-show-grants -pyourpw | grep '\_' | sed 's/\_/_/g' > fixedgrants.sql
after reviewing fixedgrants.sql, you can apply them.
```
