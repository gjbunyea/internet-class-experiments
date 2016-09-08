
cse199 -- In-class Protocol Activity -- Manual Generation of Emails -- 2016-09-07


----------
Goal: to have a locally installed SMTP server that:
	(0)  unless otherwise restricted, furnishes typical SMTP service to clients
	(1)  allows Telnet access to generate an outgoing email (i.e. a relay service)
	(2)  restricts relay service (i.e., Telnet access) to be available only to computers on a particular subnet (i.e., U.B.)
	(3)  restricts outgoing email to a specific destination address


----------
Step-by-step:

(1)  Install Postfix:  "sudo apt-get install postfix"

(2)  Perform a default (re)configuration of Postfix:  "sudo dpkg-reconfigure postfix".  Select
	"Internet Site" from the menu, and accept all default options.

(3)  Install Postfix library support for Perl regexes:  "sudo apt-get install postfix-pcre"
	This will be necessary to support outgoing whitelisting using the method described below.

(4)  Edit the /etc/postfix/main.cf file to Enable relay (Telnet) access for nodes in desired IP nets.  For example,
	to enable computers in the 128.205.36.X (timberlake is 128.205.36.8) subnet to be able to relay email, 
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

(6)  Have Postfix reload the edited configuration file(s):  "sudo postfix reload"

(7)  Test and use etc.  Check logfiles for errors (see below).


----------
Troubleshooting / FYIs:
	-The main Postfix configuration files are (usually?) located in /etc/postfix
	-The main Postfix logfile is /var/log/mail.log -- useful for detecting errors
	-This all assumes that WAN ports are open.  That is, 

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

**This has been tested with Postfix 2.11 on Ubuntu 14.04 LTS and Postfix 3.11 on Ubuntu 16.04 LTS


