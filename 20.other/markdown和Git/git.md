# git

- [提交代码](#commit)

  

## <span id="commit">提交代码</span>

顺序执行

① git init

② git add .

③ git commit -m ‘备注信息’

④ git remote add origin  你的git地址

⑤ git push -u origin master



#### github

- 查询github映射：https://www.ipaddress.com/

- 将获取的ip映射配置到本地的host。

- ```
  git config --global http.sslVerify true //开启安全验证
  git config --global user.name "UserNmae"//设置用户名
  git config --global user.email "email@qq.com" //设置邮箱
  ```

  

#### Failed to connect to github.com port 443: Timed out

```
git config --global --unset http.proxy
git config --global --unset https.proxy
```



#### SSH配置

- 设置git的user name和email

```sh
#查看配置
git config --list
#git status
git config --global user.name "docker"
git config --global user.email "xxx@yeah.net"
```

- 检查是否存在SSH Key

```
cd ~/.ssh
```

- 创建新的ssh key

  -t ：密钥的类型
  -C : 用于识别密钥的注释
  -C 一般大家都写的是Email邮箱

```
ssh-keygen -t ed25519 -C “xxx@yeah.net”
```

- 该命令会在 .ssh文件夹生成两个文件，id_ed25519和id_ed25519.pub，id_ed25519.pub是公钥，id_ed25519是私钥

- 在github -> 个人头像 -> settings -> SSH and GPG keys -> SSH keys -> New SSH Key 添加对应的公钥文件内容

- 验证是否成功

  ```
  ssh -T git@github.com
  ```

  出现   Hi youcai922! You've successfully authenticated, but GitHub does not provide shell access.  表示成功
