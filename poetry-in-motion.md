class: center, middle
layout: true
.footer[2020-11-12 / Simon Lundström, IT-avdelningen]

---

class: first-slide
# Poetry [in motion](https://www.youtube.com/results?search_query=poetry+in+motion)

#### Python packaging and dependency management made easy

---

# Agenda

1. Poetry!?
1. Why not use something else?
1. Create a Python library
1. Create a Python application

---

# Poetry!?

* [Python packaging and dependency management (made easy!)](https://python-poetry.org/)

--
* Think Maven, Gradle, Rubygems+Bundler but for Python 🐍

--
* 🧙‍♂️ Create/manage/update your virtualenv ✨automagically✨ 🧙‍♀️
<br/>https://python-poetry.org/docs/managing-environments/

--
* Uses a *standard* .~~ini~~toml file for configuration
<br/>https://python-poetry.org/docs/pyproject/

???
Makes it easy to parse, not needed to run any code to figure out things about
the project.

--
* [pyproject.toml PEP 518 standard](https://www.python.org/dev/peps/pep-0518/)
  * To quote [Black](https://black.readthedocs.io/en/stable/pyproject_toml.html#what-on-earth-is-a-pyproject-toml-file):
  > […]it can fully replace the need for setup.py and setup.cfg files.

---
# Why not Zoidberg 🦞?
---
# Why not ~~Zoidberg 🦞~~ …?

--

⚠️ These are of course my opinions and they tend to be wrong and/or outdated. ⚠️

???
There are of course alternatives
---
# Why not ~~Zoidberg 🦞~~ …?
##  setup.py
##  Pipenv
---
# Why not ~~Zoidberg 🦞~~ …?
##  setup.py

--
* Flexible with a Python-file! Very dynamic configuration possible!

--
* 👆 ^ That's also negative ^ 👆

--
* `requirements.txt` has two flaws:
  * Either you specify exact versions: `su-logging==0.2.1`

???
And then you don't get updates

--
  * Or you specify lose versions: `su-logging>=0.2`

???
* And get breakage
* unspecified behaviour
* your version in your dev env differs from what's deployed in prod

--
* `0.2.1 != 0.2.1`

???
Version tag can be overwritten. Same tag, new content. Hello malware!

PyPI doesn't allow this, but Git does. And libs can be installed straight from git.

https://mail.python.org/pipermail/distutils-sig/2015-January/025683.html

---
# Why not ~~Zoidberg 🦞~~ …?
##  setup.py
```python
import sys
from setuptools import setup, find_packages

version = "1.0.0"

setup(
    name="mixedmartialtail",
    version=version,
    description="a warlike approach to tailing logs with mixed formats",
    long_description=open("README.md").read(),
    author="Simon Lundström",
    author_email="simmel@soy.se",
    url="https://github.com/simmel/mixedmartialtail",
    license="ISC license",
    […]
```

???
Describe our package, read description shown on PyPI from README.md

---
# Why not ~~Zoidberg 🦞~~ …?
##  setup.py
```python
    entry_points={
        'mixedmartialtail.plugins.input': [
            'cat = mixedmartialtail.plugins.input.cat',
            'json = mixedmartialtail.plugins.input.json',
        ],
        'console_scripts': [
            'mmt=mixedmartialtail:main'
        ]
    },
```

???
Add entry points for:
* Configure plugins that our module and supports.
* Console scripts/binaries that our module creates/supplies when installed

---
# Why not ~~Zoidberg 🦞~~ …?
##  setup.py
```python
    install_requires=open('requirements.txt').readlines(),
```

???
Externalize requirements so they can be used with `pip install -r requirements.txt`

--
```python
    extras_require={
        'dev': [
            'check-manifest==0.35',
            'twine==1.9.1',
        ],
        'test': [
            'check-manifest==0.35',
            'pytest==3.0.7',
            'pytest-benchmark==3.0.0',
            'mock==2.0.0',
            'tox==2.6.0',
        ],
    },
```

--
```python
    packages=find_packages(exclude=['contrib', 'docs', 'tests']),
)
```

???
Find what we want to include in our Pyhon package

---
# Why not ~~Zoidberg 🦞~~ …?
##  setup.py
* Most are 50-100 lines

--
* Some are… longer…
```terminal
$ git clone --depth=1 git@github.com:numpy/numpy.git
[…]
$ cd numpy
$ find . -iname setup.py | wc -l
      19
$ find . -iname setup.py | xargs cat | wc -l
    2137
$
```

---
# Why not ~~Zoidberg 🦞~~ …?
##  setup.py
### setup.cfg

???
Not everything can be configured in setup.py, setup.cfg is needed too.

--

```ini
[check-manifest]
ignore =
    Makefile
    *.pl
    tests/*.conf
    logs
```

???
Configure check-manifest to ignore some files
check-manifest makes sure we don't publish invalid packages to PyPI.

--
```ini
[bdist_wheel]
universal = 1
```
`.whl` =~ `.deb`.

???
Publish universal Python (2&3) pure python wheels.

---
# Why not ~~Zoidberg 🦞~~ …?
##  setup.py
Obscure commands from different tools
```terminal
# Setup project for development
$ python3 -mvenv venv
$ . venv/bin/activate
$ pip3 install -e '.[dev]'
$ ./setup.py develop --user
```

---
# Why not ~~Zoidberg 🦞~~ …?
##  setup.py
Obscure commands from different tools
```terminal
# Build and deploy
$ $EDITOR setup.py # Bump version
$ pytest # Hopefully run your tests?
$ git tag -s -m $VERSION $VERSION
$ git push --tags
$ ./setup.py sdist bdist_wheel
$ twine --sign upload dist/*
```

---
# Why not ~~Zoidberg 🦞~~ …?
## Pipenv
* [Can't be used for libraries](https://pipenv.pypa.io/en/latest/advanced/#pipfile-vs-setuppy)

--
* Doesn't handle uploading to PyPI

--
* Is super slow!

???
Breaks my development flow. So slow my mind starts to wander to other things
while waiting for it...

--
* [I've gotten VCS conflicts with Pipfile.lock](https://pipenv.pypa.io/en/latest/basics/#general-recommendations-version-control)

???
So if we use multiple versions of Python Pipfile.lock shouldn't be used?
Doesn't that defeat the purpose?

--
* https://chriswarrick.com/blog/2018/07/17/pipenv-promises-a-lot-delivers-very-little/
* https://pythonspeed.com/articles/pipenv-docker/
* A lot of people have optinions https://duckduckgo.com/?q=pipenv+poetry

---

# Create a Python library
* Publicera till vår Nexus https://confluence.it.su.se/confluence/display/sysdoc/Sonatype+Nexus+-+Add+PyPi+registry

---

# Create an Python application
3. Skapa en app som använder liben
* Publicera till vår Nexus
* Installera appen via pipx

---


```terminal
root@syslog-prod-archive01:~# grep -v '^#' /etc/cron.d/sua-syslog-archive
*0 6 * * *    root    /local/scriptbox/bin/surveillance collect-logfiles /local/syslog/scripts/collect-logfiles.sh
```


```terminal
root@syslog-prod-srv01:~# grep -vE '^#|^$' /etc/cron.d/sua-syslog-server
15 0 * * * root /local/scriptbox/bin/surveillance logrotate /local/rsyslog/scripts/logrotate
 * * * * * root /local/scriptbox/bin/surveillance ucarp-status /usr/bin/pkill -USR1 ucarp
```

```terminal
root@syslog-prod-srv01:~# grep -iE 'start|end' /local/surveillance/log/logrotate/2019/04/11/20190411T001501
Surveillance started at Thu 11 Apr 00:15:01 CEST 2019.
Starting command at Thu 11 Apr 00:15:01 CEST 2019.
*Command ended at Thu 11 Apr 07:04:15 CEST 2019.
```

---

# Anledning

--

Komprimeringen av loggarna tar för lång tid. Varför?
--

(firewall-prod-faz*, vi vet.. ; )

---

```terminal
root@syslog-prod-archive01:~# xzgrep 'logrotate:.*Command ended' /archive/syslog-prod-srv01.it.su.se/2018/syslog-prod-srv01.it.su.se/syslog-prod-srv01.it.su.se-2018-04-11.log.xz
2018-04-11T08:43:20.186415+02:00 syslog-prod-srv01.it.su.se surveillance: logrotate: Command ended at Wed 11 Apr 08:43:20 CEST 2018.
```
Verkar ha varit så ett tag...

--

```terminal
root@syslog-prod-archive01:~# xzgrep 'logrotate:.*Command ended' /archive/syslog-prod-srv01.it.su.se/2017/syslog-prod-srv01.it.su.se/syslog-prod-srv01.it.su.se-2017-04-11.log.xz
2017-04-11T03:37:48.848598+02:00 syslog-prod-srv01.it.su.se surveillance: logrotate: Command ended at Tue 11 Apr 03:37:48 CEST 2017.
```

Men inte alltid!

---

###### Nyss
```terminal
root@syslog-prod-archive01:~# find /archive/syslog-prod-srv01.it.su.se/2019/ -iname "*2019-04-11*" | wc -l
1032
root@syslog-prod-archive01:~# find /archive/syslog-prod-srv01.it.su.se/2019/ -iname "*2019-04-11*" | xargs du -sch | tail -n1
3.6G    total
```

--

###### För ett år sedan
```terminal
root@syslog-prod-archive01:~# find /archive/syslog-prod-srv01.it.su.se/2018/ -iname "*2018-04-11*" | wc -l
986
root@syslog-prod-archive01:~# find /archive/syslog-prod-srv01.it.su.se/2018/ -iname "*2018-04-11*" | xargs du -sch | tail -n1
4.5G    total
```

--

###### För två år sedan
```terminal
root@syslog-prod-archive01:~# find /archive/syslog-prod-srv01.it.su.se/2017/ -iname "*2017-04-11*" | wc -l
1016
root@syslog-prod-archive01:~# find /archive/syslog-prod-srv01.it.su.se/2017/ -iname "*2017-04-11*" | xargs du -sch | tail -n1
1.3G    total
```

---

# Slutsats

--
* Antal filer spelar ingen roll (i det här fallet)

--
* Mängden data har mer än tredubblats på 2 år

---

# Vad har ökat/ändrats i storlek då?

---

## 2017

```terminal
root@syslog-prod-archive01:~# find /archive/syslog-prod-srv01.it.su.se/2017/ -iname "*2017-04-11*" | xargs du -sh | sort -h | tail
27M     /archive/syslog-prod-srv01.it.su.se/2017/shib-prod-idp04.it.su.se/shib-prod-idp04.it.su.se-2017-04-11.log.xz
29M     /archive/syslog-prod-srv01.it.su.se/2017/sukat-prod-ldapro02.it.su.se/sukat-prod-ldapro02.it.su.se-2017-04-11.log.xz
33M     /archive/syslog-prod-srv01.it.su.se/2017/sukat-prod-ldapro04.it.su.se/sukat-prod-ldapro04.it.su.se-2017-04-11.log.xz
34M     /archive/syslog-prod-srv01.it.su.se/2017/sukat-prod-ldapro05.it.su.se/sukat-prod-ldapro05.it.su.se-2017-04-11.log.xz
38M     /archive/syslog-prod-srv01.it.su.se/2017/wism4-1.su.se/wism4-1.su.se-2017-04-11.log.xz
40M     /archive/syslog-prod-srv01.it.su.se/2017/wism3-1.su.se/wism3-1.su.se-2017-04-11.log.xz
78M     /archive/syslog-prod-srv01.it.su.se/2017/e-mailfilter06.sunet.se/e-mailfilter06.sunet.se-2017-04-11.log.xz
151M    /archive/syslog-prod-srv01.it.su.se/2017/ad-prod-dc6.win.su.se/ad-prod-dc6.win.su.se-2017-04-11.log.xz
162M    /archive/syslog-prod-srv01.it.su.se/2017/ad-prod-dc7.win.su.se/ad-prod-dc7.win.su.se-2017-04-11.log.xz
165M    /archive/syslog-prod-srv01.it.su.se/2017/ad-prod-dc8.win.su.se/ad-prod-dc8.win.su.se-2017-04-11.log.xz
```

???
* AD:t loggar i JSON-format samt att EventLogen är väldigt detaljerad och har väldigt många fält

---

## 2018
```terminal
root@syslog-prod-archive01:~# find /archive/syslog-prod-srv01.it.su.se/2018/ -iname "*2018-04-11*" | xargs du -sh | sort -h | tail
24M     /archive/syslog-prod-srv01.it.su.se/2018/shib-prod-idp04.it.su.se/shib-prod-idp04.it.su.se-2018-04-11.log.xz
27M     /archive/syslog-prod-srv01.it.su.se/2018/shib-prod-idp05.it.su.se/shib-prod-idp05.it.su.se-2018-04-11.log.xz
38M     /archive/syslog-prod-srv01.it.su.se/2018/sukat-prod-ldapro08.it.su.se/sukat-prod-ldapro08.it.su.se-2018-04-11.log.xz
41M     /archive/syslog-prod-srv01.it.su.se/2018/sukat-prod-ldapro09.it.su.se/sukat-prod-ldapro09.it.su.se-2018-04-11.log.xz
60M     /archive/syslog-prod-srv01.it.su.se/2018/e-mailfilter06.sunet.se/e-mailfilter06.sunet.se-2018-04-11.log.xz
70M     /archive/syslog-prod-srv01.it.su.se/2018/firewall-prod-faz03.it.su.se/firewall-prod-faz03.it.su.se-2018-04-11.log.xz
302M    /archive/syslog-prod-srv01.it.su.se/2018/ad-prod-dc7.win.su.se/ad-prod-dc7.win.su.se-2018-04-11.log.xz
361M    /archive/syslog-prod-srv01.it.su.se/2018/ad-prod-dc6.win.su.se/ad-prod-dc6.win.su.se-2018-04-11.log.xz
365M    /archive/syslog-prod-srv01.it.su.se/2018/ad-prod-dc8.win.su.se/ad-prod-dc8.win.su.se-2018-04-11.log.xz
2.6G    /archive/syslog-prod-srv01.it.su.se/2018/firewall-prod-faz02.it.su.se/firewall-prod-faz02.it.su.se-2018-04-11.log.xz
```

???
* mailfilter håller sig på samma storlek
* faz02 loggar massor
* faz03 är upp installerad
* ad:t börjar logga mer och mer

---

## 2019
```terminal
root@syslog-prod-archive01:~# find /archive/syslog-prod-srv01.it.su.se/2019/ -iname "*2019-04-11*" | xargs du -sh | sort -h | tail
34M     /archive/syslog-prod-srv01.it.su.se/2019/sukat-prod-ldapro08.it.su.se/sukat-prod-ldapro08.it.su.se-2019-04-11.log.xz
38M     /archive/syslog-prod-srv01.it.su.se/2019/lb-bravo-shall01-t1-secondary.it.su.se/lb-bravo-shall01-t1-secondary.it.su.se-2019-04-11.log.xz
39M     /archive/syslog-prod-srv01.it.su.se/2019/shib-prod-idp01.it.su.se/shib-prod-idp01.it.su.se-2019-04-11.log.xz
40M     /archive/syslog-prod-srv01.it.su.se/2019/shib-prod-idp02.it.su.se/shib-prod-idp02.it.su.se-2019-04-11.log.xz
46M     /archive/syslog-prod-srv01.it.su.se/2019/sukat-prod-ldapro09.it.su.se/sukat-prod-ldapro09.it.su.se-2019-04-11.log.xz
65M     /archive/syslog-prod-srv01.it.su.se/2019/e-mailfilter06.sunet.se/e-mailfilter06.sunet.se-2019-04-11.log.xz
356M    /archive/syslog-prod-srv01.it.su.se/2019/ad-prod-dc8.win.su.se/ad-prod-dc8.win.su.se-2019-04-11.log.xz
454M    /archive/syslog-prod-srv01.it.su.se/2019/ad-prod-dc6.win.su.se/ad-prod-dc6.win.su.se-2019-04-11.log.xz
457M    /archive/syslog-prod-srv01.it.su.se/2019/ad-prod-dc7.win.su.se/ad-prod-dc7.win.su.se-2019-04-11.log.xz
1.1G    /archive/syslog-prod-srv01.it.su.se/2019/firewall-prod-faz03.it.su.se/firewall-prod-faz03.it.su.se-2019-04-11.log.xz
```

???
* LB börjar logga mer (tidigare 19:e plats)
* AD loggar än mer
* faz loggar mindre men fortfarande mycket. Arbete pågår i DANET-589

---

# Lösningen

---

# <s>Lösningen</s> Möjlig lösning

--

* Rotera fler filer samtidigt?

---

# Nuläge

```terminal
root@syslog-prod-srv01:~# grep -vE '^#|^$' /etc/cron.d/sua-syslog-server
*15 0 * * * root /local/scriptbox/bin/surveillance logrotate /local/rsyslog/scripts/logrotate
 * * * * * root /local/scriptbox/bin/surveillance ucarp-status /usr/bin/pkill -USR1 ucarp
```

---

# Nuläge

```terminal
root@syslog-prod-srv01:~# cat /local/rsyslog/scripts/logrotate
#!/bin/bash
#
# $Id: 33930915a34c52b6d779ec893c234077bb945fc8 $
#

set -euo pipefail

LOG_DIRS="/local/rsyslog/data/logs"
COMPRESSOR="xz"

for LOG_DIR in ${LOG_DIRS}; do
    echo "Processing ${LOG_DIR}."
    echo "Cleaning out old files..."
    # SRQ-1172011 says we need 2 years of data retention
    # -mtime +730 is 2 years ago and older
    find $LOG_DIR -type f -mtime +730 -exec rm {} +

    echo "Cleaning out empty directories..."
    find $LOG_DIR -mindepth 1 -type d -empty -exec rmdir {} +

    echo "Compressing logs..."
    # -mtime +0 is yesterday and older
    # use xargs to run 8 xz processes at the time.
*    find $LOG_DIR -type f -daystart -mtime +0 -iname "*.log" -print0 | xargs -0 -P4 $COMPRESSOR
done
```

---

# Parallellisera

[4 filer i taget](https://grafana.it.su.se/d/000000054/linux-server?orgId=1&var-node=linux_syslog-prod-srv01_it_su_se&from=1554933600000&to=1554962400000)

--

[8 filer i taget](https://grafana.it.su.se/d/000000054/linux-server?orgId=1&var-node=linux_syslog-prod-srv01_it_su_se&from=1558044000000&to=1558072800000)

--

[16 filer i taget](https://grafana.it.su.se/d/000000054/linux-server?orgId=1&var-node=linux_syslog-prod-srv01_it_su_se&from=1558303200000&to=1558332000000)


---

# <u>Lösningen</u>

--

```diff
commit 1d2de6d320589c18a02122a21c25983be0bca00f
Author: Simon Lundström <simlu+github@su.se>
Date:   Mon May 20 13:40:31 2019 +0200

    Compress more files at the same time

    No limits were found to hinder us from raising it and we have more work
    to do nowadays.

diff --git a/files/local/rsyslog/scripts/logrotate b/files/local/rsyslog/scripts/logrotate
index 3393091..7d50d19 100755
--- a/files/local/rsyslog/scripts/logrotate
+++ b/files/local/rsyslog/scripts/logrotate
@@ -20,7 +20,7 @@ for LOG_DIR in ${LOG_DIRS}; do

     echo "Compressing logs..."
     # -mtime +0 is yesterday and older
-    # use xargs to run 4 xz processes at the time.
-    find $LOG_DIR -type f -daystart -mtime +0 -iname "*.log" -print0 | xargs -0 -P4 $COMPRESSOR
+    # use xargs to run X xz processes at the time.
+    find $LOG_DIR -type f -daystart -mtime +0 -iname "*.log" -print0 | xargs -0 -P16 $COMPRESSOR
 done
```

???
Eftersom att det inte blev högre CPU, RAM eller load så fyrdubblar vi antal xz processer

---

# Framtida förbättringar

--

* Skicka metrics när vi är klara med logroteringen
  * Antingen innan och efter logrotering
  * Eller den totala tiden det tog
--

* Larma mha [Monitoring Graphite metrics with Nagios](https://confluence.it.su.se/confluence/display/sysdoc/Monitoring+Graphite+metrics+with+Nagios)

---

# Frågor?
