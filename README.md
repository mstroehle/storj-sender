# storj-miniproxy

This is a mini CDN proxy app built using libstorj and node-libstorj.
Working on setting up basic functionalities such as:
  * Listing the user's buckets
  * Adding/removing buckets
  * Uploading files to a bucket
  * Removing files from a bucket

## Sections
* [Connecting to a Bridge Server](#connecting-to-a-bridge-server)
* [Troubleshooting Bridge Access with Storj-SDK](#troubleshooting-bridge-access-with-storj-sdk)
* [Troubleshooting Bridge Access with Storj-Integration](#troubleshooting-bridge-access-with-storj-integration)
* [Verifying Bridge User Credentials](#verifying-bridge-user-credentials)
* [Initial Planning](#initial-planning)
* [Useful Docker Commands](#useful-docker-commands)

## Required Dependencies
  * [libstorj](https://github.com/Storj/libstorj)
  * [node-libstorj](https://github.com/Storj/node-libstorj)
  * dotenv

## Connecting to a Bridge Server
[Storj-SDK](https://github.com/Storj/storj-sdk) is recommended for setting up the bridge server for use with this app. See storj-sdk README for setup.
Inside your storj-sdk repo, you can check what containers are running with this command:
```bash
docker-compose ps
```
To run the bridge:
```bash
docker-compose up -d
```
To set host entries:
```bash
source ./scripts/set_host_entries.sh
```
To set the bridge address:
```bash
source ./scripts/setbr
```
You'll then be given the address to where the bridge is set. The `BRIDGE_URL` variable needs to be set to this address in your .env file.

Now, when you use the command `storj export-keys`, the resulting email, password, and encryption key need to be saved to your `.env` file respectively as `BRIDGE_EMAIL`, `BRIDGE_PASS`, and `ENCRYPT_KEY`.


### Troubleshooting Bridge Access with [Storj-SDK](https://github.com/Storj/storj-sdk)

Once inside the storj-sdk directory...
To check your hosts:
```bash
cat /etc/hosts
```
Use `mongo` to check which port that mongo is using. You should get back:
```
MongoDB shell version vx.x.x
connecting to: mongodb://127.0.0.x.yourPortNumber
```
To enter the mongo shell:
```
mongo db:yourPortNumber
```
You'll want to see what databases are available with `show dbs`, then `use storj-sandbox`.
You can view tables with `show tables`.
In order to find your activated bridge user, you can use:
```
db.users.find()
```
Then find the entry where `"activated" : true`.

In your `~/.storj` directory (for OSX) there should be a list of IP.json files. Make sure that the credentials in that file match the IP given by the `setbr` script inside `storj-sdk/`.
You may have to rename your .json file with the correct IP address.


### Troubleshooting Bridge Access with [Storj-Integration](https://github.com/Storj/integration)

When attempting to use the `bucketList` route to list buckets, I ran into the following error:
```
GET /bucketList - - ms - -
Error: Not authorized
    at Error (native)
```
This was because I had the incorrect `BRIDGE_EMAIL`, `BRIDGE_PASS`, and `ENCRYPT_KEY` in my .env file.
When using storj-integration, you need to `cd` into your storj-integration directory (wherever you cloned the repo) and then start the container that you have based on the storj-integration image. You can get its container-id as follows:
```bash
docker ps -a|grep storj-integration
```
Then you can start and attach to the container with:
```bash
docker start -ai <container-id>
```
The last step is to run the `start-everything.sh` script from the container you started/attached to:
```bash
/scripts/start-everything.sh
```
As long as you want to connect to the bridge server, you need to keep this terminal window running.
You can then use pm2 commands to view logs etc.

### Verifying Bridge User Credentials
You can check if your bridge user credentials are working using the `list-buckets` command:
```bash
storj -u http://localhost:8080 list-buckets
```
To check your current bridge username, password, and encryption key, you can also use the command:
```bash
storj -u http://localhost:8080 export-keys
```
<b>These credentials</b> are the ones that need to be in your `.env` file, respectively assigned to `BRIDGE_EMAIL`, `BRIDGE_PASS`, `ENCRYPT_KEY`.

You can also view what users are associated with the container.
First use the docker shell to get into a mongo shell:
```bash
mongo
```
Then connect to the `storj-sandbox` database and look for users:
```bash
db = connect('storj-sandbox')
db.users.find({})
```

## Development Process

#### Current Goals
- Each bucket should be a route to another page that lists files inside the bucket
- Each bucket page should also have options to upload/download from the bucket

#### Other Things to Consider
- At what point to decrypt the file? (use FlipStream.js?)

#### Issues
- Ended up creating a bunch of test user logins for both storj-integration and storj-sdk
  - Everyday some new problem with connecting to the bridge due to inconsistent credentials on my part (seems like either the sdk or integration defaults to a login other than the one in my .env file)
  - Majority of problems have been due to incorrect bridge user login

## Useful Docker Commands
To see what docker containers are running:
```bash
docker ps
```

To stop a specific container from running:
```bash
docker stop <container id>
```

To stop all running docker containers:
```bash
docker stop $(docker ps -q)
```
