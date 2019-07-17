---
layout: post
title: "Setup ssh Google Authenticator OTP"
date: 2019-07-04 12:12 +0800
categories: tools
published: true
---

### Install Google Authenticator

```sh
sudo apt-get install libpam-google-authenticator
google-authenticator
### folllow the direction and select yes
```

### Configure PAM

```sh
sudo vi /etc/pam.d/sshd
# comment below command, This tells PAM not to prompt for a password.
# Standard Un*x authentication.
@include common-auth

# add following command
auth required pam_google_authenticator.so
```

### Configure ssh

```sh
sudo vi /etc/ssh/sshd_config
```

```sh
# for key
PubkeyAuthentication yes
PasswordAuthentication no
# for authenticator
UsePAM yes

ChallengeResponseAuthentication yes

AuthenticationMethods publickey,password publickey,keyboard-interactive

```

```sh
sudo systemctl restart sshd.service
```

refer to :

- [how-to-set-up-multi-factor-authentication-for-ssh-on-ubuntu-16-04](https://www.digitalocean.com/community/tutorials/how-to-set-up-multi-factor-authentication-for-ssh-on-ubuntu-16-04)
