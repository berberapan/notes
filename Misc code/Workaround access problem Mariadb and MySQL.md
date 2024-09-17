
Connect to the database with root.

```terminal
sudo mariadb
```

Create an admin account.

```terminal
CREATE USER 'admin'@'localhost' IDENTIFIED BY 'admin';
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' WITH GRANT OPTION;
```

You can use this admin account with password admin, basically the same access rights as the root.

If you want to use root, you have to first quit the root connection and reconnect with your new admin account.

Then remove the root

```terminal
drop user root@localhost;
```

Then recreate the root user

```terminal
CREATE USER 'root'@'localhost' IDENTIFIED BY 'somepassword';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION;
```

Then you can use the same process as above to delete your admin account.

Now you should be able to connect to things like DBeaver with your root user without the Access Denied error.

[AskUbuntu forum post](https://askubuntu.com/questions/1131286/problem-in-accessing-to-mysql)