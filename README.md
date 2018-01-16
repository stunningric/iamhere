**Check MySQL Connection with PHP**
```
<?php
$servername = "IP OR DOMAIN NAME";
$username = "username";
$password = "password";


// Create connection
$conn = new mysqli($servername, $username, $password);


// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "Connected successfully";
?> 
```
**Kill remote SSH user**
```
pkill -9 -t pts/0 
pkill -9 -t "tty id" you can find tty id with "w" command or "tty" to find your ssh session tty id  
```
**PHPINFO**
```
<?php
echo "<br>";
echo "Website URL :- ";
echo  $_SERVER['HTTP_HOST'];
echo "<br>";


echo "<br>";
echo "Website DocumentRoot :- ";
echo $_SERVER['DOCUMENT_ROOT'];
echo "<br>";
phpinfo();
?>
```
**Check particular line number in big file and comment particular line in file**
```
sed '5971 {s/^/#/}' bigfilename.sql > commentbigfilename-new.sql      #comment line number

head -5971 coomentbigfilename.sql | tail -1                           # check commented line number
```
**Sample NODEJS App**
```
After install node
# mkdir myweb && cd myweb
# npm init
# npm i express --save
# nano index.js

index.js file content 
var express = require('express');
var app = express();
app.get('/', function (req, res) {
  res.send('Hello World!');
});
app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});

# node index.js
```
