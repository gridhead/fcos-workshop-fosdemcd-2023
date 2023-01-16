# Your Minecraft server

**NOTE** - These steps were confirmed to be working on the below mentioned 
guest operating system, hypervisor configurations and specifications. Certain 
amount of tweaking is necessary to make it work on distinct environments.

## Steps

### Step #1 - Setup the environment
Ensure the local environment meets the recommended requirements or the remote 
environment is reachable in the network, to begin with. In this case, we would 
setup host on a local VM.

#### Hypervisor used
**VMware® Workstation 17 Pro v17.0.0 build-20800274**  
Running on Fedora Workstation 37

#### Specifications provided
* 32GiB virtualized disk space
* 8GiB RAM
* 4 virtualized CPUs having 1 virtualized threads each
* UEFI boot
* Bridged network

#### Installation method
**Fedora CoreOS 37.20221225.3.0**  
Using the live media of STABLE stream

### Step #2 - Configure the host
Jot down the configuration variables like users, groups, keypairs, config 
files, systemd units in a convenient YML file to use and reuse it. In this 
case, We would start service containers.

#### On the host device

1. Clone the repository to a local directory.
   ```
   git clone https://github.com/t0xic0der/fcos-workshop-fosdemcd-2023.git
   ```
2. Install `butane` using the default package manager, if not installed 
   already.
   ```
   sudo dnf install butane
   ```
3. Navigate into the directory containing the `butane` configuration and the
   `ignition` files, i.e. `fcosmine.bu` and `fcosmine.ign` respectively.
   ```
   cd fcos-workshop-fosdemcd-2023/fcosmine/buteconf/
   ```
4. Modify the `butane` configuration to your liking and generate the `ignition`
   file by executing the following command.
   ```
   butane --pretty --strict fcosmine.bu > fcosmine.ign
   ```
5. Start an HTTP server in the same directory. In this case, we will use an
   intrinsic Python 3 HTTP server on port 8080.
   ```
   python3 -m http.server 8080
   ```

#### On the guest device

1. Make sure that the guest device is connected to the same network as that of
   the host device and has an active internet connection.
2. Mount the live ISO image for the Fedora CoreOS stable stream on the guest 
   device and boot into it.
3. Once booted up, type in the following command to begin configuring the 
   Fedora CoreOS installation on the storage device.
   ```
   sudo coreos-installer \
        install /dev/sda \
        --ignition-url http://<address-of-the-host-device>:8080/fcosmine.ign \
        --insecure-ignition
   ```
4. Wait for the installation to be created and configured and then, execute the
   following command to reboot the guest to the installation.
   ```
   sudo systemctl reboot
   ```
5. Once the guest installation has booted up, monitor the `systemd` service 
   `install-podman-compose-and-firewalld.service` by executing the following 
   command and wait for the guest device to reboot automatically.
   ```
   journalctl --follow --unit install-podman-compose-and-firewalld.service
   ```
6. After the guest installation has booted up, monitor the `systemd` services 
   `allow-minecraft-server-through-firewall.service` and 
   `start-minecraft-dedicated-server.service` by executing the following 
   commands and wait for the services to successfully complete.
   ```
   sudo systemctl status allow-minecraft-server-through-firewall.service
   ```
   ```
   journalctl --follow --unit start-minecraft-dedicated-server.service
   ```

### Step #3 - Become creative
That's really it! One config file and the entire host environment gets started 
with the purpose. Something looks off? Rinse and repeat. In this case, we would
connect to the server.

#### On the devices in the same network

1. Install and log into **Minecraft Java Edition** (`v1.19.2`, if the compose 
   configurations are retained).
2. From the main menu, select the **Multiplayer** option, then the **Add 
   server** option and the connect to the server address. It should be the 
   following if the compose configurations are retained.
   ```
   <address-of-the-guest-device>:9999
   ```

## Further reading
1. [Butane](https://coreos.github.io/butane/)
2. [Ignition](https://coreos.github.io/ignition/)
3. [Podman](https://podman.io/)
4. [Firewalld](https://firewalld.org/)
5. [Systemd](https://systemd.io/)
