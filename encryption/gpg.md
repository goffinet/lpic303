# GPGを使ったファイル暗号化

公開鍵と秘密鍵を使った *公開鍵暗号方式* を使用して、ファイル受け渡し時に対象ファイルを暗号化します。

今回はgpgコマンドを使用して鍵の作成と管理を行ってファイル暗号化にチャレンジしてみます。

## LPIC 303向けのメモ
  - GPGの鍵削除に使用するコマンドは `--delete-*`、失効証明書の作成は`--gen-revoke`


## 準備

### gpgコマンド入ってる?

 以下コマンドでバージョン情報が表示されればgpgコマンドはインストールされています。

 ```sh
# gpg --version
gpg (GnuPG) 2.0.22
libgcrypt 1.5.3
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: ~/.gnupg
Supported algorithms:
Pubkey: RSA, ELG, DSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
        CAMELLIA128, CAMELLIA192, CAMELLIA256
Hash: MD5, SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB, BZIP2
```

 gpgコマンドが見つからない場合は以下コマンドでインストールします。

 ```sh
yum install -y gnupg
```

### 鍵の作成

 ```sh
gpg --gen-key
```

 1. 暗号化方式を選択します

 ```
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 
```

 1. 暗号化に使用する鍵の長さを選択します

  ```
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 
Requested keysize is 2048 bits
```

 1. 鍵の有効期限を選択します

 ```
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.
```

 1. 名前、メールアドレス、コメントを入力します

 ```
Real name: Kazunori Maehata
Email address: pachi@pachi.local
Comment: pachi test
You selected this USER-ID:
    "Kazunori Maehata (pachi test) <pachi@pachi.local>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? 
```

 1. パスフレーズを設定します
   - 空パスでも可能ですが警告メッセージが出ます

 ```
┌─────────────────────────────────────────────────────┐
│ Enter passphrase                                    │
│                                                     │
│                                                     │
│ Passphrase ________________________________________ │
│                                                     │
│       <OK>                             <Cancel>     │
└─────────────────────────────────────────────────────┘
```

 gpgが鍵を生成する時にシステム内部でランダムバイトを必要としているようです。
 キーボードで適当に文字を入力するなどして鍵の生成が完了するのを待ちましょう。

 私はfindコマンドを ルート(/)から何回か実行し、ディスクアクセスを実施してました。

 ```
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
```

 しばらくすると鍵の生成が完了したことを知らせるメッセージが表示され、gpgコマンドが終了します。

 ```
 gpg: key 92B96AEF marked as ultimately trusted
 public and secret key created and signed.

 gpg: checking the trustdb
 gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
 gpg: depth: 0  valid:   2  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 2u
 pub   2048R/92B96AEF 2016-08-13
       Key fingerprint = E698 2468 D7F0 C356 0834  1358 B6E0 8587 92B9 6AEF
       uid                  Kazunori Maehata (test) <pachi@pachi.local>
       sub   2048R/4439974A 2016-08-13
```

### 鍵の確認

公開鍵と秘密鍵でオプションが異なるので注意。

 - 公開鍵

 ```sh
gpg --list-keys
```

 ```sh
# gpg --list-keys
gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
/root/.gnupg/pubring.gpg
------------------------
pub   2048R/92B96AEF 2016-08-13
uid                  Kazunori Maehata (test) <pachi@pachi.local>
sub   2048R/4439974A 2016-08-13
```

 - 秘密鍵

 ```sh
gpg --list-secret-keys
```

 ```sh
# gpg --list-secret-keys
/root/.gnupg/secring.gpg
------------------------
sec   2048R/92B96AEF 2016-08-13
uid                  Kazunori Maehata (test) <pachi@pachi.local>
ssb   2048R/4439974A 2016-08-13
```

### 鍵の削除

公開鍵と秘密鍵でオプションが異なるので注意。

 - 公開鍵

 ```sh
gpg --delete-keys <メールアドレス>
```

 - 秘密鍵

 ```sh
gpg --delete-secret-keys <メールアドレス>
```

