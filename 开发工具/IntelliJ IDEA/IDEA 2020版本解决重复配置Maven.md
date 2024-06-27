# IDEA 2020版本解决重复配置Maven

[toc]

## 问题描述

IDEA 每次open 项目,都需要手动配置maven仓库,导致无法使用本地maven依赖

## 问题解决

进入IDEA打开项目界面

![image-20230505161814811](IDEA%202020%E7%89%88%E6%9C%AC%E8%A7%A3%E5%86%B3%E9%87%8D%E5%A4%8D%E9%85%8D%E7%BD%AEMaven.assets/202305051618838.png)

![image-20230505161733278](IDEA%202020%E7%89%88%E6%9C%AC%E8%A7%A3%E5%86%B3%E9%87%8D%E5%A4%8D%E9%85%8D%E7%BD%AEMaven.assets/202305051617309.png)

重新配置maven仓库地址

![image-20230505161924100](IDEA%202020%E7%89%88%E6%9C%AC%E8%A7%A3%E5%86%B3%E9%87%8D%E5%A4%8D%E9%85%8D%E7%BD%AEMaven.assets/202305051619143.png)

由于此时进行的是全局的maven配置,所以之后打开新的项目后,都会采用当前maven配置信息