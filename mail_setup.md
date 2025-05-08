# メール送信設定
## 0.フォルダ構成 (作業後の状態)
~ (/home/radxa/)
　┗ .msmtprc
　┗ Desktop
　　┗ send_test_mail.py
　　┗ alert_sample01.mp4

## 1．msmtp（メール送信ツール）をインストール
```bash
sudo apt update
sudo apt install -y msmtp msmtp-mta
```
<br>


## 2．メール設定を作成
```bash
# 設定ファイルを新規作成
sudo nano ~/.msmtprc
```

```bash
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        ~/.msmtp.log

account gmail
host smtp.gmail.com
port 587
from system@asx.bz
user system@asx.bz
password ybsoskgwayygdprk

account default : gmail
```

```bash
# パーミッション変更
sudo chmod 600 ~/.msmtprc
```
<br>

## 3．Pythonスクリプトでメール送信処理を作成
```bash
sudo nano ~/Desktop/send_test_mail.py
```

```python
from email.message import EmailMessage
import subprocess
import mimetypes
import os

# メール基本情報（DBで管理しカスタマイズ可能にする）
from_addr = '人追跡システムサポート <system@asx.bz>'

# 宛先（DBで管理しカスタマイズ可能にする）
to_addrs = ["shotta@asx.co.jp", "rtanabe@asx.co.jp", "tikejiri@asx.co.jp"]
# タイトル（DBで管理しカスタマイズ可能にする）
subject = "人追跡サーバー警告エリアの侵入通知メッセージ"
# 本文（DBで管理しカスタマイズ可能にする）
body = """カメラ検出により指定エリアに認証されていない人が検出されました。
内容を確認し、状態を確認してください。

また、このメッセージは自動で送信されており返信はできません。
ご不明な点等ございましたらお問い合わせください。
また身に覚えのない場合は、本メールを破棄いただきますようお願いいたします。
"""

# 添付ファイルパス（例）
video_path = "/home/radxa/Desktop/alert_sample01.mp4"

# メール構築
msg = EmailMessage()
msg["From"] = from_addr
msg["To"] = ", ".join(to_addrs)
msg["Subject"] = subject
msg.set_content(body)

# 添付ファイル
if os.path.exists(video_path):
    mime_type, _ = mimetypes.guess_type(video_path)
    maintype, subtype = mime_type.split("/", 1)
    with open(video_path, "rb") as f:
        msg.add_attachment(f.read(),
                           maintype=maintype,
                           subtype=subtype,
                           filename=os.path.basename(video_path))
else:
    print("動画ファイルが見つかりません。添付なしで送信します。")

# subprocessで送信（msmtp）
proc = subprocess.run(
    ["msmtp", *to_addrs],
    input=msg.as_bytes(),
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE
)

if proc.returncode == 0:
    print("送信成功")
else:
    print("送信失敗:", proc.stderr.decode())
```
<br>

## 4．メール送信
```bash
# Pythonファイルを実行
cd ~/Desktop
python3 send_test_mail.py
```
