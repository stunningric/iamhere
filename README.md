**MySQL Backup only schema and data**
```
mysqldump -u root -p --no-create-info dbname tablename1 tablename2 > data.sql     ------------ only DATA

mysqldump -u root -p --no-data dbname tablename1 tablename2 > schema.sql   ------------ only SCHEME
```
**PM2 Commands**
```
pm2 start python.py -n "Python"						#python olery_service.py
pm2 start npm -n "appname" -- start					#npm start
pm2 start npm -n "appname" -- start					#npm start
pm2 start start.sh -n "Bash"						#sh start.sh
PORT=3001 pm2 start npm -n "appname" -- start				#npm start PARAMETER1 PARAMETER2
sudo pm2 start http.js -n "proxy" --interpreter=/home/developer/node-v6.10.3/bin/node #sudo node http.js

Once run all application execute "pm2 startup" which gives generate another command to run as below

sudo env PATH=$PATH:/usr/local/bin /usr/local/n/versions/node/6.0.0/lib/node_modules/pm2/bin/pm2 startup systemd -u dev --hp /home/dev --service-name pm2

Once execute the command it will create service pm2, which will restart all app if server restart.

pm2 start server.js --name appname -- -e=dev . ------ for dev
Run pm2 with environment file configuration. create .env file with multiple environment like .env.live , .env.dev , .env.local
```
**MySQL Import CSV with LOAD INFILE**
```
LOAD DATA INFILE 'data.csv' INTO TABLE mytables  FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '\"' LINES TERMINATED BY '\n' IGNORE 1 LINES;
```
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
s3cmd sync -v /home/ubuntu/mongodbbackup/*.tar.gz s3://domainname-mongodb-backup/   ----- IF used s3cdm
aws s3 sync /home/ec2-user/MongodbBackup/ s3://petbubs-mongodb-backup/      ----- IF used aws cli

s3cmd du -H s3://domainname-mongodb-backup/ >> /home/ubuntu/mongodbbackup/mongodb_log$TIMESTAMP.txt


# Send Email for the backup done to the owners.
#mail -s "Server MongoDB Backup" -a /home/ubuntu/mongodbbackup/mongodb_log$TIMESTAMP.txt rchauhan@theitideas.com,second@theitideas.com,third@theitideas.com  << EOF
#"MongoDB Database backup done and Database files copied to S3 bucket. Please see the attached log file for more details."
#EOF


mail -s "Domainname MongoDB Backup-$TIMESTAMP" rchauhan@theitideas.com,second@theitideas.com  << EOF
MongoDB Database backup done and Database files copied to S3 bucket below is the total size of backup bucket.
`cat /home/ubuntu/mongodbbackup/mongodb_log$TIMESTAMP.txt`
EOF


### Delete files older than 3 Days ###
find /home/ubuntu/mongodbbackup/* -mtime +5 -exec rm -rf {} \;


 
# Upload to S3

#S3_BUCKET_NAME="server-mongodb-backup" #replace with your bucket name on Amazon S3
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

keytool -genkey -alias root -keyalg RSA -keystore /etc/ssl/keystore/.tomcat-domainname-org-keystore


Make changes in below file as per your need and run as sh file 


#!/bin/bash
#Please modify these values according to your environment
certdir=/etc/letsencrypt/live/tomcat.domainname.org #just replace the domain name after /live/
keytooldir=/usr/local/jdk/bin #java keytool located in jre/bin
mydomain=tomcat.domainname.org #put your domain name here
myemail=rakesh.chauhan@domainname.com #your email
networkdevice=ens33 #your network device  (run ifconfig to get the name)
keystoredir=/etc/ssl/keystore/.tomcat-domainname-org-keystore #located in home dir of user that you Tomcat is running under - just replace jira with your user you use for Tomcat, see ps -ef to get user name if you do not know


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
			"Sid": "Allow get requests originating from www.domainname.com.",
			"Effect": "Allow",
			"Principal": "*",
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::bucketname/*",
			"Condition": {
				"StringLike": {
					"aws:Referer": [
						"https://www.domainame.com/*",
						"https://domainname.com/*",
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
$mail->Host = "smtp.domainname.se";        // sets Gmail as the SMTP server
$mail->Port = 26;                     // set the SMTP port for the GMAIL


$mail->Username = "info@domainname.com";  // Gmail username
$mail->Password = "wikmar";      // Gmail password


$mail->CharSet = 'windows-1250';
$mail->SetFrom ('info@domainname.com', 'Example.com Information');
$mail->AddBCC ( 'rakesh.chauhan@domainname.com', 'Example.com Sales Dep.');
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
**Mongodb drop database**
```
mongo dbname --eval "db.dropDatabase()"

```
**Mongodb restore database**
```
mongorestore --db dbname --verbose /home/ec2-user/MongodbBackup/mongodb-localhost-2018-03-06-1003 ---BSON JSON file path

```
**Mongodb backup database**
```
mongodump -h localhost:27017 -d dbname

```
**rethinkdb backup database**
```
rethinkdb dump -c localhost:28015 -e dbname -f dbbackupname.tar.gz

```
**Change Webmin root password**
```
sudo /usr/libexec/webmin/changepass.pl /etc/webmin root NewPAssword

```
**Restore / Backup in AWS RDS MSSQL**
```
1) Create bucket for store restore and backup file.
2) Create custome "Option Group" for RDS Instance.
3) Add new option "SQLSERVER_BACKUP_RESTORE" in newly created Option Group. In this option it will ask you to create IAM role for access bucket from RDS Service as well.
4) Update new Option Group with your RDS Instance. and use below commands to backup or restore.

Restore Procedure : 
exec msdb.dbo.rds_restore_database 
        @restore_db_name='databasename', 
        @s3_arn_to_restore_from='arn:aws:s3:::bucketname/databasebackupfilename.bak';
Once you execute restore or backup command you will get task id in resule window. e.g. you will get task_id 1 then execute below command to check status of your restoration of database.

To check status of any task :
EXEC msdb.dbo.rds_task_status 
    @task_id = 1
    GO

Take Backup of Database :
exec msdb.dbo.rds_backup_database 
        @source_db_name='databasename',
        @s3_arn_to_backup_to='arn:aws:s3:::bucketname/databasebackupfilename.bak.bak',
        @overwrite_S3_backup_file=1,
        @type='FULL';

```
**Install Lamp in AWS Amazon Image**
```
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-lamp-amazon-linux-2.html

```
**Clear RAM Memory Cache, Buffer and Swap Space on Linux**
```
Every Linux System has three options to clear cache without interrupting any processes or services.

1. Clear PageCache only
sync; echo 1 > /proc/sys/vm/drop_caches
2. Clear dentries and inodes
sync; echo 2 > /proc/sys/vm/drop_caches
3. Clear PageCache, dentries and inodes
sync; echo 3 > /proc/sys/vm/drop_caches 

```
**Jenkins CI**
```
After installation of Jenkins, Login into jenkins user and create SSH key for passwordless login 
"ssh-keygen -t rsa"
It will generate private and public key for jenkins user.
Now, Put public key contents "id_rsa.pub" into remote server's user "authorized_keys" file. They you will be able to execute command in remote server with jenkins.

ssh -T user@serverip <<EOF
 sudo yum install httpd -y
EOF

```
**Sendmail funcion with AWS SES**
```
Ref : http://www.techoism.com/send-email-using-php-script-with-amazon-smtp-authentication/

Downloads : https://drive.google.com/file/d/1OQu7oo18m3kvv_3PULQlKeCOhTAVfYr1/view?usp=sharing

After download PHPMailer.zip extract it.
Open PHPMailer folder, It has multiple file into this folder. (Send_Mail.php,class.phpmailer.php,class.pop3.php,class.smtp.php,index.php)
Now, Open Send_Mail.php file and make changes as below 

1 ) $from = "rakesh@theitideas.com";
2 ) $mail->Host       = "tls://email-smtp.ap-southeast-1.amazonaws.com"; // Amazon SES server, note "tls://" protocol
3 ) $mail->Username   = "AWSSES USER";  // SES SMTP  username
4 ) $mail->Password   = "AWSSES PASSWORD";  // SES SMTP password
5 ) $mail->SetFrom($from, 'Rakesh Chauhan');

Once done open index.php file and add To Address for email sending.

1 ) $to = "rakesh.chauhan@theitideas.com";

Once done, Check open index.php, It will send email from SES to your receipt email id with mentioned From email id.


```
Ansible 
```
####Sample inventoryfile with private key
[server]
10.160.0.4 ansible_ssh_user=rakesh ansible_ssh_private_key_file=/home/rakesh/rakesh.pem

####Sample inventoryfile with private key
[server]
10.160.0.4 ansible_ssh_user=rakesh ansible_ssh_pass=/home/rakesh/rakesh.pem

####Sample file with hosts, group and group of group with Linux and Windows host
# DB Servers
sql_db1 ansible_host=sql01.xyz.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Lin$Pass
sql_db2 ansible_host=sql02.xyz.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Lin$Pass

# Web Servers
web_node1 ansible_host=web01.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass
web_node2 ansible_host=web02.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass
web_node3 ansible_host=web03.xyz.com ansible_connection=winrm ansible_user=administrator ansible_password=Win$Pass

# Groups
[db_nodes]
sql_db1
sql_db2

[web_nodes]
web_node1
web_node2
web_node3

[boston_nodes] 
sql_db1
web_node1

[dallas_nodes]
sql_db2
web_node2
web_node3

[us_nodes:children]
boston_nodes
dallas_nodes

```
Memory Commands
```
ps -eo ppid,pid,cmd,%cpu,%mem --sort=%mem | grep -v 0.0

ps axo rss,comm,pid \
| awk '{ proc_list[$2] += $1; } END \
{ for (proc in proc_list) { printf("%d\t%s\n", proc_list[proc],proc); }}' \
| sort -n | tail -n 10 | sort -rn \
| awk '{$1/=1024;printf "%.0fMB\t",$1}{print $2}'

```
Cheat Sheet
```
https://github.com/chubin/cheat.sh

```
OpenMediaVault Password Reset
```
Known Root Password
Login to the OMV using the root user and the current password via SSH or Console
enter the following command
passwd root
The new password is now active.

Unknown Root Password, but Admin Access to OMV GUI is Available
In this scenario we still can help ourselves with the GUI. The method we use is, that we create a cron job for the root user which then resets the password.

Navigate to System -> Cron Jobs
Press the +Add button
UN-tick the enabled box, so that the cronjob does not run automatically.
put into the command field the following line, replace newpasswd with your password:
echo "root:newpasswd" | chpasswd
press okay
select the newly created cron job
Click the run button.
in the opening window click the start button. It will shortly deactivate and activate again.
open ssh or console and login as root with your new password.
Root and Admin Password Unknown
If you do not know the root password, you need to boot with a Linux live CD like Knoppix or SystemRescueCD

Download a linux boot cd like SystemRescueCD
Boot from the SystemRescueCD
Enter the following commands:
 mkdir /omvroot
 mount /dev/sda1 /omvroot
 cd /omvroot/etc
Edit with your preferred editor (nano/vim) the file /omvroot/etc/shadow
vi /omvroot/etc/shadow
You will see a line like this (first line) (your encrypted key will be much longer than this)
root:dsfDSDF!s:12581:0:99999:7:::
Remove everything between the first and second ':' which is in this example 'dsfDSDF!s'. The line should look like this:
root::12581:0:99999:7:::
Final steps:
cd /
umount /omvroot
reboot
Now connect to OMV via ssh or console and login as root.


```
setfacl
```
# -R --Recursive
# -m --Modify permission
# -d Default
# setfacl -x ACL file/directory  	# remove only specified ACL from file/directory.
# setfacl -b  file/directory   		#removing all ACL from file/direcoty
--> add permission
setfacl -R -dm u:username:rx foldername

--> Remove permission 
setfacl -dx u:kuldeepqa: dev/

Remove whole ACL Permission
setfacl -b foldername

--> Multiple users permission
setfacl -m u:user1:rwx,g:mygroup:rwx,u:user3:r dir1/


--> Copy ACL from one file to other
getfacl dir1/ > copy.txt 
setfacl -M copy.txt dir2/
getfacl dir2/ 

getfacl file1 | setfacl --set-file=- file2

--> Give default permission (After give permission, Any data will generate that will get default permission by default)

setfacl -R -m u:rakeshdev:rwx dev/
setfacl -R -m d:u:rakeshdev:rwx dev/
setfacl -R -m u:kuldeepqa:rwx,u:rakeshdev:rwx,u:gajjarstag:r dev/
setfacl -R -m d:u:kuldeepqa:rwx,d:u:rakeshdev:rwx,d:u:gajjarstag:r dev/

https://portal.tacc.utexas.edu/tutorials/acls

```
By mistake given permission -R 777 to root "/" partition
```

1) Spin up a new instance from the same OS AMI, login to the instance and run the following:
2) sudo -s
3) apt-get install acl
4) cd /
5) getfacl -R ./ > test_permissions.txt
6) Now mount the root volume of your broken instance to the new instance on "/mnt/tempvol" and run the following.
7) cp test_permissions.txt /mnt/tempvol
8) cd /mnt/tempvol
9) setfacl --restore=test_permissions.txt /mnt/tempvol


```
Clear History in Linux
```
cat /dev/null > ~/.bash_history && history -c && exit

```
Change Key - Pair AWS
```

Method 1: Launching a clone from AMI - IP address will be changed
====================================================
Step1: Create an AMI of the instance: Select instance from EC2 console >> Actions >>Image >>Create image >> Give a name,image description in the fields, select no reboot >>click on create image
Step2: It will take sometime for the AMI to be created. You can see the AMI as available once its created under: EC2 console >>Left side menu >>Images>> AMI
Step3: Launch an instance from the AMI : Select the AMI >>click on launch, select same instance type and assign same security group of old instance. At the last step, it will ask you to select a key. Choose create a new key and launch the instance
Step4: Login to the new instance using the new key file.


Method 2: Using cloud-init as user data.
===============================
step1: Generate the key-pair from EC2 console and download it. Go to EC2 console-->Key pairs-->create keypair. Once the keypair is generated download it.
step2: Find the public key using the below command:

For example:

#:Downloads $ ssh-keygen -y -f keyname.pem 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCHHOAZ+GacPymnsW+AFqtgQJ3VaDpwdJ69wjlkWOOji6Dpx1HFZIJ9zrbVcliOmATA1ozgmGo2teXGCwUrPKV/stYOLkRAUqXVdMWkWoMr8Ogg1qbMi+5Yek5GMqUGZ/I536jcKds9M2XGCsFDXfikMe13XufivMRKkNAArRzUkCT7nCXqwsKRPm4MuD1KKnUwEsy1zxeALy7AXs2uSmTLs6Qz6q7OBrJD2AtwYWjqM76ei0JO+LJd9fsStbqSsXESN89S+9ZHt0tMtt5z8iGEbApCJ601KciTaLrRX58b6Hqk3VUiVpb2mSnDhY4rS9xIbSCwtS+a3UCuvAMKK9Cd

In the above output, keyname.pem is the file which is generated in step1

Step3: Stop the instance and add userdata. Select instance-->actions--->Instance settings-->view/change userdata.

For example:

#cloud-config
cloud_final_modules:
- [users-groups,always]
users:
  - name: ec2-user
    groups: [ ec2-user ]
    sudo: [ "ALL=(ALL) NOPASSWD:ALL" ]
    shell: /bin/bash
    ssh-authorized-keys: 
    - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCHHOAZ+GacPymnsW+AFqtgQJ3VaDpwdJ69wjlkWOOji6Dpx1HFZIJ9zrbVcliOmATA1ozgmGo2teXGCwUrPKV/stYOLkRAUqXVdMWkWoMr8Ogg1qbMi+5Yek5GMqUGZ/I536jcKds9M2XGCsFDXfikMe13XufivMRKkNAArRzUkCT7nCXqwsKRPm4MuD1KKnUwEsy1zxeALy7AXs2uSmTLs6Qz6q7OBrJD2AtwYWjqM76ei0JO+LJd9fsStbqSsXESN89S+9ZHt0tMtt5z8iGEbApCJ601KciTaLrRX58b6Hqk3VUiVpb2mSnDhY4rS9xIbSCwtS+a3UCuvAMKK9Cd

   In the above output,  you can see username and group name is set ec2-user. You need to copy public key from step2 and paste it under section ssh-authorized-keys.

   step4: Give read only permission to pem file where you downloaded it , in my case it is cloud-init.pem file.

   $ chmod 400 keyname.pem


  step5: start the instance. You should be able to ssh using new SSH key pair

  For example:

 $ ssh -i "keyname.pem" ec2-user@ec2-x-x-x-x.compute-1.amazonaws.com 

For more information, you can go thorugh the below link:

https://aws.amazon.com/premiumsupport/knowledge-center/ec2-user-account-cloud-init-user-data/ 

Method 3: Using recovery instance and copying keys - IP address will be changed
==============================================================
Step1: Either Create a AMI/snapshot of the root volume before proceeding further to ensure that you have backup in case something goes wrong
Step2: Launch a recovery instance (t2.micro) with the same Linux version Specify a new key file for the same.
Step3: Stop the problematic instance and detach the root volume from : EC2 console >> volumes >>actions >>detach
Step4: Attach the volume as secondary to the recovery instance : Select volume >>attach >>select recovery instance >>attach as /dev/sdf
step5: Mount the /dev/sdf to /mnt and copy the key of recovery instance to secondary volume using commands below

sudo mount /dev/xvdf1 /mnt
sudo cd /mnt/home/ec2-user/.ssh
sudo rm /authorized_keys
sudo cp /home/ec2-user/.ssh/authorized_keys /mnt/home/ec2-user/.ssh/authorized_keys
sudo cd /mnt/home/ec2-user/.ssh/
sudo chown ec2-user:ec2-user authorized_keys
sudo chmod 600 authorized_keys

Note: In the above commands, ec2-user is the username of the instance. This may vary depending upon the Linux distribution. So, I would request you to use it as per your Linux distribution.

Step6: Stop the recovery instance
Step7: Once it has stopped completely, detach the secondary root volume from recovery instance and attach it back to the problem instance as /dev/xvda
Step8: Start the problematic instance now. Now you will be able to connect with the same key you generated for the recovery instance in the problem instance.


```
Change in sudoers file
```
sed -i '26s/.*/\%sudo   ALL\=\(ALL\:\ALL\) NOPASSWD\: ALL/' /etc/sudoers

