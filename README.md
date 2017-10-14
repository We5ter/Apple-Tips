很多小伙伴买了扩展卡，却受到了唤醒无法挂载的，这里分享一下我的挂载脚本
下载sleepwatcher_2.2.tgz
然后再同一目录运行以下脚本（先保存为sleepwatcher.sh文件）

<pre><code>
#!/bin/bash

# acquire sudo at the beginning
sudo -v

# Keep-alive: update existing `sudo` time stamp until `.osx` has finished
while true; do sudo -n true; sleep 60; kill -0 "$$" || exit; done 2>/dev/null &

# unload launch agents
sudo launchctl unload /Library/LaunchDaemons/de.bernhard-baehr.sleepwatcher.plist 2>/dev/null
launchctl unload ~/Library/LaunchAgents/de.bernhard-baehr.sleepwatcher.plist 2>/dev/null

# remove plist launchagents
sudo rm -f /Library/LaunchDaemons/de.bernhard-baehr.sleepwatcher.plist
rm -f ~/Library/LaunchAgents/de.bernhard-baehr.sleepwatcher.plist

# remove executable and man files
sudo rm -f /usr/local/sbin/sleepwatcher
sudo rm -f /usr/local/share/man/man8/sleepwatcher.8

# download sleepwatcher package, untar, and cd into directory
tar xvzf sleepwatcher_2.2.tgz 2>/dev/null
cd sleepwatcher_2.2

# create folders necessary for installation
sudo mkdir -p /usr/local/sbin /usr/local/share/man/man8

# move files into installation folders
sudo cp sleepwatcher /usr/local/sbin
sudo cp sleepwatcher.8 /usr/local/share/man/man8
sudo cp config/de.bernhard-baehr.sleepwatcher-20compatibility.plist /Library/LaunchDaemons/de.bernhard-baehr.sleepwatcher.plist

sleep 1

# load launch agent
sudo launchctl load -w -F /Library/LaunchDaemons/de.bernhard-baehr.sleepwatcher.plist

# create script in local user directory and make them executable
sudo touch /etc/rc.wakeup
sudo touch /etc/rc.sleep
sudo chmod +x /etc/rc.sleep /etc/rc.wakeup

</code></pre>

然后用终端在用户家目录创建.wakeup以及.sleep两个文件
分别写入：
<pre><code>
# .sleep
# author:zephyru5(http://lightrains.org)
#!/bin/sh
osascript -e 'tell application "Finder" to eject "你的扩展卡名"'
osascript -e 'tell application "Finder" to delay 10'
</code></pre>

<pre><code>
#.wakeup
#author: zephyru5(http://lightrains.org)
#!/bin/sh
/usr/sbin/diskutil list | grep -e ' \+[0-9]\+: \+[^ ]\+ [^ ]\+' | sed 's/.*\(disk[0-9].*\)/\1/' | xargs -I{} /usr/sbin/diskutil mount {}
</code></pre>


最后一步
在终端赋予运行权限  
<pre><code> 
sudo chmod a+x .wakeup   
sudo chmod a+x .sleep
</code></pre>

done
