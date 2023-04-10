# PySpectrometer

## Software

### OS

* Raspberry PI 64 bit Lite - buster
	* https://downloads.raspberrypi.org/raspios_lite_arm64/images/raspios_lite_arm64-2021-05-28/2021-05-07-raspios-buster-arm64-lite.zip
* Using Raspberry Pi Imager write the image to the memory card
	* Hostname: pyspectrometer
	* Enable SSH with public key authentication
	* Username: pyspectrometer
	* Password: from 1Password
	* Wireless LAN Country: US
	* Time zone: America/Los_Angeles
	* Keyboard: us
* Identify IP of pyspectrometer
* `ssh pyspectrometer@<IP>`
* `sudo dphys-swapfile swapoff`
* Edit `/etc/dphys-swapfile` and change `CONF_SWAPSIZE` to `1024`
* Run:

  ```
  sudo dphys-swapfile setup
  sudo dphys-swapfile swapon
  sudo rpi-update 39821d33e777cde9ba1a3cc8a73cfdd62fbbd2de
  ```

* Run: `sudo shutdown -r now`
* Run:

	```
	sudo apt-get update
	sudo apt-get -y install xinput plymouth plymouth-themes pix-plym-splash xorg lightdm accountsservice build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev curl libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev ninja-build cmake libjpeg-dev libsdl2-dev libevent-pthreads-2.1-6 libevent-core-2.1-6 libevent-dev libcap-dev libatlas-base-dev ffmpeg libopenjp2-7 libfmt-dev git
	git clone https://github.com/kadaan/PySpectrometer2.git
	git clone https://github.com/pyenv/pyenv.git
	git clone https://github.com/raspberrypi/libcamera
	git clone https://github.com/freedesktop/mesa-drm.git
	git clone https://github.com/tomba/kmsxx
	git clone https://github.com/raspberrypi/picamera2
	git -C PySpectrometer2 checkout develop
	sudo cp -f $HOME/PySpectrometer2/splash.png /usr/share/plymouth/themes/pix
	sudo groupadd -r autologin
	sudo gpasswd -a pyspectrometer autologin
	sudo systemctl set-default graphical.target
	sudo $HOME/pyenv/plugins/python-build/install.sh
	sudo python-build 3.9.16 /usr/local/lib/python3.9.16
	sudo update-alternatives --install /usr/bin/python python /usr/local/lib/python3.9.16/bin/python 1
	sudo update-alternatives --set python /usr/local/lib/python3.9.16/bin/python
	sudo update-alternatives --install /usr/bin/python3 python3 /usr/local/lib/python3.9.16/bin/python3 1
	sudo update-alternatives --set python3 /usr/local/lib/python3.9.16/bin/python3
	sudo update-alternatives --install /usr/bin/pip pip /usr/local/lib/python3.9.16/bin/pip 1
	sudo update-alternatives --set pip /usr/local/lib/python3.9.16/bin/pip
	sudo update-alternatives --install /usr/bin/pip3 pip3 /usr/local/lib/python3.9.16/bin/pip3 1
	sudo update-alternatives --set pip3 /usr/local/lib/python3.9.16/bin/pip3
	```

* Run: `sudo shutdown -r now`
* Run:

	```
	pip3 install --upgrade pip
	sudo pip3 install RPi.GPIO
	pip3 install jinja2 ply pyyaml meson python-prctl
	cd $HOME/PySpectrometer2 && pip3 install -r requirements.txt
	```

* Run: `sudo shutdown -r now`
* Run:

  ``` 
  sudo ln -s /usr/local/lib/python3.9.16/lib/pkgconfig/python-3.9.pc /usr/lib/pkgconfig/python3.pc
  cd $HOME/libcamera && meson setup build && meson configure -Dpipelines=raspberrypi -Dcam=enabled -Dpycamera=enabled -Dtest=false -Dwerror=false build
  sudo env PYTHONPATH=$HOME/.local/lib/python3.9/site-packages ninja -C $HOME/libcamera/build -v -j 1 install
  cd $HOME/mesa-drm && meson build
  sudo env PYTHONPATH=/home/pyspectrometer/.local/lib/python3.9/site-packages ninja -C $HOME/mesa-drm/build -v -j 1 install
  cd $HOME/kmsxx && meson build
  sudo env PYTHONPATH=/home/pyspectrometer/.local/lib/python3.9/site-packages ninja -C $HOME/kmsxx/build -v -j 1 install
  pip3 install $HOME/picamera2
  ```

* `curl -sSL get.pimoroni.com/hyperpixel4-legacy | bash`
	* Continue: `Y`
	* Version: `2`
	* Proceed: `Y`
* Run: `hyperpixel4-rotate right`
* Update `/usr/share/plymouth/themes/pix/pix.script`, comment out:
	* `message_sprite = Sprite();`
	* `message_sprite.SetPosition(screen_width * 0.1, screen_height * 0.9, 10000);`
	* `my_image = Image.Text(text, 1, 1, 1);`
	* `message_sprite.SetImage(my_image);`
* Change `/etc/lightdm/lightdm.conf` adding the following to `[Seat:*]`:

  ```
  session-wrapper=/etc/X11/Xsession
  xserver-command = X -nocursor
  autologin-guest = false
  autologin-user = pyspectrometer
  autologin-user-timeout = 0
  ```

* Remove `dtoverlay=vc4-kms-v3d` from `/boot/config.txt`
* Add the following to `/boot/config.txt`

  ```
  dtoverlay=disable-bt
  boot_delay=0
  disable_splash=1
  dtoverlay=ov5647,media-controller=1
  camera_auto_detect=0
  ```

* Change `console=tty1` to `console=tty9 loglevel=3` in `/boot/cmdline.txt`
* Add `logo.nologo vt.global_cursor_default=0` to `/boot/cmdline.txt`
* Create `$HOME/.xsession`

  ```
  exec $HOME/PySpectrometer2/run.sh
  ```
* Create `$HOME/PySpectrometer2/run.sh`

  ```
  #!/usr/bin/env bash

  export LD_LIBRARY_PATH=/usr/local/lib/aarch64-linux-gnu
  export PYTHONPATH=/usr/local/lib/python3.9.16/lib/python39.zip:/usr/local/lib/python3.9.16/lib/python3.9:/usr/local/lib/python3.9.16/lib/python3.9/lib-dynload:/usr/local/lib/python3.9.16/lib/python3.9/site-packages:/usr/local/lib/aarch64-linux-gnu/python3.9/site-packages
  python3 $HOME/PySpectrometer2/src/PySpectrometer2.py --picam --waterfall
  ```

* `chmod +x $HOME/PySpectrometer2/run.sh`
* `sudo dphys-swapfile swapoff`
* Edit `/etc/dphys-swapfile` and change `CONF_SWAPSIZE` to `100`
* Run: 

  ```
  sudo dphys-swapfile setup
  sudo dphys-swapfile swapon
  ```

* `sudo shutdown -r now`

### Calibration Data
* Create `$HOME/caldata.txt` with contents:

  ```
  800
  44,125,252,396
  405.4,436.6,487.7,546.5
  ```

### TODO

* Read only filesystem
* Longer splashscreen
* Speed boot
* Shutdown splashscreen
* Smaller banner
* Options screen