```
For loop
```
for i in `cat r.txt` ; do  ssh -o StrictHostKeyChecking=no username@$i "hostname;sudo /etc/init.d/apache2 restart;sudo /etc/inid.t/apache2 status" ; done


```
install java manually
```
After download .tag.gz file 
cd /usr/lib
tar -zxvf jdk-8u231-linux-x64.tar.gz
ln -s jdk1.8.0_231 java
vim /etc/environment
--- -->>> Add one line as below < --
JAVA_HOME="/usr/lib/java"
-----------------------------

update-alternatives --install "/usr/bin/java" "java" "/usr/lib/java/bin/java" 0
update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/java/bin/javac" 0
update-alternatives --set java /usr/lib/java/bin/java
update-alternatives --set javac /usr/lib/java/bin/javac
update-alternatives --list java
update-alternatives --list javac
java -version

```
Regular Expressions and Grep
```
^	  Match the beginning of the String
$	  Match the end of the String
*	  Match zero or more characters
?	  Match exactly one character


```
Docker Cheatsheet
```
docker run ngnix --> Download and start image
docker ps --> List all running docker container
docker ps -a --> List all docket container
docker stop container --> Stop container
docker rm container --> remove container permenantly 
docker images --> Available images
docker rmi imagename --> Remove image
docker pull nginx --> Just download image
docker exec containername cat /etc/hosts --> Execute remote command
docker run -d --name webapp ngnix --> Run in background
docker attach containerID --> Attach container again
docker run -it container bash --> login into container with bash 
docker run ngnix:4.0 --> If required purticuler version  (This is called tag) 
docker run -p 80:5000 nginx --> it will map port from host 80 to container 5000 port
docker run -v /opt/mydata:/var/lib/mysql mysql -> it will map local volume to container disk
docker inspect containerid --> For more details for container
docker logs containerid --> for logs
docker build -t webapp-color:lite . --> Build image from dockerfile
docker run --name db -e POSTGRES_PASSWORD=mysecretpassword -d postgres  --> pass environment 

