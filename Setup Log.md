# Notes:
* ![[Pasted image 20260330151201.png]]
* ![[Pasted image 20260330151229.png]]
* ![[Pasted image 20260330151421.png]]
* Is this the issue that they just fixed?:
  ![[Pasted image 20260408133035.png]]
* I am Deciding to do a LXC/LXD container to hold my xilinx install
* Resources I'm using:
	* https://www.youtube.com/watch?v=aIwgPKkVj8s
	* https://www.centennialsoftwaresolutions.com/help/set-up-lxc-for-vitis-vivado-and-petalinux-development/
	* https://threespeedlogic.com/running-vivado-on-lxc.html#the-problem
# LXD/LXC Install Info / Log:
## Install Log:
* Install LXD first:
  `sudo snap install lxd`
* Give user appropriate permissions to interact with LXD:
  `sudo usermod -aG lxd` (log out and back in)
* Then run lxd init:
  `sudo lxd init` (options selected shown below)![[Pasted image 20260330215307.png]]
* Find appropriate container using `lxc image list ubuntu: <version>`
* Then pull and launch it using:
  `lxc launch ubuntu:22.04 xilinx-2204`
* Now make the directories that Xilinx will expect when installed:
  ```
  sudo mkdir -p /tools/Xilinx/ 
  sudo chown -R $(id -u):$(id -g) /tools/Xilinx 
  mkdir -p ~/work
  ```
* Then edit the container config to attach the relevant xilinx directories
  ```lxc config edit xilinx-2204```
    * Edit the line that says:
      `devices: {}`
    * And change it to:
      ```yaml
      devices:
        sf_work:
          path: /work
          source: /home/YOURUSER/work
          type: disk
          shift: "true"
        sf_xilinx:
          path: /tools/Xilinx
          source: /tools/Xilinx
          shift: "true"
          type: disk
	    sf_downloads:
	      path: /home/ubuntu/Downloads
		  shift: "true"
		  source: /home/brooklyn/Downloads
		  type: disk

      ```
	* **you must replace YOURUSER with your actual username**
	* The "shift" options escalate permissions within these directories to the user outside of the container
	* If you decided to use a distro other than ubuntu, there might be a different default user than `ubuntu` and you will need to adjust your directories accordingly
* Identify the display number being used on the host machine (differentiates between X11 and Wayland , 0 or 1 respectively)
  `echo $DISPLAY`
* Add the following lines to the `devices:` section of your config to pass your GPU and display in, replacing the "X?" with X and the number returned by the previous command on the host PC. 
  ```YAML
  mygpu:
    type: gpu
  Xsocket:
    bind:  container
    type: proxy
    connect: unix:@/tmp/.X11-unix/X?
    listen: unix:@/tmp/.X11-unix/X0
    security.uid: "1000"
    security.gid: "1000"
  ```
	* This gives the container GPU access, and passes in an X11 connection to the container
* Also, add the line `environment.DISPLAY: :0` to the `config:` section of the file, then save and exit.
* At this point, xeyes should work
* Now to set up sound (mostly optional, but nice)
* add the following line to the `config:` section of the LXC
  `environment.PULSE_SERVER: unix:/pulse`
* Then add the following portion to the `devices:` section
  ```YAML
  PAsocket:
    bind: container
    connect: unix:/run/user/1000/pulse/native
    gid: "1000"
    listen: unix:/pulse
    mode: "0777"
    security.gid: "1000"
    security.uid: "1000"
    type: proxy
    uid: "1000"
  ```
* Finally, enter the container using ``
  ```
sudo apt install pulseaudio pavucontrol
  ``` 
* Out of convenience, and out of caution of messing up permissions of stuff by running everything as root, I am adding an alias to my bashrc to launch into a bash shell of the ubuntu user in the container: 
  ```bash
	echo 'lxcsh() { lxc exec "$1" -- su - ubuntu; }' >> ~/.bashrc
	source ~/.bashrc
  ```
* Additionally, you will need to configure the user inside of the container to have the correct X11 display variable, which can be done using the following command: 
  `lxc exec xilinx-2204 -- bash -c "echo 'export DISPLAY=:0' >> /home/ubuntu/.bashrc"`
