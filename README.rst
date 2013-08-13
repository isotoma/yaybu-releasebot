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


Build master environment
========================

You will need a ``buildbot_master`` account. The following shell commands assume you are that user.

Create a virtualenv::

    curl -O https://pypi.python.org/packages/source/v/virtualenv/virtualenv-1.10.1.tar.gz
    tar xvfz virtualenv-1.10.1.tar.gz
    cd virtualenv-1.10.1
    python virtualenv.py /Users/buildbot_master/Virtualenv

Install buildbot and its dependencies::

    ~/Virtualenv/bin/pip install -r master-requirements.txt

Create a Launchd plist for it (owned by ``root:wheel``) at ``/Library/LaunchDaemons/buildbot_master.plist``::

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>StandardOutPath</key>
        <string>twistd.log</string>
        <key>StandardErrorPath</key>
        <string>twistd-err.log</string>
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
        <string>net.buildbot.master</string>
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
        <string>/Users/buildbot_master/Instance</string>
    </dict>
    </plist>


Build slave environment
=======================

You will need a ``buildbot_slave`` account. The following shell commands assume you are that user.

Create a virtualenv::

    curl -O https://pypi.python.org/packages/source/v/virtualenv/virtualenv-1.10.1.tar.gz
    tar xvfz virtualenv-1.10.1.tar.gz
    cd virtualenv-1.10.1
    python virtualenv.py /Users/buildbot_slave/Virtualenv

Install buildbot and its dependencies::

    ~/Virtualenv/bin/pip install -r slave-requirements.txt

Create a Launchd plist for it (owned by ``root:wheel``) at ``/Library/LaunchDaemons/buildbot_slave_osx.plist``::

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>StandardOutPath</key>
        <string>twistd.log</string>
        <key>StandardErrorPath</key>
        <string>twistd-err.log</string>
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
        <string>net.buildbot.slave.osx</string>
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
        <string>/Users/buildbot_slave/Instance</string>
    </dict>
    </plist>

