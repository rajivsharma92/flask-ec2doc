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
