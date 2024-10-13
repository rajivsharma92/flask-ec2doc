Flask Calculator Setup on Amazon EC2 with Nginx (Final Documentation with Port Resolution)
Prerequisites
    • Amazon Linux 2 instance running on AWS.
    • Security group configured to allow HTTP (port 80) and SSH (port 22) access.
    • Your EC2 Key Pair file (.pem) for SSH login.

Step 1: Connect to EC2 Instance
    1. Open your terminal and SSH into your instance:
       
       
       ssh -i /path/to/your-key.pem ec2-user@<your-ec2-public-ip>

Step 2: Install Required Software
    1. Update the system and install the necessary packages:
       
       
       sudo yum update -y
       sudo yum install -y python3 python3-pip nginx git
    2. Install Flask and Gunicorn:
       
       
       sudo pip3 install flask gunicorn

Step 3: Set Up Flask Application
    1. Create the project folder for the Flask application:
       
       
       mkdir ~/flask_calculator
       cd ~/flask_calculator
    2. Create application files:
        ◦ Create app.py:
          
          
          nano ~/flask_calculator/app.py
          Add the following content:
          python
          
          from flask import Flask, request, jsonify, send_from_directory
          
          app = Flask(__name__)
          
          @app.route('/')
          def index():
              return send_from_directory('static', 'index.html')
          
          @app.route('/calculate', methods=['POST'])
          def calculate():
              data = request.json
              num1 = data.get('num1')
              num2 = data.get('num2')
              operation = data.get('operation')
          
              if operation == 'add':
                  result = num1 + num2
              elif operation == 'subtract':
                  result = num1 - num2
              elif operation == 'multiply':
                  result = num1 * num2
              elif operation == 'divide':
                  result = num1 / num2
              else:
                  return jsonify({'error': 'Invalid operation'}), 400
          
              return jsonify({'result': result})
          
          if __name__ == '__main__':
              app.run(host='0.0.0.0')
        ◦ Create wsgi.py (for Gunicorn):
          
          
          nano ~/flask_calculator/wsgi.py
          Add the following content:
          python
          
          from app import app
          
          if __name__ == "__main__":
              app.run()

Step 4: Create HTML Frontend
    1. Create a directory for static files:
       
       
       mkdir ~/flask_calculator/static
    2. Create the HTML file for the calculator interface:
       
       
       nano ~/flask_calculator/static/index.html
       Add the following content:
       html
       
       <!DOCTYPE html>
       <html lang="en">
       <head>
           <meta charset="UTF-8">
           <meta name="viewport" content="width=device-width, initial-scale=1.0">
           <title>Flask Calculator</title>
           <script>
               async function calculate() {
                   const num1 = parseFloat(document.getElementById('num1').value);
                   const num2 = parseFloat(document.getElementById('num2').value);
                   const operation = document.getElementById('operation').value;
       
                   const response = await fetch('/calculate', {
                       method: 'POST',
                       headers: {
                           'Content-Type': 'application/json',
                       },
                       body: JSON.stringify({ num1, num2, operation }),
                   });
       
                   const result = await response.json();
                   document.getElementById('result').innerText = 'Result: ' + result.result;
               }
           </script>
       </head>
       <body>
           <h1>Flask Calculator</h1>
           <input type="number" id="num1" placeholder="Number 1">
           <select id="operation">
               <option value="add">+</option>
               <option value="subtract">-</option>
               <option value="multiply">*</option>
               <option value="divide">/</option>
           </select>
           <input type="number" id="num2" placeholder="Number 2">
           <button onclick="calculate()">Calculate</button>
       
           <p id="result"></p>
       </body>
       </html>

Step 5: Configure Nginx
    1. Create an Nginx configuration file:
       
       
       sudo nano /etc/nginx/conf.d/flask_calculator.conf
    2. Add the following content to configure Nginx to act as a reverse proxy for Gunicorn:
       nginx
       
       server {
           listen 80;
           server_name <your-ec2-public-ip>;
       
           location / {
               proxy_pass http://127.0.0.1:5000;
               proxy_set_header Host $host;
               proxy_set_header X-Real-IP $remote_addr;
               proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
               proxy_set_header X-Forwarded-Proto $scheme;
           }
       }
    3. Test the Nginx configuration:
       
       
       sudo nginx -t
    4. Restart Nginx:
       
       
       sudo systemctl restart nginx

Step 6: Start Flask Application
Resolve Port Issues
Before starting the application, ensure the required port is not being used by another process:
    1. Check if port 5000 is already in use:
       
       
       sudo lsof -i :5000
    2. If a process is using port 5000, kill it:
       
       
       sudo kill -9 <PID>
       Replace <PID> with the process ID returned by the lsof command.
    3. Optionally, switch to another port (e.g., 5001) in your Gunicorn run command if you want to avoid using port 5000:
       Update the Gunicorn command in the run.sh script (next step) to use port 5001:
       
       
       gunicorn --bind 0.0.0.0:5001 wsgi:app

