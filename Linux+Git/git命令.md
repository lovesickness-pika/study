```bash
git init    在指定本地目录初始化git仓库，会生成 .git文件
git config    可以对git的一些基本命令进行配置，如缩写
git config --global alias.br branch	例：br == branch
git add file    添加指定文件到index
git add .    添加所有改动的文件到index

git status    查看local和index的区别，提供上下文信息
git status -s    查看local和index的区别，精简类型，没有上下文信息，更直观

git diff    查看local和index的具体区别
git diff --cached    查看index和snapshot的具体区别
git diff HEAD    local与index和snapshot的差异全部列出来
git diff --stat    显示改动摘要，不显示详细信息
git diff tag    显示该标签发布之后的所有改动 

git commit    提交index内容到snapshot
git commit -a    等同于先git add . 再 git commit（如果是新文件，仍然需要先add该文件）
git commit -am "xxxxx"    等同于先add、再生成快照并同事备注xxx信息

git reset HEAD -- file    取消index内的文件修改
git checkout -- file    恢复local文件，加入index之后无法恢复，要使用reset
git checkout .	撤销所有的文件修改
git reset --hard    强行回退一个版本
git reset --hard [commit id]	回退到指定版本

git rm -f file    强行删除文件，包括local区
git rm --cached file    从index删除文件，本地保留

git branch    列出所有的本地分支，当前处于的分支会有 * 提示
git branch -a    列出所有的分支，包括本地的和远程仓库的
git checkout xxx    切换到某个分支
git checkout -b "xxx"    生成并切换到新分支xxx(本地)
git push --set-upstream origin [name] 在远程仓库创建分支并上传
git branch -d "xxx"    删除某个分支，如果还有内容没有完全merge，会发出警告
git branch -D "xxx"    删除某个分支，无视merge警告

git merge xxx    把某个分支的内容合并到当前分支
git cherry-pick [commit id]	只合并某个提交点的内容到当前分支,不同于merge，merge会把所有差异的文件内容都合并过来

vim 文件    通过git直接修改某文件

git log    查看所有快照log
git log --oneline    以精简的模式查看log
git log --oneline --graph    查看log的快照，能知道合并、merge的点
git log --oneline xxx    查看xxx分支的log信息
git log --oneline xxx ^yyy    查看xxx分支的快照（但yyy分支中没有的快照）log 
git log --decorate    查看的log信息会显示tag
git log [alias]/xxx    查看远程仓库别名为alias的xxx分支log
git log --author xxx    只查看xxx的提交
git log --author xxx --oneline -5    只查看xxx的最新5个提交
git log --oneline --before={5.week.ago} --after={2018-01-01}   --no-merges 查看某个日期之前且某个日期之后的提交
git log --grep xxx    匹配备注信息里有xxx的log
git log filename	查看某个文件的每次commit记录
git log -p filename	查看某个文件每次提交的diff	
git show [commit id]	查看某次提交的修改内容

git tag -a v1.0    在当前时间点打上一个带注解的标签(v1.0)，不加-a则不会记录time，who，注解(类似commit的-m)
git tag -a v0.1 SHA    可以给某个指定的快照追加tag，SHA值为通过--oneline显示出来SHA值

git remote    列出远端仓库别名
git remote -v    列出远端仓库别名和链接地址
git remote add [alias] [url]    将url以alias的别名添加为本地的远端仓库
git remote rm [alias]    删除某个远端别名

git fetch    从远端仓库下载新分支与数据
git fetch [alias]    提取指定的远程仓库
git fetch --all    获取所有的远程仓库数据
git pull    从远端参控股提取数据并尝试合并到当前分支，相当于 fetch + merge
git push [alias] [branch]    将branch分支推送到alias远程仓库的branch分支上

```

