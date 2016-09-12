
cse199 -- In-class Protocol Activity -- Manual Generation of Emails -- 2016-09-07


----------
Goal: to have a locally installed SMTP server that:
	(0)  unless otherwise restricted (below), furnishes typical SMTP service to clients
	(1)  allows Telnet access to generate an outgoing email (i.e. a relay service)
	(2)  restricts relay service (i.e., Telnet access) to be available only to computers on a particular subnet (e.g.., U.B. 128.205.x.x)
	(3)  restricts outgoing email to a specific destination address
	(4)  listens on a nonstandard port (e.g. 10025) in lieu of the default smpt port (25)


----------
Step-by-step:

(1)  Install Postfix:  "sudo apt-get install postfix"

(2)  Perform a default (re)configuration of Postfix:  "sudo dpkg-reconfigure postfix".  Select
	"Internet Site" from the menu, and accept all default options.

(3)  Install Postfix library support for Perl regexes:  "sudo apt-get install postfix-pcre"
	This will be necessary to support outgoing whitelisting using the method described below.

(4)  Edit the /etc/postfix/main.cf file to enable relay (Telnet) access for nodes in desired IP nets.  For example,
	to enable computers in the 128.205.36.X subnet (timberlake is 128.205.36.8) to be able to relay email, 
	edit the _existing_ "mynetworks" line and add "128.205.36.0/24".  The line when finished should look something like this:

		mynetworks = 127.0.0.0/8 128.205.36.0/24 [::ffff:127.0.0.0]/104 [::1]/128

(5)  Edit the /etc/postfix/main.cf file to restrict relay destination to a particular email address.  For example,
	to restrict the destination email address to "submissions@internet-class.org", add a _new_ line:

		smtpd_recipient_restrictions = check_recipient_access pcre:/etc/postfix/receiver_whitelist

	A good place to put this line is right above the existing "smptd_relay_restrictions" line.  Then, add a new
	file /etc/postfix/receiver_whitelist (assuming the path is /etc/postfix and matching the filename "receiver_whitelist"
	as named in the "smtpd_recipient_restrictions" line above).  Include the following two lines in the file:

		/^submissions@internet-class.org$/ OK
		/^.*$/ DEFER

	These lines are regex that whitelist the target email destination and block everything else.

(6)  Change the listening port as desired.  For example, to have Postfix listen on tcp port 10025 in lieu of the default port 25,
	edit the /etc/postfix/master.cf file.  There should be a bunch of lines starting with "smpt".  Look for this one:

		smtp      inet  n       -       y       -       -       smtpd

	It is typically the first uncommented line in the file.  Change the "smpt" to the desired port number:

		10025     inet  n       -       y       -       -       smtpd

	This line specifies when to run the smptd service -- i.e., when traffic is received on port 10025 from the internet (inet).

(7)  Have Postfix reload the edited configuration file(s):  "sudo postfix reload"

(8)  Test and use etc.  Check logfiles for errors (see below).


----------
Troubleshooting / FYIs:
	-The main Postfix configuration files are (usually?) located in /etc/postfix
	-The main Postfix logfile is /var/log/mail.log -- useful for detecting errors
	-This all assumes that WAN ports are open.  That is, the default port of 25
	 cannot be blocked by host firewalls or infrastructure routers.

N.b. -- Postfix is picky about the hostname spelled out using telnet.  For a host named "experiment.internet-class.org",
	the entire fqdn needs to be spelled out.  The gotcha is that telneting into "internet-class.org" will _appear_
	to work -- ehlo handshake and mail from: work.  BUT, rcpt to: will generate a relay access denied error,
	regardless of approved ip whitelists.

N.b. -- the Postfix installation, by default, restricts relay access to  (1)  authenticated users
	("permit_sasl_authenticated") and  (2)  the list of IP nets specified in the "mynetworks" -- typically,
	only the local host ("localhost").  See the smtpd_relay_restriction line in /etc/postfix/main.cf.

N.b. -- The Postfix documentation (http://www.postfix.org/DATABASE_README.html) references a new lookup table type,
	"inline", that became available with Postfix version 3.0.  This does not work well for our needs, as it has no
	support for wildcards (needed to blacklist all addresses not on the whitelist).  The "pcre" table type
	accomplishes what we need.

N.b. -- For futurework, a script can be set to be run upon receipt of every incoming email (to, say, edit the email
	and inject a magic password).  Nutshell version:  edit /etc/postfix/master.cf and edit the first smpt line
	to run a content filter.  Set the content filter to run the script.  This has been proven in basic concept.
	Google up "run script of smpt receipt" etc.


----------

**The above has been tested with Postfix 2.11 on Ubuntu 14.04 LTS and Postfix 3.11 on Ubuntu 16.04 LTS





postconf compatibility_level=2 (=0)
postconf reload
