---
typora-root-url: ..
typora-copy-images-to: ../assets/images
layout: post
title: 使用Github Actions镜像Github仓库到Gitee
category: Git
tags: [git,mirrors,GitHub,Gitee]
---

由于国内访问Github仓库速度太慢，这里提供一个方案将Github的仓库镜像到Gitee，稍作修改还能支持各个Git仓库之间同步，或者一个仓库创建多个镜像。

### 定义

|           |                                        |
| --------- | -------------------------------------- |
| Gitee令牌 | 可用于访问Gitee仓库的凭据              |
| 原始仓库  | 已经存在，打算镜像到其他地方的上游仓库 |
| 镜像仓库  | 从原始仓库镜像出来的仓库               |



### 1. 生成Gitee令牌

*如果不需要自动同步，那可以跳过这一步*

进入Gitee-设置-[私人令牌](https://gitee.com/profile/personal_access_tokens)

![私人令牌页面](/assets/images/image-20200302223217152.png)

点击***+生成新令牌***

![生成新令牌页面](/assets/images/image-20200302223613886.png)

这里只勾选***projects***权限应该就够用了

![生成令牌验证](/assets/images/image-20200302223724503.png)

点击***提交***，输入Gitee登录密码，点击`验证`

![成功生成令牌](/assets/images/image-20200302223915999.png)

此时生成了**令牌**，复制保存下来，因为离开页面就没办法再找回**令牌**了。示例中生成的令牌是`b1946ac92492d2347c6235b4d2611184`，下文都是用**令牌**指代此字符串。

（任何时候，令牌泄露了可以删除令牌而不影响账号，如果不介意的话直接用账号密码作为令牌也可以的）

### 2. 初始化镜像仓库

进入[Gitee首页](https://gitee.com/)

![Gitee首页](/assets/images/image-20200302221201595.png)



鼠标移到右上角加号，在展开的菜单中选择***从 GitHub/GitLab 导入仓库***

![导入仓库页面](/assets/images/image-20200302221317392.png)

假设我要镜像`OpenWrt`的Github仓库，则在***Git仓库URL***输入 `https://github.com/openwrt/openwrt.git`，名称和**路径**自定义，我这里都是用`openwrt`，仓库介绍和是否开源都按需配置，然后点击***导入***

![正在导入页面](/assets/images/image-20200302221337996.png)

这时会出现正在导入的提示，通常一分钟以内就完成了。

导入完成后，这个**镜像仓库**就可以克隆到本地了，但是现在的**镜像仓库**不会自动从Github同步新的变化，所以下一步利用Github **Actions**定时同步上游仓库到**镜像仓库**

### 3. 定时同步

*如果不需要自动同步，那可以跳过这一步*

在Github中任意创建一个仓库，接下来用**任务仓库**指代这个仓库，用于配置同步任务。

![创建任务仓库](/assets/images/image-20200304201945427.png)

本例中仓库名称是`repo-sync-actions`，点击***Create repository***

![添加Secret入口](/assets/images/image-20200304204352909.png)

打开**任务仓库**的***Settings***，在左侧导航栏点击***Secrets***，点击***Add a new secret***。

![Secret内容](/assets/images/image-20200304204546928.png)

***Name***字段填入`PUSH_TARGET_PREFIX`。

***Value***字段填入Gitee仓库前缀，模式是`https://Gitee用户名:Gitee令牌@gitee.com/Gitee空间`，其中`Gitee令牌`即第一步生成的**令牌**；`Gitee空间`是第二步选择的项目归属，通常就是`Gitee用户名`；我这里最终填写的是 `https://jjm2473:b1946ac92492d2347c6235b4d2611184@gitee.com/jjm2473`（最后不要有斜杠）

点击***Add secret***。

![创建新文件](/assets/images/image-20200304203725075.png)

回到**任务仓库**的首页，点击***Create new file***。

![任务文件配置](/assets/images/image-20200304205608004.png)

上面的文件名填入`.github/workflows/openwrt+openwrt.yaml`（路径不能变，扩展名`.yaml`，文件名任意），下面的文件内容输入：

[//]: # "{% raw %}"
```yaml
name: openwrt/openwrt

on:
  watch:
    types: [started]
  schedule:
    - cron: '22 * * * *'

jobs:
  sync:
    name: Sync
    runs-on: ubuntu-latest
    steps:
      - name: Clone
        run: git clone --mirror --no-single-branch --depth=$DEPTH $SRC_REPO repo.git
        env:
          SRC_REPO: https://github.com/openwrt/openwrt.git
          DEPTH: 20
      - name: Config DownStream
        run: git -C repo.git remote add downstream "$PUSH_TARGET"
        env:
          PUSH_TARGET: ${{ secrets.PUSH_TARGET_PREFIX }}/openwrt.git
      - name: Push DownStream
        run: git -C repo.git push --all downstream && git -C repo.git push --tags downstream
      - name: Try Unshallow On Failed
        if: failure()
        run: git -C repo.git fetch --unshallow && git -C repo.git push --mirror downstream

```
[//]: # "{% endraw %}"

这里有几行需要修改，注意保持缩进：

* 第一行的name是任务名称，可自定义，无格式限制
* cron这一行指定定时任务执行的条件，`22 * * * *`表示每个小时的22分都执行一次
* SRC_REPO需要修改成你打算镜像的原始仓库地址
* PUSH_TARGET需要把斜杠后面的仓库改成第二步的**路径**加上`.git`

改完后点击***Commit new file***提交任务。

![Actions入口](/assets/images/image-20200304203809660.png)

之后可以在**Actions**页面查看任务执行情况，如果想立即执行，自己给**任务仓库**加个***Star***即可。



### 参考

[Github Actions](https://help.github.com/en/actions)

