---
title: "Intro to Git for Security Professionals"
date: "2021-06-18"
tags: []
# header:
#     overlay_image: ""
#     show_overlay_excerpt: false
published: true
---

*As seen at Wild West Hackin Fest Way West*

## Overview

This workshop is to provide an overview and introduction to the version control system Git.

Git has grown tremendously in popularity over the past 15 years since it was released, helped along especially due to code hosting services including GitHub.com, GitLab.com, and Bitbucket.org. These sites are where open-source projects most commonly live. Any time that you hear about a new open-source security tool being released, it is mostly likely to be found on one of these sites.

This workshop will help provide an introduction to security professionals that may have no background in software development, that would like to start using their favorite open-source tool, or even more, to find ways to contribute back.

No development experience is required, and participants will finish the workshop with the tools needed to make their first contribution the same day if they choose to.

## Bio

Ian Lee is a Computer Engineer and Cyber Assessment Coordinator in the High-Performance Computing (HPC) facility at Lawrence Livermore National Laboratory (LLNL); home to some of the largest supercomputers on the planet, including Sierra, currently the #3 in the world with a performance of 125.7 Pflop/s. At LLNL, Ian has created a role performing cyber assessment, penetration testing, and purple teaming duties for the facility. Ian also has a strong background as a software developer, with a passion for the use and development of open source software and practices. He leads sustainment and outreach efforts of open source software produced by the laboratory. His personal mission is to always “leave things better than you found them.

Join the WWHF Discord Channel to participate with the presenters and other attendees during the Hackin' Cast: [https://discord.gg/wwhf]

## Live Demo

Here is what we did for the live demo portion of the workshop:

```shell
docker run -it python /bin/bash
# or: `docker run -it -v ~/.ssh/id_ecdsa:/root/.ssh/id_ecdsa python /bin/bash`

# Dependencies
apt update -y && apt install -y vim less

# Basic configuration
alias ls='ls --color'
alias ll='ls -l'
git config --global user.email "IanLee1521@gmail.com"
git config --global user.name "Ian Lee"
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

# Pre-requisites
cd
python3 -m pip install poetry

# Create the project
poetry new --name awesome awesome-project
cd awesome-project

# Start the git repo
git init

# And add the things to it
git add .
git commit -m'Initial commit'
git branch -m main

# Add a new dependency
poetry add requests

# commit it
git commit -am'Added a new dependency on requests'

# Let's check out the untracked file (poetry.lock)
cat poetry.lock
echo "poetry.lock" >> .gitignore
git status
git add .gitignore && git commit -m'Added poetry lockfile to gitignore'

# start a new module
cat << DOC > awesome/stuff.py
import requests

def get_page_size(url):
    print(f'querying url: {url}')
    response = requests.get(url)

    print(f'Got status code: {response.status_code}')
    print(f"Page html is {len(response.text)} characters long")

def demo():
        get_page_size("https://google.com")
DOC
git add awesome/stuff.py
git commit -m'Added a new module to do some stuff'

# Add the script as a part of the python pacakge
cat << DOC >> pyproject.toml

[tool.poetry.scripts]
demo = 'awesome.stuff:demo'
DOC
git commit -i pyproject.toml -m'Added script entrypoint to pyproject configuration'

# Let's run it!
poetry install
poetry run demo

# Let's make it better
poetry add -D black flake8
git commit -am'Added new dev dependencies'

# Code linting
poetry run flake8 awesome/
# But let's not fix them directly

# Black the project
poetry run black awesome/
git commit -am'Blackened my awesome project'

# Create GitHub repo
git remote add git@github.com:IanLee1521/wwhf-2021-awesome-project.git
git push -u origin main
```

At this point, we have made a repo that looks like this:

```shell
root@d2b38ef4f1e9:~/awesome-project# git lg
* 72c354f - (HEAD -> main, tag: 0.1.0, origin/main) Added new file (2 hours ago) <Ian Lee>
*   65ce609 - Merge branch 'fixup' into add-linting (2 hours ago) <Ian Lee>
|\
| * 2c08fc2 - Added pycache to ignore (2 hours ago) <Ian Lee>
* | 02de1b1 - Blackened my awesome project (2 hours ago) <Ian Lee>
* | 4cf68c1 - Added new dev dependencies (2 hours ago) <Ian Lee>
* | cfe06ed - Added demo script installation to project config (2 hours ago) <Ian Lee>
* | 75e1375 - Added new stuff module (2 hours ago) <Ian Lee>
|/
* c1575c8 - Started ignoring poetry.lock and pyc files (2 hours ago) <Ian Lee>
* 3f9e69d - Added dependency on requests (2 hours ago) <Ian Lee>
* a24b127 - Updated project description (2 hours ago) <Ian Lee>
* 76a4fdc - Initial commit, boilerplate from poetry (2 hours ago) <Ian Lee>
```

## Real world examples

### Real Workflow (Rita)

Reading some documentation: [https://github.com/activecm/rita/blob/master/docs/Docker%20Usage.md]
    - Issue: `RITA_VERSION` -> `VERSION`
    - See: [https://github.com/activecm/rita/blob/master/docker-compose.yml#L10]

Clone URLs:
    - [git@github.com:activecm/rita.git]
    - [https://github.com/activecm/rita.git]

Make change and pull request:
    - Are there any contributing guidelines?
    - PR should have a clear message of why this change is being made
        - Links / references / screenshots are excellent!

### Merge Conflicts (Peirates)

```shell
git clone https://github.com/inguardians/peirates.git  # Or: git@github.com:inguardians/peirates.git
cd peirates

git merge origin/liner

git status
git diff
```

## References

- Marcello's awesome Pretty Little Python Secrets
    - Part 1: Installing Python Tools/ Libraries the Right Way: [https://www.youtube.com/watch?v=ieyRV9zQd2U]
    - Part 2: Python Development & Packaging as Beautiful as a Poem: [https://www.youtube.com/watch?v=tNlurLxcf68]

- WWHF October 2020 Workshop
    - [https://speakerdeck.com/ianlee1521/intro-to-git-for-security-professionals]
    - [https://www.youtube.com/watch?v=D8uMsXJWuRI]

- WW Hackin Cast January 2021
    - [https://speakerdeck.com/ianlee1521/releasing-your-first-python-open-source-project-to-the-masses]
    - [https://www.youtube.com/watch?v=Czupb5lzPZI]

## Things that you'll need if you want to follow along with the workshop on your own machine

Below are some notes on how you can get your local computer set up to follow along on your own with the workshop.

### Option 1 preferred: Use Docker **PREFERRED**

Installation instructions from Docker are at [https://docs.docker.com/get-docker/] for Windows, Mac, and Linux.

Once you have docker installed, we'll be using the `python:latest` image, so run `docker pull python:latest` to download in advance.

### Option 2: Use native tools

During this workshop, we will be doing almost everything at a commandline.

To fully complete everything we'll be doing, you'll need the following tools / applications:

- git [https://git-scm.com/downloads]
- python3 [https://www.python.org/downloads/]
    - pip [https://pip.pypa.io/en/stable/installing/]

If you're on a Unix like system (Linux, macOS), install the tools natively. I'm not able to describe ALL of the permutations on this, but some common ones:

#### I'm on macOS

Use Homebrew [https://brew.sh/] to install git and python3

```shell
brew install git python3
```

#### I'm on Linux

You should be able to install the tools from your local package manager (yum, apt, etc)

```shell
# EL flavors of Linux (RHEL, CentOS, etc)
yum install -y git python3 python3-pip
```

```shell
# Debian flavors of Linux (Kali, Ubuntu, Debian, etc)
apt install -y git python3 python3-pip
```

### Option 3: Follow along

I'll be going through all of the instructions, and all of the commands live during the workshop, so if you are unable or uninterested in doing the above setup, you can simply keep an eye on the workshop, and ask questions in Discord as we go along.
