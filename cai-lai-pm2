# Reinstall Node.js and PM2 using NVM (User: deploy)

This document describes how to **remove old Node.js / NPM / PM2 and
reinstall using NVM** for the `deploy` user on Linux (EC2 / Amazon
Linux).

------------------------------------------------------------------------

# 1. Remove old Node.js, NPM and PM2

Login to the server and switch to root.

``` bash
sudo su -
```

Remove old NVM and Node/NPM/PM2 binaries.

``` bash
rm -rf /home/deploy/.nvm

rm -f /usr/bin/node
rm -f /usr/bin/npm

rm -f /usr/local/bin/node
rm -f /usr/local/bin/npm

rm -f /usr/bin/pm2
rm -f /usr/local/bin/pm2
```

------------------------------------------------------------------------

# 2. Switch to deploy user

``` bash
sudo su deploy
```

------------------------------------------------------------------------

# 3. Install NVM

``` bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

Load NVM into the current shell:

``` bash
export NVM_DIR="$HOME/.nvm"
. "$NVM_DIR/nvm.sh"
```

Verify NVM:

``` bash
command -v nvm
```

Expected output:

    nvm

------------------------------------------------------------------------

# 4. Install Node.js

Install Node.js version **20**:

``` bash
nvm install 20
```

Use the installed version:

``` bash
nvm use 20
```

Verify installation:

``` bash
node -v
npm -v
```

------------------------------------------------------------------------

# 5. Install PM2

Install PM2 globally:

``` bash
npm install -g pm2
```

Verify PM2:

``` bash
pm2 ls
```

If successful, PM2 will start its daemon and display an empty process
table.

------------------------------------------------------------------------

# 6. Configure PM2 to start on boot

Run:

``` bash
pm2 startup systemd -u deploy --hp /home/deploy
```

Then execute the command suggested by PM2. Example:

``` bash
sudo env PATH=$PATH:/home/deploy/.nvm/versions/node/v20.20.1/bin pm2 startup systemd -u deploy --hp /home/deploy
```

PM2 will create the service:

    /etc/systemd/system/pm2-deploy.service

and enable it.

------------------------------------------------------------------------

# 7. Save PM2 process list

After starting your applications with PM2, save them:

``` bash
pm2 save
```

If no processes are running, you may see:

    PM2 is not managing any process

------------------------------------------------------------------------

# 8. Common PM2 commands

List processes:

``` bash
pm2 ls
```

View logs:

``` bash
pm2 logs
```

Start application:

``` bash
pm2 start app.js
```

Restart application:

``` bash
pm2 restart app_name
```

Stop application:

``` bash
pm2 stop app_name
```

Delete application:

``` bash
pm2 delete app_name
```

------------------------------------------------------------------------

# 9. Check PM2 service

``` bash
systemctl status pm2-deploy
```

------------------------------------------------------------------------

# Result

After completing these steps:

-   Node.js is managed by **NVM**
-   PM2 runs under **deploy user**
-   PM2 automatically starts when the **server reboots**
