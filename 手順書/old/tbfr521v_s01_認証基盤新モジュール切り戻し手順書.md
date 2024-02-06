# tbfr521v_s03_認証基盤新モジュール切り戻し手順書.md

------------------------------------------------------------
## 作業要件
------------------------------------------------------------
|#|項目|内容|
|:--|:--|:--|
|1|対象システム|Hybrid講座システム_BAE|
|2|対象ホスト|tbfr521v|
|3|作業日|2024/1/10-11|
|4|作業者|照井|
|5|確認者|下津・松本|

------------------------------------------------------------
## 1. 事前作業
------------------------------------------------------------
1. 作業開始連絡</br>
   関係者に作業開始の連絡を行う

2. 動作確認</br>
以下のページにアクセスできることを確認
- s03
   - https://bae-st-staging.benesse.ne.jp


3. Tera Termにログインする

4. ログファイルを作成する

5. root権限へ昇格する
```
su -
```

6. ログイン先のサーバ名を確認する
```
uname -n
```

7. root権限であることを確認する
```
id
```

8. システムログにエラーメッセージが出ていないことを確認する
```
cat /var/log/messages | egrep -i "err|warn|fail|fatal|panic|alarm" | tail -100
```

-----------------------------------------------------------------------------------
## 2. ACL設定戻し
-----------------------------------------------------------------------------------
1. ディレクトリ移動  
★ディレクトリが移動されたことと対象ディレクトリが存在することを確認
```
cd /nas/share/web/s03_1/apache/fnssso/ ;pwd ;ll ;ll bk/
```

2. 変更前差分確認  
★変更分のみ差分が出ることを確認
```
diff -r ssl_acldir bk/ssl_acldir_20231025
```
3. 切り戻し前バックアップ  
★リネームされたことを確認
```
mv ssl_acldir ./bk/ssl_acldir_kirimodoshi_$(date +%Y%m%d)
```
```
ll ;ll bk/
```

4. 切り戻し  
```
mv bk/ssl_acldir_20231025 ssl_acldir
```
```
ll ;ll bk/
```

5. 差分確認  
★「require bnapp-group」のみ差分として出ていること
```
diff -r ssl_acldir bk/ssl_acldir_kirimodoshi_$(date +%Y%m%d)
```

6. 構文チェック  
★ Syntax OK が表示されることを確認する
```
/opt/s03/apache/bin/httpd -t -f /opt/s03/apache/conf/httpd.conf
```

------------------------------------------------------------
## 3. apache環境切り戻し
------------------------------------------------------------
1. apache-tomcatのプロセスが起動していることを確認する
```
ps -ef | grep httpd | grep s03
ps -ef | grep tomcat | grep s03
```

2. apache-tomcatを停止する
```
/etc/init.d/apache-tomcat_s03 stop
```

3. プロセスが停止していることを確認する
```
ps -ef | grep httpd | grep s03
ps -ef | grep tomcat | grep s03
```

4. 切り戻し前のapache環境をバックアップする
```
mv /opt/s03/apache /opt/s03/apache_kirimodoshi_$(date +%Y%m%d)
ls -l /opt/s03/
```

5. バックアップからファイルを復元する
```
mv /opt/s03/apache_OldCert_20231025 /opt/s03/apache
ls -l /opt/s03/
```

6. 構文チェックを行い問題ないことを確認する（「Syntax OK」のみ表示されること）
```
/opt/s03/apache/bin/httpd -t -f /opt/s03/apache/conf/httpd.con
```

7. 戻し確認  
★旧認証基盤設定になっていることを確認
```
cat /opt/s03/apache/conf/global/global-ninsyo.conf
```
```
cat /opt/s03/apache/conf/virtual/s03_81_virtual-ninsyo.conf
```
```
cat /opt/s03/apache/conf/virtual/s03_80_virtual-ninsyo.conf
```

8. apache起動スクリプト戻し  
★ファイルが存在していること
```
ll /etc/init.d/apache_s03*
```

```
mv /etc/init.d/apache_s03 /etc/init.d/apache_s03_kirimodoshi.$(date +%Y%m%d)
```  

★切り戻しファイルができていること
```
ll /etc/init.d/apache_s03*
```

```
mv /etc/init.d/apache_s03.20231025 /etc/init.d/apache_s03
```  
★apache_s03ファイルが存在すること
```
ll /etc/init.d/apache_s03*
```  
★セマフォ削除処理のみ差分として出力されていること
```
diff /etc/init.d/apache_s03 /etc/init.d/apache_s03.$(date +%Y%m%d)
```

9. セマフォIDファイルの保存先ディレクトリと、対象ファイルを確認する
```
ls -ld /var/log/s03/apache/sem
ls -l /var/log/s03/apache/sem
```

10. 対象ファイルを削除する
```
rm /var/log/s03/apache/sem/sem*.id
ls -l /var/log/s03/apache/sem
```

11. 保存先ディレクトリを削除する
```
rmdir /var/log/s03/apache/sem
ls -ld /var/log/s03/apache/sem
```


12.  apache-tomcatを起動する
```
/etc/init.d/apache-tomcat_s03 start
```

13. プロセスが起動していることと、起動時刻がコマンドを実行した時刻であることを確認する
```
ps -ef | grep httpd | grep s03
ps -ef | grep tomcat | grep s03
```

14. 再起動後、apache-tomcatのログを確認する
```
cat /var/log/s03/apache/logs/error_log.`date '+%Y%m%d'`
cat /var/log/s03/tomcat/logs/catalina.out.`date '+%Y%m%d'` | grep -i "startup"
```

------------------------------------------------------------
## 99. 事後作業
------------------------------------------------------------
1. システムログにエラーメッセージが出ていないことを確認する
```
cat /var/log/messages | egrep -i "err|warn|fail|fatal|panic|alarm" | tail -100
```

2. Tera Termからログアウトする
```
exit
exit
```

3. 動作確認</br>
以下のページにアクセスできることを確認
- s03
   - https://bae-st-staging.benesse.ne.jp

4. 作業完了連絡及び動作確認依頼</br>
依頼者に変更完了の連絡とサービス動作確認依頼を行う
