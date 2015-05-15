## Git分枝介绍
我们使用业界标准的Git Flow时行开发，大概有以下几种分枝：

1. **主分支Master**
我们把原始库/master库认作为主分支，HEAD的源代码存在于此版本中，并且随时都是一个预备生产状态，也就是稳定的、可部署的版本，一般来讲是和部署在Production上的代码是相同。
2. **开发分支Develop** 
我们的日常开发都是基于develop分枝的，它的源码始终体现下个发布版的最新软件变更，当develop分支的源码到达了一个稳定状态待发布，所有的代码变更需要以某种方式合并到master分支，然后标记一个版本号。我们是不可以直接在develop分枝上进行开发的。
3. **功能分枝 Feature** 
功能分支，它是为了开发某种特定功能，从develop分支上面分出来的。开发完成后，要再并入develop。通常功能分枝是在开发者本场的，不提交到Git仓库中。它的命名，可以采用 **feature-task-name-taskId** 的形式，例如feature-build-connection-3，这样有助于准确定位每个task/bug的代码。
4. **预发布分支 Release** 
它是指发布正式版本之前（即合并到Master分支之前），我们可能需要有一个预发布的版本进行测试。
预发布分支是从Develop分支上面分出来的，预发布结束以后，必须合并进Develop和Master分支。它的命名，可以采用 **release-v1.x.x** 的形式。
5. **修补bug分支 Hotfix** 
软件正式发布以后，难免会出现bug。这时就需要创建一个分支，进行bug修补。
修补bug分支是从master分支上面分出来的。修补结束以后，再合并进master和develop分支。它的命名，可以采用 **hotfix-v1.x.x** 的形式。

## 开发流程
开发人员一般情况下必须从ticket列表中领取任务，而不能在没有ticket的情况就开始编码，当开发人员分配到一个feature或bug时，可以按照以下流程进行处理：

1. 选择要fork的分枝，如果是处于开发阶段，则从develop分枝进行拉取，如果是Release阶段，则可能要从release分枝拉取。我们假定我们是处于开发阶段，则先切换到develop分枝，然后运行git pull origin develop来拉取最新的develop分枝的代码。
2. Fork 分枝，因为我们是不允许在develop分枝上直接开发的，所以先fork一个feature分枝进行开发，例如：git checkout -b feature-add-log-15，其中15是该feature的ticket id。
3. 开发及准备提交分枝，当开发过成准备提交时，请按照以下步骤进行处理：
    * 最终提交到server上的代码只能有1个commit，如果有多个commit，则要先使用rebase命令将多个commit压缩成1个commit。
    * commit message的格式是首字母大写，在最后要包含＃ ＋ ticket id， 例如： “Add system log function #15”。
    * 切换到develop分枝，然后更新代码，执行git pull origin develop，然后切回自已的开发分枝，执行git rebase develop,这样能确保你的新的commit是在所胡develop的上的commit的后面。
    * 最后将该分枝提交到server上先进行code review，如果有问题,则在该分枝上继续改,并且重复上面的提交步骤，等代码稳定后,则由leader将该分枝merge到develop分枝。
4. 合并分枝，当code review结束后，leader需要将代码合并到develop分枝，步骤如下：先切换到develop分枝，运行git pull origin develop，然后运行git rebase feature-branch，例如:git rebase feature-signin-15。
5. 提交develop分枝到server上，git pull origin develop, git push origin develop

## Release流程
当一个Sprint内的所有功能开发完成,并且代码比较稳时,就可以进入release阶段了。

1. 切换到develop分枝, fork一个release分枝进行release开发,例如:git checkout -b release-v1.2.1。
2. 这个Sprint内的所有新加的功能以及bug都在这个release分枝上开发,这时如果有下个Sprint的开发工作就可以直接提交到develop分枝了。但是在没有进入release阶段之前是不可以提交下一个Sprint的代码到develop分枝上的。
2. 当测试完成后,需要将release分枝分别merge回develop分枝和master分枝,同时将该分枝删除。
3. 在master分枝上打个tag，例如: git tag -a v1.2.1 -m 'My version 1.2.1'。这个版本号应该和release分枝的版本号保持一致。

## Hotfix流程
1. 切换到master分枝, fork一个hotfix分枝进行修改bug,例如:git checkout -b hotfix-v1.2.2。
2. 和feature分枝一样,开发完成准备提交前如果有多个commit,就一定要将多个commit合并成一个。
3. 上传该分枝到server上进行code review，如果没有问题则部署到staging server让QA进行测试。在code review或测试过程中发现的问题都在这个分枝上提交,但是永远只有一个commit。
4. 当测试完成后,需要将hotfix分枝分别merge回develop分枝和master分枝,同时将该分枝删除。
5. 在master分枝上打个tag，例如: git tag -a v1.2.2 -m 'My version 1.2.2'。这个版本号应该和hotfix分枝的版本号保持一致。

## 其它
### .gitignore
所有不需要提交的文件/文件夹都需要在在.gitignore文件中进行申明，主要是 **上传文件目录，缓存，日志及临时文件** 。

### 处理空文件夹
使用git add命令时不会把空文件夹加入到git中去，所以我们对所有的空文件夹增加一个空的隐藏文件.gitkeep，这个文件不会对开发造成影响，又可以对空文件夹进行跟踪。

## Reference

* [介绍一个成功的 Git 分支模型](http://www.oschina.net/translate/a-successful-git-branching-model) 
* [Git分支管理策略](http://www.ruanyifeng.com/blog/2012/07/git.html)
* [Git 工具 - 重写历史](http://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E5%86%99%E5%8E%86%E5%8F%B2)
* [语义化版本控制规范](http://semver.org/lang/zh-CN/)