Run Gunicorn
    1. Create a script to start Gunicorn:
       
       
       nano ~/flask_calculator/run.sh
    2. Add the following content:
       
       
       #!/bin/
       cd ~/flask_calculator
       gunicorn --bind 0.0.0.0:5000 wsgi:app
    3. Make the script executable:
       
       
       chmod +x ~/flask_calculator/run.sh
    4. Run the application:
       
       
        ~/flask_calculator/run.sh &

Step 7: Access the Application
    1. Open your web browser and visit:
       vbnet
       
       http://<your-ec2-public-ip>/
    2. You should see the interactive calculator interface. Try performing calculations by entering numbers and selecting operations, and click Calculate to see the result.

Step 8: Optional – Keep Flask Running after Logout
To keep your Flask application running after you log out of the SSH session, use nohup (no hangup):


nohup  ~/flask_calculator/run.sh &

Troubleshooting Common Issues
    1. Port already in use:
        ◦ Check for processes using the port:
          
          
          sudo lsof -i :5000
        ◦ Kill the process using that port:
          
          
          sudo kill -9 <PID>
        ◦ Alternatively, change the port used by Gunicorn in the run.sh script to a different port (e.g., 5001).
    2. Nginx issues:
        ◦ Ensure Nginx is running:
          
          
          sudo systemctl status nginx
        ◦ Restart Nginx if needed:
          
          
          sudo systemctl restart nginx

This updated documentation resolves the port in use issue by providing steps to check for and kill processes that might be holding onto a specific port, along with the option to change the port if necessary. It ensures that the setup will work smoothly without future port conflicts.





The error you're encountering indicates an SSL issue when connecting to your MongoDB Atlas cluster. Specifically, the error tlsv1 alert internal error suggests that the SSL/TLS connection couldn't be established. Here’s how you can troubleshoot and resolve this:

1. Ensure Your IP is Whitelisted in MongoDB Atlas
Go to MongoDB Atlas Dashboard.
Navigate to Network Access in the Security section.
Add your EC2 instance's IP address to the whitelist.
You can also allow access from anywhere (0.0.0.0/0) if you are testing but keep in mind this is less secure.
2. Check for SSL/TLS Settings
The error could also occur if your connection string requires SSL and it’s either not configured properly or incompatible.

Ensure your connection string uses the correct SSL settings for Atlas:

php

mongodb+srv://<username>:<password>@<cluster-url>?retryWrites=true&w=majority&ssl=true
If you don’t want SSL, you can disable it by setting:

arduino

ssl=false
If your connection string includes ssl=true, make sure that MongoDB is properly set to accept SSL connections.

3. Update MongoDB Driver
Ensure you have the latest version of the MongoDB Node.js driver installed:



npm install mongodb@latest
4. Configure Mongoose to Use Newer TLS Versions
The error indicates that it's attempting to use tlsv1, which may be outdated for MongoDB Atlas. Ensure Mongoose uses a more recent version of TLS by explicitly setting it in your connection options:

js

mongoose.connect('your-connection-string', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  ssl: true,
  tlsAllowInvalidCertificates: false,
  tlsAllowInvalidHostnames: false,
  tlsCAFile: '/path/to/ca.pem'  // Optional if needed
});
5. Check Atlas Configuration for TLS/SSL
In your MongoDB Atlas project, ensure that the cluster is configured to support the correct TLS version. MongoDB Atlas typically supports TLS 1.2, so make sure your Node.js environment supports this as well.

6. Restart Your Application
After making these changes, restart your backend application:



node index.js
These steps should help in resolving the SSL routines: ssl3_read_bytes and Could not connect to any servers in your MongoDB Atlas cluster issues. If the issue persists after these checks, ensure there are no network-related firewalls or security groups blocking the connection.





Document: Setting up a Node.js Express Application on AWS EC2
Objective:
This document provides a step-by-step guide to deploying a Node.js Express backend on an AWS EC2 instance. It includes instructions to:
    • Set up the environment.
    • Deploy the Express application.
    • Resolve common port conflicts.

Prerequisites:
    1. AWS EC2 instance (Amazon Linux or Ubuntu).
    2. Node.js and npm installed.
    3. Application code uploaded to the EC2 instance.
    4. SSH access to the EC2 instance.
    5. Nginx installed and configured for reverse proxy (optional, but recommended for production).

