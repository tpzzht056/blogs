# 【git】 关于git初始化的事情

- pochiko 2019-05-15

### 一、如果本地没有建立任何项目
1. 到对应的网上，比如github.com新建Repository，拿到git地址 https://github.com/xxxxxx.git
2. 
   ```
   git clone https://github.com/xxxxxx.git .
   ```
   将当前的项目克隆到本地文件夹中。如果没有最后的点，则会在当前文件夹中新建repository名称的文件夹，项目全在这个里面，如果有则在本身进行git初始化。
   
### 二、如果本地建立过项目，想跟指定的git地址进行混合
假设指定git地址是 https://github.com/xxxxxx.git
1. 找到本地项目文件夹内，打命令 
   ```
   git init
   ```
2. ```
   git add .
   ```
3. ```
   git commit
   ```
4. ```
   git remote add origin https://github.com/xxxxxx.git
   ```
5. 可以现在就 
   ```
   git push -u origin master
   ``` 
   这个master是远程分支；也可以用
   ```
   git branch --set-upstream-to=origin/master master
   ```
   然后再git pull。本人用的是第二种。

对于使用branch后再git pull报出问题：
    refusing to merge unrelated histories

则使用
```
git pull origin master --allow-unrelated-histories
```
来强制允许历史合并，然后如果冲突就先去解决吧（windows的话总有诸如totoriseGit这类软件来解决冲突的）

[返回主页](./readme.md)