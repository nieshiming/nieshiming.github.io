####  Remember

- source分支存放根目录仓库下的所有文件，为网站源文件，不参与网站打包
- main分支存放public文件所有内容，编译文件文件

> source、main分支在https://github.com/nieshiming/nieshiming.github.io仓库下

#### Start
hugo server -D

#### Publish 

##### 根目录<source分支>
```ssh

    hugo -D

    git add .
    git commit -m "feat: xxx"
    git push origin/source

```


##### Public 目录

```ssh

    // 初始化项目需要建立远程仓库链接
    git init
    git checout -b main
    git remote add origin git@github.com:nieshiming/nieshiming.github.io.git

    git add .
    git commit -m "feat: xxx"
    git push origin/main

```