1. Connect to Your EC2 Instance:
    1. SSH into your EC2 instance using the terminal (replace with your instance’s public IP):
       
       
       ssh -i <your-key-file.pem> ec2-user@<EC2-IP>
    2. Ensure that Node.js and npm are installed:
       
       
       node -v
       npm -v

2. Upload Your Application to EC2:
Upload your Node.js application code to the instance using scp or through a Git repository:
    • If using Git:
      
      
      git clone <your-repo-url>
      cd <your-project-directory>
    • If using scp:
      
      
      scp -i <your-key-file.pem> <path-to-local-files> ec2-user@<EC2-IP>:/home/ec2-user/

3. Install Dependencies:
Once your application is uploaded or cloned, navigate to the project directory and install the required packages:


cd <your-project-directory>
npm install

4. Application Setup (index.js):
Ensure that your index.js (or entry-point file) is set up correctly to handle the environment port configuration. Below is a sample code to modify and use:
Corrected index.js:
javascript

const express = require('express');
const cors = require('cors');
require('dotenv').config();  // Load environment variables

const app = express();
const conn = require('./conn');  // Ensure this connection file is configured

app.use(express.json());
app.use(cors());

const tripRoutes = require('./routes/trip.routes');
app.use('/trip', tripRoutes);  // Route handling

// Test route to confirm the server is running
app.get('/hello', (req, res) => {
    res.send('Hello World!');
});

// Use environment variable for PORT or default to 3002
const PORT = process.env.PORT || 3002;

app.listen(PORT, '0.0.0.0', () => {
    console.log(`Server started at http://<EC2-IP>:${PORT}`);
});
Notes:
    • Environment Variable Setup: Ensure your .env file contains:
      makefile
      
      PORT=3002

5. Starting Your Application:
Once your code is set up and dependencies are installed, start your application using:


node index.js
The application will start on the specified port (default: 3002). You should see a message like:
arduino

Server started at http://<EC2-IP>:3002
Testing:
    1. Access your application in a browser at:
       arduino
       
       http://<EC2-IP>:3002/hello
       This should return: Hello World!
    2. You can also test other routes, like:
       arduino
       
       http://<EC2-IP>:3002/trip

6. Resolving Port Conflicts:
If you receive an error like:
perl

Error: listen EADDRINUSE: address already in use :::3001
It means the port is already being used. You can resolve this by:
    • Changing the port in your .env file.
    • Ensuring no other process is using the port:
      
      
      sudo lsof -i :3001
      Kill the process if needed:
      
      
      sudo kill -9 <process-id>

7. Running the Application in the Background:
For production or long-running applications, use pm2 or nohup to run your application in the background.
Installing pm2:


npm install pm2 -g
Starting the App with pm2:


pm2 start index.js --name "my-app"
Persisting pm2 on Reboots:


pm2 startup
pm2 save

8. Optional: Setting up Nginx Reverse Proxy (for Production)
For production environments, it’s recommended to use Nginx as a reverse proxy to handle requests on standard ports (80 or 443 for HTTPS). Here's how to set up Nginx:
Install Nginx:


sudo yum install nginx -y   # For Amazon Linux
sudo apt install nginx -y   # For Ubuntu
Nginx Configuration:
Edit the Nginx config file:


sudo nano /etc/nginx/nginx.conf
Add the following block to proxy requests to your Node.js app:
nginx

