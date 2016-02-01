
# McAfee privileged SiteList.xml leads to Active Directory domain privilege escalation

During an intern pentest, I came accross a nice way to privesc in an Active Directory domain.
I owned an employee's laptop with **McAfee Virusscan Enterprise 8.8** installed and a low privilege account.

Mcafee has a feature to customize update servers and can connect to these servers via HTTP or SMB.
The file **SiteList.xml** contains juicy informations like credentials, domain name servers, ... it looks like this :

###### C:\ProgramData\McAfee\Common Framework\SiteList.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<ns:SiteLists xmlns:ns="naSiteList" Type="Client">
<SiteList Default="1" Name="SomeGUID">
...
<HttpSite Type="fallback" Name="McAfeeHttp" Order="26" Enabled="1" Local="0" Server="update.nai.com:80">
<RelativePath>Products/CommonUpdater</RelativePath><UseAuth>0</UseAuth>
<UserName></UserName>
<Password Encrypted="1">XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX</Password>
</HttpSite>
...
<UNCSite Type="repository" Name="Paris" Order="13" Server="paris001" Enabled="1" Local="0">
<ShareName>Repository$</ShareName><RelativePath></RelativePath><UseLoggedonUserAccount>0</UseLoggedonUserAccount>
<DomainName>companydomain</DomainName>
<UserName>McAfeeService</UserName>
<Password Encrypted="1">YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY</Password>
</UNCSite>
...
<UNCSite Type="repository" Name="Tokyo" Order="18" Server="tokyo000" Enabled="1" Local="0">
<ShareName>Repository$</ShareName><RelativePath></RelativePath><UseLoggedonUserAccount>0</UseLoggedonUserAccount>
<DomainName>companydomain</DomainName>
<UserName>McAfeeService</UserName>
<Password Encrypted="1">YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY</Password>
</UNCSite>
...
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

Ty !

[@tfairane](https://twitter.com/tfairane)