CMD vs ENTRYPOINT  ::::
CMD = will run the command from dockerfile
ENTRYPOINT = we can put into dockerfile and pass the input while launching/run docker
If someone not provide the ENTRYPOINT input then we have to put both commands into dockerfile, CMD and ENTRYPOINT like below.
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
So, when user not provide ENTRYPOINT input it will take bydefault 5 because we have given 5 in CMD command

dokcer run -d --name mysql mysql
docker run -d --name nginx -p 80:80 --link db:mysql nginx
--link --> will create link and make host entry into host file to connect mysql database in mysql container

DOCKER-COMPOSE ::::
We can use docker-compose to create all containers with once and linked and mapped all ports

Like :
docker-compose.yml
db:
  image: mysql
nginx:
  image: nginx
  ports:
   - 80:80
  links: 
   - db

Docker-compose - Version ::::

docker-compose.yml (If we use service it will create dedicated network bridge and attach to each containers. So no need to put link into this version2.. It will work on DEPENDS_ON like below )
version: 2
services: 
db:
  image: mysql
nginx:
  image: nginx
  ports:
   - 80:80
  depends_on:
   - db


docker-compose.yml ( for separate networks in docker)
version: 2
services: 
db:
  image: mysql
  networks:
    - backend
nginx:
  image: nginx
  ports:
   - 80:80
  networks:
   -frontend

