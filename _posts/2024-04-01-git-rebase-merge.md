---
title: rebase和merge的区别
layout: post
---

最近项目开始变得多提交起来了,所以git相关的操作就开始变得频繁起来了,在这里简单的记录一下一些简单的git操作,以及rebase和merge的区别

#### git开发流程

对于一个成熟的开发项目来说,git的使用流程就不像我们现在这样,从主分支直接操作,而是通过新建分支合并或者变基以达到操作主分支的目的

但是对于我这种初学者来说,这种操作还是第一次听说:

组内大佬说: 一般的大企业的项目提交流程就是,从主分支拉一个新分支,从那个新分支去做你需要做的事情,比如说一个新的需求,一个bug的修复,都在这个新的分支进行,当这个修复完成之后,我们需要从主分支获取到最新的代码,因为在我们在新分支中提交内容时,可能会有别的人也提交了代码,这就意味着需要我们从主分支将代码合并或者变基到我们的新分支上去,然后提交合并请求

当然,大佬也只是说说而己,因为对于一个新项目来说,没有这么复杂的需求,只要快速完成任务,上述的内容也只在一个大型的稳定的项目中存在,我现在也接触不到,等到接触到了再看看:)

当一个多人开发的项目正在进行的时候,总会避免不了'并行'操作,也就是多个人对代码的操作,就比如,你在新分支正在写着内容,主分支那边就已经有了新的提交,这时候你就需要将主分支rebase或者merge到你的分支,并解决冲突,那rebase和merge又有什么区别呢? 

### rebase

git rebase将当前分支上的提交应用到指定分支的最新提交之后.它实际上是把你的变更暂时保存为补丁文件,在目标分支上重新应用这些变更.

假设有一个这样的初始log图
```bash
A---B---C---D (origin/main)
     \
      E---F (local/main)
```


当执行`git rebase origin main`后
```bash
A---B---C---D (origin/main)
	         \
              E'---F' (local/main)
```

可以看到,变基就是字面意思:'更改基础版本',当你的源提交后面有新的提交产生,那你的源提交就会变成最新的那个提交,当然,这中间有冲突需要解决冲突,而且,你的新提交会发生改变,其实并不是原来的提交,会新建相同的提交,实际上hash值变化了的,只是内容一样

### merge

当你执行git merge命令时,Git会创建一个新的'合并提交',这个提交有两个父提交,分别是合并前的分支尖端和被合并的分支尖端.

假设有一个这样的初始log图
```bash
A---B---C---D (origin/main)
     \
      E---F (local/main)
```

当执行`git merge origin main`后
```bash
A---B---C---D (origin/main)
     \         \
      E---F-----G (local/main after merge)
```

可以看到,多了一个G的提交出来,这就是合并提交,而且你也可以看到,主分支的和远程分支的内容都是有保存的,不同于rebase,他的提交信息是完成的,提交的hash并没有被改变

以上
