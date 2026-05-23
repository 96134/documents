git init   //git仓库初始化
git add .  //项目代码添加到缓存区
git commit -m "xxxxxx"  //添加当前的更改到仓库历史中，更新消息
git remote add github "https://xxxxx.git"  //设置别名为github的远程仓库
git push "https://xxxxxx.git"  //推送代码到别名为github的云端仓库

###### 拉取项目代码
git clone "https://xxxx.git"  //克隆一个远程仓库
git pull github master  //从别名为github的远程仓库拉取主干代码
git checkout -b cloude-previous-version github/master^ //拉取别名为github的远程仓库master分支下的上一个版本的项目代码

###### 项目代码的回溯与合并
git reset --hard 对应的历史版本的hash值  //使用相对引用回退一步
git merge 源分支别名  //将源分支的代码合并到当前的所处的分支之下

###### 进行版本控制
git checkout master  //切换分支到名称为master的分支，也即是当前工作区文件夹的体现的项目代码文件
git checkout main  //切换分支到名称为main的分支
git checkout -b dev  //创建并切换当前的git状态到dev分支

###### 推送代码
git branch <dev>  //仅仅创建一个名称为dev的分支
git push -u github dec  //推送代码到别名为github的云端仓库
//第一次提交的时候需要加上-u也就是--set-upstream的意思，追踪于上游的关系，后续提交时
//直接写为git push,默认就是将当前的文件的提交按照这一次的配置来

git status  //查看所有文件状态
git branch -a  //查看所有分支
git branch -m master main  //将当前的名为master的分支重命名为main
git log  //查看推送日志
git log --online --graph --all  //使用树形图打印提交的历史和对应的历史版本的hash值
