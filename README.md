HP P2000 Nagios Plugin
======================

This script is intended to be used with Nagios to monitor a HP P2000 SAN.
It performs status and performance checks and can output performance data.

##For this command to run the php-http module needs to be installed: http://php.net/manual/en/book.http.php or CURL libraries for version 1.6 and above. Only certain P2000 Firmware’s support the statistic commands and so the performance checks won’t work on  some SAN’s but the status check should work on all P2000 Firmware’s. Note you may need to run dos2unix check_p2000_api.php if you get an error about not finding /usr/bin/php. A Bug has been found in version 1.0.1 whereby it may not pick up a broken disk on later Firmware versions, v1.1 fixes this by using multiple different API calls to determine the health.##

By default the plugin checks the status of the P2000 SAN and so only the host address username and password.

./check_p2000_api.php -H 192.168.0.1 -U admin -P password

This will loop through all Enclosures, FAN’s, PSU’s, Temperatures, Voltages and Disks. This will not output any performance data. Below is an example of the output:

OK : Enclosure 1 OK, FAN 2 OK, PSU 2 OK, Temp 4 OK, Voltage 10 OK, Disk 10 OK,

Installing on Ubuntu
--------------------

To get the plugin working on Ubuntu you will need to install the pecl_http extension. To do this first install php-pear:

```bash
apt-get install  php-pear
apt-get install libcurl4-gnutls-dev
apt-get install make
pecl install pecl_http
```

You will be asked a few questions during this install just use all default answers. If you get an error saying it can’t find curl header make sure you have installed the curl libraries with the command above. If this installs successfully it should say “You should add “extension=http.so” to php.ini”. So lets do that using a text editor:

nano /etc/php/php

Add the line extension=http.so

If all that went well then you should now be able to use the script.

Installing on Groundwork
------------------------

Groundwork comes with its own version of PHP located in /usr/local/groundwork/php.

To use this version of PHP I had to download the pecl_http extension and compile it from source. Before that you need to make sure a few things are installed first. I used CentOS but the principle should be the same on other Distros:

```bash
yum install autoconf
yum install make
yum install zlib-devel
yum install libcurl-devel
```

Now you need to download the latest pecl_http extension source from here:

http://pecl.php.net/package/pecl_http

```bash
wget http://pecl.php.net/get/pecl_http-1.7.4.tgz
tar -xvf pecl_http-1.7.4.tgz
cd pecl_http-1.7.4.tgz
phpize
./configure
```

Make the build, test it and then install it

```bash
make
make test
make install
```

Once installed add “extension=http.so” (without the quotes) to  /usr/local/groundwork/php/etc/php.ini

Now change the first line of the script to:

```bash
#!/usr/local/groundwork/php/bin/php

Help
----
This script sends HTTP Requests to the specified HP P2000 Array to determine its health
and outputs performance data fo other checks.
Required Variables:
                -H Hostname or IP Address
                -U Username
                -P Password

Optional Variables:
                These options are not required. Both critical and warning must be specified together
                If warning/critical is specified -t must be specified or it defaults to greaterthan.
                -s Set to 1 for Secure HTTPS Connection
                -c Sets what you want to do
                                status - get the status of the SAN
                                disk - Get performance data of the disks
                                controller - Get performance data of the controllers
                                named-volume - Get the staus of an individual volume - MUST have -n volumename specified
                                named-vdisk - Get the staus of an individual vdisk - MUST have -n volumename specified
                                vdisk - Get performance data of the VDisks
                                volume - Get performance data of the volumes
                                vdisk-write-latency - Get vdisk write latency (only available in later firmwares)
                                vdisk-read-latency - Get vdisk read latency (only available in later firmwares)
                -S specify the stats to get for performance data. ONLY works when -c is specified
                -u Units of measure. What should be appended to performance values. ONLY used when -c specified.
                -w Specify Warning value
                -C Specify Critical value
                -t Specify how critical warning is calculated (DEFAULT greaterthan)
                                lessthan - if value is lessthan warning/critical return warning/critical
                                greaterthan - if value is greaterthan warning/critical return warning/critical
                -n Volume Name to be used with -c named-volume or -c named-vdisk

Examples
                Just get the status of the P2000
                ./check_p2000_api.php -H 192.168.0.2 -U manage -P !manage
                Get the status of the volume name volume1 on the P2000
                ./check_p2000_api.php -H 192.168.0.2 -U manage -P !manage -c named-volume -n volume1

                Get the CPU load of the controllers and append % to the output
                ./check_p2000_api.php -H 192.168.0.2 -U manage -P !manage -s 1 -c controller -S cpu-load -u "%"

                Get the CPU load of the controllers and append % to the output warning if its over 30 or critical if its over 60
                ./check_p2000_api.php -H 192.168.0.2 -U manage -P !manage -s 1 -c controller -S cpu-load -u "%" -w 30 -C 60

Setting option -c to anything other than status will output performance data for Nagios to process.
You can find certain stat options to use by logging into SAN Manager through the web interface and
manually running the commands in the API. Some options are iops, bytes-per-second-numeric, cpu-load,
write-cache-percent and others. If using Warning/Critical options specify a stat without any Units
otherwise false states will be returned. You can specify units yourself using -u option.
```