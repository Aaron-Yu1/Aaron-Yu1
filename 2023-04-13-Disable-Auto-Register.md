---
title: 在 Ubuntu 22.04 上安装 GitLab
thumbnailImagePosition: left
thumbnailImage: https://img.freepik.com/free-photo/person-using-laptop_53876-95246.jpg
date: 2023-04-13
categories:
- GitLab
- CI/CD
tags:
- GitLab
-CI/CD
---

默认情况下，刚刚安装好的 Gitlab 是允许任何通过自助的方式注册。但实际情况，我们使用 GitLab 都是在公司内部，因此，我们通常不需要此功能。今天我们就来看看，怎么关闭它。

<!--more-->

打开 GitLab 网页后，在登录的页面，在用户名密码窗口的下面显示着 “Don't have an account yet? Register now”。
![1](/myblog/themes/hugo-tranquilpeak-theme/exampleSite/static/img/2023-04-13-Disable-Auto-Register-1.png)

使用 root 用户登录到 GitLab，点击左上角的 Main menu 按钮（三个横杆），并选择 Admin，打开 Admin 页面。
![2](/myblog/themes/hugo-tranquilpeak-theme/exampleSite/static/img/2023-04-13-Disable-Auto-Register-2.png)

在 Admin 页面的左侧，选择 Settings 按钮，进入设置页面。
![3](/myblog/themes/hugo-tranquilpeak-theme/exampleSite/static/img/2023-04-13-Disable-Auto-Register-3.png)

在 General 页面下面，找到 Sign-up restrictions 并点击展开它。然后取消 Sign-up enabled 选项。
![4](/myblog/themes/hugo-tranquilpeak-theme/exampleSite/static/img/2023-04-13-Disable-Auto-Register-4.png)

下拉滚动菜单，找到该喷子选项的保存按钮（Save Changes），点击并保存配置。
![5](/myblog/themes/hugo-tranquilpeak-theme/exampleSite/static/img/2023-04-13-Disable-Auto-Register-5.png)


