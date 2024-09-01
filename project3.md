This project aims to set up load balancing for a static website using Nginx as a reverse proxy load balancer. The goal is to distribute traffic across multiple servers, ensuring scalability and high availability.

Key concepts covered:

- AWS (EC2 and Route 53)
- EC2 Linux (Ubuntu)
- Nginx
- DNS
- Load balancing
- SSL (certbot)
- OpenSSL command

Project tasks:

1. Deploy three servers
2. Set up static websites on two servers using Nginx
3. Differentiate between two servers by modifying index.html files
4. Set up Nginx on the third server as a load balancer
5. Configure Nginx to load balance traffic between two static websites
6. Update DNS A record with Nginx load balancer IP
7. Test the setup by accessing the website and verifying different index.html files on each reload

The project uses a round-robin algorithm by default, but can be configured for more advanced load balancing methods like least connections or IP hash.



**Documentation

-Spin up your 3 ubuntu servers. Ensure you clearly name them so you don't make mistakes.

![alt text](image-11.png)


Install Nginx and Setup Your Website
Download your website template from your preferred website by navigating to the website, locating the template you want.

Right click and select Inspect from the drop down menu.

![alt text](image.png)

![alt text](image-1.png)


](img/Interior2.png)

![alt text](<img/Mini Finance1.png>)



To install Nginx, execute the following commands on your terminal.

sudo apt update

sudo apt upgrade

sudo apt install nginx



Start your Nginx server by running the sudo systemctl start nginx command, enable it to start on boot by executing sudo systemctl enable nginx, and then confirm if it's running with the sudo systemctl status nginx command.



](<img/Screenshot 2024-08-27 202636.png>)

Visit your instances IP address in a web browser to view the default Nginx startup page.

Execute sudo apt install unzip to install the unzip tool and run the following command to download and unzip your website files sudo curl -o /var/www/html/2135_mini_finance.zip https://www.tooplate.com/zip-templates/2135_mini_finance.zip && sudo unzip -d /var/www/html/ /var/www/html/2135_mini_finance.zip && sudo rm -f /var/www/html/2135_mini_finance.zip.


![alt text](image-3.png)


![alt text](image-4.png)



To set up your website's configuration, start by creating a new file in the Nginx sites-available directory. Use the following command to open a blank file in a text editor: sudo nano /etc/nginx/sites-available/finance.

Copy and paste the following code into the open text editor.

server {
    listen 80;
    server_name example.com www.example.com;

    root /var/www/html/example.com;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
Edit the root directive within your server block to point to the directory where your downloaded website content is stored


![alt text](<img/Screenshot 2024-08-27 210049.png>)




Create a symbolic link for both websites by running the following command. sudo ln -s /etc/nginx/sites-available/finance /etc/nginx/sites-enabled/


![alt text](<img/Screenshot 2024-08-27 205437.png>)


Run the sudo nginx -t command to check the syntax of the Nginx configuration file, and when successful run the sudo systemctl restart nginx command.



![alt text](image-5.png)

Note

On your first server, run sudo rm /etc/nginx/sites-enabled/default, and on your second server, run sudo rm /etc/nginx/sites-enabled/default. This will delete the default site-enabled folders and enable Nginx to serve content from your specified website directories. If you don't delete these default folders, you'll continue to see the default Nginx page.



Run the sudo systemctl restart nginx command to restart your server.

Check both IP addresses to confirm your website is up and running.


![alt text](<img/Screenshot 2024-08-27 214504.png>)


![alt text](<img/Screenshot 2024-08-27 214656.png>)



**Configure your Load balancer

Install Nginx on the server you want to use as a load balancer, and execute sudo systemctl status nginx to ensure it's running.


![alt text](<img/Screenshot 2024-08-27 215608.png>)



Execute sudo nano /etc/nginx/nginx.conf to edit your Nginx configuration file.


Add the following within the http block.

    upstream cloudghoul {
    server 1;
    server 2;
    # Add more servers as needed
}

server {
    listen 80;
    server_name <your domain> www.<your domain>;

    location / {
        proxy_pass http://cloudghoul;
    }
}



![alt text](<img/Screenshot 2024-08-30 195535.png>)


Note

Replace the necessary placeholders as shown in the picture above. Substitute <server 1> and <server 2> with the actual private IP addresses of your servers. Also, replace <your domain> www.<your domain> with your root domain and subdomain name, and update proxy_pass and the other relevant fields accordingly.

Run sudo nginx -t to check for syntax error.

![alt text](image-6.png)


Apply the changes by restarting Nginx: sudo systemctl restart nginx



**Create An A Record
To make your website accessible via your domain name rather than the IP address, you'll need to set up a DNS record. I did this by buying my domain from Namecheap and then moving hosting to AWS Route 53, where I set up an A record.

Note

Visit Project1 for instructions on how to create a hosted zone.

Point your domain's A records to the IP address of your Nginx load balancer server.

In route 53, select the domain name and click on Create record.



![alt text](image-7.png)


Paste your IP address➀ and then click on Create records➁ to create the root domain.


![alt text](image-8.png)



Click on create record again, to create the record for your sub domain.

Paste your IP address➀, input the Record name(www➁) and then click on Create records➂.



![alt text](image-9.png)



Go to the terminal you used in setting your first website and run sudo nano /etc/nginx/sites-available/finance to edit your settings. Enter the name of your domain and then save your settings.


![alt text](<img/Screenshot 2024-08-30 213024.png>)



Restart your nginx server by running the sudo systemctl restart nginx command


Go to your domain name in a web browser to verify that your website is accessible.



Reload the webpage to ensure the load balancer distributes traffic evenly between your servers.



Install certbot and Request For an SSL/TLS Certificate
Install certbot by executing the following commands: sudo apt update sudo apt install python3-certbot-nginx



Execute the sudo certbot --nginx command to request your certificate. Follow the instructions provided by certbot and select the domain name for which you would like to activate HTTPS.




You should get a congratulatory message that says https has been successfully enabled.



Access your website to verify that Certbot has successfully enabled HTTPS.



It is recommended to renew your LetsEncrypt certificate at least every 60 days or more frequently. You can test renewal command in dry-run mode: sudo certbot renew --dry-run



![alt text](image-10.png)

