# th_apache+tomcat停止手順書.md

-----------------------------------------------------------------------------------
## 作業要件
-----------------------------------------------------------------------------------
|#|項目|内容|
|:--|:--|:--|
|1|対象システム|Hybrid講座システム_問題配信システム|
|2|対象ホスト|thap501v,thap502v,thap511v,thap521v,thap541v,thfr551v,thfr561v,thfr571v,thfr591v|
|3|作業予定日|2024/04/03|

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
## 3. 負荷分散切り離し
-----------------------------------------------------------------------------------
1. BIG-IP接続<br>
   thap~：
   - https://192.168.237.183/tmui/login.jsp
   - https://192.168.237.184/tmui/login.jsp

   thfr~：
   - https://192.168.59.5/tmui/login.jsp
   - https://192.168.59.6/tmui/login.jsp <br>
   ★接続後画面左上に「ONLINE(ACTIVE)」と表示されること
   
2. 負荷分散状態確認<br>
   Local Traffic > Network Map を選択<br>
   Searchより対象ホストを選択し、ShowMapを押下<br>
   ★「th_apa-tom停止対象調査.xlsx」を参照の元対象を選択<br>
   ★State欄が「Enabled」であること　※Disabledの場合切り離し不要
   
3. 負荷分散切り離し<br>
   対象のPoolメンバステータスをクリックで選択<br>
   StateからForced Offlineを選択しUpdateを押下<br>

4. 切り離し確認<br>
   再度Local Traffic > Network Map > ShowMapよりDisabledであることを確認する

-----------------------------------------------------------------------------------
## 4. プロセス監視無効化
-----------------------------------------------------------------------------------
1. ディレクトリ移動
   ```
   cd /etc/opt/svmon/
   ```

2. ProcessMonitor.txtファイルバックアップ<br>
   ★ProcessMonitor.txtが存在することを確認
   ```
   ll /etc/opt/svmon/
   ```
   ★コピー後差分が無いこと
   ```
   cp -p /etc/opt/svmon/ProcessMonitor.txt /etc/opt/svmon/ProcessMonitor.txt.`date '+%Y%m%d'`
   ```
   ```
   diff /etc/opt/svmon/ProcessMonitor.txt /etc/opt/svmon/ProcessMonitor.txt.`date '+%Y%m%d'`
   ```

3. ProcessMonitor.txt内s01,s02を含む設定削除<br>
   ★「プロセス監視設定削除対象_問題配信」ファイルを参照に削除する
   ```
   vi ProcessMonitor.txt
   ```

4. 設定削除後差分確認<br>
   ★コメントアウト箇所のみ差分として出力されることを確認
   ```
   diff /etc/opt/svmon/ProcessMonitor.txt /etc/opt/svmon/ProcessMonitor.txt.`date '+%Y%m%d'`
   ```

5. バックアップへ上書きコピー
   ```
   cp -p /etc/opt/svmon/ProcessMonitor.txt /etc/opt/svmon/ProcessMonitor_bak.txt
   ```
   ★コピー後に差分がないことを確認
   ```
   diff /etc/opt/svmon/ProcessMonitor.txt /etc/opt/svmon/ProcessMonitor_bak.txt
   ```

-----------------------------------------------------------------------------------
## 5. Apache Tomcat停止
-----------------------------------------------------------------------------------
1. 停止前プロセス確認<br>
   ★Apache Tomcatが起動していることを確認
   ```
   ps -ef | grep httpd | grep s01
   ```
   ```
   ps -ef | grep tomcat | grep s01
   ```

2. Apache Tomcat停止
   ```
   /etc/init.d/apache-tomcat_s01 stop
   ```

3. 停止後プロセス確認<br>
   ★Apache Tomcatが停止していることを確認
   ```
   ps -ef | grep httpd | grep s01
   ```
   ```
   ps -ef | grep tomcat | grep s01
   ```
   ※以下手順はs02環境があるホストのみ実行<br>
   対象ホスト：thap521v,thap541v,thfr571v,thfr591v

4. 停止前プロセス確認<br>
   ★Apache Tomcatが起動していることを確認
   ```
   ps -ef | grep httpd | grep s02
   ```
   ```
   ps -ef | grep tomcat | grep s02
   ```

5. Apache Tomcat停止
   ```
   /etc/init.d/apache-tomcat_s02 stop
   ```

6. 停止後プロセス確認<br>
   ★Apache Tomcatが停止していることを確認
   ```
   ps -ef | grep httpd | grep s02
   ```
   ```
   ps -ef | grep tomcat | grep s02
   ```

-----------------------------------------------------------------------------------
## 6. Apache Tomcat 起動スクリプト修正
-----------------------------------------------------------------------------------
1. ファイル確認<br>
   ★ファイルが存在すること
   - apache_s01
   - tomcat_s01
   - apache-tomcat_s01
   ```
   ll /etc/init.d/ | grep -i s01
   ```

2. 起動スクリプトリネーム
   ```
   mv /etc/init.d/apache_s01 /etc/init.d/_apache_s01_`date '+%Y%m%d'`
   ```
   ```
   mv /etc/init.d/tomcat_s01 /etc/init.d/_tomcat_s01_`date '+%Y%m%d'`
   ```
   ```
   mv /etc/init.d/apache-tomcat_s01 /etc/init.d/_apache-tomcat_s01_`date '+%Y%m%d'`
   ```

3. リネーム確認
   ```
   ll /etc/init.d/ | grep -i s01
   ```

   ※以下手順はs02環境があるホストのみ実行<br>
   対象ホスト：thap521v,thap541v,thfr571v,thfr591v

4. ファイル確認<br>
   ★ファイルが存在すること
   - apache_s02
   - tomcat_s02
   - apache-tomcat_s02
   ```
   ll /etc/init.d/ | grep -i s02
   ```

5. 起動スクリプトリネーム
   ```
   mv /etc/init.d/apache_s02 /etc/init.d/_apache_s02_`date '+%Y%m%d'`
   ```
   ```
   mv /etc/init.d/tomcat_s02 /etc/init.d/_tomcat_s02_`date '+%Y%m%d'`
   ```
   ```
   mv /etc/init.d/apache-tomcat_s02 /etc/init.d/_apache-tomcat_s02_`date '+%Y%m%d'`
   ```

6. リネーム確認
   ```
   ll /etc/init.d/ | grep -i s02
   ```

-----------------------------------------------------------------------------------
## 99. 事後作業
-----------------------------------------------------------------------------------
1. システムログ確認
   ```
   cat /var/log/messages | egrep -i "error|warn|crit|fail|fatal|panic|alert|alarm|emerg"
   ```
   
2. ログアウト
   ```
   exit
   exit
   ```
   
3. 完了連絡及び動作確認依頼<br>
   関係者に作業完了の連絡と、動作確認依頼を行う。

4. 作業エビデンス格納<br>
   作業エビデンスを案件フォルダに保存する