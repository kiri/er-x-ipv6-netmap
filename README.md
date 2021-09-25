# Edgerouter Xでipv6 netmapを使う
ルータの下にer-xをルータとして設置するが、その下にいるクライアントもipv6で通信したい。
ipv6 masquerade使えるからそれで問題ないけど興味本位でnetmapを使う。
ex-xが代理で応答する必要があるが、ndppdがnetmapでも使えるといいけど(dev版はrewriteする機能あるらしい）、neigh proxyに追加すればよいのだろうということで、conntrackの内容をみてスクリプトで追加・削除するようにしてみる。

eth0のipv6アドレスはautoconf。
switch0のipv6アドレスはfd00::/8を適当に。クライアントにはraで配布。
```
set interfaces ethernet eth0 ipv6 address autoconf
set interfaces switch switch0 ipv6 address eui64 'fd8a:bcde:1234::/64'
set interfaces switch switch0 ipv6 dup-addr-detect-transmits 1
set interfaces switch switch0 ipv6 router-advert cur-hop-limit 64
set interfaces switch switch0 ipv6 router-advert link-mtu 0
set interfaces switch switch0 ipv6 router-advert managed-flag true
set interfaces switch switch0 ipv6 router-advert max-interval 600
set interfaces switch switch0 ipv6 router-advert other-config-flag false
set interfaces switch switch0 ipv6 router-advert prefix '::/64' autonomous-flag true
set interfaces switch switch0 ipv6 router-advert prefix '::/64' on-link-flag true
set interfaces switch switch0 ipv6 router-advert prefix '::/64' valid-lifetime 2592000
set interfaces switch switch0 ipv6 router-advert reachable-time 0
set interfaces switch switch0 ipv6 router-advert retrans-timer 0
set interfaces switch switch0 ipv6 router-advert send-advert truekgg
```

いずれのファイルも/config/user-data/下に保存する。
ipv6netmapは実行権限を付ける。
```
# cd /config/user-data/
# chmod +x ipv6netmap
```
serviceは/etc/systemctl/system/へシンボリックリンク張る。
```
# ln -s /config/user-data/ipv6netmap.service /etc/systemd/system/
# systemctl daemon-reload
# systemctl enable ipv6netmap.service
# systemctl start ipv6netmap.service
```

うまくうごかなかったら、もろもろとめて、ipv6 masqueradeに戻せば良い。
systemdで起動するように書いた。
```
# systemctl stop ipv6netmap.service
# systemctl disable ipv6netmap.service

# cd /config/user-data/
# chmod +x ipv6nat
# ln -s /config/user-data/ipv6nat.service /etc/systemd/system/
# systemctl daemon-reload
# systemctl enable ipv6nat.service
# systemctl start ipv6nat.service
```
