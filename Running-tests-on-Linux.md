- clone the Brackets repository  
- checkout the release branch  
- run _npm install & grunt setup`  
- run `grunt test-integration --suite=unit --shell=/opt/brackets/brackets`  
- rename result.xml to result-unit.xml  
- run `grunt test-integration --suite=extension --shell=/opt/brackets/brackets`  
- rename result.xml to result-extension.xml  
- run `grunt test-integration --suite=integration --shell=/opt/brackets/brackets`  
- rename result.xml to result-integration.xml  

### Install Apache & PHP for server smokes  

- https://help.ubuntu.com/community/ApacheMySQLPHP  
- https://help.ubuntu.com/community/ApacheMySQLPHP#Installing_PHP_5  
- Create info.php file in /var/www/html with this content
```php
<?php
phpinfo();
?>
```