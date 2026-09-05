---
title: Dev Log — The Aliyun Server Finally Arrived
date: 2026-09-02 12:30:00
tags:
---

It finally happened. The Aliyun Ubuntu server that our school provides showed up, and honestly I've been waiting for this day for a while now. Up until this point my Didi Words project has been living entirely on my laptop — if my laptop dies, the project dies with it. So today's mission was simple: give the project a real home on the server.

Here's how the afternoon went, mistakes included.

## Step 1: SSH in and create a bare Git repo

First, I logged into the server. Since the repo is going to live under its own user, I switched over to the `git` user first, made a folder to keep remote repos organized, and created a bare repo for Didi Words:

```bash
su git
mkdir -p ~/repo/didiwords.git
cd ~/repo/didiwords.git
git init --bare
```

Quick note for future me: a **bare** repo is the right choice here. A normal repo has a working directory — actual files you can see and edit — but a bare repo only keeps Git's internal database (branches, history, objects). That's exactly what you want for a remote, because nobody is supposed to sit on the server and edit code directly. It's purely a place to push to and pull from.

The moment I ran `git init --bare`, Git threw a big wall of yellow hints at me about the initial branch name — it defaulted to `master` and reminded me that I could switch to `main` if I wanted. I just rolled with `master` for now since that's what my local project uses anyway. You can see the whole scene in the screenshot below:

![Initializing the bare repo on the server](/image/905/Screenshot%20From%202026-09-01%2019-34-10.png)

After it finished, I did an `ls` inside the repo and saw `branches  config  description  HEAD  hooks  info  objects  refs` — the anatomy of a bare repo. Weirdly satisfying. It's like the repo is sitting there saying "I'm ready, push me."

Oh, and one small stumble worth recording: before switching to root properly, I tried running `sudo` as the `git` user and got hit with `sudo: a password is required`. Right — the git user isn't in the sudoers setup. Had to `su root -` and enter the actual root password to get anything privileged done. Noted for next time.

## Step 2: Push the project up from my laptop

With the remote ready, I went back to my laptop, added the server as a remote, and pushed everything up:

```bash
git remote add origin git@8.133.224.64:/home/git/repo/didiwords.git
git push --set-upstream origin master
```

This part was a little slow, not gonna lie. It counted 809 objects, compressed them with all 12 threads, and then uploaded about 10 MiB at roughly 198 KiB/s. I just sat there watching the progress bar crawl. But it got there:

![First successful push to the server](/image/905/Screenshot%20From%202026-09-01%2019-44-00.png)

And then the line I was waiting for: `* [new branch] master -> master`. The project now officially lives on the server. There was also a warning saying the remote doesn't support the Git LFS locking API — it's harmless, and if it ever gets annoying it can be silenced with `git config lfs.<url>.lfslockverify false`, but I left it alone.

## Step 3: Set up PostgreSQL for the database

Code's on the server, but an app is nothing without its data, so next up was the database. I installed PostgreSQL on the server and created a `didiwords` database for the project to use.

For actually managing it, I didn't want to squint at `psql` in a terminal all day, so I connected to the server's PostgreSQL from my laptop using pgAdmin. Way more comfortable:

![pgAdmin connected to the didiwords database](/image/905/Screenshot%20From%202026-09-02%2011-56-20.png)

And there it is on the left: server "Didi", database "didiwords" nested inside, with all the usual suspects (Schemas, Extensions, Publications and friends) under it. On the right, the dashboard is already showing live sessions and tuple activity spikes, which basically means: it's alive. Connection works, database works, good.

## Where things stand

So after one afternoon, the server now has:

- ✅ A bare Git repo ready to receive pushes
- ✅ PostgreSQL running with the `didiwords` database created
- ⬜ Some kind of deploy hook so pushes actually build and run the app
- ⬜ Nginx sitting in front of everything
- ⬜ HTTPS (let's encrypt, literally)

Honestly, not a bad day's work. The bones are in place — next log I want the app actually *running* on this thing. Fingers crossed.
