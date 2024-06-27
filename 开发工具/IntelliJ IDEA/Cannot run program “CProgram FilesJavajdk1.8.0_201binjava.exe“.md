# Cannot run program “C:\Program Files\Java\jdk1.8.0_201\bin\java.exe“

[toc]

## 报错提示

```bash
Cannot run program “C:\Program Files\Java\jdk1.8.0_201\bin\java.exe“
Malformed argument has embedded quote: -Djava.endorsed.dirs=\"\"
```

无法获取jdk配置路径

## 问题原因

- IDEA 2020.2
- 使用插件Find bugs

原因是和Find bugs plugin冲突,

## 解决

- 方法一: 卸载find bugs

- 方法二: 增加本地自定义配置

`idea64.exe.vmoptions` 中追加

```bash
-Djdk.lang.Process.allowAmbiguousCommands=true
```

