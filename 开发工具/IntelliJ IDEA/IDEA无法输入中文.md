# IDEA无法输入中文

[toc]

## 问题描述

输入法没问题，其他软件可以输入中文，只有idea突然输入不了中文。

## 解决方案

### 方法一:重启大法

直接重启

### 方法二:自定义VM选项

`idea64.exe.vmoptions` 中追加

```bash
 -Drecreate.x11.input.method=true
```

重启IDEA