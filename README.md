**TAR exclude command**
```
tar -zcv --exclude='file1' --exclude='patter*' --exclude='file2' -f /backup/filename.tgz .	


tar -zcvf filename.tar.gz foldername --exclude="cache" --exclude="capcha" --exclude="css" --exclude="css_secure" --exclude="js" --exclude="tmp"  --exclude="catalog/product/cache"
```
**Magento Permission**
```
sudo find var/ -type f -exec chmod 600 {} \; 
sudo find media/ -type f -exec chmod 600 {} \;
sudo find var/ -type d -exec chmod 700 {} \; 
sudo find media/ -type d -exec chmod 700 {} \;
```
**S3 Bucket Allowed 2 IP**
```
{
	"Version": "2008-10-17",
	"Id": "S3PolicyId1",
	"Statement": [
		{
			"Sid": "IPDeny",
			"Effect": "Deny",
			"Principal": {
				"AWS": "*"
			},
			"Action": "s3:*",
			"Resource": "arn:aws:s3:::dblogs.domainname.com/*",
			"Condition": {
				"NotIpAddress": {
					"aws:SourceIp": ["192.168.0.12/32", "192.168.0.122/32"]
				}
			}
		}
	]
}
```
**MongoDB Backup Script**
```
#!/bin/bash
#Create Variables.
FILE="$(date +"%d-%m-%Y-%H")"
YEAR="$(date +"%Y")"
MONTH="$(date +"%m")"
DAY="$(date +"%d")"
TIMESTAMP=`date +%F-%H%M`
### System Setup ###
mkdir -p /home/ubuntu/mongodbbackup
BACKUP=/home/ubuntu/mongodbbackup
cd /home/ubuntu/mongodbbackup
 
#Force file syncronization and lock writes
#mongo admin --eval "printjson(db.fsyncLock())"
 
MONGODUMP_PATH="/usr/bin/mongodump"
MONGO_HOST="IP or Domainname" #replace with your server ip
MONGO_PORT="27017"
MONGO_DATABASE="dbname" #replace with your database name
 
 
# Create backup
$MONGODUMP_PATH -h $MONGO_HOST:$MONGO_PORT -d $MONGO_DATABASE
 
# Add timestamp to backup
mv dump mongodb-$MONGO_HOST-$TIMESTAMP
tar zcvf mongodb-$MONGO_HOST-$TIMESTAMP.tar.gz mongodb-$MONGO_HOST-$TIMESTAMP


# Backup Data to S3 Bucket
s3cmd sync -v /home/ubuntu/mongodbbackup/*.tar.gz s3://domainname-mongodb-backup/


s3cmd du -H s3://domainname-mongodb-backup/ >> /home/ubuntu/mongodbbackup/mongodb_log$TIMESTAMP.txt


# Send Email for the backup done to the owners.
#mail -s "DYRCT.COM MongoDB Backup" -a /home/ubuntu/mongodbbackup/mongodb_log$TIMESTAMP.txt rchauhan@theitideas.com,second@theitideas.com,third@theitideas.com  << EOF
#"MongoDB Database backup done and Database files copied to S3 bucket. Please see the attached log file for more details."
#EOF


mail -s "Domainname MongoDB Backup-$TIMESTAMP" rchauhan@theitideas.com,second@theitideas.com  << EOF
MongoDB Database backup done and Database files copied to S3 bucket below is the total size of backup bucket.
`cat /home/ubuntu/mongodbbackup/mongodb_log$TIMESTAMP.txt`
EOF


### Delete files older than 3 Days ###
find /home/ubuntu/mongodbbackup/* -mtime +5 -exec rm -rf {} \;


 
# Upload to S3

#S3_BUCKET_NAME="dyrct-mongodb-backup" #replace with your bucket name on Amazon S3
#S3_BUCKET_PATH=""

#s3cmd put mongodb-$HOSTNAME-$TIMESTAMP.tar 
#s3://$S3_BUCKET_NAME/$S3_BUCKET_PATH/mongodb-$HOSTNAME-$TIMESTAMP.tar
 
#Unlock database writes
#mongo admin --eval "printjson(db.fsyncUnlock())"
```
**MySQL Backup Script**
```
#!/bin/bash
 
USER="root"
PASSWORD="root"
OUTPUT="/Users/rakesh/dumps"
 
#rm "$OUTPUT/*gz" > /dev/null 2>&1
 
databases=`mysql --user=$USER --password=$PASSWORD -e "SHOW DATABASES;" | tr -d "| " | grep -v Database`
 
for db in $databases; do
    if [[ "$db" != "information_schema" ]] && [[ "$db" != _* ]] ; then
        echo "Dumping database: $db"
        mysqldump --force --opt --user=$USER --password=$PASSWORD --databases $db > $OUTPUT/`date +%Y%m%d`.$db.sql
        gzip $OUTPUT/`date +%Y%m%d`.$db.sql
    fi
done
```
**Apache script for check uptime**
```
#!/bin/sh
SERVICE='apache2'
email=rakesh.chauhan@domainname.com
time="$(date +"%r")"
date="$(date +'%d/%m/%Y')"

if ps ax | grep -v grep | grep $SERVICE > /dev/null
then
    echo "Apache service running, everything is fine"
else
    /etc/init.d/apache2 start  
    if ps ax | grep -v grep | grep $SERVICE > /dev/null  
then
subject="Apache has been started"
echo "Apache Server has been started @ "$date $time" " | mail -s "$subject" $email
else
subject="Apache is not running"
echo "Apache Server is stopped and cannot be started!!! @ "$date $time" " | mail -s "$subject" $email
  #  echo "$SERVICE started "
#    echo "$SERVICE is not running!" | mail -s "$SERVICE down" root
fi
fi

```
**Check MySQL running or not | BASH Script**
```
<?php
$servername = "localhost";
$username = "username";
$password = "password";

// Create connection
conn = new mysqli($servername, $username, $password);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "Connected successfully";
?>
```
**Gitpull Script**
```
#!/bin/sh
time="$(date +"%R")"
date="$(date +'%d%m%Y')"
cd /home/ubuntu/deploy
git checkout master
git stash
sleep 1m
git pull > /home/ubuntu/script/gitlogs/"$date"_"$time"_GIT_Pull.log 2>&1

#Upload images, CSS, JS Files to S3 bucket for Cloudfront serve
s3cmd sync "/home/ubuntu/deploy/public/js/" "s3://bucketname/js/" --add-header=Cache-Control:max-age=86400
s3cmd sync "/home/ubuntu/deploy/public/images/" "s3://bucketname/images/" --add-header=Cache-Control:max-age=86400
s3cmd sync "/home/ubuntu/deploy/public/styles/" "s3://bucketname/styles/" --add-header=Cache-Control:max-age=86400
s3cmd sync "/home/ubuntu/deploy/public/img/" "s3://bucketname/img/" --add-header=Cache-Control:max-age=86400

# Delete old log files
cd /home/ubuntu/script/gitlogs/
find /home/ubuntu/script/gitlogs/* -mtime +10 -exec rm -rf {} \;

------>>>>     Crontab <<<<-------


 
Set crontab for reboot 
@reboot sh /home/ubuntu/script/gitpull.sh

```
**Assign apache group to your user**
```
usermod -g www-data ubuntu
```
**Find Command**
```
find /var/www/ -size  +512M -exec du -sh {} \;
find /var/www/ ! -name "*.jpg" -size +10M -exec du -sh {} \;
```
**Tomcat VirtualHOST with JAVA variable**
```
</Host>
         <Host name="www.rakesh.com"  appBase="webapps/rakesh"
            unpackWARs="true" autoDeploy="true">
        <Context path="" docBase="/var/lib/tomcat8/webapps/rakesh"/>
        </Host>



JAVA_HOME=/usr/local/java/jdk1.8.0_121
PATH=$PATH:$JRE_HOME/bin:$JAVA_HOME/bin
export JAVA_HOME
export PATH
        


        sudo update-alternatives --install "/usr/bin/java" "java" "/usr/local/java/jdk1.8.0_121/bin/java" 1 
        sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/local/java/jdk1.8.0_121/bin/javac" 1
        sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/local/java/jdk1.8.0_121/bin/javaws" 1
        sudo update-alternatives --set java /usr/local/java/jdk1.8.0_121/bin/java 
        sudo update-alternatives --set javac /usr/local/java/jdk1.8.0_121/bin/javac 
        sudo update-alternatives --set javaws /usr/local/java/jdk1.8.0_121/bin/javaws 


https://www.ntu.edu.sg/home/ehchua/programming/howto/Ubuntu_HowTo.html#tomcat
https://www.ntu.edu.sg/home/ehchua/programming/howto/Tomcat_HowTo.html#configure
https://www.howtoforge.com/tutorial/how-to-install-apache-tomcat-8-5-on-ubuntu-16-04/
http://stackoverflow.com/questions/4756039/how-to-change-the-port-of-tomcat-from-8080-to-80
http://thelinuxfaq.com/191-how-to-set-or-increase-memory-heap-size-in-apache-tomcat
https://tecadmin.net/configure-ssl-certificate-in-tomcat/
https://tomcat.apache.org/tomcat-8.0-doc/ssl-howto.html
```
**Install LetsEncrypt SSL in Apache Tomcat**
```
Generate keystore file 

keytool -genkey -alias root -keyalg RSA -keystore /etc/ssl/keystore/.tomcat-indianic-org-keystore


Make changes in below file as per your need and run as sh file 


#!/bin/bash
#Please modify these values according to your environment
certdir=/etc/letsencrypt/live/tomcat.indianic.org #just replace the domain name after /live/
keytooldir=/usr/local/jdk/bin #java keytool located in jre/bin
mydomain=tomcat.indianic.org #put your domain name here
myemail=rakesh.chauhan@indianic.com #your email
networkdevice=ens33 #your network device  (run ifconfig to get the name)
keystoredir=/etc/ssl/keystore/.tomcat-indianic-org-keystore #located in home dir of user that you Tomcat is running under - just replace jira with your user you use for Tomcat, see ps -ef to get user name if you do not know


#the script itself:
cd /var/git/letsencrypt
git pull origin master




./letsencrypt-auto certonly --standalone --agree-tos --rsa-key-size 4096 -d $mydomain --standalone-supported-challenges http-01 --http-01-port 80 --renew-by-default --email $myemail --renew-by-default 




$keytooldir/keytool -delete -alias root -storepass googlecom -keystore $keystoredir
$keytooldir/keytool -delete -alias tomcat -storepass googlecom -keystore $keystoredir


openssl pkcs12 -export -in $certdir/fullchain.pem -inkey $certdir/privkey.pem -out $certdir/cert_and_key.p12 -name tomcat -CAfile $certdir/chain.pem -caname root -password pass:aaa


$keytooldir/keytool -importkeystore -srcstorepass aaa -deststorepass googlecom -destkeypass googlecom -srckeystore $certdir/cert_and_key.p12 -srcstoretype PKCS12 -alias tomcat -keystore $keystoredir
$keytooldir/keytool -import -trustcacerts -alias root -deststorepass googlecom -file $certdir/chain.pem -noprompt -keystore $keystoredir
```
**Backup CODEBASE with 3 days retation and upload to S3 Bucket**
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
mkdir -p /home/ubuntu/wwwbackup/$YEAR/$MONTH/$DAY
BACKUP=/home/ubuntu/wwwbackup/$YEAR/$MONTH/$DAY

