Dig stands for (Domain Information Groper). Dig is a network administration command-line tool for querying Domain Name System (DNS) name servers. It is useful for verifying and troubleshooting DNS problems and also to perform DNS lookups and displays the answers that are returned from the name server that were queried. dig is part of the BIND domain name server software suite. dig command replaces older tool such as **nslookup** and the host. dig tool is available in major Linux distributions.

### Basic Syntax

```bash
dig [options] [domain] [query_type]
```

- `[options]`: Various command options (optional).
- `[domain]`: The domain you want to query.
- `[query_type]`: The type of DNS record to query (optional).

### Commonly Used Options:

#### Display only the essential information (e.g., IP addresses).
  ```bash
dig +short yahoo.com
```
```Output
74.6.143.25
74.6.231.20
98.137.11.163
74.6.143.26
74.6.231.21
98.137.11.164
  ```

#### Trace the delegation path from the root name servers down to the authoritative name servers.
  ```bash
dig +trace gcuf.edu.pk
```
```Output
; <<>> DiG 9.18.18-0ubuntu0.22.04.1-Ubuntu <<>> +trace gcuf.edu.pk
;; global options: +cmd
.                       7167    IN      NS      f.root-servers.net.
.                       7167    IN      NS      j.root-servers.net.
.                       7167    IN      NS      k.root-servers.net.
.                       7167    IN      NS      e.root-servers.net.
.                       7167    IN      NS      g.root-servers.net.
.                       7167    IN      NS      a.root-servers.net.
.                       7167    IN      NS      l.root-servers.net.
.                       7167    IN      NS      h.root-servers.net.
.                       7167    IN      NS      d.root-servers.net.
.                       7167    IN      NS      b.root-servers.net.
.                       7167    IN      NS      m.root-servers.net.
.                       7167    IN      NS      c.root-servers.net.
.                       7167    IN      NS      i.root-servers.net.
;; Received 239 bytes from 127.0.0.53#53(127.0.0.53) in 0 ms

;; communications error to 2001:500:a8::e#53: timed out
;; communications error to 2001:500:a8::e#53: timed out
;; communications error to 2001:500:a8::e#53: timed out
pk.                     172800  IN      NS      root-e.pknic.pk.
pk.                     172800  IN      NS      root-s.pknic.pk.
pk.                     172800  IN      NS      root-c1.pknic.pk.
pk.                     172800  IN      NS      root-c2.pknic.pk.
pk.                     86400   IN      NSEC    pl. NS RRSIG NSEC
pk.                     86400   IN      RRSIG   NSEC 8 1 86400 20231225050000 20231212040000 46780 . kEytekEGW0twd3hopX2CcD0kq+SODV8qIkqXMekRsrNxOJBqWNkv4TrV bEpZCKMn51t/iUyL4FwlIyI1a6UCKW1BSvLSkb2nw34HtfkjAab/m30g Olpgqsu8taAEZLayFDiaoAyu8cjGLBzynCTBDvhxhiASbZsOe2G6n6AU u30ReJ93E3XQc6ygFObd1fKIZETfGjqIpZBJPesUIaG1I3O0nBbE6iab jAsAoj4FSlANJfxhGeEniWpUnn0VP5dBZZJR+virvP0nVND0vPGEH5Es 3Au/hhFf+hzavuNaYwJPAWjo5ZlboLkZ4zQdKpMqyzFBEJv8PftIG7/3 og3ylw==
;; Received 563 bytes from 199.7.83.42#53(l.root-servers.net) in 135 ms

gcuf.edu.PK.            38400   IN      NS      ns13.securednssite.com.
gcuf.edu.PK.            38400   IN      NS      ns14.securednssite.com.
gcuf.edu.PK.            38400   IN      NS      ns15.securednssite.com.
;; Received 153 bytes from 119.81.34.90#53(root-s.pknic.pk) in 99 ms

gcuf.edu.pk.            14400   IN      A       107.6.184.66
;; Received 56 bytes from 99.83.188.187#53(ns14.securednssite.com) in 107 ms
```

#### Query recursively (used by default).
  ```bash
  dig +recurse example.com
  ```

#### Specify the DNS server to query.
  ```bash
  dig @1.1.1.1 example.com
  ```

### Query Types:

#### IPv4 address.
  ```bash
  dig example.com A
  ```

#### IPv6 address.
  ```bash
  dig example.com AAAA
  ```

#### MX Mail exchange records.
  ```bash
  dig example.com MX
  ```

#### Canonical name (alias) records.
  ```bash
  dig example.com CNAME
  ```

