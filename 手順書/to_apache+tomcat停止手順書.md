# to_apache+tomcat停止手順書.md

-----------------------------------------------------------------------------------
## 作業要件
-----------------------------------------------------------------------------------
|#|項目|内容|
|:--|:--|:--|
|1|対象システム|Hybrid講座システム|
|2|対象ホスト|tofr501v,tofr502v,tofr511v,tofr521v,tofr531v,tofr901v,tofr902v|
|3|作業予定日|2024/04/11|

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
1. NetScaler接続<br>
   ★NodeがPrimaryであることを確認
   - http://192.168.58.20/menu/neo　※セカンダリ
   - http://192.168.58.21/menu/neo
  
2. 負荷分散状態確認<br>
   Traffic Management > Load Balancing > Servers を選択<br>
   Searchより対象ホストを選択<br>
   ★「to_apa-tom停止対象調査.xlsx」を参考に対象ホストを選択<br>
   ★State欄が「Enabled」であること　※Disabledの場合切り離し不要

3. 負荷分散切り離し<br>
   対象ホストを選択し、右クリック。<br>
   Disableを選択し切り離しを実施。<br>
   ★Disableであることを確認

-----------------------------------------------------------------------------------
## 4. プロセス監視無効化
-----------------------------------------------------------------------------------
※対象ホスト：「to_apa-tom停止対象調査.xlsx」を参照する<br>
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

3. ProcessMonitor.txt内s0Xを含む設定削除<br>
   ★「プロセス監視設定削除対象_to_Hybrid講座システム.xlsx」ファイルを参照に削除する
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
※対象ホスト：「to_apa-tom停止対象調査.xlsx」を参照にs0Xを修正して実行<br>

1. 停止前プロセス確認<br>
   ★Apache Tomcatが起動していることを確認
   ```
   ps -ef | grep httpd | grep s0X
   ```
   ```
   ps -ef | grep tomcat | grep s0X
   ```

2. Apache Tomcat停止
   ```
   /etc/init.d/apache-tomcat_s0X stop
   ```

   ※tofr501v_s08、tofr502v_s08の場合
   ```
   /etc/init.d/apache_s0X stop
   ```

   ※tofr501v_s04、tofr511v_s06の場合
   ```
   /etc/init.d/tomcat_s0X stop
   ```

3. 停止後プロセス確認<br>
   ★Apache Tomcatが停止していることを確認
   ```
   ps -ef | grep httpd | grep s0X
   ```
   ```
   ps -ef | grep tomcat | grep s0X
   ```

-----------------------------------------------------------------------------------
## 6. Apache Tomcat 起動スクリプト修正
-----------------------------------------------------------------------------------
※対象ホスト：「to_apa-tom停止対象調査.xlsx」を参照にs0Xを修正して実行<br>
1. ファイル確認<br>
   ★ファイルが存在すること
   - apache_s0X
   - tomcat_s0X
   - apache-tomcat_s0X
   ```
   ll /etc/init.d/ | grep -i s0X
   ```

2. 起動スクリプトリネーム
   ```
   mv /etc/init.d/apache_s0X /etc/init.d/_apache_s0X_`date '+%Y%m%d'`
   ```
   ```
   mv /etc/init.d/tomcat_s0X /etc/init.d/_tomcat_s0X_`date '+%Y%m%d'`
   ```
   ```
   mv /etc/init.d/apache-tomcat_s0X /etc/init.d/_apache-tomcat_s0X_`date '+%Y%m%d'`
   ```

3. リネーム確認
   ```
   ll /etc/init.d/ | grep -i s0X
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