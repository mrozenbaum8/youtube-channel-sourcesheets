# Complete Guide to Wordpress Migration / Backup to GCP

## Backup current site

- After installing localwp on your Mac, install local website on computer.
- While installing the website on computer, select “advanced options” during the installation process make sure the web server is set to Apache.
- Execute most other settings as per on-screen instructions.  Upload the plugin file instead of installing the all-in-one plugin via the plugin store: [https://github.com/mrozenbaum8/youtube-channel-sourcesheets/blob/main/all-in-one-wp-migration-6.7.zip]

### Increase max upload size:

- Click on filreoot to see site files in finder.
- Using [VSCode](https://code.visualstudio.com), edit the conf / php / php.ini.hbs update lines to the value you need: post_max_size = 1000M upload_max_filesize = 1000M I also upped the “memory_limit” to 512M

### Restoring backed up site to local

- Navigate to **wp-content/ai1wm-backups/** directory and upload your site’s exported backup (from the live site) file there**.**
- Click the restore option by the all-in-one plugin on wordpress and procced.
- Update permalink settings.
- Go to **Plugin editor** from the left menu panel in the admin dashboard.
- Top right, Choose **All-in-one WP Migration** from the dropdown and click select.
- Click on **constants.php** file.
- Search for the word **AI1WM_MAX_FILE_SIZE**. Change it’s value, refer below code:define( ‘AI1WM_MAX_FILE_SIZE’, 2 << 28 ); Todefine( ‘AI1WM_MAX_FILE_SIZE’, 536870912 * 5 );If you even want more than 3 GB then change value from 536870912 * 5 to 536870912 * 10( or any number).Don’t forget to update the file.

## Installing Openlitespeed Wordpress on Google Cloud

1. Go to console.google.com
2. When prompted, activate the free $300, 90-day credit.
3. Create a new project. Then select it.
4. Go to “Marketplace” on Google console.
5. Type in “openlitespeed” then click on openlitespeed-wordpress.
6. Click on Launch.
7. Choose zone near where most of your users are coming from. Note: pricing varies by zone and machine type. Not all zones / machine types, support the free trial.
8. Set boot disk to 30 Gb.
9. Make sure all the networking boxes are checked.
10. Click on IP address. Website is live - but not static. If you pause instance the IP address will change.
11. Go to external IP address.
12. Change IP address to static. Give it a name.
13. Get domain name from  Godaddy, domain, namecheap etc, and connect it to VM instance.
14. Go to DNS settings at domain.
15. Create new A record: host[ @ ] value[instance.ip.address].
16. Create new A record: host[www]  value[instance.ip.address].
17. Wait until changes have been propagated. Might take up to 24hrs.
18. Type domain into [whatsmydns.net](http://whatsmydns.net) to check propagation status.
19. Go to “deployment manager” on Google console. Click on your deployment.
20. Open SSH in browser window. Follow on-screen instructions. If you get an error then reload.
21. In new tab, go to domain name. Follow WP onscreen installation instructions.

## Increase SWAP memory limit

1. Open SSH in browser window. Copy and paste the following commands one by one.
2. sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
free -m
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab 
3. To check swap file: Type in htop

## Change upload limit

1. Go to **Plugin editor** from the left menu panel in the admin dashboard.
2. Top right, Choose **All-in-one WP Migration** from the dropdown and click select.
3. Click on **constants.php** file.
4. Search for the word AI1WM_MAX_FILE_SIZE. Change it’s value, refer below code:
define( ‘AI1WM_MAX_FILE_SIZE’, 2 << 28 );
To
define( ‘AI1WM_MAX_FILE_SIZE’, 536870912 * 5 );
If you even want more than 3 GB then change value from 536870912 * 5 to 536870912 * 10( or any number).
5. Update file.

## Modify .htaccess file via FileZilla

1. Open terminal then type `ssh-keygen -t rsa -C root -f ~/Desktop/id_rsa`
2. Type in a passphrase. You can also hit the ENTER key to accept the default (no passphrase).
3. Copy key to clipboard `pbcopy < ~/Desktop/id_rsa.pub`
4. Go to your Google Cloud account and navigate to Compute Engine >> VM Instances.
5. Click the Compute Engine Instance you need to access and click edit.
6. Scroll down to find the SSH section and click show and edit.
7. Click Add Item.
8. Paste the key in the provided field. Save.
9. Log into VM instance via SSH the paste `sudo nano /etc/ssh/sshd_config`
10. Within that file, find the line that includes `PermitRootLogin`
and modify it to `PermitRootLogin without-password`
11. Save the file by pressing ctrl+o then ctrl+x or ctrl+x >> y >> enter.
12. Reload system `sudo systemctl reload sshd.service`
13. Download and install FileZilla.
14. Open FileZilla and go to Edit >> Settings.
15. From the left settings menu navigate to Connections >> FTP >> SFTP.
16. Click **Add key file button** and select the private key you just created.
17. In the FileZilla dashboard, in the **Host** field enter `sftp://IPaddress`
18. In the **Username** field enter `root`. Click Quick Connect.
19. If the upload size has not changed go to the .htaccess file in the server root folder and add the below lines.
20. php_value upload_max_filesize 2048M
php_value post_max_size 2048M
php_value memory_limit 512M
php_value max_execution_time 300
php_value max_input_time 300