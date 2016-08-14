# mongodb-do-volume
How to install and remote connect to mongodb on DO external volume

## Extra
- To see your filesystem in Linux: df -h
- To see the systems console: dmesg

## Step 1: Create droplet
Create a droplet on DigitalOcean and add Volume.

## Step 2: See if your volume is added
See if you volume is added/connected to the droplet by commands:
```js
cd /dev/disk/by-id
ls
```

## Step 3: Configure droplet
```js
sudo parted /dev/disk/by-id/scsi-0DO_Volume_{VOLUME_NAME} mklabel gpt
sudo parted -a opt /dev/disk/by-id/scsi-0DO_Volume_{VOLUME_NAME} mkpart primary ext4 0% 100%
sudo mkfs.ext4 /dev/disk/by-id/scsi-0DO_Volume_{VOLUME_NAME}
sudo mkdir -p /mnt/{VOLUME_NAME}
echo '/dev/disk/by-id/scsi-0DO_Volume_{VOLUME_NAME} /mnt/{VOLUME_NAME} ext4 defaults,nofail,discard 0 2' | sudo tee -a /etc/fstab
sudo mount -a
```

## Step 4: Check if it worked
Check if everything working by the command:
```js
df -h
```

And search for you volume with the right (or a little bit less GB than you configuration)

## Step 5: Install MongoDB (ubuntu 16.04)
```js
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
sudo apt-get update
sudo apt-get install -y mongodb-org
```

## Step 6: Change mongodb config
Open config file:
```js
sudo nano /etc/mongod.conf
```

Change path name & hide bindIp
```js
dbPath: /mnt/{VOLUME_NAME}
#bindIp: 127.0.0.1
```

## Step 7: Create auth
Start mongo + create auth

Command line 1:
```js
mongod --config /etc/mongod.conf
```

Command line 2:
```js
user {DATABASE}
// Just read/write
db.createUser({user:"{USERNAME}", pwd:"{PASSWORD}", roles:["readWrite","dbAdmin"]})

// Admin:
db.createUser({user:"{USERNAME}", pwd:"{PASSWORD}", roles:["readWrite","clusterAdmin","readAnyDatabase"]})

// Test
db.auth("{USERNAME}", "{PASSWORD}"); // if worked returns 1 ortherwise error
```

## Step 8: Start mongo
```js
  mongod --auth --fork --logpath /var/log/mongod.log --config /etc/mongod.conf  
```

## Step 9: Connect
Connect with mongoose
```js
  // (mongo) mongodb://{USERNAME}:{PASSWORD}@{IP}:{PORT:27017}/{DATABASE}
  mongoose.connect("mongodb://admin_user:1234@139.59.215.224:27017/admin");
```

Connect in shell
```js
  mongo mongodb://admin_user:1234@139.59.215.224:27017/admin
```

Connect on localhost
```js
  mongo admin -u {USERNAME} -p {PASSWORD}
```

## Todo:
// start firewall => sudo ufw enable
// First see this: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04
sudo ufw allow from your_other_server_ip/32 to any port 27017
sudo ufw status

## Interesting sources:
- https://docs.mongodb.com/manual/reference/method/db.createUser/
- https://www.digitalocean.com/community/tutorials/how-to-use-block-storage-on-digitalocean
