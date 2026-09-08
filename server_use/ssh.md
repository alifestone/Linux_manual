## 設定 SSH 伺服器

### 1. 安裝 SSH server

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install openssh-server

# Arch/EndeavourOS
sudo pacman -S openssh
```

### 2. 啟動並設定開機自動啟動

```bash
sudo systemctl enable --now sshd
sudo systemctl status sshd   # 確認是否 active (running)
```

### 3. 確認防火牆放行 22 埠

```bash
# ufw (Ubuntu)
sudo ufw allow ssh

# firewalld
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

### 4. 基本安全設定（`/etc/ssh/sshd_config`）

```
Port 22                      # 可改成非標準埠減少掃描
PermitRootLogin no           # 禁止直接以 root 登入
PasswordAuthentication no    # 改用金鑰登入，關閉密碼登入
PubkeyAuthentication yes
```

改完後重啟服務：
```bash
sudo systemctl restart sshd
```

### 5. 設定金鑰登入

**本機（client）產生金鑰：**
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

**上傳公鑰到伺服器：**
```bash
ssh-copy-id user@server_ip
# 或手動
cat ~/.ssh/id_ed25519.pub | ssh user@server_ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### 6. 測試連線

```bash
ssh user@server_ip
# 或指定 port
ssh -p 2222 user@server_ip
```

---

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
