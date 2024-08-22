# MYSql setup()

1. visit **_mysql.com_** and head to downloads section and click on mqsql installer.
2. once downloaded open the exe and click on custom install --> select **sql server, mysql work-bench & mysql shell** and click next and in next page click on execute and click next.
3. In **Type and Networking** leave it as default and click next.
4. In **Authentication Method** use **strong password as recommended** and hit next.
5. In **Accounts and Roles** give the **password** and remember it, as it the root password and hit next.
6. In **Windows and Services** keep it as it is and hit next
7. In **Server File Permission** click on the first option and click next.
8. In **Apply configurations** click on execute and click next.
9. In **Installation complete screen** click on finish
10. Once done we have to add the env variable for that click on **_windows_** --> **_edit system environment variables_** --> click on **_Envirnoment variables_** --> in **systems variables** double click on **path** --> click on **New** and paste the path which might look like cdrive/ProgrammingFiles/MYSQL/mySql-server/bin --> click ok --> click ok --> click ok.
11. Open cmd and check version my using command **_mysql --version_**.
12. In cmd enter `mysql -u root -p` and enter the root passowrd in my case 8**\*\*\***96@A\*i and hit enter then you can see as below.
    ```
    Welcome to the MySQL monitor. Commands end with ; or \g.
    Your MySQL connection id is 11
    Server version: 8.0.39 MySQL Community Server - GPL
    ```

Copyright (c) 2000, 2024, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

```

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```
