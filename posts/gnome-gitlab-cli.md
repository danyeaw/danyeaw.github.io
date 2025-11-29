<!--
.. title: GNOME GitLab CLI
.. slug: gnome-gitlab-cli
.. date: 2025-11-28 15:54:51 UTC-05:00
.. tags: GNOME
.. category: Linux
.. link: 
.. description: Configure the glab CLI tool for use with the GNOME GitLab.
.. type: text
-->

Git authentication has evolved quite a bit since 2005 when SSH keys were the
go-to method for connecting to remote repositories.

Throughout the 2010s, GitHub made SSH the standard while HTTPS quietly gained
traction since it played nicer with corporate firewalls. When GitHub
eventually required personal access tokens instead of passwords, tools like
Git Credential Manager emerged to handle the credential storage, making HTTPS
authentication more practical.

In 2020 when GitHub released their CLI tool (`gh`),
which uses OAuth for authentication and suddenly made getting started way
easier—no more fumbling through SSH key generation tutorials for newcomers.

I used to have SSH keys installed on my Yubikey, but they were always a hassle
to set up and use. These days authenticating with `gh` is definitely the way
to go.

I also do quite a lot of open source contribution to GNOME's GitLab instance,
which seems to have completely missed the change to HTTPS that GitHub went
through. GitLab does have a CLI tool as well called `glab`.

Clement Sam started `glab` as an open source project, and then GitLab adopted
it in 2022. By default it wants to connect to gitlab.com using SSH, here's 
how you can set it up using GNOME GitLab and HTTPS.

First [create a new personal access token](https://gitlab.gnome.org/-/user_settings/personal_access_tokens?scopes=api%2Cwrite_repository)
with `write_repository` and `api` scopes.


```bash
glab config set -g host gitlab.gnome.org
glab config set -g git_protocol https
glab auth login --use-keyring --hostname gitlab.gnome.org
```

Authenticate with your token that you just created. That's it-enjoy
contributing to GNOME!