#### TXT Text records.
  ```bash
  dig example.com TXT
  ```

#### Name server records.
  ```bash
  dig example.com NS
  ```

#### Start of authority record.
  ```bash
  dig example.com SOA
  ```

### Examples:

- **Querying IPv4 address:**
  ```bash
  dig example.com A
  ```

- **Querying IPv6 address:**
  ```bash
  dig example.com AAAA
  ```

- **Querying mail exchange records:**
  ```bash
  dig example.com MX
  ```

- **Querying name servers:**
  ```bash
  dig example.com NS
  ```

- **Querying text records:**
  ```bash
  dig example.com TXT
  ```

These are just some basic examples, and there are many other options and record types you can use with the `dig` command. Feel free to explore the `man dig` command or the 
[official documentation](https://www.man7.org/linux/man-pages/man1/dig.1.html) for more details and options.

#### Query Domain "A" Record
```shell
dig yahoo.com
```
```Ouput
; <<>> DiG 9.18.18-0ubuntu0.22.04.1-Ubuntu <<>> yahoo.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15198
;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;yahoo.com.                     IN      A

;; ANSWER SECTION:
yahoo.com.              622     IN      A       98.137.11.163
yahoo.com.              622     IN      A       74.6.231.21
yahoo.com.              622     IN      A       98.137.11.164
yahoo.com.              622     IN      A       74.6.231.20
yahoo.com.              622     IN      A       74.6.143.26
yahoo.com.              622     IN      A       74.6.143.25

;; Query time: 0 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Tue Dec 12 14:29:29 PKT 2023
;; MSG SIZE  rcvd: 134
```

Above command causes dig to look up the "A" record for the domain name yahoo.com. Dig command reads the /etc/resolv.conf file and querying the DNS servers listed there. The response from the DNS server is what dig displays.

#### Understand the Output
Lines beginning with ; are comments not part of the information.
The first line tell us the version of dig (9.8.2) command.
Next, dig shows the header of the response it received from the DNS server
Next comes the question section, which simply tells us the query, which in this case is a query for the "A" record of yahoo.com. The IN means this is an Internet lookup (in the Internet class).

The answer section tells us that yahoo.com has a list of IP addresses.
Lastly, there are some stats about the query. You can turn off these stats using the +nostats option.

#### Query Domain "A" Record with +nostats
```shell
dig yahoo.com +nostats
```
```Output
; <<>> DiG 9.18.18-0ubuntu0.22.04.1-Ubuntu <<>> yahoo.com +nostats
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37733
;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;yahoo.com.                     IN      A

;; ANSWER SECTION:
yahoo.com.              629     IN      A       74.6.143.25
yahoo.com.              629     IN      A       98.137.11.164
yahoo.com.              629     IN      A       74.6.231.21
yahoo.com.              629     IN      A       74.6.231.20
yahoo.com.              629     IN      A       74.6.143.26
yahoo.com.              629     IN      A       98.137.11.163
```
#### Query MX Records
```shell
dig yahoo.com mx
```
```Output
; <<>> DiG 9.18.18-0ubuntu0.22.04.1-Ubuntu <<>> yahoo.com mx
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 21673
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;yahoo.com.                     IN      MX

;; ANSWER SECTION:
yahoo.com.              713     IN      MX      1 mta6.am0.yahoodns.net.
yahoo.com.              713     IN      MX      1 mta7.am0.yahoodns.net.
yahoo.com.              713     IN      MX      1 mta5.am0.yahoodns.net.

;; Query time: 43 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Tue Dec 12 14:36:17 PKT 2023
;; MSG SIZE  rcvd: 117
```
#### Query only Answer Section
```shell
dig yahoo.com +nocomments +noquestion +noauthority +noadditional +nostats
```
```Output
; <<>> DiG 9.18.18-0ubuntu0.22.04.1-Ubuntu <<>> yahoo.com +nocomments +noquestion +noauthority +noadditional +nostats
;; global options: +cmd
yahoo.com.              1783    IN      A       74.6.143.25
yahoo.com.              1783    IN      A       74.6.231.20
yahoo.com.              1783    IN      A       74.6.231.21
yahoo.com.              1783    IN      A       98.137.11.163
yahoo.com.              1783    IN      A       98.137.11.164
yahoo.com.              1783    IN      A       74.6.143.26
```
#### Reverse IP Look
```shell
$ dig -x 74.6.231.20 +short
```
```Output
media-router-fp73.prod.media.vip.ne1.yahoo.com.
```
