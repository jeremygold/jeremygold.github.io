---
layout: post
title: Hexapod autostart on boot
---

So this week's goal is to get the hexapod to automatically startup when the Raspberry PI boots. The hexapod has two key services that need to be running for it to work:

* The [ServoBlaster](https://github.com/richardghirst/PiBits/tree/master/ServoBlaster) daemon
* The ROS web_control.launch file

The servoblaster initialization is pretty straight forward - I've just created a simple startup script, then another script that will install it as a service, and ensure that it runs by default when the device boots.

```bash
#! /bin/sh
# /etc/init.d/servoblaster

### BEGIN INIT INFO
# Provides:          servoblaster
# Required-Start:    $syslog
# Required-Stop:     $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Startup script for ServoBlaster daemon
# Description:       A simple script for the Hexapod project that will start / stop the ServoBlaster Daemon
### END INIT INFO

case "$1" in
  start)
    echo "Starting servoblaster daemon"
    /home/pi/hexapod/Rpi/ServoBlaster/user/servod --p1pins="15,13,12,19,21,22,23,24,26,29,31,32,33,35,36,37,38,40"
    ;;
  stop)
    echo "Stopping servoblaster daemon"
    killall servod
    ;;
  *)
    echo "Usage: /etc/init.d/servoblaster {start|stop}"
    exit 1
    ;;
esac

exit 0
```

The installation script copies this file in to /etc/init.d, sets it as executable, then uses update-rc.d so that it runs by default on startup:

```bash
#!/bin/bash

if [[ $EUID -ne 0 ]]; then
   echo "This script must be run as root" 1>&2
   exit 1
fi

# Install and run servoblaster init script
cp servoblaster /etc/init.d/
chmod 755 /etc/init.d/servoblaster
/etc/init.d/servoblaster start

# Update to run by default
update-rc.d servoblaster defaults
```

The ROS initialization script was a bit more fiddly... I wanted to have the ROS system start on boot, but to be able to attach to the terminal, and debug / tweak things after boot as well... I decided to start up a tmux session, with the relevant ROS shells initialized...

The actual /etc/init.d/hexapod_ros script is pretty straight-forward:

```bash
#! /bin/sh
# /etc/init.d/hexapod_ros

### BEGIN INIT INFO
# Provides:          hexapod_ros
# Required-Start:    $syslog
# Required-Stop:     $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Startup script for Hexapod ROS services
# Description:       A simple script for the Hexapod project that will start / stop the ROS web service
### END INIT INFO

case "$1" in
  start)
    echo "Starting Hexapod ROS services"
    /home/pi/hexapod/Rpi/StartupScripts/HexapodRosTmux.sh
    ;;
  stop)
    echo "Stopping Hexapod ROS services"
    /usr/bin/tmux kill-session
    ;;
  *)
    echo "Usage: /etc/init.d/hexapod_ros {start|stop}"
    exit 1
    ;;
esac

exit 0
```

The fun begins in the HexapodRosTmux.sh script - I start up a new session, make sure that windows will hang around on exit (so I can see the debug trace if something goes wrong), and then fire up a separate window for roscore, and roslaunch for my main program.

HexapodRosTmux.sh:

```bash
#!/bin/zsh

# Ref http://raspberrypi.stackexchange.com/questions/58897/run-script-in-terminal-after-boot
/usr/bin/tmux new-session -d -s HexapodSession
/usr/bin/tmux set-option -t HexapodSession set-remain-on-exit on
/usr/bin/tmux new-window -d -n "RosCore" -t HexapodSession:1 -c '/home/pi/hexapod/ROS' 'sudo -u pi /home/pi/hexapod/Rpi/StartupScripts/HexapodRosCore.zsh'
sleep 10
/usr/bin/tmux new-window -d -n "RosWebControl" -t HexapodSession:2 -c '/home/pi/hexapod/ROS' 'sudo -u pi /home/pi/hexapod/Rpi/StartupScripts/HexapodRosWebControl.zsh'

exit 0
```

I had to create dedicated scripts for each of the windows, since they're executed as root, and the environment gets reset between command otherwise (and things like environment variables don't stick.

I also needed a dedicated 10 second sleep after starting roscore, and before starting web_control.launch, to be sure that the core services were ready... Would be great to somehow detect that roscore is initialized from the tmux startup script so we don't have to do an arbitrary sleep...

HexapodRosCore.sh:

```bash
#!/bin/zsh

. ./initRos.sh
roscore
```

HexapodRosWebControl.sh:

```bash
#!/bin/zsh

. ./initRos.sh
roslaunch web_control web_control.launch

```

After the PI boots, I can get into the tmux session by doing:

```bash
$ sudo su
# tmux a
```

If it's worked properly, I can point the browser at http://hexapodpi:5000/, and control the robot from here (more details on the ROS web server later)

![config.yml](/images/HexapodWebControl.png)