### Binaries ###
TAR="$(which tar)"
GZIP="$(which gzip)"
FTP="$(which ftp)"
### Today + hour in 24h format ###
#NOW=$(date +"%d%H")

### Create hourly dir ###


### Create dir for each databases, backup tables in individual files ###


cd /var/www/html/
tar zcvf $BACKUP/project.tar.gz project/

ls -lh /home/ubuntu/wwwbackup/$YEAR/$MONTH/$DAY/*.gz | awk '{print $5, $9}' > /home/ubuntu/wwwbackup/$YEAR/$MONTH/$DAY/www_log_$YEAR_$MONTH_$DAY.txt

# Backup Data to S3 Bucket
s3cmd sync -v /home/ubuntu/wwwbackup/ s3://project.com-code/

s3cmd du -H s3://project.com-code/wwwbackup/$YEAR/$MONTH/$DAY >> /home/ubuntu/wwwbackup/www_log.txt

# Send Email for the backup done to the owners.
mail -s "Project.com Code Backup" -a /home/ubuntu/mysqlbackup/mysql_log.txt rakesh.chauhan@project.com  << EOF
"Project.com Code backup has been done and Code files copied to project.com S3 bucket."
EOF

# Delete files older than 7 Days
find /home/ubuntu/wwwbackup/* -mtime +1 -exec rm -f {} \;
```
**SSH Tunnel**
```
ssh -C -L 3306:remote.mysql.com:3306 -i myaws.pem ubuntu@remoteserver
```
**SET JAVA_HOME**
```
export JAVA_HOME=/usr/local/jdk
export PATH=$PATH:$JAVA_HOME/bin
export JAVA_OPTS="-Xms128m -Xmx512m"
```
**Install JAVA in Ubuntu**
```
Introduction
As a lot of articles and programs require to have Java installed, this article will guide you through the process of installing and managing different versions of Java.
Installing default JRE/JDK
This is the recommended and easiest option. This will install OpenJDK 6 on Ubuntu 12.04 and earlier and on 12.10+ it will install OpenJDK 7.
Installing Java with apt-get is easy. First, update the package index:
sudo apt-get update
Then, check if Java is not already installed:
java -version
If it returns "The program java can be found in the following packages", Java hasn't been installed yet, so execute the following command:
sudo apt-get install default-jre
This will install the Java Runtime Environment (JRE). If you instead need the Java Development Kit (JDK), which is usually needed to compile Java applications (for exampleApache Ant, Apache Maven, Eclipse and IntelliJ IDEA execute the following command:
sudo apt-get install default-jdk
That is everything that is needed to install Java.
All other steps are optional and must only be executed when needed.
Installing OpenJDK 7 (optional)
To install OpenJDK 7, execute the following command:
sudo apt-get install openjdk-7-jre 
This will install the Java Runtime Environment (JRE). If you instead need the Java Development Kit (JDK), execute the following command:
sudo apt-get install openjdk-7-jdk
Installing Oracle JDK (optional)
The Oracle JDK is the official JDK; however, it is no longer provided by Oracle as a default installation for Ubuntu.
You can still install it using apt-get. To install any version, first execute the following commands:
sudo apt-get install python-software-properties
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
Then, depending on the version you want to install, execute one of the following commands:
Oracle JDK 6
This is an old version but still in use.
sudo apt-get install oracle-java6-installer
Oracle JDK 7
This is the latest stable version.
sudo apt-get install oracle-java7-installer
Oracle JDK 8
This is a developer preview, the general release is scheduled for March 2014. This external article about Java 8 may help you to understand what it's all about.
sudo apt-get install oracle-java8-installer
Managing Java (optional)
When there are multiple Java installations on your Droplet, the Java version to use as default can be chosen. To do this, execute the following command:
sudo update-alternatives --config java
It will usually return something like this if you have 2 installations (if you have more, it will of course return more):
There are 2 choices for the alternative java (providing /usr/bin/java).

Selection    Path                                            Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-7-oracle/jre/bin/java          1062      auto mode
  1            /usr/lib/jvm/java-6-openjdk-amd64/jre/bin/java   1061      manual mode
  2            /usr/lib/jvm/java-7-oracle/jre/bin/java          1062      manual mode

Press enter to keep the current choice[*], or type selection number:
You can now choose the number to use as default. This can also be done for the Java compiler (javac):
sudo update-alternatives --config javac
It is the same selection screen as the previous command and should be used in the same way. This command can be executed for all other commands which have different installations. In Java, this includes but is not limited to: keytool, javadoc and jarsigner.
Setting the "JAVA_HOME" environment variable
To set the JAVA_HOME environment variable, which is needed for some programs, first find out the path of your Java installation:
sudo update-alternatives --config java
It returns something like:
There are 2 choices for the alternative java (providing /usr/bin/java).

Selection    Path                                            Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-7-oracle/jre/bin/java          1062      auto mode
  1            /usr/lib/jvm/java-6-openjdk-amd64/jre/bin/java   1061      manual mode
  2            /usr/lib/jvm/java-7-oracle/jre/bin/java          1062      manual mode

Press enter to keep the current choice[*], or type selection number:
The path of the installation is for each:
/usr/lib/jvm/java-7-oracle
/usr/lib/jvm/java-6-openjdk-amd64
/usr/lib/jvm/java-7-oracle
Copy the path from your preferred installation and then edit the file /etc/environment:
sudo nano /etc/environment
In this file, add the following line (replacing YOUR_PATH by the just copied path):
JAVA_HOME="YOUR_PATH"
That should be enough to set the environment variable. Now reload this file:
source /etc/environment
Test it by executing:
echo $JAVA_HOME
If it returns the just set path, the environment variable has been set successfully. If it doesn't, please make sure you followed all steps correctly.
```
**Check IP of Instances behind ELB**
```
<?php
exec('aws elb describe-instance-health --load-balancer-name benefit |grep "InstanceId"', $out);
$string = '';
foreach($out as $k=> $v) {
$string .= ' '. trim( str_replace('"InstanceId": ','', $v)).' ';
}

$new_str = str_replace(',','', $string);


exec('aws ec2 describe-instances --instance-ids '.$new_str.' | grep "PublicDnsName"',$put);

$arr = array();
foreach($put as $kk=> $vv) {
	$string1 = ''. trim( str_replace('"PublicDnsName": ' ,'', $vv)).'';
	if(!in_array($string1, $arr)) {
		array_push($arr, $string1);
	}
}

print_r($arr);
die;
?>
```
**GIT Basic Command**
```
git clone https://github.com/stunningric/testrepo.git
nano index.php
git commit -m "index.php"
git add index.php
git commit -m "index.php"
git push

Create Remote branch 
git branch dev
git push origin dev
```
**S3 Policy**
```
--------------------------------------------------------------------------------------------------------
BUCKET POLICY in S3 PERMISSION

{
	"Version": "2012-10-17",
	"Id": "Policy123453331623",
	"Statement": [
		{
			"Sid": "Stmt14445678325671",
			"Effect": "Allow",
			"Principal": "*",
			"Action": [
				"s3:GetObject",
				"s3:PutObject"
			],
			"Resource": "arn:aws:s3:::bucketname/*"
		}
	]
}


PERMISSION IN IAM 

Give Read only permission and put this policy in below option

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt144953456000",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::bucketname/*",
                "arn:aws:s3:::bucketname-code/*"
            ]
        }
    ]
}
---------------------------------------------------------------------------------------------------------------------
{
	"Version": "2012-10-17",
	"Id": "Policy1451885867327",
	"Statement": [
		{
			"Sid": "Stmt1451885860743",
			"Effect": "Allow",
			"Principal": "*",
			"Action": [
				"s3:DeleteObject",
				"s3:GetObject",
				"s3:PutObject"
			],
			"Resource": "arn:aws:s3:::bucketname.com/*"
		}
	]
}


IAM 


{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:ListAllMyBuckets",
            "Resource": "*"
        },
        {
            "Action": "s3:*",
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::bucketname.eu",
                "arn:aws:s3:::bucketname.eu/*"
            ]
        }
    ]
}


{
	"Version": "2012-10-17",
	"Id": "http referer policy example",
	"Statement": [
		{
			"Sid": "Allow get requests originating from www.alservicelink.com.",
			"Effect": "Allow",
			"Principal": "*",
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::bucketname/*",
			"Condition": {
				"StringLike": {
					"aws:Referer": [
						"https://www.domainame.com/*",
						"https://alservicelink.com/*",
								]
				}
			}
		}
	]
}
```
**Screen Help**
```
screen -ls						#Check screen
screen -r screenno.					#Attach screen	
Ctrl+ad							#Dettach screen
screen -X -S screenno. quit				#Quit screen 
```
**Apache config file in Linux Distro**
```
ServerRoot              ::      /etc/httpd
Primary Config Fle      ::      /etc/httpd/conf/httpd.conf
Other Config Files      ::      /etc/httpd/conf.d
Module Locations        ::      /usr/lib/httpd/modules
DocumentRoot            ::      /var/www/html
ErrorLog                ::      /var/log/httpd/error_log
AccessLog               ::      /var/log/httpd/access_log
cgi-bin                 ::      /var/www/cgi-bin (empty and disabled by default)
binary                  ::      /usr/sbin/httpd
runtime directory       ::      /etc/httpd/run
start/stop              ::      /sbin/service httpd {start|stop|restart|condrestart|reload|status|fullstatus|graceful|help|configtest}


Notes:
There is an extra config file in /etc/sysconfig/httpd which can be used to change to the worker mpm /usr/sbin/httpd.worker. 
Extra
 config files named *.conf are loaded from /etc/httpd/conf.d. This 
directory is used by packages like mod_python for drop-in configs 
If
 you're having issues with authorization and your permissions are 
correct, you might have problems with SELinux permissions. Take a look 
at httpd_selinux(8) and related documentation. Particularly sealert(8) can be used for analysis and suggested solutions. 


RedHat 9.0 and older:

ServerRoot              ::      /etc/httpd
Primary Config Fle      ::      /etc/httpd/conf/httpd.conf
DocumentRoot            ::      /var/www/html
ErrorLog                ::      /var/log/httpd/error_log
AccessLog               ::      /var/log/httpd/access_log
cgi-bin                 ::      /var/www/cgi-bin (empty and disabled by default)
binary                  ::      /usr/sbin/httpd
start/stop              ::      /sbin/service httpd {start|stop|restart|condrestart|reload|status|fullstatus|graceful|help|configtest}


Mandriva (Apache httpd 2.2):

ServerRoot              ::      /etc/httpd
Primary Config Fle      ::      /etc/httpd/conf/httpd.conf
DocumentRoot            ::      /var/www/html
ErrorLog                ::      /var/log/httpd/error_log
AccessLog               ::      /var/log/httpd/access_log
cgi-bin                 ::      /var/www/cgi-bin
binary                  ::      /usr/sbin/httpd
start/stop              ::      /sbin/service httpd
{start|stop|restart|reload|graceful|condreload|closelogs|update|condrestart|status|extendedstatus|configtest|configtest_vhosts|semcleanrestart|debug|show_defines}
```
**Check DB Size MySQL**
```
SELECT table_schemaTABLENAME, sum( data_length + index_length ) / 1024 / 1024 "Data Base Size in MB",sum( data_free )/ 1024 / 1024 "Free Space in MB" FROM information_schema.TABLES GROUP BY table_schema ;
```
**Setup SSL Certificate in Apache**
```
<VirtualHost *:443>
        ServerName www.domainname.com
        DocumentRoot /home/ubuntu/deploy
        <Directory />
        Options FollowSymLinks
        AllowOverride All
        Require all granted
        </Directory>
ErrorLog ${APACHE_LOG_DIR}/error_ssl.log
CustomLog ${APACHE_LOG_DIR}/access_ssl.log combined
SSLEngine on
SSLCertificateFile /etc/ssl/domainname/domainname.crt
SSLCertificateKeyFile /etc/ssl/domainname/domainname.key
SSLCertificateChainFile /etc/ssl/domainname/gd-bundle-ca.crt
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
Header set Access-Control-Allow-Origin "*"
```
**Install AWS ELB SSL Certificate**
```
aws iam upload-server-certificate --server-certificate-name certificate2017 --certificate-body file://domainname.crt --private-key file://domainname.com.key --certificate-chain file://IntermediateCA.crt --path /cloudfront/certificate/
```
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