* you should now be able to use the command `lxcsh xilinx-2204` to open your xilinx-2204 container into the normal user
* For versions of Vitis starting with 2023, it is allowed to use unlimited ram. . . this means it can nuke your computer if you don't limit it, vitis doesn't provide tools to do this, but you can limit the amount of ram available to the container (I'd recommend no more than 2/3rds of your system memory) with the following command:
  ```bash
  lxc config set xilinx-2204 limits.memory <# of gigs>GB
  ```
## Relevant LXC info:
* Commands to start, stop, and view the container:
  `lxc start xilinx-2204`
  `lxc stop xilinx-2204`
* To pass a command into the container:
  `lxc exec xilinx-2204 -- <command here>`
* To create a snapshot:
  `lxc snapshot xilinx-2204 <nameOfSnapShot>'
* To list some info about a container: 
  `lxc info xilinx-2204`
* Restore from a snapshot:
  `lxc restore xilinx-2204 <nameOfSnapShot>`
* Set container to start always when the system starts:
  `lxc config set xilinx-2204 boot.autostart 1`
* Set container max memory:
  ```lxc config set xilinx-2204 limits.memory <# of gigs>GB```
* Show the config of the container:
  `lxc config show xilinx-2204`
# Xilinx Tooling Install Inside Container 2023.2:
* First, on the host, download the self extracting web installer, leave it in your downloads folder (which has been made accessible to the user in the container during config) [link](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools.html) 
* Now enter the container with `lxcsh xilinx-2204`, navigate to the downloads folder, and make the downloaded file executable by modifying its file permissions to add the executable flag. `chmod +x FPGAs_Adaptive(....).bin`
* Run the installer by prefacing its filename with ./
* Enter your email and password:![[Pasted image 20260331175809.png]]
* Then select the entire vitis suite:![[Pasted image 20260331175851.png]]
* Select only the portions Relevant to this Project:![[Pasted image 20260331175938.png]]
* Install to /tools/Xilinx:![[Pasted image 20260331180027.png]]
* Run this special command (as user in container) or vitis will hang when creating new projects:
  ``` bash
	  sudo /tools/Xilinx/Vitis/2023.2/scripts/installLibs.sh
  ```
* add vivado shortcuts to the shell (run as user inside the container):
  ```bash

	echo 'source /tools/Xilinx/Vitis/2023.2/settings64.sh' >> ~/.bashrc
  ```

# Git Setup:
Looking to properly set up credential management so that I don't have to deal with ssh inside of the container
- [Git Credential Manager](https://github.com/git-ecosystem/git-credential-manager) seems to be the best way to do it without setting up an ssh key pair.
- Their install instructions are found [here](https://github.com/git-ecosystem/git-credential-manager) 
- TLDR, grab the latest deb package from their releases, install it using the following commands:
```bash
sudo dpkg -i <path-to-package>
git-credential-manager configure
```
* Tell git to use it:
  ```bash
	git config --global credential.helper manager
  ```
* Install pass to manage the gpg keys you're going to use to authenticate:
  ```bash
	  sudo apt install pass
  ```
* Init a gpg key for it to use (replace fields with desired values):
  ```bash
  cat > /tmp/keygen.params << 'EOF'
  Key-Type: RSA
  Key-Length: 4096
  Passphrase:<YOUR_ENCRYPTION_KEY>
  Key-Usage: sign
  Subkey-Type: RSA
  Subkey-Length: 4096
  Subkey-Usage: encrypt
  Name-Real: <YOUR_REAL_NAME>
  Name-Email: <YOUR_REAL_EMAIL>
  Expire-Date: 0
  %commit
EOF
  gpg --batch --gen-key /tmp/keygen.params
  ```
* List the key (this step is just to make copying the key easier), the line after pub is the key-id
  ```shell
  gpg --list-keys --keyid-format LONG
  ```
  ![[Pasted image 20260407210155.png]]
* Then add the gpg key to pass:
  ```bash
  pass init <gpg-id>
  ```
* Tell git to store the credentials as a gpg
  ```bash
	git config --global credential.credentialStore gpg
  ```
* Repeat the same process inside of the container (if we hadn't set up a gui connection, we would also have to pass in a config option `git config --global credential.guiPrompt false`)
* The first time you clone, you'll be asked for two factor, subsequent attempts will not require the two factor

# Install Boards With Tcl:
Attempting to use the [Xilinx Board Store](https://github.com/Xilinx/XilinxBoardStore), following their [install guide](https://github.com/Xilinx/XilinxBoardStore/wiki/Accessing-the-Board-Store-Repository)
1) Open the Vivado TCL console (it can be exited by typing `exit`):
   ```bash
   vivado -mode tcl
   ```
2) Get the meta data (this hanged for like 15 seconds):
   ```tcl
   xhub::refresh_catalog [xhub::get_xstores xilinx_board_store]
   ```
3) Install everything because its easier (going to install a lot of boards, took like a minute), (added the uninstall of the iwave boards because they're misconfigured or smth):
   ```tcl
   xhub::install [xhub::get_xitems -filter {company == "digilentinc.com"}]
   xhub::uninstall [xhub::get_xitems -filter {company == "iwavesystems.com"}  -of_objects [xhub::get_xstores xilinx_board_store] 
   ```
4) Tell vivado where the boards are (these boards are installed at a user level instead of a system level, so you need to point vivado in the right direction):
   ```tcl
   set_param board.repoPaths [get_property LOCAL_ROOT_DIR [xhub::get_xstores xilinx_board_store]]
   ```

# Remove A Dead / Missing File Using Tcl:
Got a warning that a file was missing, and I knew that I didn't want it, so I went to go remove it from the project.
```tcl
remove_files sources_1/imports/imports/TopLevel/MainMemory.v
```
![[Pasted image 20260407214410.png]]


# 1st Bringup log:
### Issue 1: vivado dies when I click synthesize "realloc(): invalid pointer"
vivado would error only when I clicked synthesize, (synthesis would still complete in the background?)
#### What it looked like:
* ```bash
realloc(): invalid old size
Abnormal program termination (6)
Please check '/home/ubuntu/work/Caravel_NPU_FPGA_2025/hs_err_pid2247.log' for details
```

In the log it appeared to be an issue with libudev.so.1:
```
#
# An unexpected error has occurred (6)
#
Stack:
/lib/x86_64-linux-gnu/libc.so.6(+0x42520) [0x7f9264c42520]
/lib/x86_64-linux-gnu/libc.so.6(pthread_kill+0x12c) [0x7f9264c969fc]
/lib/x86_64-linux-gnu/libc.so.6(raise+0x16) [0x7f9264c42476]
/lib/x86_64-linux-gnu/libc.so.6(abort+0xd3) [0x7f9264c287f3]
/lib/x86_64-linux-gnu/libc.so.6(+0x89677) [0x7f9264c89677]
/lib/x86_64-linux-gnu/libc.so.6(+0xa0cfc) [0x7f9264ca0cfc]
/lib/x86_64-linux-gnu/libc.so.6(+0xa4c6c) [0x7f9264ca4c6c]
/lib/x86_64-linux-gnu/libc.so.6(realloc+0x122) [0x7f9264ca5862]
/lib/x86_64-linux-gnu/libudev.so.1(+0x15707) [0x7f923f873707]
/lib/x86_64-linux-gnu/libudev.so.1(+0x1bb1b) [0x7f923f879b1b]
/lib/x86_64-linux-gnu/libudev.so.1(+0x75ff) [0x7f923f8655ff]
/lib/x86_64-linux-gnu/libudev.so.1(+0x7b6b) [0x7f923f865b6b]
/lib/x86_64-linux-gnu/libudev.so.1(+0x10192) [0x7f923f86e192]
/lib/x86_64-linux-gnu/libudev.so.1(+0x105d3) [0x7f923f86e5d3]
/lib/x86_64-linux-gnu/libudev.so.1(udev_enumerate_scan_devices+0x2a1) [0x7f923f86f341]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libXil_lmgr11.so(+0x12e165) [0x7f925c52e165]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libXil_lmgr11.so(xilinxd_52bd866351b78202+0x9) [0x7f925c52e5e9]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libXil_lmgr11.so(+0xdb467) [0x7f925c4db467]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libXil_lmgr11.so(xilinxd_52bd862318b59a70+0x86) [0x7f925c4db226]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libXil_lmgr11.so(+0xc879f) [0x7f925c4c879f]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libXil_lmgr11.so(xilinxd_52bd9e9e1c8e52fb+0x1b) [0x7f925c4d253b]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libXil_lmgr11.so(xilinxd_52bd700d1bd3c616+0x30) [0x7f925c4d25d0]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_commonxillic.so(XilReg::Utils::GetHostInfo[abi:cxx11](XilReg::Utils::HostInfoType, bool) const+0x1a0) [0x7f926147fe20]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_commonxillic.so(XilReg::Utils::GetHostInfoFormatted[abi:cxx11](XilReg::Utils::HostInfoType, bool) const+0x59) [0x7f9261486799]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_commonxillic.so(XilReg::Utils::GetHostInfo[abi:cxx11]() const+0x103) [0x7f9261486a53]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_commonxillic.so(XilReg::Utils::GetRegInfo(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, bool, bool)+0x96) [0x7f926148a406]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_commonxillic.so(XilReg::Utils::GetRegInfoWebTalk(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)+0x60) [0x7f926148a630]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_project.so(HAPRWebtalkHelper::getRegistrationId[abi:cxx11]() const+0x3d) [0x7f92407e2f4d]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_project.so(HAPRWebtalkHelper::HAPRWebtalkHelper(HAPRProject*, HAPRDesign*, HWEWebtalkMgr*, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)+0x178) [0x7f92407e5648]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_tcltasks.so(+0x1cd17a5) [0x7f92570d17a5]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_tcltasks.so(+0x1cdb03c) [0x7f92570db03c]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_common.so(+0xd7d9bf) [0x7f926657d9bf]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libtcl8.5.so(+0x3356f) [0x7f926083356f]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libtcl8.5.so(+0x76945) [0x7f9260876945]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libtcl8.5.so(+0x7e0f9) [0x7f926087e0f9]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libtcl8.5.so(TclEvalObjEx+0x76) [0x7f9260835216]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_common.so(+0xd7b3b3) [0x7f926657b3b3]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libtcl8.5.so(Tcl_ServiceEvent+0x7f) [0x7f92608a7bef]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libtcl8.5.so(Tcl_DoOneEvent+0x154) [0x7f92608a7f24]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_commontasks.so(+0x2bdc07) [0x7f92596bdc07]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_commontasks.so(+0x2c689e) [0x7f92596c689e]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_commontasks.so(+0x2c717f) [0x7f92596c717f]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_common.so(+0xd7d9bf) [0x7f926657d9bf]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libtcl8.5.so(+0x3356f) [0x7f926083356f]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libtcl8.5.so(+0x76945) [0x7f9260876945]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libtcl8.5.so(+0x7e0f9) [0x7f926087e0f9]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libtcl8.5.so(TclEvalObjEx+0x76) [0x7f9260835216]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_commonmain.so(+0xe752) [0x7f9267376752]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/libtcl8.5.so(Tcl_Main+0x1d0) [0x7f92608a02f0]
/tools/Xilinx/Vivado/2023.2/lib/lnx64.o/librdi_common.so(+0xda6f7b) [0x7f92665a6f7b]
/lib/x86_64-linux-gnu/libc.so.6(+0x94ac3) [0x7f9264c94ac3]
/lib/x86_64-linux-gnu/libc.so.6(+0x1268d0) [0x7f9264d268d0]
```
#### Solution
* According to the xilinx forum, this is a common issue when running in a container, and the initial solution is to always launch vivado with the following command in order to ensure it had the correct library loaded.
`LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libudev.so.1 vivado`
* According to the xilinx forum, this is an issue possibly caused by a 3rd party library thats used for internet stuff. . . (I think its an attempted telemetry call over udev devices. . . )
### Issue 2: 160% of IO used:
160% of all of the io was used, and it seemed like every single pin file was misconfigured, ![[Pasted image 20260408145902.png]]so I am attempting to install the digilent-vivado-boards repo boards instead of the boards store ones, because the board store doesn't seem to have the xc7a100t, atleast not in the normal install directory? /tools/Xilinx/Vivado/2023.2/data/xhub/boards/XilinxBoardStore,
![[Pasted image 20260408145902.png]] 
![[Pasted image 20260408150214.png]]
#### SOLUTION:
#### Actual solution:
* the top level is caravel, not TopLevel.v . . . 
* because of this, all of the IO of the userspace were trying to find matching pin definitions in the board file, these didn't exist, because they never existed, because it isn't the actual top level, & these connections are virtual/internal
* After switching to caravel everything was fine, build took a long time though
#### The boards part:
* The boards actually were installed, but boards installed through the tcl commands after vivado is installed get installed on a **user level**, so you need to run this command to tell vivado where they are (in the ~/.Xilinx folder) (**NEED TO RUN THIS WITH PROJECT OPEN**):
  ```   tcl
  set_param board.repoPaths [get_property LOCAL_ROOT_DIR [xhub::get_xstores xilinx_board_store]]
  ```

### Issue 3: "could not find the file" when opening project
#### Error Message:
```tcl
CRITICAL WARNING: [Project 1-311] Could not find the file '/home/ubuntu/work/Caravel_NPU_FPGA_2025/CARAVEL/CARAVEL.srcs/sources_1/imports/imports/TopLevel/MainMemory.v', nor could it be found using path '/home/ubuntu/work/Caravel_NPU_FPGA_2025/CARAVEL/C:/Xilinx/CaravelFPGA/Caravel_FPGA_2025/CARAVEL/CARAVEL.srcs/sources_1/imports/imports/TopLevel/MainMemory.v'.
```
#### Error description:
File was deleted but not properly removed from vivado project, so it pops up with a warning every time you open the project, but cant fix it in the GUI
#### Solution:
run this tcl command to remove a specific file (with project open)
```tcl
remove_files <absolutefilepath>
```

### Issue 4: IP has a different revision in the IP catalog
#### Error Message:
```tcl
WARNING: [IP_Flow 19-2162] IP 'clk_fix' is locked:
* IP definition 'Clocking Wizard (6.0)' for IP 'clk_fix' (customized with software release 2019.2) has a different revision in the IP Catalog.
```

#### Error Description:
vivado updates are evil, specific IP cores are out of date, and have applicable updates
#### Solution:
run this tcl command to upgrade **all IP**
```tcl
upgrade_ip [get_ips]

```

### Issue 5: Bajillion jou files updating even after gitignore added:
#### Description
Gitignores only prevent files that are *not yet tracked* from having their changes committed, changes to files that have already been committed will continue to register forever unless following commands:

#### Solution: (nuclear)
* remove *everything* from the git and add it all back, only files which the gitignore doesn't apply to will be re-added
```bash
git rm -r --cached .
git add .
git commit -m "Fixing gitignore"
```
* compress everything that remains
```bash
git gc --prune=now --aggressive

```
* eventually, properly clean the files out using https://github.com/newren/git-filter-repo to remove them from the history for ever

## Cross Compiling Caravel Firmware:
Used this guide in addition to what was present in the repo
https://riscv.epcc.ed.ac.uk/documentation/how-to/install-toolchain/

Ran into error when initializing submodules of toolchain repo:
#### Error:
```bash
Submodule path 'binutils': checked out '49d4d3fafa4ec4ff5a3460d91d5b1ed5286487db'
error: Server does not allow request for unadvertised object 12ec17e7f192febdf4a316e5bffd1a5d4a9ea698
fatal: Fetched in submodule path 'dejagnu', but it did not contain 12ec17e7f192febdf4a316e5bffd1a5d4a9ea698. Direct fetching of that commit failed.
fatal: 
```
#### Resolution:
pinned to hashes that don't exist anymore
```bash
git submodule update --init --recursive --remote
```

```bash
git clone https://github.com/riscv/riscv-gnu-toolchain
cd riscv-gnu-toolchain
git submodule update --init --recursive binutils gcc glibc linux-headers
```

```
git submodule sync --recursive #important part
git submodule update --init --recursive --remote
```

Then built with 

```bash 
sudo mkdir -p /opt/riscv sudo chown -R $(whoami) /opt/riscv
./configure --prefix=/opt/riscv --with-arch=rv32gc --with-abi=ilp32d
make linux -j10 #adjust down if stuff crashes
```