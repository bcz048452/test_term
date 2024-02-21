# tofr521v_Sorry化リハ手順書
-----------------------------------------------------------------------------------
## 作業要件
-----------------------------------------------------------------------------------
|#|項目|内容|
|:--|:--|:--|
|1|対象システム|高HOME|
|2|対象ホスト|tofr521v|
|3|作業日|2024/2/20|
|4|変更管理|-|

-----------------------------------------------------------------------------------
## 1. 事前作業
-----------------------------------------------------------------------------------
1. 作業開始連絡<br>
   関係者に作業開始連絡を行う

2. 動作確認<br>
   ログインページにアクセスできることを確認<br>
   ★社内NW（wifi）接続済のPC及び社内NW未接続のスマホでそれぞれ確認する
   - Hybrid講座システム(s07)
   - https://d-k-st.benesse.ne.jp

-----------------------------------------------------------------------------------
## 2. サーバログイン
-----------------------------------------------------------------------------------
1. サーバにログインする

2. ログ取得開始

3. ログインホスト確認
   ```
   uname -n
   ```

4. rootユーザに切替
   ```
   su -
   ```

5. 切替確認
   ```
   id
   ```

6. システムログ確認
   ```
   cat /var/log/messages | egrep -i "error|warn|crit|fail|fatal|panic|alert|alarm|emerg"
   ```

7. アクセスログ確認<br>
   ★500系が返っていないこと
   ```
   tail -f /var/log/s07/apache/ssl_logs/s07_81_ssl_access_log.`date '+%Y%m%d'` | awk '{print $4,$5,$1,$6,$7,$9,$11}'
   ```

-----------------------------------------------------------------------------------
## 3. 負荷分散切り離し
-----------------------------------------------------------------------------------
1. NetScaler接続<br>
   ★NodeがPrimaryであることを確認
   - http://192.168.58.20/menu/neo　※セカンダリ
   - http://192.168.58.21/menu/neo
  
2. 負荷分散状態確認<br>
   Traffic Management > Load Balancing > Servers を選択<br>
   Searchより"tofr521vp7"を選択<br>
   ★State欄が「Enabled」であること　※Disabledの場合切り離し不要<br>
   ★IPAddrress/Domain欄が「172.22.131.209」であること
   
3. 負荷分散切り離し<br>
   tofr521vp7を選択し、右クリック。<br>
   Disableを選択し切り離しを実施。<br>
   ★Disableであることを確認

4. アクセスログ確認<br>
   ★アクセスが来なくなったこと
   ```
   tail -f /var/log/s07/apache/ssl_logs/s07_81_ssl_access_log.`date '+%Y%m%d'` | awk '{print $4,$5,$1,$6,$7,$9,$11}'
   ```

5. 動作確認<br>
   共通sorryページが表示されること<br>
   ★社内NW接続済のPC及び社内NW未接続のスマホでそれぞれ確認する
   - Hybrid講座システム(s07)
   - https://d-k-st.benesse.ne.jp

-----------------------------------------------------------------------------------
## 4. 手動Sorry化
-----------------------------------------------------------------------------------
1. 以下ファイルをsftpで作業/tmpにアップロードする
   ```
   s07_81_virtual-mod_rewrite.conf.2024.sorry
   ```

2. ファイルを移動する
   ```
   mv /tmp/s07_81_virtual-mod_rewrite.conf.2024.sorry /opt/s07/apache/conf/virtual
   ```

3. ファイルの権限を確認する
   ```
   ls -l /opt/s07/apache/conf/virtual/s07_81_virtual-mod_rewrite.conf.2024.sorry
   ```
4. ファイルの所有者を変更する<br>
   ★root:rootではないときのみ実行
   ```
   chown root:root /opt/s07/apache/conf/virtual/s07_81_virtual-mod_rewrite.conf.2024.sorry; ll
   ```
5. ファイルの権限を変更する
   ★書き込み権限がない場合のみ実行
   ```
   chmod 664 /opt/s07/apache/conf/virtual/s07_81_virtual-mod_rewrite.conf.2024.sorry; ll
   ```

6. ディレクトリ移動
   ```
   cd /opt/s07/apache/conf/virtual/; pwd
   ```
  
7. ファイルバックアップ
   ```
   cp -p ./s07_81_virtual-mod_rewrite.conf ./bk/s07_81_virtual-mod_rewrite.conf.$(date '+%Y%m%d') && ls -l
   ```

8. 差分確認<br>
   ★差分がないことを確認する
   ```
   diff ./s07_81_virtual-mod_rewrite.conf ./bk/s07_81_virtual-mod_rewrite.conf.$(date '+%Y%m%d')
   ```

9.  sorry化実施
   ```
   cp -p ./s07_81_virtual-mod_rewrite.conf.2024.sorry ./s07_81_virtual-mod_rewrite.conf
   ```

