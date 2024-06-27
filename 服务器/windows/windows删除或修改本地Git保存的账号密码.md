# windows删除或修改本地Git保存的账号密码

由于在公司中途更换电脑和git账户信息，导致项目不能clone.后来发现是本地win10系统保存原来的git账户信息所致。

![img](windows%E5%88%A0%E9%99%A4%E6%88%96%E4%BF%AE%E6%94%B9%E6%9C%AC%E5%9C%B0Git%E4%BF%9D%E5%AD%98%E7%9A%84%E8%B4%A6%E5%8F%B7%E5%AF%86%E7%A0%81.assets/202306082157854.png)![img](windows%E5%88%A0%E9%99%A4%E6%88%96%E4%BF%AE%E6%94%B9%E6%9C%AC%E5%9C%B0Git%E4%BF%9D%E5%AD%98%E7%9A%84%E8%B4%A6%E5%8F%B7%E5%AF%86%E7%A0%81.assets/202306082157763.png)

![img](windows%E5%88%A0%E9%99%A4%E6%88%96%E4%BF%AE%E6%94%B9%E6%9C%AC%E5%9C%B0Git%E4%BF%9D%E5%AD%98%E7%9A%84%E8%B4%A6%E5%8F%B7%E5%AF%86%E7%A0%81.assets/202306082157807.png)



修改或删除git凭据

```bash
git config --global user.name "zhangsan"
git config --global user.email "123@qq.com"
```