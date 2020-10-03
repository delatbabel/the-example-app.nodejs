## The node.js example app

The node.js example app teaches the very basics of how to work with Contentful:

- consume content from the Contentful Delivery and Preview APIs
- model content
- edit content through the Contentful web app

The app demonstrates how decoupling content from its presentation enables greater flexibility and facilitates shipping higher quality software more quickly.

<a href="https://the-example-app-nodejs.herokuapp.com/" target="_blank"><img src="https://images.contentful.com/qz0n5cdakyl9/4GZmvrdodGM6CksMCkkAEq/700a527b8203d4d3ccd3c303c5b3e2aa/the-example-app.png" alt="Screenshot of the example app"/></a>

You can see a hosted version of `The node.js example app` on <a href="https://the-example-app-nodejs.contentful.com/" target="_blank">Heroku</a>.

## What is Contentful?

[Contentful](https://www.contentful.com) provides a content infrastructure for digital teams to power content in websites, apps, and devices. Unlike a CMS, Contentful was built to integrate with the modern software stack. It offers a central hub for structured content, powerful management and delivery APIs, and a customizable web app that enable developers and content creators to ship digital products faster.

## Requirements

* Node 8
* Git
* Contentful CLI (only for write access)

Without any changes, this app is connected to a Contentful space with read-only access. To experience the full end-to-end Contentful experience, you need to connect the app to a Contentful space with read _and_ write access. This enables you to see how content editing in the Contentful web app works and how content changes propagate to this app.

## Common setup

Clone the repo and install the dependencies.

```bash
git clone https://github.com/contentful/the-example-app.nodejs.git
cd the-example-app.nodejs
```

```bash
npm install
```

## Steps for read-only access

To start the express server, run the following

```bash
npm run start:dev
```

Open [http://localhost:3000](http://localhost:3000) and take a look around.


## Steps for read and write access (recommended)

Step 1: Install the [Contentful CLI](https://www.npmjs.com/package/contentful-cli)

Step 2: Login to Contentful through the CLI. It will help you to create a [free account](https://www.contentful.com/sign-up/) if you don't have one already.
```
contentful login
```
Step 3: Create a new space
```
contentful space create --name 'My space for the example app'
```
Step 4: [Seed](https://github.com/contentful/contentful-cli/tree/master/docs/space/seed) the new space with the example content model [`the-example-app`](https://github.com/contentful/content-models/tree/master/the-example-app). Replace the `SPACE_ID` with the id returned from the create command executed in step 3
```
contentful space seed -s '<SPACE_ID>' -t the-example-app
```
Step 5: Head to the Contentful web app's API section and grab `SPACE_ID`, `DELIVERY_ACCESS_TOKEN`, `PREVIEW_ACCESS_TOKEN`.

Step 6: Open `variables.env` and inject your credentials so it looks like this

```
NODE_ENV=development
CONTENTFUL_SPACE_ID=<SPACE_ID>
CONTENTFUL_DELIVERY_TOKEN=<DELIVERY_ACCESS_TOKEN>
CONTENTFUL_PREVIEW_TOKEN=<PREVIEW_ACCESS_TOKEN>
PORT=3000
```

Step 7: To start the express server, run the following
```bash
npm run start:dev
```
Final Step:

Open [http://localhost:3000?editorial_features=enabled](http://localhost:3000?editorial_features=enabled) and take a look around. This URL flag adds an “Edit” button in the app on every editable piece of content which will take you back to Contentful web app where you can make changes. It also adds “Draft” and “Pending Changes” status indicators to all content if relevant.

## Deploy to Heroku
You can also deploy this app to Heroku:

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)


## Use Docker
You can also run this app as a Docker container:

Step 1: Clone the repo

```bash
git clone https://github.com/contentful/the-example-app.nodejs.git
```

Step 2: Build the Docker image

```bash
docker build -t the-example-app.nodejs .
```

Step 3: Run the Docker container locally:

```bash
docker run -p 3000:3000 -d the-example-app.nodejs
```

If you created your own Contentful space, you can use it by overriding the following environment variables:

```bash
docker run -p 3000:3000 \
  -e CONTENTFUL_SPACE_ID=<SPACE_ID> \
  -e CONTENTFUL_DELIVERY_TOKEN=<DELIVERY_ACCESS_TOKEN> \
  -e CONTENTFUL_PREVIEW_TOKEN=<PREVIEW_ACCESS_TOKEN> \
  -d the-example-app.nodejs
```


## Performance Testing

This branch of the code includes some features to allow performance testing for the purpose of my training course.

This is the setup script that I have been using to set this application up on a CentOS 8 minimal install VM running under VirtualBox:

### Steps to Install as root

root installation is only used to install the base system packages.

```
# Install Node.JS, npm and PM2 (https://pm2.keymetrics.io/)
dnf module enable nodejs:12
dnf -y install nodejs
node --version
npm install pm2@latest -g

# Install git and other tools
dnf -y install git
dnf -y install psmisc
dnf -y update

# Enable port 3000 in the firewall
# This is the port on which the application runs
# If you want to have the application listening on port 80/443 then set up nginx or
# Apache as a front end reverse proxy.
firewall-cmd --zone=public --permanent --add-port=3000/tcp
firewall-cmd --reload
firewall-cmd --list-all

# Things needed to make the application build
dnf -y install python38 make
dnf -y groupinstall 'Development Tools'

# Performance tools
npm install -g chrome2calltree

# PM2 doesn't work under SELinux so disable that for the time being
# See https://stackoverflow.com/questions/62814539/pm2-keeps-getting-killed-every-90-seconds-on-centos-8
cat /etc/selinux/config | sed 's/=enforcing/=permissive/g' > /tmp/selinux.tmp
/bin/mv /tmp/selinux.tmp /etc/selinux/config
setenforce permissive
```

### Steps to Install as Non-Privileged User

As any other user on the system (I use the username "del"), run the following commands:

```
# Grab the branch with profiling support from github
git clone git@github.com:delatbabel/the-example-app.nodejs.git
cd the-example-app.nodejs/
git checkout feature/add-profiling-support
git pull

# Setup
npm install
pm2 startup

# This command makes pm2 run at boot time
# Change the username here as needed
sudo env PATH=$PATH:/usr/bin /usr/local/lib/node_modules/pm2/bin/pm2 startup systemd -u del --hp /home/del

# Start the app via pm2
pm2 --name contentful start node ./bin/www
pm2 list all

# Save the pm2 config so it restarts on boot
pm2 save

# Check that port 3000 is listening
netstat -plunt

# Now you can view the logs!
pm2 logs

# If you need to restart the app, do this:
pm2 restart all

# A simple terminal based dashboard to see what's going on while you're running jMeter
pm2 monit
```
