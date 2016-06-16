#### Hack 100.000$ with one line Bash ~ freaky PayPal API (educational purpose only)

## Disclaimer

First of all, this paper aims to alarm ppl about security issues concerning **data leakage**.

**IT COULD HAVE FINANCIAL IMPACT && COST TO YOUR LIFE/COMPANY!!!**

The fact is, I'm clearly a white hat but to make computer security move, I feel compelled to publicly disclose.
Also, I would like to call software developers : "Guys ! Github is not your f*** dirty room" don't push sensitive data -- or bad code. This writeup is not about technical but education, about fixing mentality on how security concern is perceived.

Conclusion, **please DO NOT ROB innocent ppl** (imagine It could be your *grandma*).

- - -

#### Give me your token, I will tell you who you are

[**OAuth**](https://tools.ietf.org/html/rfc6749) is an authorization framework widely used on social media.
it's mainly used to authenticate communication between applications.
For example, your Twitter account can be managed by your Spotify
application to share your top musics, etc.

Between them you find a token : a unique string able to authorize tasks in a scope.
This token is actually as important as your couple of login/password -- or family jewels.
I admit that stock it and prevent it from being compromised is a hard responsibility.

<p align="center">
<img height="300px" width="300px" src="https://upload.wikimedia.org/wikipedia/commons/b/b7/Unico_Anello.png"/>
</p>

#### PayPal NVP

In the same idea as OAuth tokens, we will focus on other kind of API authentication. PayPal offer an [**Name-Value Pair** (NVP)](https://developer.paypal.com/docs/classic/api/NVPAPIOverview/) API to payout.
NVP is basically credentials to authenticate you during API operation - as OAuth.
Noticed that PayPal provided a sandbox for development test purpose.

PayPal print this warning :

```
Important: You must protect the values for USER, PWD, and SIGNATURE in your
implementation. Consider storing these values in a secure location other than
your web server document root and setting the file permissions so that only the
system user that executes your ecommerce application can access it.
```

One method named [**MassPay**](https://developer.paypal.com/docs/classic/mass-pay/gs_MassPay/) allows ecommerce application to pay people giving a valid PayPal email address.

```
curl https://api-3t.sandbox.paypal.com/nvp \
  -s \
  --insecure \
  -d USER=AccountUsername         # Caller UserName \
  -d PWD=AccountPassword          # Caller Pswd \
  -d SIGNATURE=AccountSignature   # Caller Sig \
  -d METHOD=MassPay \             # MassPay Method
  -d VERSION=90 \
  -d RECEIVERTYPE=EmailAddress \
  -d CURRENCYCODE=USD \
  -d L_EMAIL0=foo@bar.com           # The first payout starts with "0" \
  -d L_AMT0=13.37                   # Profit ...
```

Another method give us the balance of an account with a method named [**GetBalance**](https://developer.paypal.com/docs/classic/api/merchant/GetBalance_API_Operation_NVP/).

```
curl https://api-3t.sandbox.paypal.com/nvp \
  -s \
  --insecure \
  -d USER=AccountUsername         # Caller UserName \
  -d PWD=AccountPassword          # Caller Pswd \
  -d SIGNATURE=AccountSignature   # Caller Sig \
  -d METHOD=GetBalance \          # GetBalance Method
  -d VERSION=90 \
  -d RETURNALLCURRENCIES=1
```

#### Collect credsxxx

To collect the raw material we will need the [Github vanilla search bar](https://github.com/search) or [GitMiner](https://github.com/danilovazb/GitMiner).

By profiling NVP credentials, we can notice that all signatures have a predictable pattern.

With some keywords we can find plenty of candies.

* [```api-3t.paypal.com``` ```api1```](https://github.com/search?utf8=✓&q=api-3t.paypal.com+api1&type=Code&ref=searchresults) ```We’ve found 4,800 code results```

* [```api-3t.paypal.com``` ```api1``` ```live```](https://github.com/search?utf8=✓&q=api-3t.paypal.com+api1+live&type=Code&ref=searchresults) ```We’ve found 3,300 code results```

* [```api-3t.paypal.com``` ```api1``` ```live``` ```masspay```](https://github.com/search?utf8=✓&q=api-3t.paypal.com+api1+live+masspay&type=Code&ref=searchresults) ```We’ve found 42 code results```

#### Heat

*```"Get on your knees! Get on your knees! We want to hurt no one! We're here for the
bank's money, not your money. Your money is insured by the federal government,
you're not gonna lose a dime! Think of your families, don't risk your life.
Don't try and be a hero!" Neil McCauley```*

So, let's feed China's cyber espionage units with bunch of token and one shitty line Bash
*(actually I just summed the total of each account)*.

```
~ ./Summing.sh creds.csv
Total: 101513,24 USD /O_O\
```

#### "Seriously, how can I prevent from data leakage ?"

* Do not disclose sensitive information on source code management system (*Github*) and/or social media (*Stackoverflow*).
* Work with private repository and/or custom Git.
* When you push configuration file : **DOUBLE CHECK if any sensitive data is in**.

#### If you pushed code with sensitive data

* **CONSIDER YOURSELF AS COMPROMISE!!!**.
* Revoke immediately your token.
* ***"I guess you could call it a "failure", but I prefer the term "learning experience"." — Andy Weir***

### PayPal POV

```
Hello,
 
Thank you for participating in our PayPal Bug Bounty Program.
Bugs are classified according to the data impacted, ease of exploit and overall risk to PayPal customers and the PayPal brand. Your submission was found to be invalid and not actionable due to the following:

Paypal has no control over our users mishandling their personal information. Merchants/Customers sign user agreements to not expose their tokens/paswords in any way and are responsible for any losses that result.
 
We take pride in keeping PayPal the safer place for online payment.
 
Thanks,
PayPal Bug Bounty Team
```

<p align="center">
<img height="600px" width="450px" src="http://i0.kym-cdn.com/photos/images/newsfeed/000/234/719/c7c.jpg"/>
</p>
