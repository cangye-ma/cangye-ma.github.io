---
layout: post
title: Common Command in Linux
date: 2021-02-06 20:20:30 +0800
category: Linux
---
## 解压与压缩文件
- .tar
    - 解包：tar xvf FileName.tar
    - 打包：tar cvf FileName.tar DirName
- .gz
    - 解压1：gunzip FileName.gz
    - 解压2：gzip -d FileName.gz
    - 压缩：gzip FileName
- .tar.gz和.tgz
    - 解压：tar zxvf FileName.tar.gz
    - 压缩：tar zcvf FileName.tar.gz DirName
- .bz2
    - 解压1：bzip2 -d FileName.bz2
    - 解压2：bunzip2 FileName.bz2
    - 压缩： bzip2 -z FileName
- .tar.bz2
    - 解压：tar jxvf FileName.tar.bz2
    - 压缩：tar jcvf FileName.tar.bz2 DirName
- .bz
    - 解压1：bzip2 -d FileName.bz
    - 解压2：bunzip2 FileName.bz
    - 压缩：未知
- .tar.bz
    - 解压：tar jxvf FileName.tar.bz
    - 压缩：未知
- .Z
    - 解压：uncompress FileName.Z
    - 压缩：compress FileName
- .tar.Z
    - 解压：tar Zxvf FileName.tar.Z
    - 压缩：tar Zcvf FileName.tar.Z DirName
- .zip
    - 解压：unzip FileName.zip
    - 压缩：zip FileName.zip DirName
- .rar
    - 解压：rar x FileName.rar
    - 压缩：rar a FileName.rar DirName
