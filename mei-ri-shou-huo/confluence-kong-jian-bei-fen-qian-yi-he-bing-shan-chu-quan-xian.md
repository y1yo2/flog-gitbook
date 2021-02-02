---
description: 2021/02/02
---

# Confluence空间备份迁移、合并、删除权限

## 1 Confluence空间的备份迁移

### 1.1 导出【空间团队】进行备份

* 在相应空间中，进入【空间管理】-【内容工具】；

![](../.gitbook/assets/image%20%287%29.png)

* 进入【导出】，选择以XML格式导出；

![XML&#x5BFC;&#x51FA;&#x7A7A;&#x95F4;](../.gitbook/assets/image%20%289%29.png)

* 得到空间的备份文件 Confluence-space-export-123456-123.xml.zip；

### 1.2 导入备份文件进行空间迁移

* 进入【站点管理】-【备份与还原】，导入空间的备份文件；

![](../.gitbook/assets/image%20%2810%29.png)

* 可以直接上传备份文件\(小于 25 MB\)，或将备份文件复制到主机的`/var/atlassian/application-data/confluence/restore` 目录下导入；

![&#x4E0A;&#x4F20;&#x6216;&#x5BFC;&#x5165;&#x5907;&#x4EFD;&#x6587;&#x4EF6;](../.gitbook/assets/image%20%286%29.png)

## 2 Confluence空间的合并

如何将一个空间的内容，完全迁移到其他空间呢？例如将A空间的内容，迁移到B空间的一个页面下；

* 进入空间页面，选择右上的`...`中的移动，将页面及其子页面移动至其他空间的页面下；

![&#x79FB;&#x52A8;&#x9875;&#x9762;](../.gitbook/assets/image%20%2811%29.png)

* 移动完成后可删除导出的空间；

## 3 Confluence空间-删除权限

* 空间权限分为用户组权限，用户权限，匿名用户权限；

{% hint style="info" %}
空间的权限是附加的。如果一个用户以个人的方式或者以一个用户组成员的方式赋予了权限，Confluence 将会把这些权限合并在一起。
{% endhint %}

* 如何撤销权限？将用户或用户组**取消选择**所有权限然后单击保存修改。保存后这些用户和用户组将不再显示在列表中。

![&#x64A4;&#x9500;&#x6743;&#x9650;](../.gitbook/assets/image%20%288%29.png)

> 参考资料：​[Assign Space Permissions](https://www.cwiki.us/display/CONFLUENCEWIKI/Assign+Space+Permissions)

