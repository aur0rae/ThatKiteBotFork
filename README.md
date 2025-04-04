# ThatKiteBot - A Discord bot for science and nonsense

# Section 1. Installing docker
The instructions below are for installing Docker specifically on **Debian Linux**, and will cover the current official methods for adding Docker's repositories to your installation. In section 4, you will also find **unnofficial** instructions for running on Arch Linux (btw). **If you get stuck, reference [Docker's documentation](https://docs.docker.com/engine/install/)** - these instructions may not be up to date!

> 📝 **Note:** If you are on a distribution other than **Debian Linux** or **Arch Linux**, please reference the aforementioned Docker docs, then skip to step 2. Note also that SELinux may cause issues and is not recommended for use with this project, nor is `wsl`.

### System requirements:
It is recommended you run the bot with **no less** than 2GB of RAM, though for the best experience, your host device should have at least 4GB of RAM. Raspberry Pi and other non-`amd64` computers are currently not supported, though this should be fixed soon.

## 1.1 Preinstallation
Verify that you do not have the unnofficial Docker packages from Debian's default repositories installed.
```sh
sudo apt-get remove docker.io docker-doc docker-compose podman-docker containerd runc
```
This will likely result in no packages being removed, especially on a fresh install. This is to be expected, and according to the documentation will not remove any images, containers, volumes, or networks stored in `/var/lib/docker/`.

## 1.2 Installing from the official Docker repositories
This step will add Docker's repositories to `apt` and will import their signing key. 

If you not on mainline Debian, substitute the line `$(. /etc/os-release && echo "$VERSION_CODENAME")` with the codename of the release it is based on. As of the time of writing, this is most likely `bookworm` (current stable) or potentially `trixie` (current testing).
```sh
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
To install the most recent version, run the following command:
```sh
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 1.3 Verify the installation
To ensure everything installed correctly, run Docker's "Hello, world!" script.
```sh
sudo docker run hello-world
```

## 1.4 Check your docker compose verison
Before running, make sure that your version of compose is at least 2.X. Note that `docker compose` is a more recent build than `docker-compose`, so if something breaks it may be worth attempting the older syntax.
```sh
docker compose version
```

It is also recommended for security that you add yourself to the docker group.
```sh
groupadd docker
usermod -aG docker $USER
```

---

# Section 2. Setup and first run

This will walk you through setting up and installing specifically the bot now that you have Docker installed. For the smoothest experience, you may want to set up a bot through Discord's Developer Portal prior to configuring the bot, however there is a simple guide later on that should get you through the basics. That said, you'll be on your own as far as authorizing it and adding it to your server.

## 2.1 Clone the repository
This will download the necessary files from GitHub for you to be able to build and run the Docker container.
```sh
git clone https://github.com/ThatRedKite/thatkitebot.git
```

## 2.2 Building the image
This will tell docker to grab all of the necessary dependencies and build the image, then run it. This step may take a while, and the bot should throw a token error; once this occurs everything should stop updating, and you'll be ready to close it with `ctrl+c` and move to the next step.
```sh
cd thatkitebot
docker compose up --build
```

## 2.3 Set API keys
Now you need to open up the bot's configuration file and give it a bot token so it can connect and run.
```sh
nano ./data/init_settings.json 
```
This will open the `nano` text editor where you will see something like this:
```json
{
    "discord token": "",
    "tenor api key": "",
    "prefix": "+",
}
```
`discord_token` is the discord bot API token you can get from the [Discord Developer Portal](https://discord.com/developers/) after making a bot. To do this, create an application, then go to the "bot" option in the settings bar to the left. From here, you'll generate a token and paste it inside the double quotes. 

> 📝 Note: Feel free to customize the bot while you're here - give it a name, profile picture, etc. 

### ⚠️ Warning: NEVER share your API keys with anyone, and do NOT edit this file in the GitHub interface!

The tenor api key is not necessary for operation.
After that is done hit `ctrl + x`, `y` and `enter`. The settings will be saved.

## 2.4 Starting the bot 
Start the bot from a stopped state (like we have right now).
```sh
docker compose up -d thatkitebot
```
This will start the bot in the background. If you do not wish to start the bot in the background, you can omit the `-d` and any logs will be printed to the terminal. To view the logs, run:
```sh
docker logs thatkitebot-thatkitebot-1
```

---

# Section 3. Maintenance
For security and stability, it is advised to allot some downtime to updating the bot whenever a new release comes out. This will prevent breakage should a dependency change upstream and should give access to all the latest functionality.

## 3.1 Updating
To update, you will need to sync your local copy of the bot with the `git` repo and rebuild the container. This is required to get any potential missing libraries.
```sh
git pull
docker compose up --build -d thatkitebot
```

To check the status of the container run `docker container ls`. You should see 2 containers: `redis:alpine` and `thatkitebot_thatkitebot`. That will mean everything is running. Assuming everything is functioning correctly, you should now have an online and functional bot. To double check, you can run `+help` in your server (unless you set a different prefix in an above step).
## 3.2 Stopping
To stop the bot, run
```
docker compose stop
```

---

# Section 4. Other distributions
As of the time of writing, Arch Linux is the only other supported distribution for the bot - and is the main one development is done on - though it will likely run on most others fine. Note that the method of installation is **NOT**, however, supported by Docker and will involve third-party packages.

## 4.1 Arch Linux

Install the `docker` and `docker-compose` packages from the `Extra` repository.
```sh
sudo pacman -S docker docker-compose
```

Enable either the Docker service or socket. Enabling the service will start the daemon on boot, while the socket will allow it to be started whenever a container is run.
```sh
sudo systemctl enable docker.service
```
or
```sh
sudo systemctl enable docker.socket
```

Next, add yourself to the `docker` group to allow commands to run without `sudo` permissions.
```sh
groupadd docker
usermod -aG docker $USER
```

---

Made with ❤️