server {
    listen 80;
    server_name <your-domain-or-EC2-IP>;

    location / {
        proxy_pass http://localhost:3002;  # Forward requests to your Node app
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
Restart Nginx:


sudo systemctl restart nginx

9. Conclusion:
Now your Node.js Express app should be successfully running on your EC2 instance, accessible via the EC2 IP address or domain. Ensure that proper firewall rules are configured in the AWS console to allow traffic on port 80 or 3002.
Common Commands:
    • Start app: node index.js
    • Start with pm2: pm2 start index.js --name "my-app"
    • Check logs: pm2 logs

This document should guide you through the process of deploying your Node.js Express app on AWS EC2 with the necessary steps to avoid port conflicts and ensure smooth operation.






Flask App Deployment on AWS EC2 with Nginx and Gunicorn
1. Set Up EC2 Instance
    • Launch an AWS EC2 instance (Amazon Linux, Ubuntu, etc.).
    • Ensure your security group allows access to:
        ◦ SSH (TCP 22) – For remote login.
        ◦ HTTP (TCP 80) – For accessing the web server.
        ◦ Custom TCP for your app's port (e.g., 3002).
2. Update and Install Necessary Packages
    • Connect to your EC2 instance via SSH:
      bash
      Copy code
      ssh -i your-key.pem ec2-user@<EC2-PUBLIC-IP>
    • Update the instance and install dependencies:
      bash
      Copy code
      sudo yum update -y  # For Amazon Linux
      sudo apt update -y  # For Ubuntu
    • Install Python, pip, Flask, Gunicorn, and Nginx:
      bash
      Copy code
      sudo yum install python3 nginx -y  # Amazon Linux
      sudo apt install python3-pip nginx -y  # Ubuntu
      sudo pip3 install flask gunicorn
3. Create Your Flask Application
    • On your EC2 instance, create your Flask app directory:
      bash
      Copy code
      mkdir flask_app
      cd flask_app
    • Create app.py with the following content:
      python
      Copy code
      from flask import Flask
      app = Flask(__name__)
      
      @app.route('/')
      def hello():
          return "Hello, World!"
    • Run the Flask app for testing:
      bash
      Copy code
      python3 app.py
      You should see the message about Flask running on http://127.0.0.1:5000. Test it by visiting <EC2-PUBLIC-IP>:5000 from your browser.
4. Set Up Gunicorn
    • Gunicorn is a production-ready server that can serve your Flask app. Use the following command to run your app with Gunicorn:
      bash
      Copy code
      gunicorn --bind 127.0.0.1:3002 app:app
    • Verify Gunicorn is running on port 3002:
      bash
      Copy code
      sudo netstat -tuln | grep 3002
5. Configure Nginx as a Reverse Proxy
    • Nginx will act as a reverse proxy to forward client requests to Gunicorn. Create an Nginx config file for your Flask app:
      bash
      Copy code
      sudo nano /etc/nginx/conf.d/flask_app.conf
    • Add the following configuration:
      nginx
      Copy code
      server {
          listen 80;
          server_name <EC2-PUBLIC-IP>;
      
          location / {
              proxy_pass http://127.0.0.1:3002;
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
              proxy_set_header X-Forwarded-Proto $scheme;
          }
      }
    • Test the Nginx configuration:
      bash
      Copy code
      sudo nginx -t
    • Restart Nginx:
      bash
      Copy code
      sudo systemctl restart nginx
    • Now, accessing the EC2 instance’s public IP should show your Flask app served via Nginx and Gunicorn.
6. Configure Gunicorn to Run as a Service (Optional)
    • To ensure that Gunicorn runs on reboot or as a service, create a systemd service file:
      bash
      Copy code
      sudo nano /etc/systemd/system/gunicorn.service
    • Add the following content:
      ini
      Copy code
      [Unit]
      Description=Gunicorn instance to serve Flask app
      After=network.target
      
      [Service]
      User=ec2-user
      Group=nginx
      WorkingDirectory=/home/ec2-user/flask_app
      ExecStart=/usr/local/bin/gunicorn --workers 3 --bind 127.0.0.1:3002 app:app
      
      [Install]
      WantedBy=multi-user.target
    • Enable and start the Gunicorn service:
      bash
      Copy code
      sudo systemctl start gunicorn
      sudo systemctl enable gunicorn
7. Common Troubleshooting
If your app doesn’t work after deployment, follow these troubleshooting steps:
    1. Check Gunicorn is Running
        ◦ Ensure Gunicorn is running and listening on the right port (e.g., 3002):
          bash
          Copy code
          sudo netstat -tuln | grep 3002
    2. Check Nginx Logs for Errors
        ◦ If you get a "502 Bad Gateway" error, check the Nginx error logs for clues:
          bash
          Copy code
          sudo tail -f /var/log/nginx/error.log
        ◦ Possible errors include:
            ▪ Connection refused: This usually happens when Gunicorn is not running or not listening on the correct port.
            ▪ Permission issues: Ensure proper permissions for Nginx and Gunicorn.
    3. Firewall and Security Group Rules
        ◦ Ensure your EC2 security group allows inbound traffic on the necessary ports (e.g., 80 for HTTP and 3002 if accessed directly).
        ◦ Check if the firewall (if enabled) allows traffic:
          bash
          Copy code
          sudo ufw status  # Ubuntu
          sudo iptables -L  # Amazon Linux
    4. Gunicorn Misconfiguration
        ◦ If Gunicorn fails to start, ensure the app:app is correct and points to the Flask app.
        ◦ Check Gunicorn logs if running as a systemd service:
          bash
          Copy code
          sudo journalctl -u gunicorn
8. Clean Up and Security
    • Once your app is working:
        ◦ Ensure you close unnecessary ports (like 3002 if it’s only used internally).
        ◦ Consider securing your app with HTTPS by adding an SSL certificate with Let's Encrypt:
          bash
          Copy code
          sudo yum install certbot python3-certbot-nginx
          sudo certbot --nginx -d <your-domain>

By following this guide, you should have a solid understanding of how to deploy a Flask app using Gunicorn and Nginx, and troubleshoot common issues during deployment.
Output:
