# er-x-ipv6-netmap
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
config-ipv6-netmapとip6neighproxyは実行権限付ける。
```
# cd /config/user-data/
# chmod +x config-ipv6-netmap
# chmod +x ip6neighproxy
```
serviceは/etc/systemctl/system/へシンボリックリンク張る。
```
# ln -s /config/user-data/ip6neighproxy.service /etc/systemd/system/
# systemctl daemon-reload
# systemctl enable ip6neighproxy.service
```

/config/scripts/post-config.d/にシンボリックリンク張る。
```
# ln -s /config/user-data/config-ipv6-netmap /config/scripts/post-config.d/
```

うまくうごかなかったら、もろもろとめて、ipv6 masqueradeに戻せば良い。
```
ip6tables -t nat -A POSTROUTING -o eth0 -s fd8a:bcde:1234::/64 -j MASQUERADE
```
