# BAE_ルーティング追加手順

-----------------------------------------------------------------------------------
## 作業要件
-----------------------------------------------------------------------------------
|#|項目|内容|
|:--|:--|:--|
|1|対象システム|BAE|
|2|対象ホスト|tbfr901v,tbfr921v,tbfr931v|
|3|作業予定日|2024/01/15|

-----------------------------------------------------------------------------------
## 1. 事前作業
-----------------------------------------------------------------------------------
1. 作業開始連絡  
    関係者に作業開始の連絡を行う

-----------------------------------------------------------------------------------
## 2. サーバログイン
-----------------------------------------------------------------------------------
1. Tera Termにログインする

2. root権限へ昇格する
    ```
    su -
    ```

3. ログイン先のサーバ名を確認する
   ```
   uname -n
   ```

4. root権限であることを確認する
   ```
   id
   ```

5. システムログ確認
   ```
   cat /var/log/messages | egrep -i "error|warn|crit|fail|fatal|panic|alert|alarm|emerg"
   ```

-----------------------------------------------------------------------------------
## 3. 新AccessCheckとのルーティング追加
-----------------------------------------------------------------------------------
1. バックアップ取得<br>
   ```
   cp -p /etc/sysconfig/network-scripts/route-eth1 /etc/sysconfig/network-scripts/_route-eth1.`date '+%Y%m%d'`
   ```
   差分比較<br>
   ★差分がないことを確認
   ```
   diff /etc/sysconfig/network-scripts/route-eth1 /etc/sysconfig/network-scripts/_route-eth1.`date '+%Y%m%d'`
   ```

2. ルーティング追加<br>
   route-eth1を編集する
   ```
   vi /etc/sysconfig/network-scripts/route-eth1
   ```
   ファイルの最下部に以下の記述を追加する<br>
   ★tbfr901vの場合
   ```
   ## ascot1-01-01v
   GATEWAY16=172.21.96.1
   NETMASK16=255.255.255.255
   ADDRESS16=172.26.42.121

   ## ascot1-01-02v
   GATEWAY17=172.21.96.1
   NETMASK17=255.255.255.255
   ADDRESS17=172.26.42.122

   ## ascot1-01-03v
   GATEWAY18=172.21.96.1
   NETMASK18=255.255.255.255
   ADDRESS18=172.26.42.123

   ## ascot1-01-04v
   GATEWAY19=172.21.96.1
   NETMASK19=255.255.255.255
   ADDRESS19=172.26.42.124

   ## Reserved IP Address for AccessCheck-1
   GATEWAY20=172.21.96.1
   NETMASK20=255.255.255.255
   ADDRESS20=172.26.42.126

   ## Reserved IP Address for AccessCheck-2
   GATEWAY21=172.21.96.1
   NETMASK21=255.255.255.255
   ADDRESS21=172.26.42.127

   ## Reserved IP Address for AccessCheck-3
   GATEWAY22=172.21.96.1
   NETMASK22=255.255.255.255
   ADDRESS22=172.26.42.128

   ## Reserved IP Address for AccessCheck-4
   GATEWAY23=172.21.96.1
   NETMASK23=255.255.255.255
   ADDRESS23=172.26.42.129
   ```
   
   ★tbfr921vの場合
   ```
   ## ascot1-01-01v
   GATEWAY15=172.21.96.1
   NETMASK15=255.255.255.255
   ADDRESS15=172.26.42.121

   ## ascot1-01-02v
   GATEWAY16=172.21.96.1
   NETMASK16=255.255.255.255
   ADDRESS16=172.26.42.122

   ## ascot1-01-03v
   GATEWAY17=172.21.96.1
   NETMASK17=255.255.255.255
   ADDRESS17=172.26.42.123

   ## ascot1-01-04v
   GATEWAY18=172.21.96.1
   NETMASK18=255.255.255.255
   ADDRESS18=172.26.42.124

   ## Reserved IP Address for AccessCheck-1
   GATEWAY19=172.21.96.1
   NETMASK19=255.255.255.255
   ADDRESS19=172.26.42.126

   ## Reserved IP Address for AccessCheck-2
   GATEWAY20=172.21.96.1
   NETMASK20=255.255.255.255
   ADDRESS20=172.26.42.127

   ## Reserved IP Address for AccessCheck-3
   GATEWAY21=172.21.96.1
   NETMASK21=255.255.255.255
   ADDRESS21=172.26.42.128

   ## Reserved IP Address for AccessCheck-4
   GATEWAY22=172.21.96.1
   NETMASK22=255.255.255.255
   ADDRESS22=172.26.42.129
   ```

   ★tbfr931vの場合
   ```
   ## ascot1-01-01v
   GATEWAY11=172.21.96.1
   NETMASK11=255.255.255.255
   ADDRESS11=172.26.42.121

   ## ascot1-01-02v
   GATEWAY12=172.21.96.1
   NETMASK12=255.255.255.255
   ADDRESS12=172.26.42.122

   ## ascot1-01-03v
   GATEWAY13=172.21.96.1
   NETMASK13=255.255.255.255
   ADDRESS13=172.26.42.123

   ## ascot1-01-04v
   GATEWAY14=172.21.96.1
   NETMASK14=255.255.255.255
   ADDRESS14=172.26.42.124

   ## Reserved IP Address for AccessCheck-1
   GATEWAY15=172.21.96.1
   NETMASK15=255.255.255.255
   ADDRESS15=172.26.42.126

   ## Reserved IP Address for AccessCheck-2
   GATEWAY16=172.21.96.1
   NETMASK16=255.255.255.255
   ADDRESS16=172.26.42.127

   ## Reserved IP Address for AccessCheck-3
   GATEWAY17=172.21.96.1
   NETMASK17=255.255.255.255
   ADDRESS17=172.26.42.128

   ## Reserved IP Address for AccessCheck-4
   GATEWAY18=172.21.96.1
   NETMASK18=255.255.255.255
   ADDRESS18=172.26.42.129
   ```

   差分確認<br>
   ★変更箇所のみ差分が出力されることを確認
   ```
   diff /etc/sysconfig/network-scripts/route-eth1 /etc/sysconfig/network-scripts/_route-eth1.`date '+%Y%m%d'`
   ```

3. 設定反映前ルーティング確認<br>
   ★追加したIPがないことを確認
   ```
   netstat -nr
   ```
   ★確認するIP
   - 172.26.42.121
   - 172.26.42.122
   - 172.26.42.123
   - 172.26.42.124
   - 172.26.42.126
   - 172.26.42.127
   - 172.26.42.128
   - 172.26.42.129

4. 設定反映
   ```
   /etc/sysconfig/network-scripts/ifup-routes eth1
   ```

5. 設定反映後ルーティング確認<br>
   ★追加したIPがあることを確認
   ```
   netstat -nr
   ```
   ★確認するIP
   - 172.26.42.121
   - 172.26.42.122
   - 172.26.42.123
   - 172.26.42.124
   - 172.26.42.126
   - 172.26.42.127
   - 172.26.42.128
   - 172.26.42.129

6. システムログ確認
   ```
   cat /var/log/messages | egrep -i "error|warn|crit|fail|fatal|panic|alert|alarm|emerg"
   ```

-----------------------------------------------------------------------------------
## 99. 事後作業
-----------------------------------------------------------------------------------
1. Tera Termからログアウトする
   ```
   exit
   exit
   ```

2. サーバログイン確認
   新AccessCheck経由でサーバにログインできることを確認する
   ホスト(T)：ac10

3. 作業完了連絡
   依頼者に変更完了の連絡を行う

4. 作業エビデンス格納
   作業エビデンスを資料格納先に保存する
