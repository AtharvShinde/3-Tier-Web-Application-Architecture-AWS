1. Create VPC for your project 2 public subnet 4 private subnet and 1 NAT gateway for private subnet for communicate with internet

2. Create an S3 Bucket and upload application code folder containing app-tier web-tier

3. Create IAM Role and Give Permission AmazonEC2RoleforSSM for EC2 instance for accessing another EC2 instance present in private subnet

4. Rename Subnet For better understanding as APP1 and APP2 and DB1 and DB2 for private subnet of VPC

7. Create Security Groups for (1 Web Server, 1 App Server, 1 App internal Load Balancer, 1 Database)
- DB Security Group Allow for port 3306 (MySQL) access from VPC CIDR Range 192.168.0.0/22 or Security from of APP-Server
- Web-ALB-SG Allow http and https from anywhere ipv4
- Web Server Allow http from sg of Web-ALB-SG or VPC CIDR
- APP Server allow Custom TCP for react port that is 4000 from VPC CIDR

5. Create Subnet Groups in RDS for DB1 and DB2

6. Create MySQL Database in RDS db.t4g.micro
 
7. Add MySQL Database Endpoint to DbConfig

8. Create Ec2 Insatnce with Amazon Linux Image 2 with appropriate security group that is created above and IAM Role Created in Step 1

    App Server Setup: Launch an ec2 instance in APP subnet of Custom VPC

    Install MySQL
    On the app server, install MySQL:

        sudo yum install mysql -y
    
    Configure MySQL Database
    Connect to the database and perform basic configuration: Replace below info with your DB information

        mysql -h DB_ENDPOINT -u admin -p
    
    In the MySQL shell, execute the following commands:

        CREATE DATABASE webappdb;
        SHOW DATABASES;
        USE webappdb;

        CREATE TABLE IF NOT EXISTS transactions(
        id INT NOT NULL AUTO_INCREMENT,
        amount DECIMAL(10,2),
        description VARCHAR(100),
        PRIMARY KEY(id)
        );

        SHOW TABLES;
        INSERT INTO transactions (amount, description) VALUES ('400', 'groceries');
        SELECT * FROM transactions;

    Update Application Configuration to with DB information
    Update the **application-code/app-tier/DbConfig.js** file with your database credentials.

    Install and Configure Node.js and PM2
    Install Node.js and PM2:

        curl -o- https://raw.githubusercontent.com/avizway1/aws_3tier_architecture/main/install.sh | bash
        source ~/.bashrc

        nvm install 16
        nvm use 16

        npm install -g pm2

    Download application code from S3 and start the application:

        cd ~/
        aws s3 cp s3://S3-Bucket-Name/application-code/app-tier/ app-tier --recursive

        cd ~/app-tier
        npm install
        pm2 start index.js

        pm2 list
        pm2 logs
        pm2 startup
        pm2 save

    Verify that the application is running by executing:

        curl http://localhost:4000/health

    It should return: This is the health check.

9. Create Target Group for port 4000

10. Create Internal Load Balancer attach security group already created for it

11. Add internal load balancer endpoint to nginx.conf

12. Create Web Tier Setup: Launch EC2 Instance in Web Subnets we have created in Custom VPC
    
    Web Tier Installation.
    Install Node.js and Nginx on the web tier:

        curl -o- https://raw.githubusercontent.com/avizway1/aws_3tier_architecture/main/install.sh | bash
        source ~/.bashrc
        nvm install 16
        nvm use 16

        cd ~/
        aws s3 cp s3://3tierproject-avinash/application-code/web-tier/ web-tier --recursive
        
        cd ~/web-tier
        npm install
        npm run build
        
        sudo amazon-linux-extras install nginx1 -y

    Update Nginx configuration:

        cd /etc/nginx
        ls

        sudo rm nginx.conf
        sudo aws s3 cp s3://3tierproject-avinash/application-code/nginx.conf .

        sudo service nginx restart

        chmod -R 755 /home/ec2-user

        sudo chkconfig nginx on

 13. Access the web tier by visiting http://<Web-Server-Public-IP> in your browser.