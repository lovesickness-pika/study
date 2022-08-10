### GIT

- 分布式版本控制软件



#### git流程

- git在每个开发人员本地有一个版本库，本地目录与版本库之间有个暂存区
- git add：本地目录增加到暂存区
- git commit：暂存区提交到本地分支（本地会有很多分支，默认分支为**main**）
- git push：分支推送到远程分支
- git pull：将远程分支拉取到本地分支（pull实际上等于fetch+merge）



#### 连接

- git config user.name	git config user.email	或者直接在.gitconfig文件修改（此文件也在c盘用户目录下）
- 生成本地key：

```shell
ssh-keygen -t rsa -C lovesickness.pika@gmail.com		//生成本地key
```

```shell
ssh -T git@github.com		//连通git
```

- 发送给远程：登录github->ssh->将本地生成秘钥的公钥（.pub）提交



#### 生成git项目

- 在项目根路径输入git init后，项目根目录生成文件夹：.git

```shell
git init
```

- 创建远程仓库地址别名

```shell
git remote add origin git@github.com:lovesickness-pika/git-test.git		//添加远程库并命名别名
```

- 查看远程仓库

```bash
git remote -v
```

#### git使用

- 发布项目（第一次提交项目）

```shell
git add .		//将根目录（.）全部提交到暂存区
git commit -m "发布项目"		//将暂存区内容提交到本地分支		-m后面为注释
git push -u origin main(远程分支名)		//将本地分支推送到远程分支
```

- 克隆项目

```shell
git clone git@github.com:lovesickness-pika/git-test.git
```

- 提交

```she
git add <文件路径>
git commit -m
git push origin main		//不需要-u
```

- 显示当前工作目录下的提交文件状态

```she
git status
```

- 查看提交的历史信息

```she
git log	//完整
git log --pretty=online		//每个版本用一行显示，且只显示当前指针之前的版本
git log --online		//使用前7位hash值
git reflog		//显示离各版本需要回退或前进的次数
```

- 切换版本

  ```shell
  git reset --hard [索引值前7位]		//基于索引值跳版本
  git reset --hard head^		//根据^的个数后退版本
  git reset --hard head~n		//后退n个版本
  
  
  ```

  reset 有三个不同的参数：soft、mixed、hard

  - soft 仅仅移动本地库的HEAD指针（这样的话就只更新本地库，可能我不想删了我刚敲的代码和刚提交到暂存区的代码）
  - mixed移动本地库HEAD指针和重置暂存区
  - hard移动本地库的指针和重置暂存区与工作区

- 查看区别

```ba
git diff <文件名>		//将工作区文件与暂存区进行比较
git diff Head <文件名>		//将工作区	文件与本地库指针指向的版本进行比较
```



#### 合并提交

- 简单方式：pull会将远程仓库的提交合并的本地仓库



#### 分支

- 分支的好处：并行推进多个功能开发，提高开发效率
- 各个分支在开发过程中，如果某一个分支开发失败，不会影响其他分支



##### 分支命令

```bash
git branch -v		//查看分支
git checkout [分支名]		//切换分支
git branch [分支名]		//创建分支
```

 ##### 合并分支

1. 切换到需要合并内容的分支

2. 执行命令并指定需要同步的分支

   ```bash
   git merge <指定分支名>		//当前分支同步指定分支的内容
   ```

##### 合并分支时产生的冲突及解决方法

- 冲突的产生：当前分支使用merge时发现需要同步的内容在当前分支进行了修改，此时git不会直接替换，而是交由开发人员解决

合并时的状态：

![image-20210715201950807](C:\Users\皮卡丘\AppData\Roaming\Typora\typora-user-images\image-20210715201950807.png)

![image-20210715202018618](C:\Users\皮卡丘\AppData\Roaming\Typora\typora-user-images\image-20210715202018618.png)



- 修改test文件后，使用 git commit提交修改

#### git基本原理

- 使用hash算法进行摘要，来保证数据的完整性，git底层使用的是SHA-1
- Git使用的历史版本是采用快照方式存储的，即每个历史版本保存的都是完整的历史文件（SVN采用增量保存，每次保存修改部分），每个历史版本都使用hash算法计算hash值来区分版本

##### 分支

- 创建分支的原理实际上是多一个指向当前版本的指针