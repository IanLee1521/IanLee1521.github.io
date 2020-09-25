---
title: "Workshop: Intro to Git for Security Professionals"
date: "2020-09-25"
tags: []
# header:
#     overlay_image: ""
#     show_overlay_excerpt: false
published: true
---

Here lies the workshop content for my WWHF 2020 workshop!

## Setup

To get set up for the workshop, you will need:

- Git
- (optionally) A GitHub.com account

### Installing Git

There are many paths you could get to install git. You may also have it already (especially if you're using macOS or Linux). To check, try running

```sh
git --version
```

And see if anything comes out. Example:

```sh
$ git --version
git version 2.25.1
```

If you don't have Git installed, then check out the [Git Downloads](https://git-scm.com/downloads) page.

### Installing Git (Docker Edition)

Another option if you're inclined to use Docker is to do something like this:

```sh
$ docker run -it --name wwhf-git ubuntu:latest /bin/bash
...
root@716ffa2959f6:/#
root@716ffa2959f6:/# apt -y update && apt -y install git
...
root@716ffa2959f6:/# git --version
git version 2.25.1
```

### Making a GitHub.com Account

While not required for the workshop, before long you'll likely want to look at making a GitHub.com account to contribute back to the community.

You may want to read through the GitHub Help pages on Setting up and managing your GitHub profile.

1. [Create an account on GitHub](https://github.com/join).

2. [Update your profile information](https://github.com/settings/profile).

    - **Photo**: A headshot photo, or image that is uniquely you.
    - **Name**: Your first and last name.
    - **Bio**: Include a few words about yourself! Don't forget to mention @LLNL.
    - **URL**: This might be your [people.llnl.gov](https://people.llnl.gov) page, or a personal website if you prefer.
    - **Company**: Probably `Lawrence Livermore National Laboratory, @LLNL`.
    - **Location**: Your primary location.

3. Add your email address (and any aliases) to your [Email Settings](https://github.com/settings/emails) page. This will link any commits done via [your Git identity](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup#Your-Identity) to your GitHub account.

4. [Setup an SSH key](https://docs.github.com/articles/generating-an-ssh-key/).

5. [Enable two-factor authentication (2FA)](https://github.com/settings/security).
