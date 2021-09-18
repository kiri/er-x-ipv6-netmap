# er-x-ipv6-netmap
ルータの下にer-xをルータとして設置するが、その下にいるクライアントもipv6で通信したい。
ipv6 masquerade使えるからそれで問題ないけど興味本位でnetmapを使う。

eth0のipv6アドレスはautoconf。
switch0のipv6アドレスはfd00::/8を適当に。クライアントにはraで配布。

いずれのファイルも/config/user-data/下に保存する。
serviceとtimerは/etc/systemctl/system/へシンボリックリンク張る。
config-ipv6-netmapとupdate-ndproxyは実行権限付ける。
/config/scripts/post-config.d/にシンボリックリンク張る。

eth0がndに代理で答えるために時々proxyにアドレスを追加するためにtimer使う。
```
/config/user-data# systemctl daemon-reload
/config/user-data# systemctl enable update-ndproxy.timer
/config/user-data# systemctl start update-ndproxy.timer
```