networks: 
  frontend:
  backend:


YAML:::: MUST have same Spaces everywhere

KEY-Pair Value 

Key: Value
Fruit: Apple
Vegetable: Carrot
Liquid: Water
Meat: Chicken


Array/ Lists
Fruits: 
- Orange
- Apple
- Banana

Vegetable:
- Carrot
- Cauliflower
- Tomato


Dictionary/ Map
Banana:
   Calories: 105
   Fat: 0.4 g
   Carbs: 27 G
Grape:
   Calories: 109
   Fat: 0.6 g
   Carbs: 12 G 


KeyValue / Dictionary/List
Fruits:
    - Banana:
        Calories: 105
        Fat: 0.4 g
        Carbs: 27 G
    - Grape:
        Calories: 109
        Fat: 0.6 g
        Carbs: 12 G 


docker -H=remoteIP-OR-domainname:2375 run nginx --> download image from private hosted docker registry

docker run --cpus=.5 ubuntu --> Limit container cpu usage
docker run --memory=100m ubuntu --> Limit container memory usage

docker stored data into host machine "/var/lib/docker"

docker volume create data_volume --> to create volume for docker

docker run -v data_volume:/var/lib/mysql mysql --> attach volume, and it will not delete if you delete docker

docker run --mount type=bind,source=/data/volume1,target=/var/lib/mysql mysql

