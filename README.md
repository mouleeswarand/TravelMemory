# Travel Memory Application Deployment - Connecting Backend and Frontend  + Scaling the application using LoadBalancer + Cloudflare DNS

## Launch Instance (2 Instance - One for frontend and One for backend)
1. Log into the AWS Management Console and navigate to the EC2 Dashboard.
2. Click on 'Launch Instance'. Choose 'Ubuntu Server 20.04 LTS' from the list of Amazon Machine Images (AMIs).
3. Select an Instance Type. Choose the 't2.micro' instance type (eligible for the free tier).
4. Configure Instance Details. Ensure the default VPC and public subnet are selected.
5. Add Storage. Allocate 8 GB of storage for your instance. This is the default size but can be adjusted based on your application needs.

## Configuring EC2
1. Connect the EC2 Instance
2. Frontend1 `ssh -i "mouli-key.pem" ubuntu@ec2-13-203-231-108.ap-south-1.compute.amazonaws.com`
3. Backend1 `ssh -i "mouli-key.pem" ubuntu@ec2-3-110-156-174.ap-south-1.compute.amazonaws.com`
4. Up-to-date `sudo apt update && sudo apt upgrade -y`
5. ### Install Node.js 22 and npm
   `curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs`
6. `node -v
npm -v`

## Configure the Mongo Database
1. Sign in to your MongoDB Atlas account
2. Create a new cluster - free tier option (M0).
3. Add a new Database `travelmemory`
4. Under Network Access - Allow Access from Anywhere or add your specific IP address
5. Copy and save the URI `URI: mongodb+srv://<username>:<password>@cluster0.mongodb.net/travelmemory`
6. Download MongoDB Compass
7. Once connected, click on 'Create Database', name it 'travelmemory', and create a collection (e.g., 'trips')

## Deploying Backend Application - Backend server
1. `cd TravelMemory/backend`
2. Create `.env` file
3.  Add the following URI
   `MONGO_URI='ENTER_YOUR_URL'
PORT=3001`
4. `npm install` - This will install necessary packages
5. `node index.js` or `pm2 start index.js pm2 save` - Run the Backend Application use `pm2` for detach mode
6. Test the API Endpoint `https://backend.tigredge.net/hello` or `3.110.156.174:3001/hello`

## Deploying the Frontend - Frontend Server
1. `cd TravelMemory/frontend`
2. create a `.env` file to store environment variables
3. `echo "REACT_APP_BACKEND_URL=http://EC2_PUBLIC_IP:3001" > .env`
4. `npm install`
5. `npm start` or `npm start &` - use & for detach mode to run the application
6. ### Configure NGINX reverse proxy to access the VM
7. `server {
    server_name memory.tigredge.net;

    # Frontend
    location / {

        proxy_pass http://localhost:3000/;
        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;

        proxy_cache_bypass $http_upgrade;
    }

    listen 443 ssl; # certbot managed
   
    ssl_certificate /etc/letsencrypt/live/memory.tigredge.net/fullchain.pem; # certbot managed
   
    ssl_certificate_key /etc/letsencrypt/live/memory.tigredge.net/privkey.pem; # certbot managed
   
    include /etc/letsencrypt/options-ssl-nginx.conf; # certbot managed
   
    }`

8. ### Configure SSL
  - `sudo snap install --classic certbot`
  - `sudo ln -s /snap/bin/certbot /usr/local/bin/certbot`
  - `sudo certbot --nginx`
  - enter the domain name to obtain the SSL from Let's encrypt


9. ## convert the image into AMI
10. ## create a new instances from AMI - One for Frontend and One for Backend
11. ## Configure the right Configuration, IP address, NGINX and Security groups
    - Run the VMs
12. ## Create a Load balancer for Scaling the VM
    - First create `Target group(Frontend-TG)` for Frontend Server and `Target group(Backend-TG)` for Backend Server
    - map the ports 3000 - Frontend Server (2 VMs)
    - map the ports 3001 - Backend Server (2 VMs)
    - Now create a Loadbalance using HTTP(80) and HTTPS(443) Listeners that redirect to the Targetgroups
    - Once the port numbers are `Healthy` we can test the Loadbalancer Frontend LB `Fronntend-LB-1887701079.ap-south-1.elb.amazonaws.com` or `https://travel.tigredge.net/`
    - Once the port numbers are `Healthy` we can test the Loadbalancer Backend LB `Backend-LB-1219613108.ap-south-1.elb.amazonaws.com`
    - Now, Map the CNAME records to the DNS on Cloudflare or any domain provider, I have mapped the domain called `https://travel.tigredge.net/` for SSL i use AWS certificate manager or Cloudflare proxy enabled SSL for easy use
    - For further scaling, Configure ASG
### Now check the URL to see both the Frontend, Backend and We add any Experience all the entries will be stored in `travelmemory` database
### MERN Application is up & running with Load Balancing the VMs 2 for Frontend and 2 for Backend with SSL enabled

`Frontend APP URL: https://memory.tigredge.net/

Backend APP URL: https://backend.tigredge.net/hello

Load balancer URL

Frontend LB  URL: Fronntend-LB-1887701079.ap-south-1.elb.amazonaws.com

Backend LB URL :  Backend-LB-1219613108.ap-south-1.elb.amazonaws.com

DNS Record / CNAME: https://travel.tigredge.net/`




