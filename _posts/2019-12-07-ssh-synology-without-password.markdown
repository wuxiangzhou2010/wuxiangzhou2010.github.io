---
layout: post
title: "SSH Synology NAS without password"
date: 2019-12-07 12:12 +0800
categories: tools
published: false
---

- Follow carefully with [official doc](https://www.synology.com/en-global/knowledgebase/DSM/tutorial/Management/How_to_log_in_to_DSM_with_key_pairs_as_admin_or_root_permission_via_SSH_on_computers)

- If you breaks, you can still fix using telnet to get a shell access

```sh
brew install telnet
telnet ${nas-ip}
# fill your account and passwords
```

- Disable password authentication

```sh
sudo vi /etc/ssh/sshd_config
# change PasswordAuthentication yes to PasswordAuthentication no
```

- Remember to disable telnet

- Reference
  - [how restart SSH server](https://community.synology.com/enu/forum/17/post/6347)