docker history imageid --> to check command layer of image

docker system df --> check images, container and local volume size 

docker run -v /opt/data/:/var/lib/mysql -d --name mysql-db -e MYSQL_ROOT_PASSWORD=db_pass123 mysql  --> attach volume | set mysql password 

docker network ls --> check network 

docker network create --driver bridge --subnet 182.18.0.1/24 --gateway 182.18.0.1 wp-mysql-network ---> Create new network

docker run --network=wp-mysql-network -e DB_Host=mysql-db -e DB_Password=db_pass123 -p 38080:8080 --name webapp --link mysql-db:mysql-db -d nginx

```
Kubernetes Basics
```
Kubernetes 
 Nodes : Instances (Servers)
 Pods : Docker running inside Nodes

Kubernetes Components :

API Server : 
etcd : distributed reliable key-value store DB
Scheduler :  responsible for distributing work or containers across multiple nodes
Controller : 
Container Runtime : underlying framework that is responsible for running application in containers like Docker, rkt, CRI-O 
Kubelet : 

In Master 
API-Server
ETCD
Controller
Schedular

In Node 
Kubelet
Container Runtime

kubectl :  command line utility used to manage a kubernetes cluster
Minikube : To setup kubernetes (for single node)
Kubeadm : To setup kubernetes (for multi node)


