# Домашнее задание к занятию "3.6. Компьютерные сети, лекция 1"

### 1. Работа c HTTP через телнет.
- Подключитесь утилитой телнет к сайту stackoverflow.com
  `telnet stackoverflow.com 80`
- отправьте HTTP запрос
```bash
GET /questions HTTP/1.0
HOST: stackoverflow.com
[press enter]
[press enter]
```
- В ответе укажите полученный HTTP код, что он означает?

```bash
telnet stackoverflow.com 80
Trying 151.101.193.69...
Connected to stackoverflow.com.
Escape character is '^]'.
GET /questions HTTP/1.0
HOST: stackoverflow.com

HTTP/1.1 301 Moved Permanently
cache-control: no-cache, no-store, must-revalidate
location: https://stackoverflow.com/questions
...
```

Код ответа: `301 Moved Permanently` - постоянный редирект на `https`-версию страницы `https://stackoverflow.com/questions`

### 2. Повторите задание 1 в браузере, используя консоль разработчика F12.
- откройте вкладку `Network`
- отправьте запрос http://stackoverflow.com
- найдите первый ответ HTTP сервера, откройте вкладку `Headers`
- укажите в ответе полученный HTTP код.
- проверьте время загрузки страницы, какой запрос обрабатывался дольше всего?
- приложите скриншот консоли браузера в ответ.

```bash
Request URL: http://stackoverflow.com/
Request Method: GET
Status Code: 307 Internal Redirect
Referrer Policy: strict-origin-when-cross-origin
Location: https://stackoverflow.com/
Non-Authoritative-Reason: HSTS
```

Код `307 Internal Redirect` - временный редирект с сохранением исходных параметров запроса, редиректит сам хром, согласно [Strict Transport Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security).

Дольше всего грузился js-скрипт `https://winterbash2021.stackexchange.com/api/is-participating` – **999ms**.

![](Screenshot%202022-01-03%20at%2000.26.57.png)

### 3. Какой IP адрес у вас в интернете?

```bash
curl ifconfig.me
2.61.152.44
```

### 4. Какому провайдеру принадлежит ваш IP адрес? Какой автономной системе AS? Воспользуйтесь утилитой `whois`

```bash
vagrant@vagrant:~$ whois -h whois.radb.net 2.61.152.44
route:          2.61.0.0/16
descr:          Rostelecom networks
origin:         AS12389
notify:         ripe@rt.ru
mnt-by:         ROSTELECOM-MNT
created:        2018-09-28T12:53:21Z
last-modified:  2018-09-28T12:53:21Z
source:         RIPE
```

Провайдер: **Rostelecom**, автономная система: **AS12389**.

### 5. Через какие сети проходит пакет, отправленный с вашего компьютера на адрес 8.8.8.8? Через какие AS? Воспользуйтесь утилитой `traceroute`

```bash
root@vagrant:~# traceroute -An 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  10.0.2.2 [*]  0.180 ms  0.149 ms  0.132 ms
 2  10.254.210.1 [*]  124.468 ms  124.817 ms  124.796 ms
 3  163.172.215.1 [AS12876]  124.780 ms  127.367 ms  128.380 ms
 4  * * 195.154.2.126 [AS12876]  128.329 ms
 5  51.158.8.24 [AS12876]  131.765 ms  132.145 ms 51.158.8.171 [AS12876]  132.126 ms
 6  * * *
 7  108.170.241.193 [AS15169]  135.076 ms 108.170.241.161 [AS15169]  141.554 ms 108.170.241.193 [AS15169]  140.947 ms
 8  172.253.66.187 [AS15169]  141.364 ms 142.251.48.177 [AS15169]  141.512 ms 142.251.66.241 [AS15169]  141.501 ms
 9  8.8.8.8 [AS15169]  141.604 ms  143.778 ms  141.697 ms
```

### 6. Повторите задание 5 в утилите `mtr`. На каком участке наибольшая задержка - delay?

```bash
vagrant (10.0.2.15)                                                                                                                                                                  2022-01-07T09:33:41+0000
Keys:  Help   Display mode   Restart statistics   Order of fields   quit
                                                                                                                                                                     Packets               Pings
 Host                            Loss%   Snt   Last   Avg  Best  Wrst StDev
 1. AS???    10.0.2.2             0.0%   121    0.3   0.3   0.2   0.6   0.1
 2. AS???    10.254.210.1         0.0%   121  123.7 121.6 113.5 154.7   8.5
 3. AS12876  163.172.215.1        0.0%   121  120.0 122.7 114.3 154.4   8.9
 4. AS12876  195.154.2.128       25.8%   121  116.5 121.6 113.5 173.9  10.8
 5. AS12876  51.158.8.171         0.0%   121  117.1 123.9 114.5 213.3  13.4
 6. (waiting for reply)
 7. AS15169  108.170.241.193      0.0%   121  118.0 122.7 114.6 164.5   9.8
 8. AS15169  142.250.211.91       0.0%   121  117.6 121.4 114.8 156.1   6.9
 9. AS15169  8.8.8.8              0.0%   121  117.9 120.8 115.0 164.7   7.1
```

Наибольшая задержка на участке до пятого хоста:
```bash
5. AS12876  51.158.8.171         0.0%   121  117.1 123.9 114.5 213.3  13.4
```

### 7. Какие DNS сервера отвечают за доменное имя dns.google? Какие A записи? воспользуйтесь утилитой `dig`

