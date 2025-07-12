---
layout: post

tags: tips
---

/etc/sysctl.confにこれを追加

```
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.lo.disable_ipv6=1
```

で、以下のオマジナイを唱える

`sudo sysctl -p`

調べてみると、どうやらAAAAレコードを優先するけどV6をしゃべってないって話らしいんだけど、じゃあなんでAAAAレコードあるんやと思った。