### 失効証明書の作成
  - パスフレーズの漏洩/忘れ、秘密鍵の漏洩などがあった場合は対象の鍵自体を無効にする
  - 失効証明書を作成しただけでは効果はない
    - 失効証明書を有効にしたい場合は、`gpg --import`で失効証明書をインポートする
  - 作成した失効証明書は外部デバイスなどで厳重に保管しましょう

 失効証明書の作成コマンドは以下の通り。

 ```sh
gpg -o <失効証明書パス> --gen-revoke <メールアドレス>
```

 先ほど作成した鍵の失効証明書を作成してみます。

 ```sh
# gpg -o ~/pachi.revoke --gen-revoke pachi@pachi.local

sec  2048R/92B96AEF 2016-08-13 Kazunori Maehata (test) <pachi@pachi.local>

Create a revocation certificate for this key? (y/N) y
Please select the reason for the revocation:
  0 = No reason specified
  1 = Key has been compromised
  2 = Key is superseded
  3 = Key is no longer used
  Q = Cancel
(Probably you want to select 1 here)
Your decision? 0
Enter an optional description; end it with an empty line:
> revoke test
> 
Reason for revocation: No reason specified
revoke test
Is this okay? (y/N) y

You need a passphrase to unlock the secret key for
user: "Kazunori Maehata (test) <pachi@pachi.local>"
2048-bit RSA key, ID 92B96AEF, created 2016-08-13

ASCII armored output forced.
Revocation certificate created.

Please move it to a medium which you can hide away; if Mallory gets
access to this certificate he can use it to make your key unusable.
It is smart to print this certificate and store it away, just in case
your media become unreadable.  But have some caution:  The print system of
your machine might store the data and make it available to others!
```

 ```sh
# ls -l ~/pachi.revoke 
-rw-r--r--. 1 root root 573 Aug 13 22:08 /root/pachi.revoke
```


## 鍵サーバへ鍵の登録

 公開鍵をクライアントが取得するためには、鍵サーバへ公開鍵を登録する必要があります。

 ```sh
gpg --send-keys <登録したい公開鍵のkey ID>
```

 ```sh
# gpg --list-keys
/root/.gnupg/pubring.gpg
------------------------
pub   2048R/92B96AEF 2016-08-13
uid                  Kazunori Maehata (test) <pachi@pachi.local>
sub   2048R/4439974A 2016-08-13

# gpg --send-keys 92B96AEF
gpg: sending key 92B96AEF to hkp server keys.gnupg.net
```

## 鍵サーバに公開鍵が登録されていることを確認する

 公開鍵が鍵サーバに登録されたことを確認します。`gpg --send-keys`から10分程度時間開けたほうがよいかもしれません。

 ```sh
gpg --keyserver hkp://keys.gnupg.net --search-keys <メールアドレス>
```

 ```sh
# gpg --keyserver hkp://keys.gnupg.net --search-keys pachi@pachi.local
gpg: searching for "pachi@pachi.local" from hkp server keys.gnupg.net
(1)     Kazunori Maehata (test) <pachi@pachi.local>
          2048 bit RSA key 92B96AEF, created: 2016-08-13
Keys 1-1 of 1 for "pachi@pachi.local".  Enter number(s), N)ext, or Q)uit > q
```

## 鍵サーバから公開鍵を取得する(クライアント)

 鍵サーバに登録した公開鍵をファイルを暗号化するマシン上に取得します。

 ```sh
gpg --keyserver hkp://keys.gnupg.net --recv-keys <登録した公開鍵のkey ID>
```

 今回は鍵を作成したマシンで実行したので既に存在するため、`not changed`と表示されます。

 ```sh
# gpg --keyserver hkp://keys.gnupg.net --recv-keys 92B96AEF
gpg: requesting key 92B96AEF from hkp server keys.gnupg.net
gpg: key 92B96AEF: "Kazunori Maehata (test) <pachi@pachi.local>" not changed
gpg: Total number processed: 1
gpg:              unchanged: 1
```
