---
title: GitLab 初始化密码
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/free-photo/turned-gray-laptop-computer_400718-47.jpg
date: 2023-04-13
categories:
- GitLab
- CI/CD
tags:
- GitLab
- CI/CD
---

GitLab root 初始密码有两种情况：
- 在配置文件中自定义
- 系统自动分配

<!--more-->

在配置文件中自定义，需要在启动 GitLab 实例之前在 /etc/gitlab/gitlab.rb 文件中，去掉 gitlab_rails['initial_root_password'] 的注释，并配置一个值作为 GitLab 的初始密码。
```bash
root@jenkins:~# cat /etc/gitlab/gitlab.rb | grep -Ev '^$|#' | grep initial_root_password
gitlab_rails['initial_root_password'] = "P@ssw0rd"
```

如果在启动 GitLab 实例之前，你并没有配置 gitlab_rails['initial_root_password']，则你可以使用系统自动分配的。在 /etc/gitlab/initial_root_password  文件中，Password 的值就是你当前环境的初始密码。需要注意的是，该值得有效时间只有 24 小时，所以你在登录的 GitLab 后，尽快更改密码。
```bash
root@jenkins:~# cat /etc/gitlab/initial_root_password 
# WARNING: This value is valid only in the following conditions
#          1. If provided manually (either via `GITLAB_ROOT_PASSWORD` environment variable or via `gitlab_rails['initial_root_password']` setting in `gitlab.rb`, it was provided before database was seeded for the first time (usually, the first reconfigure run).
#          2. Password hasn't been changed manually, either via UI or via command line.
#
#          If the password shown here doesn't work, you must reset the admin password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

Password: hs4MEjhUtbxwB+2/UfPK+cy2h7Ul/KrfyTU/67mCUXI=

# NOTE: This file will be automatically deleted in the first reconfigure run after 24 hours.
```