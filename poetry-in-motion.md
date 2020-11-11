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
* Initialize with Poetry
* Git init
* Add dev tools
* Add hello function
* Make release
* Upload to Nexus

---

# Create a Python library
* Initialize with Poetry
???
```terminal
$ poetry new --src hellolib
$ asdf local python 3.5.3
$ $EDITOR pyproject.toml
# s/3.7/3.5/ Poetry is installed with python 3.7 hence the default
# BSD-3-Clause
$ poetry env use 3
$ poetry env list --full-path
```

--

* Hey, where's my virtualenv!?

--
```terminal
$ poetry env list --full-path
```

--
* Want them in the project folder?
```terminal
$ poetry config virtualenvs.in-project true
```
See https://python-poetry.org/docs/configuration/#virtualenvsin-project-boolean

---
# Create a Python library
* Initialize with Poetry
* Git init

???
```terminal
$ git init
$ git add *
$ curl -JL -o .gitignore 'https://raw.githubusercontent.com/github/gitignore/master/Python.gitignore'
$ git commit -m 'init'
```
--
  * Install deps
???
```terminal
$ poetry install
$ git status
$ git add poetry.lock
```
--
* Add dev tools

???
```terminal
$ poetry add --dev isort==4.3.9 yapf pylint
$ git status
$ git diff pyproject.toml
$ git commit -am 'Add dev tools'
$ echo -m 'ignore=missing-module-docstring,missing-function-docstring' > .pylintrc
```
--

* Add hello function

???
```terminal
$ poetry run python3
```

--
* Make release

???
```terminal
$ poetry version -s
$ git tag 0.1.0 -m 0.1.0
$ poetry build
```

---
* Upload to Nexus

--
* Configure proxy repo
???
https://python-poetry.org
https://confluence.it.su.se/confluence/display/sysdoc/Sonatype+Nexus+-+Add+PyPi+registry

`pyproject.toml`:
```toml
[[tool.poetry.source]]
name = "su"
url = "https://maven.it.su.se/repository/su-pypi-group/simple"
default = true
```
--

* Configure our Nexus repo
```terminal
$ poetry config http-basic.su simlu@su.se $(pass show work/nexus) # Doesn't work ATM
$ poetry config repositories.su https://maven.it.su.se/repository/su-pypi/
```

--

* Publish
```terminal
$ poetry publish --repository su
```

---

# Create an Python application
3. Skapa en app som använder liben
* Publicera till vår Nexus
* Installera appen via pipx

---

# Questions?
