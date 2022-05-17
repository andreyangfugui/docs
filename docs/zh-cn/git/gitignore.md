## 1、给项目添加.gitignore
.gitignore可以忽略你不想上传的文件，比如doc,target,classes等等

只需要在.git同目录下新增.gitignore文件，然后添加不需要上次的目录即可，比如

```shell script
/venv
.idea
```

2、清除已经上传的多余文件
如果你添加.gitignore的时候，git里面已经上传了很多不需要的文件，则使用下面两个命令干掉他们
如果是文件夹：git rm -r --cached 文件夹名
如果是文件：git rm --cached 文件名

## 如果已经上传

如果之前忘记定义 .gitignore 文件，执行了 git add . 命令之后，
发现加入的文件太多了，想要定义 .gitignore 并重新存储更改的文件。

```shell script
# 清除记录的所有文件管理信息，注意行末的点表示当前目录，一定要写
git rm -r --cached .
# 添加该仓库的所有文件，注意行末的点表示当前目录，一定要写
git add .
# 提交注释
git commit -m "....."
# 推送到仓库
git push -u origin master
```

## git删除远程仓库的文件或目录

```shell script
#删除mydir目录
git rm -r --cached mydir

#删除mydir目录下的hello.txt文件
git rm -r --cached mydir/hello.txt

git commit -m "删除说明" 

# 推送
git push
```

用-r参数删除目录, git rm --cached hello.txt 删除的是仓库中的文件，且本地工作区的文件会保留且不再与远程仓库发生跟踪关系，如果本地仓库中的文件也要删除则用git rm hello.txt
