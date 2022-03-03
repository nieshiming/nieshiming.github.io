####  Remember

- source分支 => 根目录仓库下的所有文件，包含hugo配置，网站源文件等，不参与网站打包
- main分支 => public文件所有内容，是实际发布网站文件

> source、main分支在https://github.com/nieshiming/nieshiming.github.io

#### Start
hugo server -D

#### Publish 

##### Source Branch <根目录>
```ssh

    hugo -D

    git add .
    git commit -m "feat: xxx"
    git push origin/source

```


##### Main Branch <public 目录>

```ssh

    // 初始化项目需要建立远程仓库链接
    git init
    git checout -b main
    git remote add origin git@github.com:nieshiming/nieshiming.github.io.git

    git add .
    git commit -m "feat: xxx"
    git push origin/main

```