Setup multinode kubernetes cluster with KUBEADM 
install docker in all master and node servers
install kubeadm|kubelet|kubectl in all master and node servers
kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=APISERVERIP (MASTER) --> In this stage need to define which network package need to use like flannel etc.
Few instruction will be printed when compelted. Follow that command to join worker node to Master
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml  --> It will create kube-flannel pod 
now run the kubeadm join command which we copied into kubeadm init step
kubectl run nginx --image=nginx --> Launch single pod
kubectl get pods --> List pods (container)



PODS: 
kubectl get nodes --> list nodes (servers/instances)
kubectl run nginx --image=nginx
kubectl describe pods --> It will describe all details of pods
kubectl get pods -o wide --> It will show IP and Nod of pods



etcd port : 2379

kubectl get pods -n kube-system --> check kubernetes own pods

Controller :
watch status for all node and pods
node monitor period = 5s default
node monitor grace period = 40s default
POD eviction timeout = 5m default

Scheduler :
Will decide which pod (container) goes which nodes
It will make sure right pods (Memory and CPU) are place on right nodes

Kube-Proxy :
Is process to check for new services and add network rules to forward proper traffic to all pods

Kubernetes YAML top level property

vim pod.yml

apiVersion: v1
kind: Pod OR Service OR ReplicaSet OR Deployment
metadata:
  name: myapp-pod
  labels: 
    app: myapp
    type: front-end
