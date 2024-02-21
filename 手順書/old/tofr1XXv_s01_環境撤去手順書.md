# 【Hybrid講座システム_本番】s01環境撤去手順書

## 作業要件
---
本手順の作業要件は以下の通り。

|#|項目|内容|
|:--|:--|:--|
|1|CAB番号|-|
|2|対象システム|Hybrid講座システム|
|3|対象サーバ|tofr901v,tofr911v|
|4|作業予定日|2023/9/19|
|5|作業フォルダ|-|

## 関連ドキュメント
---
なし

## 作業実施手順
---
本作業手順は以下の通り。(★:確認観点)

---
### 事前作業
---
#### 1. 開始連絡
**・関係者に開始連絡を行う**

#### 2. Tera Termで対象サーバへ接続
★ログが取得されているか確認する。取得できていない場合は、以下の手順でログ取得を開始する。

```
[ファイル(F)] > [ログ(L)] > ファイル名と保存先を指定して[保存(S)]を選択
ファイル名：YYYYMMDD_HHMMSS_"ホスト名".log
```
#### 3. ログイン先のサーバ名を確認
```bash
uname -n
```
#### 4. root権限へ昇格
```bash
su -
```
#### 5. rootであることを確認
```bash
id
```
#### 6. エラーログを確認する。
★問題となるエラーが出力されていない事を確認
```bash
cat /var/log/messages | egrep -i "err|warn|fail|fatal|panic|alarm" | tail -100
```

---
### 負荷分散切り離し
---

#### **1. NetScaler接続**
★NodeがPrimaryであることを確認
```
http://192.168.58.21/menu/neo
```

#### **2. 負荷分散状態確認**
Traffic Management > Load Balancing > Servers を選択
Searchより"tofr901v|tofr911v"を選択
★Enableであること　※Disableの場合切り離し不要

![Netscaler_image](image1.png)

#### **3. 負荷分散切り離し**
対象設定を選択し、右クリック。Disableを選択し切り離しを実施。
★Disableであることを確認

![Netscaler_image](image2.png)


---
### Apache Tomcat停止
---

#### **1. 停止前プロセス確認**
★Apache Tomcatが起動していることを確認
```bash
ps -ef |grep httpd |grep s01
```
```bash
ps-ef |grep tomcat |grep s01
```

#### **2. Apache Tomcat停止**
```bash
/etc/init.d/apache-tomcat_s01 stop
```

#### **3. 停止後プロセス確認**
★Apache Tomcatが停止していることを確認
```bash
ps -ef |grep httpd |grep s01
```
```bash
ps-ef |grep tomcat |grep s01
```

---
### Apache Tomcat 起動スクリプト修正
---

#### **1. ファイル確認**
★ファイルが存在すること
（apache_s01、tomcat_s01、apache-tomcat_s01）
```bash
ll /etc/init.d/*s01 
```

#### **2. 起動スクリプトリネーム**　
```bash
mv /etc/init.d/apache_s01 /etc/init.d/_apache_s01_`date '+%Y%m%d'`
```
```bash
mv /etc/init.d/tomcat_s01 /etc/init.d/_tomcat_s01_`date '+%Y%m%d'`
```
```bash
mv /etc/init.d/apache-tomcat_s01 /etc/init.d/_apache-tomcat_s01_`date '+%Y%m%d'`
```

#### **3. リネーム確認**
★ファイルがリネームされていること
（_apache_s01_20230921、_tomcat_s01_20230921、_apache-tomcat_s01_20230921）

```bash
ll /etc/init.d/*s01 
```

---
### プロセス監視無効化
---

#### **1. ProcessMonitor.txtファイルバックアップ**
★ProcessMonitor.txtが存在することを確認
```bash
ll /etc/opt/svmon/
```

★コピー後差分が無いこと
```bash
cp -p /etc/opt/svmon/ProcessMonitor.txt /etc/opt/svmon/ProcessMonitor.txt.`date '+%Y%m%d'`
```
```bash
diff /etc/opt/svmon/ProcessMonitor.txt /etc/opt/svmon/ProcessMonitor.txt.`date '+%Y%m%d'`
```

#### **2. ProcessMonitor.txt内s01を含む設定削除**　
※「プロセス監視設定削除対象」ファイル参照
```bash
vi /etc/opt/svmon/ProcessMonitor.txt
```

#### **3. 設定削除後差分確認**
★s01を含む削除分のみ差分として出力されることを確認

```bash
diff /etc/opt/svmon/ProcessMonitor.txt /etc/opt/svmon/ProcessMonitor.txt.`date '+%Y%m%d'`
```

---
### コンテンツディレクトリリネーム
---

#### **1. コンテンツディレクトリの確認**
★リネーム対象ディレクトリが存在することを確認
```bash
ll /nas/share/web/s01_*/apache/
```
```bash
ll /nas/share/web/s01_*/tomcat/
```

#### **2. リネーム**　
※「プロセス監視設定削除対象」ファイル参照
```bash
mv /nas/share/web/s01_1/apache/cgi-bin /nas/share/web/s01_1/apache/_cgi-bin_`date '+%Y%m%d'`
```
```bash
mv /nas/share/web/s01_1/apache/htdocs /nas/share/web/s01_1/apache/_htdocs_`date '+%Y%m%d'`
```
```bash
mv /nas/share/web/s01_1/apache/ssl_cgi-bin /nas/share/web/s01_1/apache/_ssl_cgi-bin_`date '+%Y%m%d'`
```
```bash
mv /nas/share/web/s01_1/apache/ssl_htdocs /nas/share/web/s01_1/apache/_ssl_htdocs_`date '+%Y%m%d'`
```
```bash
mv /nas/share/web/s01_1/tomcat/webapps /nas/share/web/s01_1/tomcat/_webapps_`date '+%Y%m%d'`
```
```bash
mv /nas/share/web/s01_2/apache/cgi-bin /nas/share/web/s01_2/apache/_cgi-bin_`date '+%Y%m%d'`
```
```bash
mv /nas/share/web/s01_2/apache/htdocs /nas/share/web/s01_2/apache/_htdocs_`date '+%Y%m%d'`
```
```bash
mv /nas/share/web/s01_2/apache/ssl_cgi-bin /nas/share/web/s01_2/apache/_ssl_cgi-bin_`date '+%Y%m%d'`
```
```bash
mv /nas/share/web/s01_2/apache/ssl_htdocs /nas/share/web/s01_2/apache/_ssl_htdocs_`date '+%Y%m%d'`
```
```bash
mv /nas/share/web/s01_2/tomcat/webapps /nas/share/web/s01_2/tomcat/_webapps_`date '+%Y%m%d'`
```


#### **3. 設定削除後差分確認**
★対象ディレクトリがリネームされていることを確認

```bash
ll /nas/share/web/s01_*/apache/
```
```bash
ll /nas/share/web/s01_*/tomcat/
```


---
### 事後作業
---
#### 1. エラーログ確認
★問題となるエラーが出力されていないこと
```bash
cat /var/log/messages | egrep -i "err|warn|fail|fatal|panic|alarm" | tail -100

```
#### 2. ログアウト
```bash
exit
exit
```
**・取得したTeraTermログを変更管理フォルダに実績として格納する。**

#### 3. 完了連絡
**・関係者に作業完了の連絡、および動作確認依頼を行う。**
