# Xiaomi-Router-BE7000-Install-Docker

相信很多用家都會忽略了BE7000有內置Docker的功能, 因為要啟用這功能, 需先執行一些事前操作, 現在我便將這些操作講解一編吧.
1. 準備一個讀寫速度比較快的USB磁碟
2. 透過DiskGenius這類磁碟管理工具將USB碟格式化為Ext4文件系統
3. 完成格式化後, 將USB碟插入路由器背面的USB端口
4. 在瀏覽器中輸入路由器的IP地址, 預設是192.168.31.1
5. 登入管理頁面後, 點選存儲功能, 開啟USB3.0及創建虛擬內存(256MB)
6. 然後點選DOCKER頁面, 再點選安裝Docker
7. 完成安裝後點選運行Docker, 再點選安裝第三方管理
8. 管理工具安裝後便能點選管理Docker進入SimpleDocker頁面, 預設登入帳戶及密碼均為admin, 進入Docker管理頁面後先點選賬戶去修改密碼
9. 在首頁的Docker版本信息下有項Docker目錄, 其中/mnt/usb-xxxxxxxx/要好好記下, 於部署Docker應用時會運用到
10. 然後點選容器管理, 再點選simple-docker左手邊的終端命令圖示
11. 進入CLI後執行以下指令:
```
apk update
apk add docker-cli
```
這樣便完成安裝Docker了, 我們來為小米路由器部署一些應用吧.

首先小米路由器原本提供的Samba功能因沒有提供使用者密碼的設定, 故Windows 11預設是不能掛載的, 現我們先以Docker再部署一個Samba服務, 
指令中以*...*包括的地方請替換成你相對的內容
```
docker run -it --name samba \
-p 139:139 -p 445:445 \
-v /mnt/*usb_disk_name*:/share \
-d --restart unless-stopped \
dperson/samba \
-u "*user*;*password*" \
-s "*conf_name*;/share;yes;no;no;*user*;*user*;;routershare"
```
然後以下是我常用到的服務, 假如有需要你亦可輸入以下指令去安裝

部署Snapdrop一個分享剪貼內容及檔案的服務:
```
docker run -d \
  --name=snapdrop \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Asia/Hong_Kong \
  -p 8080:80 \
  -p 4043:443 \
  -v /mnt/*usb_disk_name*/mi_docker/snapdrop/config:/config \
  --restart unless-stopped \
  linuxserver/snapdrop:version-b8b78cc2
```
部署Aria2 Pro及AriaNg一個下載工具服務:
```
docker run -d \
    --name aria2-pro \
    --restart unless-stopped \
    --log-opt max-size=1m \
    -e PUID=$UID \
    -e PGID=$GID \
    -e UMASK_SET=022 \
    -e RPC_SECRET=*password* \
    -e RPC_PORT=6800 \
    -p 6800:6800 \
    -e LISTEN_PORT=6888 \
    -p 6888:6888 \
    -p 6888:6888/udp \
    -v /mnt/*usb_disk_name*/mi_docker/aria2/config:/config \
    -v /mnt/*usb_disk_name*/downloads:/downloads \
    p3terx/aria2-pro
```
```
docker run -d \
    --name ariang \
    --log-opt max-size=1m \
    --restart unless-stopped \
    -p 6880:6880 \
    p3terx/ariang
```
部署Tailscale內網穿透及遠端使用內網服務的工具:
```
docker run -d \
--name tailscale \
--network host \
--restart unless-stopped \
-v /mnt/*usb_disk_name*/mi_docker/tailscale/data:/var/lib/tailscale \
-e TS_AUTHKEY=*token_key* \
-e TS_STATE_DIR=/var/lib/tailscale \
-e TS_ROUTES=192.168.31.0/24 \
-e TS_EXTRA_ARGS="--advertise-exit-node" \
tailscale/tailscale:latest
```
部署OpenSpeedTest測試連接路由器的裝置連接速度的工具:
```
docker run -d \
--restart=unless-stopped \
--name openspeedtest \
-p 3000:3000 \
-p 3001:3001 \
openspeedtest/latest
```
