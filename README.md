# utone-linux

English | [日本語](README.ja.md)

This repository provides a Debian Live environment to easily run edge devices for the on-campus DJ event "The Utopia Tone". It includes [moip-toolkit](https://github.com/TechnoTUT/moip-toolkit) (NDI receive/transmit and digital signage) and [booth-display](https://github.com/TechnoTUT/booth-display) (booth display controller).  

## How to use
Prebuilt images are not distributed, because a built image contains the NDI runtime, which is covered by the NDI SDK license (see [License](#license)). Build the ISO image yourself as described in [How to build](#how-to-build), then write it to a DVD or USB drive. To write to a USB drive, use a tool like [Rufus](https://rufus.ie/).  
If you want to keep settings after reboot, specify the size of the persistent storage in Rufus settings. 1GB is sufficient for most cases. If you plan to use NDI transmission or signage, setting up persistent storage is recommended.  

Insert the completed DVD or USB drive into your PC and power it on to boot into the Live environment.  

### Receive NDI
To receive NDI signals, run the following command:  
```bash
nrx
```

To change the connection destination, edit the `.config/systemd/user/nrx.service` in your home directory and enable the service:  

```bash
vim ~/.config/systemd/user/nrx.service
systemctl --user enable --now nrx.service
```

`nrx` renders through KMS/DRM by default, which bypasses the desktop compositor and draws directly to the display for the lowest latency. To show it on a desktop session instead, use the commented-out `ExecStart` line with `--video-driver auto`. To choose a source manually:  
```bash
nrx -s "<NDI Source Name>"
```
- No desktop session may be running on that display (console / TTY state). `sudo systemctl set-default multi-user.target` makes the machine boot without a GUI.
- Add the user to the `video` and `input` groups: `sudo usermod -aG video,input $USER`
- Output is always fullscreen. If a desktop session owns the display, it falls back to a window on the desktop.

### Transmit NDI
To transmit NDI signals from a capture board, run the following command:  
```bash
ntx
```
Transmits NDI signals as `<HOSTNAME> (TX)`.  

If you want to change arguments such as the transmission name, resolution, or frame rate, edit `.config/systemd/user/ntx.service` in the home directory and enable the service. List of arguments can be checked with `ntx --help`.   
```bash
vim ~/.config/systemd/user/ntx.service
systemctl --user enable --now ntx.service
```
The default hostname is `debian`. To change the hostname, use the command `sudo hostnamectl set-hostname <new hostname>`.  

### Signage
Signage is provided by moip-toolkit. Start the Web UI service, then upload an image from your browser to show it fullscreen on the connected display:  
```bash
systemctl --user enable --now nweb.service
```
Open `http://<IP address>:8000/` in a browser (the IP address is shown at login). To keep the setting after reboot, use persistent storage.  

You can also show an image directly from the command line:  
```bash
moip signage /path/to/image.png
```
Add `--framebuffer` to draw directly to the Linux framebuffer without a GPU (`/dev/fb0`, the live user is in the `video` group).  
See the [moip-toolkit README](https://github.com/TechnoTUT/moip-toolkit) for details.

### Booth display
The [booth-display](https://github.com/TechnoTUT/booth-display) controller streams test patterns, video files or NDI sources to the tiled booth displays. Start it with:  
```bash
bth
```
To start it automatically, enable the service:  
```bash
systemctl --user enable --now booth.service
```
Open `http://<IP address>:8080/` in a browser to use the Web GUI. Display addresses and crop positions are set in `/opt/booth-display/config.yaml`.  

NDI input uses the NDI runtime (`libndi.so.6`) from the official NDI SDK for Linux. The ISO build downloads the SDK, accepts its license non-interactively and installs the runtime to `/usr/local/lib`, so read the NDI SDK license before you build.  
See the [booth-display README](https://github.com/TechnoTUT/booth-display) for details.

## How to build
### Using Dev Container (Recommended)
You can easily build the ISO using VS Code and Dev Containers (Docker/Podman required).  

1. Open this repository in VS Code.
2. Click "Reopen in Container" when prompted (or via Command Palette `Ctrl+Shift+P` -> `Dev Containers: Reopen in Container`).
3. In the terminal inside the container, run:
   ```bash
   make build
   ```
   To clean build artifacts:
   ```bash
   make clean
   ```

### Using Debian / WSL2
To build the ISO image, you need a Debian environment. If you are using Windows, you can easily set up a Debian environment using WSL2.    
```pwsh
wsl --update
wsl --list --online
wsl --install -d Debian
```

Install `live-build` and `make` in a Debian environment:  
```bash
sudo apt update
sudo apt install live-build make
```
To build the ISO image, clone this repository with the `--recursive` option:  
```bash
git clone https://github.com/TechnoTUT/utone-linux-public.git --recursive
cd utone-linux-public
```
Build the ISO image with `make`:  
```bash
make build
```
Successful execution will create `utone-linux-amd64.hybrid.iso` in the `work` directory.  
To clean up build artifacts, run `make clean`:  
```bash
make clean
```

## License
The files in this repository are released under the [MIT License](LICENSE). [moip-toolkit](https://github.com/TechnoTUT/moip-toolkit) and [booth-display](https://github.com/TechnoTUT/booth-display) are also MIT-licensed.  

A built ISO image also contains software downloaded during the build under its own license, including the NDI runtime from the NDI SDK for Linux. The NDI SDK license applies to it, so do not redistribute images you build. NDI® is a registered trademark of Vizrt NDI AB.  
