# SQL alkalmazás

## Feladat

1. Adatbázis létrehozása
2. Tábla létrehozása (id,név, neptunkód, kor, )
3. Adatok felvitele
4. Adatok kiolvasása, és megjelenítése, a múltórán létrehozott Apache szerveren

## SQL

1. Létrehoztam egy új adatbáist sqlben
    - ```sql
        CREATE DATABASE db;
        ```
1. Létrehoztam az új táblát
    - ```sql
        CREATE TABLE `student` (
        `id` int NOT NULL,
        `name` varchar(255) DEFAULT NULL,
        `neptuncode` varchar(6) DEFAULT NULL,
        `age` int DEFAULT NULL,
        PRIMARY KEY (`id`)
        );
      ```
1. Feltöltöttem a táblát adatokkal
    - ```sql 
        INSERT INTO student VALUES(2, 'Gipsz Jakab', 'ABC123', 55 );
        ```
2. Létrehoztam egy új felhasznlót, amely a táblát csak olvasni tudja. Ezt fogja haszálni a webszerver.
    - ```sql 
        CREATE USER 'web_user'@'localhost' IDENTIFIED BY 'passw0rd';
        ```
    - ```sql
        GRANT SELECT ON db.student TO 'web_user'@'localhost';
        ```
    - ```sql
        FLUSH PRIVILEGES;
        ```

## PHP

Létrehoztam egy PHP szkriptet a webszerveren, amely kiolvassa az adatbázisból az adatokat, és megjeleníti egy táblázatban. 

```php
<html>
   <head>
      <title>Selecting Records</title>
   </head>
   <body>
      <?php
         $dbhost = 'localhost';
         $dbuser = 'web_user';
         $dbpass = 'passw0rd';
         $conn = mysqli_connect($dbhost, $dbuser, $dbpass);

         if(! $conn ) {
            die('Could not connect: ' . mysqli_error($conn));
         }
         
         mysqli_select_db( $conn, 'db' );
         $sql = "SELECT * FROM student WHERE id=2";
         $retval = mysqli_query( $conn, $sql );
         if(! $retval ) {
            die('Could not get data: ' . mysqli_error($conn);
         }
         echo "<table BORDER=1><tr><th>id</th><th>name</th><th>neptuncode</th><th>age</th></tr>";
         while($row = mysqli_fetch_array($retval)) {
                echo "<tr>";
                echo "<td>" . $row['id'] . "</td>";
                echo "<td>" . $row['name'] . "</td>";
                echo "<td>" . $row['neptuncode'] . "</td>";
                echo "<td>" . $row['age'] . "</td>";
                echo "</tr>";
         } 
         echo "</table>";
 
         mysqli_close($conn);
      ?>
   </body>
</html>

```


### Források:

<https://www.hostinger.com/tutorials/mysql/how-create-mysql-user-and-grant-permissions-command-line>
<https://www.tutorialspoint.com/php_mysql/php_mysql_select_records.htm>