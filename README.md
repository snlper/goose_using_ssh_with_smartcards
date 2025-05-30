# goose_using_ssh_with_smartcards
Спизженая статья

Using OpenSSH with smartcards
OpenSSH is the most popular connection system to remote computers. It is free software available under Mac OS X, GNU/Linux and even Windows. OpenSSH is probably used by hundred thousands of users.

In this HOWTO, you will connect from your computer (later called "client") to a remote computer (later called "server") using a smart card or a token, for complete security.

This tutorial covers:

+ How to install OpenSSH version 5.5p1 server and client under GNU/Linux and Mac OS X.
+ How to upgrade OpenSSH client under Mac OS X without breaking your box.
+ Secure shell access (ssh).
+ Secure Copy (scp).
+ Secure File Transfer Protocol (sftp), a replacement for scp.
+ OpenSSH ssh-agent usage.
+ Gnome Keyring Daemon, Gnome replacement for ssh-agent
In the tutorial, we print commands for GNU/Linux and Mac OS X, so can simply copy and paste witout modification.

Software prerequisites
Although patches were submitted years ago, smart card support in OpenSSH is very recent:

Only OpenSSH version 5.4p1 supports smartcard using PKCS#11 authentication. Furthermore, there is a bug in OpenSSH v5.4p1 release preventing ssh from reading pkcs#11 config in /etc/ssh/ssh_config. Although this is a minor bug, this proves to be tedious.

Therefore we recommand using OpenSSH 5.5p1.

To check OpenSSH version on your computer, running client:

$ ssh -v
OpenSSH_5.4p1, OpenSSL 0.9.8m 25 Feb 2010
In this example, we are running OpenSSH 5.4p1.
You should upgrade to OpenSSH 5.5p1.

In the tutorial, you will learn how to upgrade OpenSSH.

Understanding OpenSSH client and server logic
When using smartcards, you will connect from a client (usually your computer) to a server (a remote computer).

OpenSSH should be installed on the client and on the server:

Client is running SSH client, running from console using "ssh" syntax.
Server is running SSH server, usually started automatically on boot by a daemon process.
Of course, smartcard authentication is performed on client.

Installing OpenSSH on client
This section describes how to install OpenSSH client on your station.

Installing OpenSSH on server
This section describes how to install OpenSSH server on your station.

OpenSSH version on the server is not very important, although it is preferable to have a recent version.

Reading SSH public key on card (client side)
Connect the smart card reader and insert a smart card.
If you are using a token, connect the USB key.

In the next paragraphs, '*' indicate that text was shortened for readability.

Query the available RSA keys:

$ pkcs15-tool --list-public-keys
Using reader with a card: OmniKey CardMan 4321 00 00
Public RSA Key [Public Key]
Com. Flags : 2
Usage : [0x4], sign
Access Flags: [0x0]
ModLength : 2048
Key ref : 0
Native : no
Path : 3f0050153000
Auth ID : 01
ID : 7645d913d5b4e03f3fe5*****f02324c23a7ebf
In our example, the public key ID is 7645d913d5b4e03f3fe5*****f02324c23a7ebf4.

Sometimes, there is no public key, only a private key. Try:

$ pkcs15-tool --list-keys
Using reader with a card: Feitian ePass2003 00 00
Private RSA Key [Private Key]
Object Flags : [0x3], private, modifiable
Usage : [0x2E], decrypt, sign, signRecover, unwrap
Access Flags : [0x0]
ModLength : 2048
Key ref : 0 (0x0)
Native : yes
Path : 3f0050152900
Auth ID : 01
ID : 7645d913d5b4e03f3fe5*****f02324c23a7ebf
Now extract the RSA key in SSH format:

$ pkcs15-tool --read-ssh-key 7645d913d5b4e03f3fe5*****f02324c23a7ebf4
Using reader with a card: OmniKey CardMan 4321 00 00
Please enter PIN [User PIN]:
2048 65537 258115708996235*****134757454178319
ssh-rsa AAAAB3NzaC*****ed0aZdx9FFu/w6l7P5KsndWgP
Notice the RSA public key in SSH format:

ssh-rsa AAAB3NzaC*****ed0aZdx9FFu/w6l7P5KsndWgP

Installing public key on OpenSSH server
In this section we will copy your public RSA key to the ~/.ssh/authorized_keys file on server.

The .ssh notation denotes a hidden folder.
This folder should be inside the home of the user connecting.

Please notice that on the server, the .ssh folder may not exist.
In which case you will need to create it.

In our example, we check that the .ssh exist:

$ ls -lh /home/fperou/.ssh
authorized_keys
If the .ssh folder does not exist, create it using the user account:

$ mkdir ~/.ssh
$ touch ~/.ssh/authorized_keys
Copy the content of the public key (on smart card) to the ~/.ssh/authorized_keys file (on server). In our example:

ssh-rsa AAAAB3NzaC1yc2EAAAADAQ********9FFu/w6l7P5KsndWgP

Using ssh with smart cards
In our example, we log using ssh client, user 'fperou' on remote server 'remotehost':

 GNU/Linux:

$ ssh -I /usr/lib/opensc-pkcs11.so fperou@remotehost
Enter PIN for 'FRANCOIS PEROU (User PIN)':****
francois@remotehost:~$
To ease connection, you may add this line to /etc/ssh/ssh_config:

PKCS11Provider /usr/lib/opensc-pkcs11.so
 Mac OS X:

$ ssh -I /Library/OpenSC/lib/opensc-pkcs11.so fperou@remotehost
Enter PIN for 'FRANCOIS PEROU (User PIN)':****
francois@remotehost:~$
To ease connection, you may add this line to /etc/ssh_config file (Mac OS X 10.6 / 10.7) or /opt/local/etc/ssh/ssh_config (Mac Ports):

PKCS11Provider /Library/OpenSC/lib/opensc-pkcs11.so
Exit and run the same command using verbose output:

ssh -v francois@remotehost
OpenSSH_5.5p1, OpenSSL 0.9.8m 25 Feb 2010
debug1: Reading configuration data /usr/etc/ssh_config
debug1: Connecting to ****.*****.com [88.160.168.33] port 22.
debug1: Connection established.
debug1: manufacturerID OpenSC (www.opensc-project.org cryptokiVersion 2.20 libraryDescription libraryVersion 0.0
debug1: label manufacturerID \ model serial \<299851151317110> flags 0x40d
debug1: have 1 keys
debug1: Remote protocol version 2.0, remote software version OpenSSH_5.3p1 Debian-3
debug1: match: OpenSSH_5.3p1 Debian-3 pat OpenSSH*
debug1: Enabling compatibility mode for protocol 2.0
debug1: Local version string SSH-2.0-OpenSSH_5.4
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
debug1: kex: server->client aes128-ctr hmac-md5 none
debug1: kex: client->server aes128-ctr hmac-md5 none
debug1: SSH2_MSG_KEX_DH_GEX_REQUEST(1024<1024<8192) sent
debug1: expecting SSH2_MSG_KEX_DH_GEX_GROUP
debug1: SSH2_MSG_KEX_DH_GEX_INIT sent
debug1: expecting SSH2_MSG_KEX_DH_GEX_REPLY
debug1: Host '*************' is known and matches the RSA host key.
debug1: Found key in /home/******/.ssh/known_hosts:1
debug1: ssh_rsa_verify: signature correct
debug1: SSH2_MSG_NEWKEYS sent
debug1: expecting SSH2_MSG_NEWKEYS
debug1: SSH2_MSG_NEWKEYS received
debug1: Roaming not allowed by server
debug1: SSH2_MSG_SERVICE_REQUEST sent
debug1: SSH2_MSG_SERVICE_ACCEPT received
debug1: Authentications that can continue: publickey
debug1: Next authentication method: publickey
debug1: Offering public key: /usr/lib/opensc-pkcs11.so
debug1: Server accepts key: pkalg ssh-rsa blen 279
Enter PIN for 'François Pérou' (User PIN)':
debug1: pkcs11_provider_unref: 0x153fb90 refcount 2
debug1: Authentication succeeded (publickey).
debug1: channel 0: new [client-session]
debug1: Requesting no-more-sessions@openssh.com
debug1: Entering interactive session.
Linux firewall 2.6.32-trunk-486 #1 Sun Jan 10 05:53:18 UTC 2010 i686
The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Apr 1 16:57:20 2010 from xxxxxxxxxxxx

Make sure that OpenSSH is asking for PIN and not using local keys in ~/.ssh on the client side.

Using scp with smartcards
scp allows to specify any OpenSSH syntax using -o switch.

 GNU/Linux:
To use smart cards, add this switch to your scp command line:

-o PKCS11Provider=/usr/lib/opensc-pkcs11.so
Therefore a common file transfer using scp would be:

$ scp -v -o PKCS11Provider=/usr/lib/opensc-pkcs11.so filename user@host:path
To ease connection, you may add this line to /etc/ssh/ssh_config:

PKCS11Provider /usr/lib/opensc-pkcs11.so
 Mac OS X:
To use smart cards, add this switch to your scp command line:

-o PKCS11Provider=/Library/OpenSC/lib/opensc-pkcs11.so
Therefore a common file transfer using scp would be:

$ scp -v -o PKCS11Provider=//Library/OpenSC/lib/opensc-pkcs11.so filename user@host:path
To ease connection, you may add this line to /opt/local/etc/ssh/ssh_config:

PKCS11Provider /Library/OpenSC/lib/opensc-pkcs11.so
It is possible to enter several filenames or wildcards to avoid sending multiple commands:

$ scp -v filename1 filename2 filename-other* user@host:path

Using ssh authentication agent ssh-add with smartcards
Using ssh-agent allows to use smartcards easily, as you just enter your PIN code once in a session.

Adding keys from PKCS#11 provider
If you are running OpenSSH in a shell environment, to load keys, type:

 GNU/Linux:

$ ssh-add -s /usr/lib/opensc-pkcs11.so
Enter passphrase for PKCS#11:
 Mac OS X:

$ ssh-add -s /Library/OpenSC/lib/opensc-pkcs11.so
Enter passphrase for PKCS#11:
Enter PIN code to authenticate.

Now verify that keys have been loaded:

$ ssh-add -l
2048 75:9e:dd:32:aa:*************:fb:57:1f:ad:2e /usr/lib/opensc-pkcs11.so (RSA)
2048 41:16:d5:c0:37:*************:75:d6:f1:81:dc /usr/lib/opensc-pkcs11.so (RSA)
You will be able to use SSH, SCP, SFTP without entering PIN code again.

Now you may also comment this line, which becomes useless:

# PKCS11Provider /usr/lib/opensc-pkcs11.so
as ssh-agent will load RSA keys from smartcards.

Removing keys provided by PKCS#11 provider
Using the usual command does not work:

$ ssh-add -D
This will remove all identities, but the smartcard system will be left in a unusable state.

Instead, you should run:
 GNU/Linux:

$ ssh-add -e /usr/lib/opensc-pkcs11.so
 Mac OS X:

$ ssh-add -e /Library/OpenSC/lib/opensc-pkcs11.so


Using Gnome-keyring with smartcards in Gnome
Gnome includes a advanced password and key manager called Gnome-keyring, which acts as a replacement for ssh-agent.

To use smartcards without problem, you will need at least Gnome 2.6.30 and Gnome-keyring-daemon 2.6.30. Our tests show that Gnome 2.6.28 keyring-manager is not able to load keys from PKCS#11 smartcards.

After starting Gnome 2.6.30, run gconf-editor to enable PKCS11 and ssh agent:

Type gconf-edit and open /apps/gnome-keyring/daemon-components

$ gconf-editor


Make sure that pkcs11 and ssh are enabled.
In our tests, we found that Gnome 2.6.30 needed some additional information on startup.

Exit Gconfig and return to desktop.
In the main menu bar, select System->Preferences->Startup Applications.

Startup applications preferences dialog is displayed:


Although Gnome-Keyring-Daemon is running on startup, you need to inform the daemon to load pkcs#11 and ssh extensions.

Find the Certificate and Key storage icon. Make sure it is enabled:

If you click on Edit, the command should be:

gnome-keyring-daemon --start --components=pkcs11
Find the Gnome SSH agent icon. Make sure it is enabled:

If you click on Edit, the command should be:

gnome-keyring-daemon --start --components=ssh
Now load your public SSH keys from your smartcard:

$ ssh-add -s /usr/lib/opensc-pkcs11.so
On prompt, enter PIN code:

Enter passphrase for PKCS#11: ******
Card added: /usr/lib/opensc-pkcs11.so
You can now list public keys loaded by Gnome-Keyring and ssh-agent:

$ ssh-add -L
ssh-rsa AAAAB3NzaC1yc2EAAAADAQA**********XRVVUYDKsndWgP /usr/lib/opensc-pkcs11.so
ssh-rsa AAAAB3NzaC1yc2EAAAADA*********R9EQ7MeKHsfot4xotz6YqE/RPve+1dAvTl /usr/lib/opensc-pkcs11.so
You can now use your smartcard in Gnome.

We did not test sftp attachment in Nautilus 2.6.30, but it should work smoothly with RSA keys on smartcard.

Why use OpenSSH with smart cards?
SSH has the usual leaks of computer security:

Passwords may be lost or stolen.
RSA keys: if the computer is lost or hacked, the secret keys may be compromised.
In a professional environment, you may log from several computers to several servers. You may need to copy the secret keys to several computers, which may be a security issue.
Smart cards solve these issues, providing a very single sign-on (SSO) solution. Smart cards are able to store the RSA private key without displaying it, which means that the key cannot leave the smart card. The public key can be read from smart card after login in using a PIN code.



Smartcard Prerequisites
As a prerequisite, you should read our smart card quickstarter guide, in order to learn how to install and configure smartcards.

Hereafter, we consider that you installed a smart card reader and configured a smart card either with a self-signed certificate or your existing RSA key in OpenSSH format like explained in the guide.

Dump the content of your smartcard to make sure your RSA certificates are installed:

$ pkcs15-tool --dump
PKCS#15 Card [François Pérou]:
Version : 1
Serial number : 2851294610040810
Manufacturer ID: EnterSafe
Last update : 20100919114626Z
Flags : EID compliant
PIN [User PIN]
Com. Flags: 0x3
ID : 01
Flags : [0x32], local, initialized, needs-padding
Length : min_len:4, max_len:16, stored_len:16
Pad char : 0x00
Reference : 1
Type : ascii-numeric
Path : 3f005015

Private RSA Key [Private Key]
Com. Flags : 3
Usage : [0x4], sign
Access Flags: [0x0]
ModLength : 2048
Key ref : 1
Native : yes
Path : 3f005015
Auth ID : 01
ID : f7af721c8db60f82d726930ccf7d253e73aa45c6

Public RSA Key [Public Key]
Com. Flags : 2
Usage : [0x4], sign
Access Flags: [0x0]
ModLength : 2048
Key ref : 0
Native : no
Path : 3f0050153000
Auth ID :
ID : f7af721c8db60f82d726930ccf7d253e73aa45c6

