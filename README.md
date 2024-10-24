# Deploy-Node.js-app-on-EC2

# 1. create AWS account 
# 2. Launch EC2 instance
# 3. Install Node and NPM
    curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    sudo apt install nodejs

    node --version
# 4. Clone your project from Github
    git clone https://github.com/piyushgargdev-01/short-url-nodejs
# 5. Install dependencies and test app
    sudo npm i pm2 -g
    pm2 start index

    # Other pm2 commands
    pm2 show app
    pm2 status
    pm2 restart app
    pm2 stop app
    pm2 logs (Show log stream)
    pm2 flush (Clear logs)

    # To make sure app starts when reboot
    pm2 startup ubuntu
# Configure mongodb
    You will not be able to connect to mongodb as it is not configured for ec2 
    To configure MongoDB for your Node.js app running on an EC2 instance, you have two primary options:

    1. Use a Managed MongoDB Service (e.g., MongoDB Atlas)
        Sign up at MongoDB Atlas.
        Follow the steps to create a new cluster. Choose a free tier if available.
    2. Set Up MongoDB on the EC2 Instance 
        Incase of ubuntu follow the process at :https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/#:~:text=MongoDB%208.0%20Community%20Edition%20supports%20the%20following,22.04%20LTS%20(%22Jammy%22)%20*%2020.04%20LTS%20(%22Focal%22)
    Configure MongoDB
    By default, MongoDB binds to localhost only. If your Node.js app is running on the same EC2 instance, the default configuration is fine.

    However, if you need to allow external connections:
    1. Edit the MongoDB configuration file:
        sudo nano /etc/mongod.conf
    2. Modify the bindIp setting under the net section:
        net:
        bindIp: 0.0.0.0
        port: 27017
    3. Restart MongoDB:
        sudo systemctl restart mongod
    4. Update the Security Group
        In the AWS Management Console, navigate to your EC2 instance's security group.
        Add a new inbound rule to allow traffic on port 27017 (MongoDB's default port). You can restrict the source IP to your specific IP address or a range for security.
    Update Your Application's MongoDB Connection String
        If your Node.js app is running on the same instance, use:
        MONGO_URI=mongodb://localhost:port/yourDatabaseName
        If your app is running on a different server, use the public IP or DNS of your EC2 instance:
        MONGO_URI=mongodb://<ec2-public-ip>:port/yourDatabaseName
    Test the Connection
        Make sure your Node.js application is configured correctly to connect to MongoDB.



# 6. Setup Firewall
    sudo ufw enable
    sudo ufw status
    sudo ufw allow ssh (Port 22)
    sudo ufw allow http (Port 80)
    sudo ufw allow https (Port 443)
# 7. Install NGINX and configure
    sudo apt install nginx

    sudo nano /etc/nginx/sites-available/default
    Add the following to the location part of the server block

    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:8001; #whatever port your app runs on
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
    # Check NGINX config
    sudo nginx -t

    # Restart NGINX
    sudo nginx -s reload
# 8. Add SSL with LetsEncrypt
    sudo add-apt-repository ppa:certbot/certbot
    sudo apt-get update
    sudo apt-get install python3-certbot-nginx
    sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

    # Only valid for 90 days, test the renewal process with
    certbot renew --dry-run

