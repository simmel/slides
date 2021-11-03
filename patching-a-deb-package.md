layout: true
.footer[2021-11-02 / Simon Lundström, IT-avdelningen]

---

class: first-slide
# Patching a Debian package [for fun^Wsanity and profit](https://english.stackexchange.com/questions/25205/what-is-the-origin-of-phrase-for-fun-and-profit)

1. Experience a bug
2. Cry
3. ???
4. Profit

---

# Agenda

1. The problem
1. The workaround
1. The bug
1. The solution

---
# The problem

```terminal
meow:~$ kgetcred smtp/smtp.su.se
kgetcred: krb5_get_creds: Matching credential (smtp/smtp.su.se@SU.SE) not found
```
???
Skulle skicka mail och helt plötsligt så kunde jag inte autentisera mot SMTP-servern
Varför säger den att smtp principalen inte finns? Det lär den ju göra annars kan ingen maila.
--
```terminal
kadmin> get -s smtp/smtp.su.se
Principal        Expiration  PW-exp  PW-change   Max life  Max renew
smtp/smtp.su.se  never       never   2014-04-09  10 hours  1 day
kadmin>
```
???
Dubbelkolla i KDC:n, i fall att.

---
<style>.huge { font-size: 250pt; color: white; margin-top: -10pt;</style>
# The problem
#### What does the Internet say about this?
https://duckduckgo.com/?q=heimdal+%22Matching+credential%22+%22not+found%22

???
Vad säger Internet om felmeddelandet?
--
background-image: url(https://www.absolut.com/globalassets/campaign-page/absolut-nothing/screenshot-2020-12-17-at-11.53.52.png)

--
.center.huge[…]
???
Klassiker...
---
<style>
.big-hand img { height: 100px; width: 100px; }
.big-hand { font-size: 70pt; }
</style>
# The problem
#### Call a friend

.big-hand[🤙![Patrik Lundin](https://avatars.githubusercontent.com/u/306237?v=4)]

![Call a friend](https://buzzerblog.com/wp-content/uploads/2014/06/Screen-Shot-2014-06-03-at-9.37.02-AM.png)

???
Dax att rubberducka lite i Vem vill bli miljonär-anda
---
# The problem

```terminal
meow:~$ kgetcred smtp/smtp.su.se
kgetcred: krb5_get_creds: Matching credential (smtp/smtp.su.se@SU.SE) not found
meow:~$ klist -v | head
Credentials cache: KCM:1000:1
        Principal: simlu@SU.SE
    Cache version: 0
  KDC time offset: 56 years 3 weeks 1 day 21 hours 42 minutes 57 seconds
meow:~$
```
???
Samla ihop information och skicka till patlu
---
# The problem

```terminal
meow:~$ kgetcred smtp/smtp.su.se
kgetcred: krb5_get_creds: Matching credential (smtp/smtp.su.se@SU.SE) not found
meow:~$ klist -v | head
*Credentials cache: KCM:1000:1
        Principal: simlu@SU.SE
    Cache version: 0
  KDC time offset: 56 years 3 weeks 1 day 21 hours 42 minutes 57 seconds
meow:~$
```
???
Den observante ser kanske att jag inte kör FILE som CC

---
<style>
.lolpatlu{ float: left;}
.lol-kcm {
height: 120px; width: 150px; background-repeat: no-repeat; background-size: cover; text-align: center; float: left;
margin-top: -70pt;
}
.gurka {
margin-top: 50pt;
float: right;
}
</style>
# The problem
<div class="gurka">
.big-hand.lolpatlu[![Patrik Lundin](https://avatars.githubusercontent.com/u/306237?v=4)]
<div class="lol-kcm" style="background-image: url(http://www.pngall.com/wp-content/uploads/2016/07/Speech-Bubble-Free-Download-PNG.png);"><br/>lol KCM</div>
</div>
```terminal
meow:~$ cat /etc/krb5.conf
[libdefaults]
# MIT
default_ccache_name = KCM:
# Heimdal
default_cc_type = KCM:%{uid}
meow:~$
```
???
KCM sparar CC i en daemons minne istället för i filer.
Används i OS X man kallas där "API".

---
<style>
.nohilight .remark-code-line-highlighted { background-color: revert; }
</style>
# The problem
#### Why KCM?
* No need to manage different CC files, just `kinit $USER && kinit $USER/root`
* `kswitch` between them, e.g. `kswitch -p $USER/root`.

.nohilight[
```terminal
meow:~$ klist -l
    Name               Cache name     Expires
  simlu/root@SU.SE   KCM:1000:3     Nov  3 17:58:20 2021
** simlu@SU.SE        KCM:1000:1     Nov  3 17:58:13 2021
meow:~$ kswitch -p $USER/root
meow:~$ klist
Credentials cache: KCM:1000:3
        Principal: simlu/root@SU.SE
[...]
meow:~$
```
]

---
# The problem

```terminal
meow:~$ kgetcred smtp/smtp.su.se
kgetcred: krb5_get_creds: Matching credential (smtp/smtp.su.se@SU.SE) not found
meow:~$ klist -v | head
Credentials cache: KCM:1000:1
        Principal: simlu@SU.SE
    Cache version: 0
  KDC time offset: 56 years 3 weeks 1 day 21 hours 42 minutes 57 seconds
meow:~$
```
???
patlu pekar ut något konstigt som jag missat

---
# The problem

```terminal
meow:~$ kgetcred smtp/smtp.su.se
kgetcred: krb5_get_creds: Matching credential (smtp/smtp.su.se@SU.SE) not found
meow:~$ klist -v | head
Credentials cache: KCM:1000:1
        Principal: simlu@SU.SE
    Cache version: 0
* KDC time offset: 56 years 3 weeks 1 day 21 hours 42 minutes 57 seconds
meow:~$
```

---
# The problem
```terminal
meow:~$ man krb5.conf
[...]
                      kdc_timesync = boolean
                           Try to keep track of the time differential between the local machine
                           and the KDC, and then compensate for that when issuing requests.
```
???
Sidospår som inte var rätt

patlu: Men köra en FILE credential cache funkar väl?

---
# The workaround
```terminal
meow:~$ kgetcred -c FILE:/tmp/gurka123 smtp/smtp.su.se
meow:~$ klist -c FILE:/tmp/gurka123 -v
Credentials cache: FILE:/tmp/gurka123
        Principal: simlu@SU.SE
    Cache version: 4

[...]

Server: smtp/smtp.su.se@SU.SE
Client: simlu@SU.SE
Ticket etype: des3-cbc-sha1, kvno 2
Ticket length: 321
Auth time:  Nov  1 15:18:50 2021
Start time: Nov  1 15:19:43 2021
End time:   Nov  2 01:18:50 2021
Ticket flags: transited-policy-checked, pre-authent, forwardable
Addresses: addressless

meow:~$
```

---
# The bug

![No one else has ever reported that problem](https://assets.amuniversal.com/d8afc5906cc901301d50001dd8b71c47)

https://github.com/heimdal/heimdal/issues?q=kcm+offset

---
# The solution

https://duckduckgo.com/?q=debian+rebuild+package

---
# The solution
```terminal
meow:~$ sudo vi /etc/apt/sources.list
[...]
deb http://se.archive.ubuntu.com/ubuntu/ focal main restricted
deb-src http://se.archive.ubuntu.com/ubuntu/ focal main restricted
[...]
meow:~$ sudo apt update
```
???
Se till att vi kan hämta källkoden och paketet

---
# The solution
```terminal
meow:~$ sudo apt-get build-dep heimdal
meow:~$ mkdir src/heimdal-rebuild && cd $_
meow:~$ apt-get source heimdal
```
???
Installera build deps och hämta källkoden och paketet

---
# The solution
```terminal
meow:~$ cd heimdal-7.7.0+dfsg
meow:~$ curl -sL https://github.com/heimdal/heimdal/pull/390.patch | patch -p1
meow:~$ dpkg-source --commit
meow:~$ dpkg-buildpackage -us -uc -rfakeroot
```
???
Dra in patchen, "commita" och bygga paketet

---
# The solution
```terminal
meow:~$ cd ..
meow:~$ sudo dpkg -i heimdal-kcm_7.7.0+dfsg-1ubuntu1_amd64.deb
meow:~$ sudo systemctl restart heimdal-kcm.service
```

---

```terminal
meow:~$ klist -v
Credentials cache: KCM:1000:1
        Principal: simlu@SU.SE
    Cache version: 0

Server: krbtgt/SU.SE@SU.SE
Client: simlu@SU.SE
Ticket etype: aes256-cts-hmac-sha1-96, kvno 2
Ticket length: 301
Auth time:  Nov  2 11:59:41 2021
End time:   Nov  2 21:59:41 2021
Ticket flags: enc-pa-rep, pre-authent, initial, forwardable
Addresses: addressless

meow:~$ kswitch -p $USER/root
meow:~$ klist -v | head
Credentials cache: KCM:1000:3
        Principal: simlu/root@SU.SE
    Cache version: 0

Server: krbtgt/SU.SE@SU.SE
Client: simlu/root@SU.SE
Ticket etype: aes256-cts-hmac-sha1-96, kvno 2
Ticket length: 307
Auth time:  Nov  2 11:59:49 2021
End time:   Nov  2 21:59:49 2021
meow:~$
```

---
# More questions?
--

Slides at https://github.com/simmel/slides via https://github.com/stockholmuniversity/su-remark-template/