```bash
root@vagrant:~# dig +trace dns.google

; <<>> DiG 9.16.1-Ubuntu <<>> +trace dns.google
;; global options: +cmd
.			7032	IN	NS	l.root-servers.net.
.			7032	IN	NS	k.root-servers.net.
.			7032	IN	NS	j.root-servers.net.
.			7032	IN	NS	a.root-servers.net.
.			7032	IN	NS	i.root-servers.net.
.			7032	IN	NS	h.root-servers.net.
.			7032	IN	NS	g.root-servers.net.
.			7032	IN	NS	f.root-servers.net.
.			7032	IN	NS	e.root-servers.net.
.			7032	IN	NS	d.root-servers.net.
.			7032	IN	NS	c.root-servers.net.
.			7032	IN	NS	b.root-servers.net.
.			7032	IN	NS	m.root-servers.net.
;; Received 262 bytes from 127.0.0.53#53(127.0.0.53) in 0 ms

google.			172800	IN	NS	ns-tld1.charlestonroadregistry.com.
google.			172800	IN	NS	ns-tld2.charlestonroadregistry.com.
google.			172800	IN	NS	ns-tld3.charlestonroadregistry.com.
google.			172800	IN	NS	ns-tld4.charlestonroadregistry.com.
google.			172800	IN	NS	ns-tld5.charlestonroadregistry.com.
google.			86400	IN	DS	6125 8 2 80F8B78D23107153578BAD3800E9543500474E5C30C29698B40A3DB2 3ED9DA9F
google.			86400	IN	RRSIG	DS 8 1 86400 20220120050000 20220107040000 9799 . erEwaa9Pf2kOSxUbP6GUvfnR+vfObD4Ma+uY+KmbtyzyuWtIe6SkW+d7 LKWXToDmCVc31BrZ7CQj4OX1G0KTZBfM4v0DW2THUCrSxQ9SLKyYfV6r w20Zz/JcoEVvRv6jBJusGxxgkXtlb9ZTZgXHk8p6lGVoiVybGesFWvDO FE7cIvsfjdziFkZZd/rNSTh0fVHOT1cOdnXCehTatWtrHoLpceY+WB+N 283SlR/hg9WF50waG+aeQJx0nJytdMO99xA7NesYGocdnG02tMXYAFVV HdbUtIuBgBi7K/g3yxGm4sb97FylsvHCuXCr7B0QhAn/4lv1us9AhMpt n9KEpg==
;; Received 730 bytes from 199.7.91.13#53(d.root-servers.net) in 120 ms

dns.google.		10800	IN	NS	ns2.zdns.google.
dns.google.		10800	IN	NS	ns3.zdns.google.
dns.google.		10800	IN	NS	ns4.zdns.google.
dns.google.		10800	IN	NS	ns1.zdns.google.
dns.google.		3600	IN	DS	56044 8 2 1B0A7E90AA6B1AC65AA5B573EFC44ABF6CB2559444251B997103D2E4 0C351B08
dns.google.		3600	IN	RRSIG	DS 8 2 3600 20220125203415 20220103203415 15006 google. eJso8reujxB3H789+KmpQwRoH1Gc2kzsqqtTkGGAOoxY/kET7bzViVlm FscHCNgPUZQKjCXeoUbLIFosepBE8whGyGMuR2Ls75loP06uDnJmzyGe xKMokEBkgHml66lbb60UN/MGdwqGVp6MpS+y/qFOBNR0vuWe7pJXQKxY 90c=
;; Received 506 bytes from 216.239.60.105#53(ns-tld5.charlestonroadregistry.com) in 220 ms

dns.google.		900	IN	A	8.8.4.4
dns.google.		900	IN	A	8.8.8.8
dns.google.		900	IN	RRSIG	A 8 2 900 20220126043112 20220104043112 1773 dns.google. IyUNv1qiI/N51ShtCzkgjwxktQbFzPSRls2Ts7kkNsjzmcyR4i0V5RQx NYCbm6IIwjV08+Jsz9etqiKc7QcKsWyChcdQyL1V+j6uLg7jv7lA/+9N zVIx0eok+nnoeAH6hYNr1+0imzpftl3GvOYdA0Jur2TamvJq3rL0OdGY qW8=
;; Received 241 bytes from 216.239.32.114#53(ns1.zdns.google) in 136 ms
```

`127.0.0.53` => `d.root-servers.net` => `ns-tld5.charlestonroadregistry.com` => `ns1.zdns.google`

А-записи для `dns.google`:
```bash
root@vagrant:~# dig +short dns.google A
8.8.4.4
8.8.8.8
```

### 8. Проверьте PTR записи для IP адресов из задания 7. Какое доменное имя привязано к IP? воспользуйтесь утилитой `dig`

```bash
root@vagrant:~# dig -x 8.8.4.4
...
;; ANSWER SECTION:
4.4.8.8.in-addr.arpa.	6756	IN	PTR	dns.google.
```

```bash
root@vagrant:~# dig -x 8.8.8.8

; <<>> DiG 9.16.1-Ubuntu <<>> -x 8.8.8.8
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 14755
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;8.8.8.8.in-addr.arpa.		IN	PTR

;; ANSWER SECTION:
8.8.8.8.in-addr.arpa.	6956	IN	PTR	dns.google.

;; Query time: 4 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Fri Jan 07 11:32:13 UTC 2022
;; MSG SIZE  rcvd: 73
```

К обоим адресам привязано имя `dns.google.`