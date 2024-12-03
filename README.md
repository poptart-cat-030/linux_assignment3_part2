# linux_assignment3
Assignment 3 for ACIT 2420 (Linux System Administration)

## Introduction 

Hi, I'm Hillary. This repository will allow you to set up a nginx web server that will serve an HTML document containing the current date and my system information.
<br>
The server also has an uncomplicated firewall (UFW) to help secure it.

## Creating a system user

Run the following command:
```bash
useradd -r -m -d /var/lib/webgen -s /usr/bin/nologin webgen
```
- `-r` flag is for making a system user
- `-m` flag is for creating the home directory
- `-d` flag is for setting the home directory to `/var/lib/webgen`
- `-s` flag is for setting the shell to `/usr/bin/nologin`
- `webgen` is the name of the user

> [!Note]
We are creating a system user for this task instead of using a regular user or root to make our system more secure by preventing anyone from logging into that user. If the service is compromised, attackers are limited in what they can do. <br> Reference: https://www.devdungeon.com/content/how-create-secure-linux-system-user

## Moving files and creating directories

1. Locate the `generate-index.service` and `generate-index.timer` files from this repository
2. Move the files to `/etc/systemd/system` by running the following command for each file:
```bash
sudo mv name-of-file /etc/systemd/system
```

3. Locate the `generate_index` file from this repository
4. Create a directory `bin` to put this file in:
```bash
sudo mkdir /var/lib/webgen/bin
```

5. Move `generate_index` to the `/var/lib/webgen/bin` directory you just created:
```bash
sudo mv generate_index /var/lib/webgen/bin
```

6. Create a directory `HTML` in `/var/lib/webgen/`:
```bash
sudo mkdir /var/lib/webgen/HTML
```
The `HTML` directory will store the html that will be created when the `generate_index` file is ran

## Setting permissions

1. Make the user `webgen` have full permissions to its home directory and all the files within:
```bash
chown -R webgen:webgen /var/lib/webgen
```

2. Add execute permissions to the `generate_index` script:
```bash
chmod +x /var/lib/webgen/bin/generate_index
```

## Starting and enabling timers and services

1. Run the following command:
```bash
sudo systemctl daemon-reload
```

2. Start `generate-index.timer`:
```bash
sudo systemctl start generate-index.timer
```

3. Test that `generate-index.timer` is active:
```bash
sudo systemctl list-timers
```
You should see a `generate-index.timer` in the resulting list

4. Test if `generate-index.service` is able to run:
```bash
sudo systemctl status generate-index.service
```

If you see no error messages then the service ran successfully. If you want to see the system logs for that service directly, then run this command instead:
```bash
journalctl -u generate-index.service
```

5. Enable `generate-index.service` to start upon booting:
```bash
sudo systemctl enable generate-index.service
```

## Configuring server with nginx

1. Install `nginx`:
```bash
sudo pacman -S nginx
```
Installing the package will create a new directory `/etc/nginx` and a new file `nginx.conf` in that folder

2. Navigate into `/etc/nginx`
3. Create a backup of the `nginx.conf` file and name it `nginx-backup.conf`:
```bash
sudo cp nginx.conf nginx-backup.conf
```

4. Create a `sites-available` and a `sites-enabled` directory:
```bash
sudo mkdir name-of-directory
```

5. Locate the `nginx.conf` file from this repository
6. Move the `nginx.conf` file to `/etc/nginx`:
```bash
sudo mv nginx.conf /etc/nginx
```
This will override the original `nginx.conf` file

7. Locate the `server-block.conf` file from this repository
8. Move the `server-block.conf` file to `/etc/nginx/sites-available`:
```bash
sudo mv server-block.conf /etc/nginx/sites-available
```

9. Create a symbolic link to `/etc/nginx/sites-available/server-block.conf`:
```bash
ln -s /etc/nginx/sites-available/server-block.conf /etc/nginx/sites-enabled/server-block.conf
```

10. Run the following command:
```bash
sudo systemctl daemon-reload
```

11. Test `nginx.conf`:
```bash
sudo nginx -t
```

12. Restart `nginx`:
```bash
sudo systemctl restart nginx.service
```

13. Enable `nginx` to start upon booting:
```bash
sudo systemctl enable nginx.service
```

14. Check status of `nginx`:
```bash
sudo systemctl status nginx.service
```

> [!Note]
It's important to use a separate server block file instead of modifying the main nginx.conf file directly because nginx server blocks allow you to run multiple websites with different configurations on the same server. <br> Reference: https://linuxize.com/post/how-to-set-up-nginx-server-blocks-on-ubuntu-22-04/ 

## Setting up UFW

1. Install `ufw`:
```bash
sudo pacman -S ufw
```

2. Enable `ufw.service` to start upon booting:
```bash
sudo systemctl enable ufw.service
```

3. Add the following rules to the firewall by running the following commands:
```bash
sudo ufw allow ssh
# allows ssh connection
```
```bash
sudo ufw allow http
# allows http connection
```
```bash
sudo ufw limit ssh
# sets rate limit on ssh for security purposes
```

4. Enable the firewall (`ufw`):
>[!warning]
>Do not enable the firewall before allowing ssh connection in the previous step or else you will lock yourself out of your own server

```bash
sudo ufw enable
```

5. Check status of `ufw`:
```bash
sudo ufw status verbose
```

