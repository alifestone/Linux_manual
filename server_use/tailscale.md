# server 端
安裝
``` bash
sudo pacman -S tailscale
```

啟用背景服務並設為開機自動啟動
``` bash
sudo systemctl enable --now tailscaled
```

登入（會給你一個網址，用瀏覽器開啟後用 Google/GitHub/Microsoft 帳號登入）
``` bash
sudo tailscale up
```

# client 端
Tailscale SSH — 讓 SSH 認證直接走 Tailscale 的身分驗證，不用管 authorized_keys：
``` bash
sudo tailscale up --ssh
```

# 相關指令
``` bash
tailscale status
tailscale ip -4      # 會是 100.x.x.x 的位址
```
