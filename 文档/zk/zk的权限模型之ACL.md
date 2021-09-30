## ACL:

ACL：Access Control List 访问控制列表

```java
ACL 权限控制，使用：scheme:id:perm 来标识，主要涵盖 3 个方面：
```

权限模式（Scheme）：授权的策略
授权对象（ID）:授权的对象
权限（Permission）:授予的权限

其特性如下：
　　ZooKeeper的权限控制是基于每个znode节点的，需要对每个节点设置权限
　　每个znode支持设置多种权限控制方案和多个权限
　　子节点不会继承父节点的权限，客户端无权访问某节点，但可能可以访问它的子节点

example:

```java
setAcl /test2 ip:128.0.0.1:crwda
```

1. ### scheme 采用何种方式授权

   **world：**默认方式，相当于全部都能访问

   **auth**：代表已经认证通过的用户(cli中可以通过addauth digest user:pwd 来添加当前上下文中的授权用户)

   **digest**：即用户名:密码这种方式认证，这也是业务系统中最常用的。用 *username:password* 字符串来产生一个MD5串，然后该串被用来作为ACL ID。认证是通过明文发送*username:password* 来进行的，当用在ACL时，表达式为*username:base64* ，base64是password的SHA1摘要的编码。

   **ip**：使用客户端的主机IP作为ACL ID 。这个ACL表达式的格式为*addr/bits* ，此时addr中的有效位与客户端addr中的有效位进行比对。

2. ### ID  给谁授予权限

   ![img](https://img2018.cnblogs.com/blog/1196212/201903/1196212-20190312150144821-1097483460.png)

3. ### permission  授予什么权限

   **CREATE、READ、WRITE、DELETE、ADMIN** 也就是 **增、删、改、查、管理**权限，这5种权限简写为crwda

   注意:

   **这5种权限中，delete是指对子节点的删除权限，其它4种权限指对自身节点的操作权限**

   **更详细的如下:**

   　　**CREATE**  c 可以创建子节点
   　　**DELETE**  d 可以删除子节点（仅下一级节点）
   　　**READ**    r 可以读取节点数据及显示子节点列表
   　　**WRITE**   w 可以设置节点数据
   　　**ADMIN**   a 可以设置节点访问控制列表权限

   

4. ### **CREATE、READ、WRITE、DELETE、ADMIN** 也就是 **增、删、改、查、管理**权限，这5种权限简写为crwda

## 2.ACL 相关命令

getAcl    getAcl <path>   读取ACL权限
setAcl    setAcl <path> <acl>   设置ACL权限
addauth   addauth <scheme> <auth>   添加认证用户

