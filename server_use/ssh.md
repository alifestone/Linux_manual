# <font color="32a653"> 基礎操作 </font>
## <font color="96d0ff">使用者相關操作 </font>
### 增加使用者至伺服器
```bash
sudo adduser <username> 
```

### 將使用者加入 ssh key 白名單
```bash
cd /home/<username>
sudo mkdir .ssh
sudo vim .ssh/authorized_keys
```

### 更改(檔案/資料夾)的(擁有者/群組)
> 把 .ssh 跟 .ssh/authorized_keys 的擁有者給使用者
```bash
sudo chown <username>:<usergroup> .ssh 
sudo chown <username>:<usergroup> .ssh/authorized_keys
```

### 將使用者從伺服器移除
>加上 --remove-home 會把使用者的家目錄包括裡面的資料全部移除，請謹慎使用

```bash
sudo deluser --remove-home <username>
```

### 將使用者從 sudo 身分組移除
```bash
sudo deluser <username> sudo
```

### 重新啟動 ssh
```bash
sudo /etc/init.d/ssh restart
```

### 更改 docker 權限
```bash
sudo chmod 777 /var/run/docker.sock
```

### 加入 docker group
```bash
sudo usermod -aG docker $USER
getent group docker
```
## <font color="96d0ff">管理相關操作</font>
### 查看檔案大小
```bash
sudo ncdu <folder>
```
