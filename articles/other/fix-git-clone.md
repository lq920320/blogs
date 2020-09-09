
当出现如下错误时：
```
Receiving objects:  34% (2929/8599), 16.36 MiB | 20.00 KiB/s
error: RPC failed; curl 18 transfer closed with outstanding read data remaining
fatal: the remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
```

第一步：
    设置缓存大小： `git config --global http.postBuffer 524288000`。
    可使用 `git config --list` 查看设置是否成功。

接着出现如下错误：
```
Receiving objects:  34% (2929/8599), 13.59 MiB | 92.00 KiB/s
error: RPC failed; curl 56 OpenSSL SSL_read: Connection was reset, errno 10054
fatal: the remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
```

第二步：
	设置 ssl 验证为 "false"： `git config http.sslVerify "false"`。
	如果出现 `fatal: not in a git directory`，执行 `git init` 进行初始化，然后再次进行设置。

解决方案总结：
```
git config --global http.postBuffer 524288000

git init

git config http.sslVerify "false"
```

