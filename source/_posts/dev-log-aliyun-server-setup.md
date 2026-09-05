---
title: Dev Log — The Aliyun Server Finally Arrived
date: 2026-09-02 12:30:00
tags:
---

Finally! The Aliyun Ubuntu server our school provided arrived, so today I spent the whole afternoon setting it up. The goal is simple: give my Didi Words project a real home instead of living only on my laptop. Here's what I did.

## Step 1: Create a bare Git repo on the server

First things first — I SSH'd into the server, switched to the `git` user, and created a directory to hold my remote repos. A bare repo is perfect here since nobody needs to work on the code directly on the server; it's just a place to push to.

```bash
su git
mkdir -p ~/repo/didiwords.git
cd ~/repo/didiwords.git
git init --bare
```

Git greeted me with a wall of friendly hints about `master` vs `main` (see screenshot below), but the repo was initialized just fine:

![Initializing the bare repo on the server](image/905/Screenshot%20From%202026-09-01%2019-34-10.png)

That `branches  config  description  HEAD  hooks  info  objects  refs` listing always feels satisfying — like the repo is saying "I'm ready, push me."

One small hiccup: I tried `sudo` as the `git` user first and got `sudo: a password is required`. Oops. Had to `su root -` to get things done. Classic.

## Step 2: Push the project from my laptop

Back on my machine, I added the server as a remote and pushed the whole project up:

```bash
git remote add origin git@8.133.224.64:/home/git/repo/didiwords.git
git push --set-upstream origin master
```

It took a bit — 809 objects, about 10 MiB, crawling up at ~198 KiB/s — but it worked:

![First successful push to the server](image/905/Screenshot%20From%202026-09-01%2019-44-00.png)

Seeing `* [new branch] master -> master` never gets old. There was a harmless warning about Git LFS locking API not being supported, which I can safely ignore (or disable with `lfs.<url>.lfslockverify false` if it gets annoying).

## Step 3: Set up PostgreSQL

The app needs a database, so next I installed PostgreSQL on the server and created a `didiwords` database for the project. To make my life easier, I connected to it from my laptop with pgAdmin instead of fumbling around in `psql` all the time:

![pgAdmin connected to the didiwords database](image/905/Screenshot%20From%202026-09-02%2011-56-20.png)

Look at that — server "Didi", database "didiwords", dashboard showing live sessions. It's alive!

## What's next

The server now has:

- ✅ A bare Git repo to receive pushes
- ✅ PostgreSQL up and running with the `didiwords` database
- ⬜ A proper deploy hook / CI to build and run the app on push
- ⬜ Nginx in front of everything
- ⬜ HTTPS (let's encrypt, literally)

Not a bad day's work. Next log I'll hopefully have the app actually running on the server. Fingers crossed.
