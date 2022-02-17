# 问题：上传失败，不允许使用密码验证

> 报错信息：remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.

原因：Github不再允许使用密码上传，需要去github网页生成token

步骤：
1. Github界面:settings / Developer settings/Personal access tokens
重新生成Token
2. SourceTree界面:工具/选项/验证,删除掉后使用Https+Basics验证,弹窗里输入生成的Token


## 后续问题:上述处理之后依旧报错
报错信息和之前一样
>  报错信息：remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead
> 
步骤:
1. SourceTree界面:工具/选项/Git,最下方采用系统的Git,不使用内置的Git

---
---
