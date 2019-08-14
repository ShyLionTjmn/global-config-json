This is a perl script, which load JSON associative array of devices, and by filter ssh to them and issue commands from script

Usage: global-config-json [-j json_data_file] [-f] [-v] [-l] [-a arg0 [-a arg1 [ ... -a arg9 ] ] script_file [ip/name/id ...]

	-f	do not fork, will slow work one by one
	-v	verbose
	-d	debug
	-r	use \\r instead \\n (for Mikrotiks)
	-l	save full log in any case
	-a argX	argument %X (0-9) to be replaced with argX

	script file syntax:

match data_key regexp
notch data_key regexp

user username, or will be asked for, once
pass password

l	login
e NN regex	expect for NN seconds
p command	print command
d	disconnect

Script vars:

# %n - short_name
# %Y,%M,%D year,month,day
# %h,%m,%s hour,minute,second
# %t time()


	if ip/name/id set, only these devices will procceed (script filter will apply anyway)
	By default /etc/global-config-json.json data file used.

Example data JSON:
==================
{
  "r1": {
    "short_name": "router1",
    "overall_status": "ok",
    "sysObjectID": ".1.3.6.1.4.1.40418.7.5",
    "data_ip": "1.2.3.4"
  },
  "r2": {
    "short_name": "router2",
    "overall_status": "ok",
    "sysObjectID": ".1.3.6.1.4.1.1.2.3",
    "data_ip": "1.2.3.5"
  }
}
==================

Example script:
============
match sysObjectID ^\.1\.3\.6\.1\.4\.1\.40418\.7\.5$
nomatch overall_status ^error$
#
user robot
pass somepass
# script rules
# %n - short_name
# %Y,%M,%D year,mont,day
# %h,%m,%s hour,minute,second
# %t time()
l
e 30 %n#
p conf t
e 30 %n\(config\)#
p int eth1/1-28
e 30 %n\(config-if-port-range\)#
p lldp management-address tlv
e 30 %n\(config-if-port-range\)#
p end
e 60 %n#
p write
e \[Y\/N\]:
p Y
e 60 %n#
p exit
d
==============

This example script will skip all devices with error overall status, and process only those, matching sysObjectID .1.3.6.1.4.1.40418.7.5


==============
Mikrotik specific
==============

Use username+cte as login, to disable color injections into output
Use -r parameter to send \r instead \n newline, otherwise it will not accept commands.

==============
JSON
==============
data_ip and short_name are mandatory fields
