# tzfr511v_Sorry化リハ手順書
-----------------------------------------------------------------------------------
## 作業要件
-----------------------------------------------------------------------------------
|#|項目|内容|
|:--|:--|:--|
|1|対象システム|高ゼミ会員サイト|
|2|対象ホスト|tzfr511v|
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
   - 高ゼミ会員サイト(s01)
   - https://kzemi-t4.benesse.ne.jp/op/index.html

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
   tail -f /var/log/s01/apache/ssl_logs/s01_81_ssl_access_log.`date '+%Y%m%d'` | awk '{print $4,$5,$1,$6,$7,$9,$11}'
   ```

-----------------------------------------------------------------------------------
## 3. mod_rewrite_sorry_on.conf修正
-----------------------------------------------------------------------------------
1. 以下ファイルをsftpで作業/tmpにアップロードする
   ```
   mod_rewrite_sorry_on.conf.new
   ```

2. ファイルの権限を確認する
   ```
   ls -l /tmp/mod_rewrite_sorry_on.conf.new
   ```
3. ファイルの所有者を変更する<br>
   ★root:tzz0001ではないときのみ実行
   ```
   chown root:tzz0001 /tmp/mod_rewrite_sorry_on.conf.new; ll
   ```
4. ファイルの権限を変更する
   ★書き込み権限がない場合のみ実行
   ```
   chmod 664 /tmp/mod_rewrite_sorry_on.conf.new; ll
   ```

5. ディレクトリ移動
   ```
   cd /opt/s01/apache/conf/virtual/sorry/; pwd
   ```
  
6. ファイルバックアップ
   ```
   cp -p ./mod_rewrite_sorry_on.conf ./mod_rewrite_sorry_on.conf.$(date '+%Y%m%d') && ls -l
   ```

7. mod_rewrite_sorry_on.conf差し替え
   ```
   mv /tmp/mod_rewrite_sorry_on.conf.new /opt/s01/apache/conf/virtual/sorry/mod_rewrite_sorry_on.conf; ls -l
   ```

8. 差分確認<br>
   ★変更箇所のみ出力されることを確認する
   ```
   diff ./mod_rewrite_sorry_on.conf ./mod_rewrite_sorry_on.conf.$(date '+%Y%m%d')
   ```

9. 構文チェック<br>
   ★Syntax OKであること
   ```
   /opt/s01/apache/bin/httpd -t -f /opt/s01/apache/conf/httpd.conf
   ```

-----------------------------------------------------------------------------------
## 4. sorryジョブ実行
-----------------------------------------------------------------------------------
1. SystemWalkerログイン
   ★tmjb501v:rootにログイン
   
2. プロジェクト選択<br>
   TZAAD1プロジェクトを開く
   
3. ジョブ実行
   ・「sorry_on_kzemi-t4.benesse.ne.jp」を選択し右クリック
   ・「操作」⇒「起動」を選択し実行

4. 動作確認<br>
   以下のページにアクセスできることを確認<br>
   ★社内NW接続済のPCからログインページが表示されること<br>
   ★社内NW未接続のスマホで共通sorryページが表示されること
   - 高ゼミ会員サイト(s01)
   - https://kzemi-t4.benesse.ne.jp/op/index.html

5. アクセスログ確認<br>
   ★500系が返ってこないこと
   ```
   tail -f /var/log/s01/apache/ssl_logs/s01_81_ssl_access_log.`date '+%Y%m%d'` | awk '{print $4,$5,$1,$6,$7,$9,$11}'
   ```

-----------------------------------------------------------------------------------
## 5. sorry手動実行　※上記## 4.手順が実行できない場合のリカバリ手順
-----------------------------------------------------------------------------------
1. ディレクトリ移動
   ```
   cd /opt/s01/apache/conf/virtual/sorry/
   ```

2. sorry化解除状態の確認<br>
   ★sorry_off.confとの差分がないこと
   ```
   diff ./mod_rewrite_sorry.conf ./mod_rewrite_sorry_off.conf
   ```

3. sorry化実施
   ```
   cp -p ./mod_rewrite_sorry_on.conf ./mod_rewrite_sorry.conf
   ```
   
4. sorry化状態の確認<br>
   ★sorry_on.confとの差分がないこと
   ```
   diff ./mod_rewrite_sorry.conf ./mod_rewrite_sorry_on.conf
   ```

5. 構文チェック<br>
   ★Syntax OKであること
   ```
   /opt/s01/apache/bin/httpd -t -f /opt/s01/apache/conf/httpd.conf
   ```

6. プロセス確認
   ```
   ps -efwww | grep httpd | grep s01
   ```
   
7. apache再起動<br>
   親プロセスを選択し"kill -HUP"を行う
   ```
   kill -HUP [親プロセス]
   ```
   
8. プロセス確認
   ★子プロセスの起動時刻が現在時刻であること
   ```
   ps -efwww | grep httpd | grep s01
   ```

9. apacheエラーログ確認
   ```
   tail -100 /var/log/s01/apache/logs/error_log.`date '+%Y%m%d'`
   ```

10. 動作確認<br>
    以下のページにアクセスできることを確認<br>
    ★社内NW接続済のPCからログインページが表示されること<br>
    ★社内NW未接続のスマホで共通sorryページが表示されること
    - 高ゼミ会員サイト(s01)
    - https://kzemi-t4.benesse.ne.jp/op/index.html

11. アクセスログ確認<br>
    ★500系が返ってこないこと
    ```
    tail -f /var/log/s01/apache/ssl_logs/s01_81_ssl_access_log.`date '+%Y%m%d'` | awk '{print $4,$5,$1,$6,$7,$9,$11}'
    ```

-----------------------------------------------------------------------------------
## 6. sorry解除ジョブ実行
-----------------------------------------------------------------------------------
1. SystemWalkerログイン
   ★tmjb501v:rootにログイン

2. プロジェクト選択<br>
   TZAAD1プロジェクトを開く
   
3. ジョブ実行
   ・「sorry_off_kzemi-t4.benesse.ne.jp」を選択し右クリック
   ・「操作」⇒「起動」を選択し実行

4. アクセスログ確認<br>
   ★200系が返ってくること
   ```
   tail -f /var/log/s01/apache/ssl_logs/s01_81_ssl_access_log.`date '+%Y%m%d'` | awk '{print $4,$5,$1,$6,$7,$9,$11}'
   ```

5. 動作確認<br>
   以下のページにアクセスできることを確認<br>
   ★社内NW接続済のPC及び社内NW未接続のスマホでログインページが表示されること
   - 高ゼミ会員サイト(s01)
   - https://kzemi-t4.benesse.ne.jp/op/index.html

6. アクセスログ確認<br>
   ★500系が返ってこないこと
   ```
   tail -f /var/log/s01/apache/ssl_logs/s01_81_ssl_access_log.`date '+%Y%m%d'` | awk '{print $4,$5,$1,$6,$7,$9,$11}'
   ```

-----------------------------------------------------------------------------------
## 7. sorry解除手動実行　※上記## 6.手順が実行できない場合のリカバリ手順
-----------------------------------------------------------------------------------
1. ディレクトリ移動
   ```
   cd /opt/s01/apache/conf/virtual/sorry/
   ```

2. sorry化状態の確認<br>
   ★sorry_on.confとの差分がないこと
   ```
   diff ./mod_rewrite_sorry.conf ./mod_rewrite_sorry_on.conf
   ```

3. sorry化実施
   ```
   cp -p ./mod_rewrite_sorry_off.conf ./mod_rewrite_sorry.conf
   ```
   
4. sorry化状態の確認<br>
   ★sorry_off.confとの差分がないこと
   ```
   diff ./mod_rewrite_sorry.conf ./mod_rewrite_sorry_off.conf
   ```

5. 構文チェック<br>
   ★Syntax OKであること
   ```
   /opt/s01/apache/bin/httpd -t -f /opt/s01/apache/conf/httpd.conf
   ```

6. プロセス確認
   ```
   ps -efwww | grep httpd | grep s01
   ```

7. apache再起動<br>
   親プロセスを選択し"kill -HUP"を行う
   ```
   kill -HUP [親プロセス]
   ```
   
8. プロセス確認
   ★子プロセスの起動時刻が現在時刻であること
   ```
   ps -efwww | grep httpd | grep s01
   ```

9. apacheエラーログ確認
   ```
   tail -100 /var/log/s01/apache/logs/error_log.`date '+%Y%m%d'`
   ```

10. 動作確認<br>
    以下のページにアクセスできることを確認<br>
    ★社内NW接続済のPC及び社内NW未接続のスマホでログインページが表示されること
    - 高ゼミ会員サイト(s01)
    - https://kzemi-t4.benesse.ne.jp/op/index.html

11. アクセスログ確認<br>
    ★500系が返ってこないこと
    ```
    tail -f /var/log/s01/apache/ssl_logs/s01_81_ssl_access_log.`date '+%Y%m%d'` | awk '{print $4,$5,$1,$6,$7,$9,$11}'
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