spec: 
  containers:
     - name: nginx-container
       image: nginx

kubectl create -f pod.yml --> Create pod
kubectl get pods  --> Check pods
kubectl describe pod myapp-pod  --> Describe everything for pod


ReplicaSet :
vim rc-defination.yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels: 
    app: myapp
    type: front-end
spec: 
  template: 
      metadata:
        name: myapp-pod
        labels: 
          app: myapp
          type: front-end
      spec: 
        containers:
        - name: nginx-container
          image: nginx
  replicas: 3


vim replicaset-defination.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels: 
    app: myapp
    type: front-end
spec: 
  template: 
      metadata:
        name: myapp-pod
        labels: 
          app: myapp
          type: front-end
      spec: 
        containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
     matchLabels:
       type: front-end

Upgrade replica size from 3 to 6 
Just update the yml file and execute "kubectl replace -f replicaset-defination.yml" OR "kubectl scale -replicas=6 -f replicaset-defination.yml" 

kubectl edit replicaset replicaname --> to change the in replicset parameters like replica or image

kubectl get replicaset
kubectl delete replicaset myapp-replicaset


DEPLOYMENT :: 

vim deployment-defination.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80


NAMESPACE :::

How we can connect db from one NS to another NS.

Local NS
db-service
Just put container name 

Remote NS
db-service.dev.svc.cluster.local
ServerName(Container).NameSpace.Service(default by kubernetes).domainname(Default by kubernetes)

kubectl create namespace dev
kubectl get pods --namespace=dev
kubectl get pod -A  OR kubectl get pods --all-namespace --> All pods from all namespace
kubectl config set-context $(kubectl config current-context) --namespace-dev
kubectl get all --all-namespaces   --> All details like deployment,service pod etc.

SERVICE:::

Service Tyoe : 
NodePort
ClusterIP
LoadBalancer

Nodeport:

Application running in POD on 80 port --> Service will connect with POD 80 to Service pod 80 --> Node will connect with Node port (any port which is exposed)

vim service-nodeport-defination.yml

apiVersion: v1
kind: Service
metadata:
  name: my-service-meta
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 8080
  selector:
      app: myapp
      type: front-end
      
vim service-defination.yml
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  ports:
    - targetPort: 80
      port: 80
  selector:
      app: myapp
      type: back-end
      
-------------------------------------------------------------------------------------------------------------------------------------
POD

Create an NGINX Pod

