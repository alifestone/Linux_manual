# 遠端桌面設定（wayvnc + Tailscale）

透過 [wayvnc](https://github.com/any1/wayvnc)（wlroots 系列 Wayland compositor 專用的 VNC server）搭配 [Tailscale](https://tailscale.com/) 建立的私有網路，讓筆電可以安全連進桌機看到 Hyprland 的 GUI 畫面。

## 為什麼選這組合

- Hyprland 是 Wayland compositor，傳統的 `ssh -X`／X11 forwarding 不適用，必須用支援 wlroots 的 VNC server。`wayvnc` 是目前對 wlroots-based compositor 支援最完整的選擇。
- 直接把 VNC port 開放給整個區網風險較高；改用 Tailscale 建立點對點的加密私有網路，不需要在路由器開 port，也不用擔心密碼被外部掃到。

## 1. 安裝

```bash
sudo pacman -S wayvnc
```

## 2. 設定 wayvnc

設定檔位置：`~/.config/wayvnc/config`

```ini
address=100.<>.<>.<>   # 本機的 Tailscale IP，只監聽這個介面
port=5900
enable_auth=true
username=<username>
password=<隨機產生的強密碼>
```

重點：

- `address` 設成 **Tailscale IP**（用 `tailscale ip -4` 查），而不是 `0.0.0.0`，這樣只有透過 Tailscale 連進來的裝置才連得到，不會暴露給整個區網。
- `enable_auth=true` 一定要開，並且密碼不要跟帳號一樣（原本 config 裡密碼跟帳號同名，已改成隨機字串）。
- `--gpu` / `--max-fps` 這類選項**只能當 CLI 參數**，不能寫進 config 檔（config 只接受 `address`、`port`、`enable_auth`、`username`、`password`、`certificate_file` 等固定 keyword，寫錯 key 會直接報錯 `Failed to load config`）。

## 3. 加入 Hyprland 自動啟動

這份 dotfiles 用 Lua 腳本管理 Hyprland 設定，自動啟動的程式列在 `.config/hypr/conf/autostart.lua`：

```lua
hl.on("hyprland.start", function()
    -- ...其他自動啟動的程式...
    hl.exec_cmd("wayvnc --gpu")
end)
```

`--gpu` 會啟用 GPU 加速的畫面擷取／編碼，比純 CPU 編碼快很多，能明顯改善更新率。

改完設定後**登出重新登入 Hyprland（或重啟）**讓 autostart 生效——因為 wayvnc 必須在圖形工作階段內啟動才能拿到正確的 `WAYLAND_DISPLAY` 環境變數，用 SSH 之類的非圖形化 shell 手動測試會噴 `WAYLAND_DISPLAY is not set in the environment`。

## 4. 防火牆

檢查是否有 `firewalld` 之類的服務在跑：

```bash
systemctl status firewalld
```

如果有，且 Tailscale 介面（`tailscale0`）沒有被歸類到信任的 zone，外部連線會被擋掉（ping 得通、TCP port 卻 timeout）。最簡單的作法是把整個 Tailscale 介面設成信任：

```bash
sudo firewall-cmd --permanent --zone=trusted --change-interface=tailscale0
sudo firewall-cmd --reload
```

## 5. 筆電（client）端連線

1. 筆電安裝 Tailscale，登入同一個帳號，加入同一個 tailnet。
2. 安裝 VNC client，例如 [RealVNC Viewer](https://www.realvnc.com/en/connect/download/viewer/)、TigerVNC，或 macOS 內建的「畫面共享」。
3. 連線位址填桌機的 Tailscale IP + port，例如：`100.116.76.81:5900`。
4. 輸入 `~/.config/wayvnc/config` 裡設定的帳號密碼。

## 6. 高延遲網路下的效能調校

如果筆電跟桌機不在同一個區網（跨地點連線），Tailscale 的實際 RTT 可能有數百 ms（可用 `tailscale ping <對方 IP>` 確認），這是 VNC（RFB 協定）先天上比較吃虧的情境。可以調整的地方：

- **wayvnc 端**：加 `--gpu` 讓擷取／編碼走 GPU，減少每一幀的處理時間。
- **RealVNC client 端**：連線內容（Properties）裡把 **Picture quality** 改成 **Bandwidth efficient / Low**，犧牲畫質換取更新率。
- **降低桌面負擔**：關掉不必要的動畫效果（waybar 特效、swaync 動畫），或連線時暫時降低螢幕解析度。
- **根本限制**：VNC 是為低延遲區網設計的協定，跨地點高延遲下體感一定比不上本地。如果需要更流暢的遠端操作體驗（例如遠端玩遊戲、長時間工作），建議改用 UDP-based、專為高延遲/不穩定網路設計的方案，例如 **Sunshine（server）+ Moonlight（client）**，用硬體編碼的 H.264/HEVC 串流，抗延遲與抗丟包能力遠優於傳統 VNC。

## 疑難排解

| 症狀 | 原因 | 解法 |
|---|---|---|
| `Failed to load config` | config 檔案路徑用 `--config=~/...` 這種 `=` 連接寫法，`~` 沒有展開成實際路徑 | 改用空格分隔或 `$HOME`：`--config "$HOME/.config/wayvnc/config"` |
| `WAYLAND_DISPLAY is not set in the environment` | 從 SSH 或非圖形化 shell 手動執行 wayvnc | 直接在桌機本機終端機測試，或手動 `export WAYLAND_DISPLAY=wayland-1`（socket 名稱用 `ls $XDG_RUNTIME_DIR | grep wayland` 確認） |
| ping 得通但 VNC port timeout | 防火牆（如 firewalld）擋掉 Tailscale 介面的連線 | 見上方「防火牆」章節 |
| 連上但更新率很低 | 通常是網路延遲（尤其跨地點連線），其次才是編碼效能 | 見上方「高延遲網路下的效能調校」章節 |
