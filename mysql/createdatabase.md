```
UPDATE user SET password=PASSWORD('ergfdks6624jkho4') WHERE user='sdk';

CREATE DATABASE `mail` CHARACTER SET utf8 COLLATE utf8_general_ci; 
CREATE USER 'mail'@'%' IDENTIFIED BY 'y2*%478XS^Sg%asV'; 
GRANT ALL ON `mail`.* TO 'mail'@'%' WITH GRANT OPTION; 
FLUSH PRIVILEGES;

# create a readonly
CREATE USER 'guest'@'%' IDENTIFIED BY '123456'; 
GRANT SELECT ON *.* TO 'guest'@'%' WITH GRANT OPTION; 
FLUSH PRIVILEGES;
```