kubectl run --generator=run-pod/v1 nginx --image=nginx


Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml


Deployment

Create a deployment

kubectl create deployment --image=nginx nginx


Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

kubectl create deployment --image=nginx nginx --dry-run -o yaml


Generate Deployment YAML file (-o yaml). Don't create it(--dry-run) with 4 Replicas (--replicas=4)

kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml

The usage --generator=deployment/v1beta1 is deprecated as of Kubernetes 1.16. The recommended way is to use the kubectl create option instead.


IMPORTANT:

kubectl create deployment does not have a --replicas option. You could first create it and then scale it using the kubectl scale command.


Save it to a file - (If you need to modify or add some other details)

kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml > nginx-deployment.yaml


OR

kubectl create deployment --image=nginx nginx --dry-run -o yaml > nginx-deployment.yaml

You can then update the YAML file with the replicas or any other field before creating the deployment.


Service

Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379

kubectl expose pod redis --port=6379 --name redis-service --dry-run -o yaml

(This will automatically use the pod's labels as selectors)

Or

kubectl create service clusterip redis --tcp=6379:6379 --dry-run -o yaml  (This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)


Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

kubectl expose pod nginx --port=80 --name nginx-service --dry-run -o yaml

(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run -o yaml

(This will not use the pods labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.
-------------------------------------------------------------------------------------------------------------------------------------

Imperative command :::

kubectl run --generator=run-pod/v1 nginx-pod --image=nginx:alpine    --> Launch pod
kubectl run --generator=run-pod/v1 redis --image=redis:alpine -l tier=db --> launch pod with label
kubectl expose pod redis --port=6379 --name redis-service  --> create service for existing pod
kubectl create deployment webapp --image=kodekloud/webapp-color --> Create deployment 
kubectl scale deployment/webapp --replicas=3 --> The scale the webapp to 3 using command 
kubectl expose deployment webapp --type=NodePort --port=8080 --name=webapp-service --dry-run -o yaml > webapp-service.yaml --> to generate a service definition file. Then edit the nodeport in it and create a service.

------------------------- If scheduler not available-------------------
Need to add nodeNade parameter for assigning node manually

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: node-01
  
  
  Label selectors ::::
  
  kubectl get pods -l env=dev,bu=digital
  kubectl get pods -l env=dev,bu=digital,tier=frontend
  
vim labels.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
	

Set Tolerations for pod -->

apiVersion: v1
kind: Pod
metadata:
  name: bee
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "spray"
    value: "mortein"
    effect: "NoSchedule"
  
Create POD with name "mosquito" and container name with "nginx"
apiVersion: v1
kind: Pod
metadata:
  name: mosquito
spec:
  containers:
  - name: nginx
    image: nginx

Remove taint from node
kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-

Example
kubectl taint nodes node01 key=value:NoSchedule
kubectl taint nodes node01 spray=mortein:NoSchedule

Check Taint for node --> 
kubectl describe node kubemaster | grep Taint

Taint Effect Type :
NoSchedule
PreferNoSchedule
NoExecute


Node Selectors :::::

apiVersion: v1
kind: Pod
metadata:
  name: mosquito
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    size: Large

kubectl label nodes nodename label-key=label-value
kubectl label nodes node1 size=Large
    

Node Affinity::::::

apiVersion: v1
kind: Pod
metadata:
  name: mosquito
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
           nodeSelectorTerms:
           -  matchExpression:
              - key: size
                operator: In
                value:
                - Large
                - Medium
  

apiVersion: v1
kind: Pod
metadata:
  name: mosquito
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
           nodeSelectorTerms:
           -  matchExpression:
              - key: size
                operator: NotIn
                value:
                - Small

apiVersion: v1
kind: Pod
metadata:
  name: mosquito
spec:
  containers:
  - name: nginx
    image: nginx
  affinity:
    nodeAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
           nodeSelectorTerms:
           -  matchExpression:
              - key: size
                operator: Exists
		
		
Apply new label to Node
kubectl label node node01 color=blue OR kubectl edit node node01   


kubectl run blue --image=nginx --replica=6


Set nodeAffinity :::::

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: red
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/master
                operator: Exists
