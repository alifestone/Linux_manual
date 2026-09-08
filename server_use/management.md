# <font color="96d0ff">管理相關操作</font>
## 查看檔案大小
```bash
sudo ncdu <folder>
```

## 查看進程
```bash
# 查看近期進程
ps -eo pid,ppid,lstart,etime,tty,stat,args --sort=start_time
# 查看特定進程
ps aux | grep "<pid> or <script name>"
```

## 查看 GPU 使用
`nvidia-smi`

## 查看資源使用率
`btop`
