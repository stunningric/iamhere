**Check MongoDB connection with PHP**
```
<?php
//$server = "mongodb://username:password@localhost:27017/dbname";
$server = "mongodb://rakesh:rakesh123@localhost:27017/dbname";

$c = new MongoClient( $server );
 
if($c->connected)
    echo "Connected successfully";
else
    echo "Connection failed";
 ?>
 ```
**Check Mail without Authentication with Sendmail Package**
```
<?php
 $to = "rchauhan@theitideas.com";
 $subject = "Testmail !";
 $headers = "From: webmaster@example.com";
 $body = "Hi,\n\nThis is test mail !!!";
 if (mail($to, $subject, $body, $headers)) {
   echo("<p>Email successfully sent!</p>");
  } else {
   echo("<p>Email delivery failed…</p>");
  }
 ?>
 ```
**Check Mail with Authentication**
```
<?php
require_once "Mail.php";
$from = "Web Master <info@domainname.com>";
$to = "Rakesh <rchauhan@theitideas.com>";
$subject = "Test email using PHP SMTP\r\n\r\n";
$body = "This is a test email message";
$host = "smtp.domainname.com";
$username = "info@domainname.com";
$password = "password";
$headers = array ('From' => $from,
  'To' => $to,
  'Subject' => $subject);
$smtp = Mail::factory('smtp',
  array ('host' => $host,
    'auth' => true,
    'username' => $username,
    'password' => $password));
$mail = $smtp->send($to, $headers, $body);
if (PEAR::isError($mail)) {
  echo("<p>" . $mail->getMessage() . "</p>");
} else {
  echo("<p>Message successfully sent!</p>");
}
?>
-----------------------------------------------SECOND SCRIPT------------------------------------------------------------
require_once('scripts/phpmailer/class.phpmailer.php');


$mail = new PHPMailer();


$mail->IsSMTP();                       // telling the class to use SMTP


$mail->SMTPDebug = 0;
// 0 = no output, 1 = errors and messages, 2 = messages only.


$mail->SMTPAuth = true;                // enable SMTP authentication
$mail->SMTPSecure = "tls";              // sets the prefix to the servier
$mail->Host = "smtp.fsdata.se";        // sets Gmail as the SMTP server
$mail->Port = 26;                     // set the SMTP port for the GMAIL


$mail->Username = "info@moosek.com";  // Gmail username
$mail->Password = "wikmar";      // Gmail password


$mail->CharSet = 'windows-1250';
$mail->SetFrom ('info@moosek.com', 'Example.com Information');
$mail->AddBCC ( 'rakesh.chauhan@indianic.com', 'Example.com Sales Dep.');
$mail->Subject = $subject;
$mail->ContentType = 'text/plain';
$mail->IsHTML(false);


$mail->Body = $body_of_your_email; 
// you may also use $mail->Body = file_get_contents('your_mail_template.html');


$mail->AddAddress ('your.recipient@domain.com', 'Recipients Name');     
// you may also use this format $mail->AddAddress ($recipient);


if(!$mail->Send())
{
        $error_message = "Mailer Error: " . $mail->ErrorInfo;
} else 
{
        $error_message = "Successfully sent!";
}


// You may delete or alter these last lines reporting error messages, but beware, that if you delete the $mail->Send() part, the e-mail will not be sent, because that is the part of this code, that actually sends the e-mail
```
**Check MemCache with PHP**
```
<?php
$mc = new Memcached();
$mc->addServer("127.0.0.1", 11211);
 
$result = $mc->get("test_key");
 
if($result) {
  echo $result;
} else {
  echo "No data on Cache. Please refresh page pressing F5";
  $mc->set("test_key", "test data pulled from Cache!") or die ("Failed to save data at Memcached server");
}
?>
```
**compress file with bzip recursively.**
```
"find . -type f -exec bzip2 {} +"
```
**IP TABLES**
```
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT   :: not disconnect current ssh connection
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```
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
