**Interactive Mysql create user for existing db**
```
#!/bin/bash
BKDIR=/home/username/mysqltest
USER=username
PASS=password

#Ask user to enter database name and save input to dbname variable
read -p "Please Enter Database Name:" dbname

#checking if database exist
mysql -u$USER -p$PASS -Bse "USE $dbname" 2> /dev/null

#if database exist:
if [ $? -eq 0 ]; then

#ask user about username
read -p "Please enter the username you wish to create : " username

#ask user about allowed hostname
read -p "Please Enter Host To Allow Access Eg: %,ip or hostname : " host

#ask user about password
read -p "Please Enter the Password for New User ($username) : " password

#mysql query that will create new user, grant privileges on database with entered password
query="GRANT ALL PRIVILEGES ON $dbname.* TO $username@'$host' IDENTIFIED BY '$password'";

#ask user to confirm all entered data
read -p "Executing Query : $query , Please Confirm (y/n) : " confirm

#if user confims then
if [ "$confirm" == 'y' ]; then

#run query
mysql -u$USER -p$PASS -e "$query"

#update privileges, without this new user will be created but not active
mysql -u$USER -p$PASS -e "flush privileges"
else
#if user didn't confirm entered data
read -p "Aborted, Press any key to continue.."
#just exit
fi
else
#If database not exit – warn user and exit
echo "The Database: $dbname does not exist, please specify a database that exists";
fi
```
**Apache Installation with basic configuration**
```
-----------------------------------------BASH SCRIPT FOR INSTALLATION------------------------------------
#!/bin/bash
#sudo apt-get update
sudo apt-get install apache2 -y
sudo apt-get install php5 libapache2-mod-php5 php5-mcrypt php5-mysql php-pear php5-gd php5-curl php5-cgi php5-cli php5-common php5-json php5-memcache php5-memcached -y
sudo apt-get install sendmail
a2enmod rewrite
a2enmod headers
a2enmod expires
/etc/init.d/apache2 restart
touch /var/www/html/phpinfo.php
echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php
echo "Installation completed Check phpinfo with http://localhost/phpinfo.php"

-------------------------------------------Config--------------------------------------------------------
ServerAdmin webmaster@localhost
	DocumentRoot /home/ubuntu/deploy
        <Directory />
        Options FollowSymLinks
        AllowOverride All
        Require all granted	
        </Directory>

-------------------------------------------Apache 7 Installation ------------------------------------------
sudo apt-get install -y apache2
sudo apt-get install -y php7.0 libapache2-mod-php7.0 php7.0-mysql php7.0-curl php7.0-gd php7.0-intl php-pear php-imagick php7.0-imap php7.0-mcrypt php-memcache php-memcached php7.0-mbstring php-redis


ServerAdmin webmaster@localhost
	DocumentRoot /home/ubuntu/deploy
        <Directory />
        Options FollowSymLinks
        AllowOverride All
        Require all granted	
        </Directory>


sudo apt-get install apache2 libapache2-mod-php7.0 php7.0-cli php7.0-common php7.0-curl php7.0-gd php7.0-json php7.0-mcrypt php7.0-mbstring php7.0-mysql php7.0-xml php7.0-xmlrpc php7.0-zip php-apcu php-mongodb php-memcache php-memcached php-redis
sudo a2enmod headers expires rewrite vhost_alias




<VirtualHost *:80>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.
	#ServerName www.example.com
	UseCanonicalName Off

	ServerAdmin webmaster@localhost
	VirtualDocumentRoot /home/ubuntu/deploy/%1/public
        <Directory /home/ubuntu/deploy/>
        Options FollowSymLinks
        AllowOverride All
        Require all granted	
        </Directory>


	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```
**MySQL backup and upload to S3 Bucket**
```
#!bin/bash
#Create Variables.
FILE="$(date +"%d-%m-%Y-%H")"
YEAR="$(date +"%Y")"
MONTH="$(date +"%m")"
DAY="$(date +"%d")"
#########################
######TO BE MODIFIED#####
### System Setup ###
mkdir -p /root/backup/mysql_db/$YEAR/$MONTH/$DAY
BACKUP=/root/backup/mysql_db/$YEAR/$MONTH/$DAY

### MySQL Setup ###
MUSER="backup"
MPASS="p@ssw0rd123"
MHOST="localhost"

######DO NOT MAKE MODIFICATION BELOW#####
#########################################
### Binaries ###
TAR="$(which tar)"
GZIP="$(which gzip)"
FTP="$(which ftp)"
MYSQL="$(which mysql)"
MYSQLDUMP="$(which mysqldump)"

### Today + hour in 24h format ###
#NOW=$(date +"%d%H")

### Create dir for each databases, backup tables in individual files ###
for db in $(mysql --user=$MUSER --password=$MPASS -e 'show databases' -s --skip-column-names|grep -vi information_schema|grep -vi performance_schema);
do mysqldump --add-drop-table --allow-keywords --routines --skip-lock-tables -q -c --user=$MUSER --password=$MPASS --opt $db | gzip > "$BACKUP/$db.gz";
done

### Create txt for each databases, write file size and directory location to the txt file ###

ls -lh /root/backup/mysql_db/$YEAR/$MONTH/$DAY/*.gz | awk '{print $5, $9}' > /root/backup/mysql_log.txt 

### Backup Data to S3 Bucket ###

s3cmd sync -v /root/backup/ s3://bucketname/ 

### Write total data backed (MB’s) up to S3 bucket ###
s3cmd du -H s3://nanserverbackup/mysql_db/$YEAR/$MONTH/$DAY >> /root/backup/mysql_log.txt

### Send Email for the backup done to the owners ###
mail -s "DataBase (Subject Server) Rsync" -a /root/backup/mysql_log.txt email@gmail.com,email@gmail.com  << EOF
"Mysql Database BackUp Done and Database files copied to S3 bucket. Please see the attached log file for more details."
EOF


echo "Backup Done SucessFully"

### Delete files older than 3 Days ###
find /root/backup/mysql_db/* -mtime +3 -exec rm -rf {} \;
```
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
