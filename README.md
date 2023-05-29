# VM Management Scripts for auto-shutoff of VMs when not in use

This script uses **Redis** to set a default time of 15 mins, after the 15 mins are over the machine will be shutdown automatically. 

To increase the time, the user has to ssh into the VM which will prompt for a duration for the technical session. Once the technical session is over the user can manually shut it down or wait for the set duration for the script to shut it down.

## Add to GCP Startup Script
Advanced Options > Management > Automation Script

```
sudo apt update
sudo apt install redis-server -y
sudo sed -i 's/supervised no/supervised systemd/g' /etc/redis/redis.conf
sudo sed -i 's/port 6379/port 9000/g' /etc/redis/redis.conf
sudo systemctl restart redis.service
```

## Add autoset.sh script
1. `cd /etc/init.d`
2. `sudo vi autoset.sh`
3. add the following:
```
#!/bin/sh
### BEGIN INIT INFO
# Provides: autoset.sh
# Required-Start: $all
# Required-Stop: $all
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: Start autoset.sh at boot time
### END INIT INFO


# Default time being 15 mins
redis-cli -p 9000 set shut test EX 900
```
4. Make it executable: `sudo chmod +x /etc/init.d/autoset.sh`
5. close and run this command `sudo update-rc.d autoset.sh defaults`

## Add to .bashrc to take input from user
1. run: `vi ~/.bashrc`
```
echo "Hi There; please provide how long you want to have this technical session running in mins:"

read ttl

tts=$((ttl*60))

redis-cli -p 9000 set shut test EX $tts
```

## Autoshut using crontab
1. add the following `autoshut.sh` script in your home
```
#!/bin/sh

ttl=$(redis-cli -p 9000 ttl shut)

if [ $ttl -lt 0 ]; then
        $(sudo shutdown)
fi
```
2. run `crontab -e`
3. update the file with: `*/5 * * * *  bash /path/to/autoshut.sh`
4. reboot: `sudo reboot`

## To extend the time
1. If the time is over user will see the following message:
```
Broadcast message from root@dj-test-1 (Thu 2023-05-18 05:05:02 UTC):

The system is going down for poweroff at Thu 2023-05-18 05:06:02 UTC!
```
2. run `vi ~/timeextend.sh`
3. Add the following:

```
sudo shutdown -c
redis-cli -p 9000 set shut test EX 3600
```
4. Make it executable: `sudo chmod +x ~/timeextend.sh`
