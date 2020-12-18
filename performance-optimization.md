## Performance Optimization 

### Slow query (good read) 

https://mariadb.com/kb/en/slow-query-log-overview/

## Innodb and general optimization 
https://www.percona.com/blog/2016/10/12/mysql-5-7-performance-tuning-immediately-after-installation/
http://schulung.t3isp.de/documents/pdfs/mysql/mysql-performance.pdf

## Is innodb_buffer_pool big enough 

  * you should always enough free pages 
  * 1 page is 16 KBytes. 

```
MariaDB [training]> pager grep -i  'free buffers '
PAGER set to 'grep -i  'free buffers ''
MariaDB [training]> show engine innodb status \G
Free buffers       15015
1 row in set (0.000 sec)

MariaDB [training]> pager
Default pager wasn't set, using stdout.
```
