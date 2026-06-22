# MySQL Debug Commands

## Check Current User

```sql
SELECT CURRENT_USER();
```

---

## Check MySQL Hostname

```sql
SELECT @@hostname;
```

---

## List All Users

```sql
SELECT user, host FROM mysql.user;
```

---

## Check Authentication Plugin

```sql
SELECT user, host, plugin
FROM mysql.user;
```

---

## Check Specific User Plugin

```sql
SELECT user, host, plugin
FROM mysql.user
WHERE user='root';
```

---

## Check User Permissions

```sql
SHOW GRANTS FOR 'root'@'%';
```

---

## Check Database List

```sql
SHOW DATABASES;
```

---

## Check Current Database

```sql
SELECT DATABASE();
```

---

## Check MySQL Version

```sql
SELECT VERSION();
```

---

## Check Bind Address

```sql
SHOW VARIABLES LIKE 'bind_address';
```

---

## Check Skip Name Resolve

```sql
SHOW VARIABLES LIKE 'skip_name_resolve';
```

---

## View Active Connections

```sql
SHOW PROCESSLIST;
```

---

## View Active Connections with Hosts

```sql
SELECT ID, USER, HOST, DB, COMMAND, TIME, STATE
FROM information_schema.PROCESSLIST;
```

---

## Create Application User

```sql
CREATE USER 'app_user'@'%' IDENTIFIED BY 'strong_password';
```

---

## Grant Permissions

```sql
GRANT ALL PRIVILEGES
ON app_database.*
TO 'app_user'@'%';
```

---

## Reload Privileges

```sql
FLUSH PRIVILEGES;
```

---

## Change Password

```sql
ALTER USER 'app_user'@'%' IDENTIFIED BY 'new_password';
FLUSH PRIVILEGES;
```

---

## Verify User Exists

```sql
SELECT user, host
FROM mysql.user
WHERE user='app_user';
```

---

## Test Connection Information

```sql
STATUS;
```

---

## Check Server Variables

```sql
SHOW VARIABLES;
```
