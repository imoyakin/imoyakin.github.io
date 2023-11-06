---
title: "使用DockerDev开发"
date: 2022-12-27T08:56:33+08:00
draft: true
---
  关于公共DNS服务器，之前写过一篇文章，记录了下主要方便以后自己查找，不过那篇只有IPV4与IPV6，没有更加安全的DoT/DoH这类的加密DNS，DNS over TLS(DoT)和DNS over Https(DoH)是新一代的DNS，它们通过TLS或者https协议加密了你和DNS服务器之间的连接，从而使第三方无法得知你的域名申请请求，也无法污染DNS结果，所以整理了这篇文章。  

  注意：1、加密DNS传输过程中第三方无法查看、无法污染DNS结果不代理DNS服务器本身不作恶；2、在国内使用国外加密DNS有可能无法上网，能使用8.8.8.8上网不代表DoH(https://8.8.8.8/dns-query)就可以访问，经测试大多数国外DoH在国内都被屏蔽，能使用的解析速度也很慢。


阿里公共DNS:
IPv4：223.5.5.5
IPv4：223.6.6.6
IPv6：2400:3200::1
IPV6: 2400:3200:baba::1
DoH：https://223.5.5.5/dns-query
DoH：https://223.6.6.6/dns-query
DoH：https://dns.alidns.com/dns-query
DoT：dns.alidns.com
DoT：223.5.5.5
DoT：223.6.6.6

腾讯公共DNS(DNSPod):
IPv4：119.29.29.29
IPV6:2402:4e00::
DoH：https://doh.pub/dns-query
DoH：https://1.12.12.12/dns-query
DoH：https://120.53.53.53/dns-query
DoH：https://sm2.doh.pub/dns-query (国密)
DoT：dot.pub
DoT：1.12.12.12
DoT：120.53.53.53

百度公共DNS：
IPV4: 180.76.76.76
IPV6: 2400:da00::6666

360公共DNS:
电信/铁通/移动IPv4: 101.226.4.6
电信/铁通/移动IPv4: 218.30.118.6
联通IPv4: 123.125.81.6
联通IPv4: 140.207.198.6
DoH: https://doh.360.cn
DoT: dot.360.cn

CNNIC DNS:
IPV4: 1.2.4.8
IPV4: 210.2.4.8
IPV6: 2001:dc7:1000::1


台湾Quad 101(twnic):
IPV4: 101.101.101.101

IPV4: 101.102.103.104

IPV6: 2001:de4::101

IPV6: 2001:de4::102

DoH: https://dns.twnic.tw/dns-query


Google公共DNS
IPv4：8.8.8.8
IPv4：8.8.4.4
IPV6: 2001:4860:4860::8888
IPV6: 2001:4860:4860::8844)
DoH地址：https://dns.google/dns-query
DoH地址：https://8.8.8.8/dns-query
DoH地址：https://8.8.4.4/dns-query
DoT:dns.google
DoT:8.8.8.8
DoT:8.8.4.4

Cloudflare公共DNS:
IPV4: 1.1.1.1
IPV4: 1.0.0.1
IPV6: 2606:4700:4700::1111
IPV6: 2606:4700:4700::1001
DoH: https://1.1.1.1/dns-query
DoH: https://1.0.0.1/dns-query
DoH: https://cloudflare-dns.com/dns-query
DoT：1.1.1.1
DoT：1.0.0.1

DoT：1dot1dot1dot1.cloudflare-dns.com

DoT：cloudflare-dns.com

DoT：one.one.one.one


DNS.SB公共DNS:
IPv4: 185.222.222.222
IPV4: 45.11.45.11
IPV6: 2a09::
IPV6: 2a11::
DoH: https://doh.sb/dns-query (或者：https://doh.dns.sb/dns-query两个指向一样，只是域名不一样)

DoH: https://45.11.45.11/dns-query
DoH: https://185.222.222.222/dns-query
DoT: dot.sb
DoT: 185.222.222.222
DoT: 45.11.45.11

DoH香港节点: https://hk-hkg.doh.sb/dns-query  (更多分节点：https://dns.sb/doh/，可以根据自己的位置选择速度快一些的节点。)


AdGuard 公共DNS:
IPV4默认过滤广告: 94.140.14.14
IPV4默认过滤广告：94.140.15.15
IPV4无过滤: 94.140.14.140
IPV4无过滤:94.140.14.141
IPV4家庭保护(过滤广告与成人内容)：94.140.14.15
IPV4家庭保护(过滤广告与成人内容)：94.140.15.16
DoH默认过滤广告：https://dns.adguard.com/dns-query
DoH家庭保护(过滤广告与成人内容)：https://dns-family.adguard.com/dns-query
DoH非过滤：https://dns-unfiltered.adguard.com/dns-query
DoT默认过滤广告：dns.adguard.com
DoT家庭保护(过滤广告与成人内容)：dns-family.adguard.com
DoT非过滤：dns-unfiltered.adguard.com

OpenDNS(Cisco):
IPv4：208.67.222.222
IPv4：208.67.220.220
IPV4 Family: 208.67.222.123
IPV4 Family: 208.67.220.123
IPV6: 2620:0:ccc::2
IPV6: 2620:0:ccd::2
DoH：https://doh.opendns.com/dns-query
DoH Family：https://doh.familyshield.opendns.com/dns-query
DoT: dns.umbrella.com

IBM Quad9:
IPv4默认安全：9.9.9.9
IPv4默认安全：149.112.112.112
IPv6默认安全: 2620:fe::fe
IPv6默认安全: 2620:fe::fe:9
DoH默认：https://dns.quad9.net/dns-query
DoT默认：dns.quad9.net
IPv4无过滤：9.9.9.10
IPv4无过滤：149.112.112.10
IPv6无过滤: 2620:fe::10
IPv6无过滤: 2620:fe::fe:10
DoH无过滤：https://dns10.quad9.net/dns-query
DoT无过滤：dns10.quad9.net
IPv4 ECS保护：9.9.9.11
IPv4 ECS保护：149.112.112.11
IPv6 ECS保护: 2620:fe::11
IPv6 ECS保护: 2620:fe::fe:11
DoH ECS保护：https://dns11.quad9.net/dns-query
DoT ECS保护：dns11.quad9.net

> https://www.iewb.net/content/html/dns.html