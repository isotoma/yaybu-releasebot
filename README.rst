========================
Yaybu Release Automation
========================

This buildbot primarily exists to ensure reliable builds of the nightly channel for OSX.


Build environment
=================

 * Fairly vanilla Mac Mini
 * Mountain Lion
 * Sleep turned off
 * Set to auto-start after power failure
 * Full XCode installation (from App Store)
 * Install XCode command line tools (via preferences pane)


Users accounts
==============

The following accounts are needed::

  * ``buildbot_master``
  * ``buildbot_slave``

They are hidden from the login window (for cosmetic reasons)::

    sudo defaults write /Library/Preferences/com.apple.loginwindow HiddenUsersList -array-add buildbot_master buildbot_slave


Build master environment
========================

Become the ``buildbot_master`` user::

    sudo su - buildbot_master

Clone this repository::

    git clone git://github.com/yaybu/yaybu-releasebot ~/ReleaseBot

Create a virtualenv::

    curl -O https://pypi.python.org/packages/source/v/virtualenv/virtualenv-1.10.1.tar.gz
    tar xvfz virtualenv-1.10.1.tar.gz
    cd virtualenv-1.10.1
    python virtualenv.py /Users/buildbot_master/Virtualenv
    cd ..
    rm -rf virtualenv-*

Install buildbot and its dependencies::

    ~/Virtualenv/bin/pip install -r ~/ReleaseBot/master/requirements.txt

Create the sqlite database::

    ~/Virtualenv/bin/buildbot upgrade-master ~/ReleaseBot/master/

Create a Launchd plist for it (owned by ``root:wheel``) at ``/Library/LaunchDaemons/buildbot_master.plist``::

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>StandardOutPath</key>
        <string>twistd.stdout.log</string>
        <key>StandardErrorPath</key>
        <string>twistd.stderr.log</string>
        <key>EnvironmentVariables</key>
        <dict>
                <key>PATH</key>
                <string>/Users/buildbot_master/Virtualenv/bin:/opt/local/bin:/sbin:/usr/sbin:/bin:/usr/bin</string>
        </dict>
        <key>GroupName</key>
        <string>daemon</string>
        <key>KeepAlive</key>
        <dict>
                <key>SuccessfulExit</key>
                <false/>
        </dict>
        <key>Label</key>
        <string>com.yaybu.releasebot.master</string>
        <key>ProgramArguments</key>
        <array>
                <string>twistd</string>
                <string>--nodaemon</string>
                <string>-y</string>
                <string>./buildbot.tac</string>
        </array>
        <key>RunAtLoad</key>
        <true/>
        <key>UserName</key>
        <string>buildbot_master</string>
        <key>WorkingDirectory</key>
        <string>/Users/buildbot_master/ReleaseBot/master</string>
    </dict>
    </plist>

Start the service::

    sudo launchctl load -F /Library/LaunchDaemons/buildbot_master.plist

There should be a buildbot visible at::

    http://hostname:8080/


Build slave environment
=======================

Become the ``buildbot_slave`` user::

    sudo su - buildbot_slave

We will be using Xcode tools as the ``buildbot_slave`` user so need to agree to its T&C::

    xcodebuild -license

Clone this repository::

    git clone git://github.com/yaybu/yaybu-releasebot ~/ReleaseBot

Create a virtualenv::

    curl -O https://pypi.python.org/packages/source/v/virtualenv/virtualenv-1.10.1.tar.gz
    tar xvfz virtualenv-1.10.1.tar.gz
    cd virtualenv-1.10.1
    python virtualenv.py /Users/buildbot_slave/Virtualenv
    cd ..
    rm -rf virtualenv-*

Install buildbot and its dependencies::

    ~/Virtualenv/bin/pip install -r ~/ReleaseBot/slave/requirements.txt

Create a Launchd plist for it (owned by ``root:wheel``) at ``/Library/LaunchDaemons/buildbot_slave_osx.plist``::

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>StandardOutPath</key>
        <string>twistd.stdout.log</string>
        <key>StandardErrorPath</key>
        <string>twistd.stderr.log</string>
        <key>EnvironmentVariables</key>
        <dict>
                <key>PATH</key>
                <string>/Users/buildbot_slave/Virtualenv/bin:/opt/local/bin:/sbin:/usr/sbin:/bin:/usr/bin</string>
        </dict>
        <key>GroupName</key>
        <string>daemon</string>
        <key>KeepAlive</key>
        <dict>
                <key>SuccessfulExit</key>
                <false/>
        </dict>
        <key>Label</key>
        <string>yaybu.com.releasebot.slave.osx</string>
        <key>ProgramArguments</key>
        <array>
                <string>twistd</string>
                <string>--nodaemon</string>
                <string>-y</string>
                <string>./buildbot.tac</string>
        </array>
        <key>RunAtLoad</key>
        <true/>
        <key>UserName</key>
        <string>buildbot_slave</string>
        <key>WorkingDirectory</key>
        <string>/Users/buildbot_slave/ReleaseBot/slave</string>
    </dict>
    </plist>

Start it::

    sudo launchctl load -F /Library/LaunchDaemons/buildbot_slave_osx.plist

Slave should show as connected at this URL::

    http://hostname:8080/buildslaves


Setting up a Windows 7 slave (WIP)
==================================

First part of setup was done as the Administrator user (the user that is created when setting up Windows 7 from scratch).

Follow the instructions in the Yaybu.exe README (https://github.com/yaybu/yaybu.exe) up to (and including) installing the requirements.

Install pywin32 using the binary installer.

Install buildbot_slave::

    C:\Python27\python -m pip install buildbot_slave
    C:\Python27\Scripts\buildslave --version

Create a new slave::

    mkdir C:\ReleaseBot\
    mkdir C:\ReleaseBot\win7_slave
    C:\Python27\Scripts\buildslave -r C:\ReleaseBot\win7_slave cherrytree:9989 win7_slave pass

Test slave works::

    C:\Python27\Scripts\buildslave start .

Should see it connect in buildbot web UI.

Now to install the buildbot slave Windows service.

Create an account for the buildbot slave - ``buildbot_slave`` is ideal. It shouldn't be an adminstrator. But it does need to have the SeServiceLoginRight permission, which you can grant with ntrights from the Win2k3 resource kit. NOTE: Even when an adminstrator, you need to run this with an Administrator terminal or the command will fail::

    ntrights +r SeServiceLogonRight -u buildbot_slave

This next bit is a bit fiddly as it needs to create some registry keys that it doesnt have permission to \o/.

Create the service::

    C:\Python27\Scripts\buildbot_serice.py --user .\buildbot_spave --password password --startup auto install

Start the service. It will fail to start::

    net start buildbot

Using regedit find ``HKEY_LOCAL_MACHINE\SystemCurrentControlSet\services\Buildbot`` and give the ``buildbot_slave`` user ``Full Control`` in the permissions context menu.

Again, try to start the service. It will fail to start once more::

    net start buildbot

Refresh regedit (with F5) and you should now see a ``Parameters`` folder within the ``Buildbot`` key. It will be empty. Add a string key named ``directories`` and set it to ``C:\ReleaseBot\win7_slave``.

Now try and start it again. This time it should start, and you should see it connect in the buildbot web ui::

    net start buildbot


Final checks
============

First of all, reboot and make sure everything comes up::

    sudo reboot

