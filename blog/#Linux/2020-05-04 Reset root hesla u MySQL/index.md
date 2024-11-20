# Reset root hesla u MySQL/MariaDB databáze

Zapomněli jste root heslo do své MySQL nebo MariaDB databáze? Pokud používáte password managera, tak se nic takového přece nemůže stát. Jenže, to by ho tam musel taky někdo uložit, že? No, naštěstí existuje jednoduchý způsob, jak root heslo pro databázi změnit.

Nejprve zastavíme databázový server:

```shell
sudo systemctl stop mysql
```

nebo

```shell
sudo systemctl stop mariadb
```

A spustíme ho znovu v safe režimu a bez načítání uživatelských oprávnění (každý se pak může přihlásit bez hesla, tak bacha na to):

```shell
sudo mysqld_safe --skip-grant-tables &
```

Nyní se přihlásíme do databáze bez hesla:

```shell
mysql -u root
```

V MySQL konzoli spustíme příkazy, které nastaví pro roota nové heslo:

```sql
FLUSH PRIVILEGES;
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('NOVE_HESLO');
FLUSH PRIVILEGES;
```

Nové heslo je nastavené a můžeme server bezpečně vypnout:

```shell
mysqladmin -u root -p shutdown
```

Následně už jen databázový server opět spustíme v normálním režimu a je dokonáno. Uživatel root se již může přihlásit pod novým heslem.

```shell
sudo systemctl start mariadb
```
