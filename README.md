UserLAnd allows you to run a Linux distribution (like Ubuntu) on your Android without root access, creating a Linux environment where you can install and run SearXNG.

### Prerequisites

1. **Android Device with Sufficient Storage and RAM**: Ensure your device has enough storage (at least 4GB for UserLAnd, Ubuntu, and SearXNG) and ideally 4GB of RAM for smoother performance.
2. **UserLAnd App**: Install [UserLAnd from the Play Store](https://play.google.com/store/apps/details?id=tech.ula).
3. **Terminal and SSH Clients**: UserLAnd has its terminal interface, but you may also want Termux or JuiceSSH if you prefer them.
4. **Patience**: Setting up a server environment on Android may take time due to limited hardware resources.

### Step-by-Step Installation Guide

1. **Set up Ubuntu with UserLAnd**:
   - Open UserLAnd and select **Ubuntu** as the distribution.
   - Create a username, password, and VNC password when prompted (remember these for later).
   - UserLAnd will download and install Ubuntu on your device, creating a root filesystem.

2. **Open a Terminal Session in UserLAnd**:
   - Once Ubuntu is set up, start a terminal session. You’ll now be inside an Ubuntu environment where you can run commands.

3. **Update and Install Necessary Packages**:
   - Run the following commands to update the system and install dependencies for SearXNG:
     ```bash
     sudo apt update && sudo apt upgrade -y
     sudo apt install -y python3-dev python3-babel python3-venv uwsgi uwsgi-plugin-python3 git build-essential libxslt-dev zlib1g-dev libffi-dev libssl-dev
     ```

4. **Create a New User for SearXNG**:
   - This step is optional but recommended for security and organization.
   - Run the following commands to create a new user for SearXNG and set up the necessary directory:
     ```bash
     sudo useradd -m -d /usr/local/searxng searxng
     sudo mkdir "/usr/local/searxng"
     sudo chown -R "searxng:searxng" "/usr/local/searxng"
     ```

5. **Install SearXNG and Dependencies**:
   - Switch to the new user and clone the SearXNG repository:
     ```bash
     sudo -u searxng -i
     git clone "https://github.com/searxng/searxng" "/usr/local/searxng/searxng-src"
     ```
   - Create a virtual environment for SearXNG and activate it:
     ```bash
     python3 -m venv "/usr/local/searxng/searx-pyenv"
     echo ". /usr/local/searxng/searx-pyenv/bin/activate" >> "/usr/local/searxng/.profile"
     ```
   - Exit and start a new session to ensure the virtual environment is active:
     ```bash
     exit
     sudo -u searxng -i
     ```
   - Confirm the Python environment and install additional packages:
     ```bash
     pip install -U pip setuptools wheel pyyaml
     cd "/usr/local/searxng/searxng-src"
     sudo pip install --use-pep517 --no-build-isolation -e .
     ```

6. **Configure SearXNG**:
   - Copy the default settings file:
     ```bash
     sudo mkdir -p "/etc/searxng"
     sudo cp "/usr/local/searxng/searxng-src/utils/templates/etc/searxng/settings.yml" "/etc/searxng/settings.yml"
     ```
   - Edit the settings file (`/etc/searxng/settings.yml`) to customize it for your use. The key configurations include:
     - `server.secret_key`: Set a unique secret key. You can use this command `python3 -c 'import secrets; print(secrets.token_hex(32))'` to generate a random secret. Remember, you will need to set this or searxng will not run.
     - `general.instance_name`: Customize the instance name.
     - Adjust other settings as needed, such as safe search, image proxy, and enabled plugins.

7. **Check the SearXNG Setup**:
   - Enable debugging to test if everything is working correctly:
     ```bash
     sudo sed -i 's/debug: false/debug: true/' "/etc/searxng/settings.yml"
     ```
   - Start the SearXNG web application:
     ```bash
     export SEARXNG_SETTINGS_PATH="/etc/searxng/settings.yml"
     python searx/webapp.py
     ```
   - Open your Android browser and go to [http://127.0.0.1:8888](http://127.0.0.1:8888) to see the SearXNG interface. If it loads correctly, SearXNG is up and running.

8. **Access from Your PC**:
   - To access SearXNG from your PC, you’ll need to connect over the local network. You may need to set up port forwarding or configure UserLAnd to make the SearXNG instance accessible via your device’s IP address on port `8888`.
   - Open your browser on the PC and navigate to `http://[your Android device's IP]:8888`.

9. **Disable Debugging and Exit**:
   - Once confirmed, disable debugging by reversing the change:
     ```bash
     sudo sed -i 's/debug: true/debug: false/' "/etc/searxng/settings.yml"
     ```

### Important Precautions

- **System Resources**: UserLAnd may limit performance, so avoid running other heavy apps while using SearXNG.
- **Data Persistence**: Backup your configuration files, as UserLAnd environments may reset under some circumstances.
- **Security**: Running SearXNG on a public network may expose your device to risks. Ensure it's only accessible on your local network or use a VPN.


## Find your Android device's IP address

1. **Go to Settings** on your Android device.
2. **Open Wi-Fi Settings**:
   - Tap on **Wi-Fi**.
   - Connect to your Wi-Fi network if you aren’t already.
3. **View Network Details**:
   - Tap on your connected Wi-Fi network’s name to open its details.
   - Look for **IP address** under the network details.

The IP address listed is usually something like `192.168.x.x` or `10.0.x.x` for local networks. Use this IP to access your SearXNG instance from other devices on the same network.

Alternatively, if you’re in a terminal in UserLAnd, you can find the IP address by running:

```bash
ip a
```

Look for the `inet` address under `wlan0` or `eth0` (the wireless or ethernet interface), which will show your local IP.

## Make SearXNG accessible from any device

To make SearXNG accessible from any device on your network, you need to set its IP address to `0.0.0.0` in the configuration. Here’s how to do it:

1. **Open the SearXNG settings file**:
   ```bash
   sudo nano /etc/searxng/settings.yml
   ```

2. **Configure the server to listen on all IP addresses**:
   Add or modify the following lines in the `settings.yml` file under the `server` section:
   ```yaml
   server:
     bind_address: "0.0.0.0"
     port: 8888
   ```

   - `bind_address: "0.0.0.0"` allows the server to listen on all network interfaces.
   - `port: 8888` is the port SearXNG will use, but you can adjust it if needed.

3. **Save and exit** the file (in nano, press `CTRL+X`, then `Y`, then `Enter`).

4. **Restart SearXNG** to apply the changes.

Now, you should be able to access your SearXNG instance from any device on the same network by navigating to `http://[your Android device's IP]:8888`. To find your Android device’s IP, you can use the `ip a` command in UserLAnd.
