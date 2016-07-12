---
layout: post
title: "centos7 下安装ffmpeg"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### centos7 下安装ffmpeg

[http://www.crucialp.com/resources/tutorials/server-administration/how-to-install-ffmpeg-centos-rhel-redhat-enterprise-easy-way/](http://www.crucialp.com/resources/tutorials/server-administration/how-to-install-ffmpeg-centos-rhel-redhat-enterprise-easy-way/)

[http://www.tecmint.com/enable-rpmforge-repository/](http://www.tecmint.com/enable-rpmforge-repository/)


失败了， 照上面的方式配置了yum.reps.d/dag.repo和rpmforge.repo 后还是`No package ffmpeg available.`

崩溃啊。。。

哈哈哈，找到了
[http://ask.xmodulo.com/enable-nux-dextop-repository-centos-rhel.html](http://ask.xmodulo.com/enable-nux-dextop-repository-centos-rhel.html)

nux-dextop 这个repo里有
