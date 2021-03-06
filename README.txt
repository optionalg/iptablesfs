iptablesfs
==========

iptablesfs is an easy, intuitive, filesystem-based interface to managing your iptables firewall on Linux. Protecting your server or workstation is as easy as this:

1) Run the installer. This copies the "firewall" shell script to /etc/init.d/ and creates a symlink into /etc/rcS.d/ so that the firewall is started on boot.
2) Run "/etc/init.d/firewall setup" to create the configuration directory.
3) If you don't want to reboot, just run "/etc/init.d/firewall start".

This will DROP all incoming packets except for those which belong to established connections.

Adding rules:

Want to allow access to HTTPS from anywhere on the Internet?
echo ANY_IP > /etc/firewall/eth0/tcp/443

Want to add access to HTTP from your home and work IP address ranges?
echo homeaddress > /etc/firewall/eth0/tcp/80
echo 172.16.80.1-20 >> /etc/firewall/eth0/tcp/80

DNS?
echo ANY_IP > /etc/firewall/eth0/udp/53
echo homeaddress > /etc/firewall/eth0/tcp/53

Allow ICMP ECHO REQUESTs from your home address but nowhere else?
echo homeaddress > /etc/firewall/eth0/icmp/echo-request

Allow access to SSH only from a specific source port at home?
echo homeaddress:50022 > /etc/firewall/eth0/tcp/22

Just remember when you have finished your configuration changes to apply them with:
/etc/init.d/firewall restart

ACTIONS
=======
The script takes an action as its argument. During the boot process, the operating system will specify the "start" action.

> start
	This will firstly apply the default policy, which is DROP, to the INPUT and FORWARD chains, for both IPv4 and IPv6 (if supported). The INPUT, FORWARD and OUTPUT chains are all flushed, so there are no rules. It will then apply the rules configured in your files, for each interface and for tcp, udp and icmp, in that order.

> stop
	No operation

> open
	This flushes all rules from INPUT FORWARD and OUTPUT, and then sets their policies to ACCEPT.
	
> seal
	This flushes all rules from INPUT FORWARD and OUTPUT, and and sets their policies to DROP. If you are logged in over SSH,
	there will be a warning, and the firewall rules will not be changed unless you run the command again with the "realseal"
	action. Realseal does not check where the user is logged in from before sealing the firewall. This will immediately and
	permanently interrupt SSH and all other connections.

> rules
	This will list all the current IPv4 and IPv6 rules in place, regardless of whether they were added by this tool or not.

> restart
	This is a synonym for start.

NOTES
=====
> Source addresses can be provided in any format iptables understands. It is bad security practice to use hostnames which require DNS lookups to resolve. It's OK to use hostnames which are in your /etc/hosts file.

> If you're installing this on a remote server over SSH, MAKE SURE YOU ADD A RULE to let you get back in once your established connection has been terminated:
echo $(echo $SSH_CLIENT | awk {print'$1'}) > /etc/firewall/eth0/tcp/22 && /etc/init.d/firewall restart

> /etc/init.d/firewall setup will create multiple directories under /etc/firewall - one for each interface that's up when you run the script. However, you can add rules for further interfaces, even if they aren't up yet. 

For example, to add control for VPN interfaces, you can make a tun+ directory under /etc/firewall, and then create the icmp, tcp and udp directories beneath it.

> To accept ESTABLISHED connections for an interface, simply touch the file ESTABLISHED within the directory for the interface. For example, /etc/firewall/eth0/ESTABLISHED.

You can limit this to just one protocol. For example:
rm /etc/firewall/eth0/ESTABLISHED && touch /etc/firewall/eth0/tcp/ESTABLISHED
or
rm /etc/firewall/eth0/ESTABLISHED && touch /etc/firewall/eth0/icmp/ESTABLISHED

> It's the same for ACCEPTing ALL packets on an interface or protocol. Just touch the ACCEPT file.
For example, to ACCEPT all packets coming over a VPN, touch /etc/firewall/tun+/ACCEPT.

If you ACCEPT all packets for an interface or protocol, no further rules are processed for that interface or
protocol, as the ACCEPT rule will always override any subsequent rules granting access.

> Don't forget - you need to re-run /etc/init.d/firewall restart after you've finished your changes.

> IPv6: in all modes EXCEPT "open", ALL IPv6 packets are dropped in the INPUT and FORWARD chains.
There is no mechanism currently for managing an IPv6 firewall with this script; it merely ensures
that the machine will not respond to IPv6.

Contact: jonnyhightower@funkygeek.com
Updated: 02/04/2013