10. 差分確認<br>
    ★変更箇所のみ出力されることを確認する
    ```
    diff ./s07_81_virtual-mod_rewrite.conf ./bk/s07_81_virtual-mod_rewrite.conf.$(date '+%Y%m%d')
    ```

11. 構文チェック<br>
    ★Syntax OKであること
    ```
    /opt/s07/apache/bin/httpd -t -f /opt/s07/apache/conf/httpd.conf
    ```

12. プロセス確認
    ```
    ps -efwww | grep httpd | grep s07
    ```
    
13. apache再起動<br>
    親プロセスを選択し"kill -HUP"を行う
    ```
    kill -HUP [親プロセス]
    ```

14. プロセス確認
    ★子プロセスの起動時刻が現在時刻であること
    ```
    ps -efwww | grep httpd | grep s07
    ```

15. apacheエラーログ確認
    ```
    tail -100 /var/log/s07/apache/logs/error_log.`date '+%Y%m%d'`
    ```

-----------------------------------------------------------------------------------
## 5. 負荷分散繋ぎ込み
-----------------------------------------------------------------------------------
1. NetScaler接続<br>
   ★NodeがPrimaryであることを確認
   - http://192.168.58.20/menu/neo　※セカンダリ
   - http://192.168.58.21/menu/neo
   
2. 負荷分散状態確認<br>
   Traffic Management > Load Balancing > Servers を選択<br>
   Searchより"tofr521vp7"を選択<br>
   ★State欄が「Disable」であること<br>
   ★IPAddrress/Domain欄が「172.22.131.209」であること

3. 負荷分散繋ぎ込み<br>
   tofr521vp7を選択し、右クリック。<br>
   Enabledを選択し繋ぎ込みを実施。<br>
   ★Enabledであることを確認

4. 動作確認<br>
   以下のページにそれぞれアクセスできることを確認<br>
   ★社内NW接続済のPCからログインページが表示されること<br>
   ★社内NW未接続のスマホで共通sorryページが表示されること
   - Hybrid講座システム(s07)
   - https://d-k-st.benesse.ne.jp

5. アクセスログ確認<br>
   ★500系が返ってこないこと
   ```
   tail -f /var/log/s07/apache/ssl_logs/s07_81_ssl_access_log.`date '+%Y%m%d'` | awk '{print $4,$5,$1,$6,$7,$9,$11}'
   ```

-----------------------------------------------------------------------------------
## 7. sorry解除
-----------------------------------------------------------------------------------
1. ディレクトリ移動
   ```
   cd /opt/s07/apache/conf/virtual/
   ```

2. sorry化状態の確認<br>
   ★バックアップと差分があることを確認
   ```
   diff ./s07_81_virtual-mod_rewrite.conf ./bk/s07_81_virtual-mod_rewrite.conf.$(date '+%Y%m%d')
   ```

3. sorry解除
   ```
   cp -p ./bk/s07_81_virtual-mod_rewrite.conf.$(date '+%Y%m%d') ./s07_81_virtual-mod_rewrite.conf
   ```

4. 差分確認<br>
   ★バックアップとの差分がないことを確認
   ```
   diff ./s07_81_virtual-mod_rewrite.conf ./bk/s07_81_virtual-mod_rewrite.conf.$(date '+%Y%m%d')
   ```

5. 構文チェック<br>
   ★Syntax OKであること
   ```
   /opt/s07/apache/bin/httpd -t -f /opt/s07/apache/conf/httpd.conf
   ```

6. プロセス確認
   ```
   ps -efwww | grep httpd | grep s07
   ```

7. apache再起動<br>
   親プロセスを選択し"kill -HUP"を行う
   ```
   kill -HUP [親プロセス]
   ```
   
8. プロセス確認
   ★子プロセスの起動時刻が現在時刻であること
   ```
   ps -efwww | grep httpd | grep s07
   ```

9.  apacheエラーログ確認
   ```
   tail -100 /var/log/s07/apache/logs/error_log.`date '+%Y%m%d'`
   ```

10. 動作確認<br>
    以下のページにアクセスできることを確認<br>
    ★社内NW接続済のPC及び社内NW未接続のスマホでログインページが表示されること
    - Hybrid講座システム(s07)
    - https://d-k-st.benesse.ne.jp

11. アクセスログ確認<br>
    ★500系が返ってこないこと
    ```
    tail -f /var/log/s07/apache/ssl_logs/s07_81_ssl_access_log.`date '+%Y%m%d'` | awk '{print $4,$5,$1,$6,$7,$9,$11}'
    ```

-----------------------------------------------------------------------------------
## 99. 事後作業
-----------------------------------------------------------------------------------
1. システムログ確認
   ```
   cat /var/log/messages | egrep -i "error|warn|crit|fail|fatal|panic|alert|alarm|emerg"
   ```

2. 作業終了連絡<br>
   関係者に作業終了の連絡を行う
