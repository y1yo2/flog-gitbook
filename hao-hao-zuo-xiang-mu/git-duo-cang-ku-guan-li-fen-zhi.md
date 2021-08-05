---
description: 同一个工程，多个仓库的分支如何管理？
---

# Git多仓库管理分支

仓库a，开发仓库，别名dev；仓库b，线上仓库，别名online；

需求：本地仓库拉取dev仓库，修改后推送dev仓库及online仓库；

### 方案一：推送线上仓库新分支

dev：分支dev\_master

online：分支online\_master

```text
git clone dev
git checkout dev_master

git remote add online online.git

git push online dev_master:new_master
```

online：分支online\_master；分支new\_master；

通过在online库 Merge online\_master，同步分支的内容和历史记录。若想合并历史记录并重做基础分支，推荐使用 rebase 合并。

适合场景：适合一次性的跨库同步操作，方便快捷。

缺点：造成online库的分支增多且混乱。

### 方案二：本地伪造分支合并，推送线上仓库

dev：分支dev\_master

online：分支online\_master

```csharp
git clone dev
git checkout dev_master

git remote add online online.git

git checkout -b fake_onlmaster online/online_master

git merge dev_master --allow-unrelated-historie
# 处理冲突
git add .
git commit -m 'update fix conflict'

git checkout online/online_master
git merge fake_onlmaster
git push online/online_master

git branch -D fake_onlmaster
```

### merge request 与 pull request

MR：同一仓库中，两条分支合并发起 merge request，审核通过后，分支merge成功。

PR：同一工程，fork生成两个仓库（作者仓库、fork仓库）。将fork仓库的更改，推送至作者仓库时，发起 pull request（对作者来说，是一个拉取别人代码的请求），审核通过后，fork仓库的更改会merge进作者仓库。

### merge 与 rebase

merge 和 rebase，都是分支合并命令。有什么区别，分别适合什么场景？

1.处理冲突

merge 处理冲突时，只需要处理一次，处理合并后有一个commit。

rebase 处理冲突时，每个历史commit都需处理一次，处理合并后无commit。

2.历史commit记录

merge：历史commit记录分别归属两条分支。

rebase：历史commit记录会归属到被合并的分支。

#### 适用场景

1. 其他本地仓库有副本的分支，不建议执行rebase。
2.  多分支合并时，`git merge --no-ff`，自动生成merge commit。
3. 拉取远程仓库代码时，`git pull --rebase`，等同于将本地commit推送远程分支，而不产生额外commit（git pull默认使用merge）。但，需要每次commit都处理一遍冲突。

> rebase 实际是在原分支重新commit一遍，需要合并的分支的commit。
>
> git merge other\_branch -no-ff
>
> -no-ff：即使无需处理冲突，也生成一个merge的commit节点





