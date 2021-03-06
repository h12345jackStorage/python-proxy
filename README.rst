python-proxy
============

|made-with-python| |PyPI-version| |Hit-Count|

.. |made-with-python| image:: https://img.shields.io/badge/Made%20with-Python-1f425f.svg
   :target: https://www.python.org/
.. |PyPI-version| image:: https://badge.fury.io/py/pproxy.svg
   :target: https://pypi.python.org/pypi/pproxy/
.. |Hit-Count| image:: http://hits.dwyl.io/qwj/python-proxy.svg
   :target: https://pypi.python.org/pypi/pproxy/

HTTP/Socks4/Socks5/Shadowsocks/ShadowsocksR/Redirect asynchronous tunnel proxy implemented in Python3 asyncio.

QuickStart
----------

.. code:: rst

  $ pip3 install pproxy
  Successfully installed pproxy-1.6.4
  $ pproxy
  Serving on :8080 by http,socks4,socks5
  ^C
  $ pproxy -i ss://chacha20:abc@:8080
  Serving on :8080 by ss (chacha20-py)

Optional: (better performance with C ciphers)

.. code:: rst

  $ pip3 install pproxy[accelerated]
  Successfully installed pycryptodome-3.6.4

Apply OS system-wide proxy: (MacOS, Windows)

.. code:: rst

  $ pproxy -r ss://chacha20:abc@server_ip:8080 --sys -vv
  Serving on :8080 by http,socks4,socks5 
  System proxy setting -> socks5 localhost:8080
  socks5 ::1:57345 -> ss server_ip:8080 -> slack.com:443
  socks5 ::1:57345 -> ss server_ip:8080 -> www.google.com:443
  ..... (all local traffic log) ......

Apply CLI proxy: (MacOS, Linux)

.. code:: rst

  $ export http_proxy=http://localhost:8080 
  $ export https_proxy=http://localhost:8080 

Features
--------

- Single-thread asynchronous IO with high availability and scalability.
- Lightweight (~500 lines) and powerful by leveraging python builtin *asyncio* library.
- No additional library is required. All codes are in pure Python.
- Auto-detect incoming traffic.
- Tunnel by remote proxy servers.
- Tunnel and relay with several layers.
- Unix domain socket.
- Basic authentication for all protocols.
- Regex pattern file to route/block by hostname.
- SSL/TLS client/server support.
- Built-in encryption ciphers. (chacha20, aes-256-cfb, etc)
- Shadowsocks OTA (One-Time-Auth_).
- SSR plugins. (http_simple, verify_simple, tls1.2_ticket_auth, etc)
- Statistics by bandwidth and traffic.
- PAC support for javascript configuration.
- Iptables NAT redirect packet tunnel.
- PyPy3 support with JIT speedup.
- System proxy auto-setting support.

.. _One-Time-Auth: https://shadowsocks.org/en/spec/one-time-auth.html

Python3
-------

*Python 3.5* added new syntax **async def** and **await** to make asyncio programming easier. *Python 3.6* added new syntax **formatted string literals**. This tool was to demonstrate these new syntax, so the minimal Python requirement was **3.6**.

From **pproxy** 1.1.0, the minimal Python requirement is **3.3**, since old python versions are still widely used and PyPy3 only has 3.3 support currently. *Python 2* will not be supported in the future.

From **proxy** 1.3.0, the minimal Python requirement is **3.6**, since **Python 3.7** make the **async**/**await**/ reserved words, we cannot make pproxy compatible with older versions.

PyPy3
-----

.. code:: rst

  $ pypy3 -m ensurepip
  $ pypy3 -m pip install asyncio pproxy

Requirement
-----------

pycryptodome_ is an optional library to enable faster (C version) cipher. **pproxy** has many built-in pure python ciphers. They are lightweight and stable, but slower than C ciphers. After speedup with PyPy_, the pure python ciphers can get similar performance as C version. If the performance is important and don't have PyPy_, install pycryptodome_ instead.

These are some performance benchmarks between Python and C ciphers (dataset: 8M):

+---------------------+----------------+
| chacha20-c          | 0.64 secs      |
+---------------------+----------------+
| chacha20-py (pypy3) | 1.32 secs      |
+---------------------+----------------+
| chacha20-py         | 48.86 secs     |
+---------------------+----------------+

.. _pycryptodome: https://pycryptodome.readthedocs.io/en/latest/src/introduction.html
.. _PyPy: http://pypy.org

Usage
-----

.. code:: rst

  $ pproxy -h
  usage: pproxy [-h] [-i LISTEN] [-r RSERVER] [-b BLOCK] [-a ALIVED] [-v]
                [--ssl SSLFILE] [--pac PAC] [--get GETS] [--sys]
                [--test TESTURL] [--version]
  
  Proxy server that can tunnel among remote servers by regex rules. Supported
  protocols: http,socks4,socks5,shadowsocks,shadowsocksr,redirect
  
  optional arguments:
    -h, --help     show this help message and exit
    -i LISTEN      proxy server setting uri (default: http+socks://:8080/)
    -r RSERVER     remote server setting uri (default: direct)
    -b BLOCK       block regex rules
    -a ALIVED      interval to check remote alive (default: no check)
    -v             print verbose output
    --ssl SSLFILE  certfile[,keyfile] if server listen in ssl mode
    --pac PAC      http PAC path
    --get GETS     http custom {path,file}
    --sys          change system proxy setting (mac, windows)
    --test TEST    test this url for all remote proxies and exit
    --version      show program's version number and exit
  
  Online help: <https://github.com/qwj/python-proxy>

URI Syntax
----------

.. code:: rst

  {scheme}://[{cipher}@]{netloc}/[@{localbind}][,{plugins}][?{rules}][#{auth}]

- scheme

  - Currently supported scheme: http, socks, ss, ssl, secure. You can use + to link multiple protocols together.

    +--------+-----------------------------+
    | http   | http protocol               |
    +--------+-----------------------------+
    | socks4 | socks4 protocol             |
    +--------+-----------------------------+
    | socks5 | socks5 protocol             |
    +--------+-----------------------------+
    | ss     | shadowsocks protocol        |
    +--------+-----------------------------+
    | ssr    | shadowsocksr (SSR) protocol |
    +--------+-----------------------------+
    | redir  | redirect (iptables nat)     |
    +--------+-----------------------------+
    | ssl    | unsecured ssl/tls (no cert) |
    +--------+-----------------------------+
    | secure | secured ssl/tls (cert)      |
    +--------+-----------------------------+
    | direct | direct connection           |
    +--------+-----------------------------+

  - Valid schemes: http://, http+socks4+socks5://, http+ssl://, ss+secure://, http+socks5+ss://

  - Invalid schemes: ssl://, secure://

- cipher

  - Cipher's format: "cipher_name:cipher_key". Cipher can be base64-encoded. So cipher string with "YWVzLTEyOC1nY206dGVzdA==" is equal to "aes-128-gcm:test".

  - Full cipher support list:

    +-----------------+------------+-----------+-------------+
    | Cipher          | Key Length | IV Length | Score (0-5) |
    +=================+============+===========+=============+
    | table-py        | any        | 0         | 0 (lowest)  |
    +-----------------+------------+-----------+-------------+
    | rc4             | 16         | 0         | 0 (lowest)  |
    +-----------------+------------+-----------+-------------+
    | rc4-md5         | 16         | 16        | 0.5         |
    +-----------------+------------+-----------+-------------+ 
    | chacha20        | 32         | 8         | 5 (highest) |
    +-----------------+------------+-----------+-------------+
    | chacha20-ietf   | 32         | 12        | 5           |
    +-----------------+------------+-----------+-------------+
    | chacha20-ietf-  |            |           |             |
    | poly1305-py     | 32         | 32        | AEAD        |
    +-----------------+------------+-----------+-------------+
    | salsa20         | 32         | 8         | 4.5         |
    +-----------------+------------+-----------+-------------+
    | aes-128-cfb     | 16         | 16        | 3           |
    |                 |            |           |             |
    | aes-128-cfb8    |            |           |             |
    |                 |            |           |             |
    | aes-128-cfb1-py |            |           | slow        |
    +-----------------+------------+-----------+-------------+
    | aes-192-cfb     | 24         | 16        | 3.5         |
    |                 |            |           |             |
    | aes-192-cfb8    |            |           |             |
    |                 |            |           |             |
    | aes-192-cfb1-py |            |           | slow        |
    +-----------------+------------+-----------+-------------+
    | aes-256-cfb     | 32         | 16        | 4.5         |
    |                 |            |           |             |
    | aes-256-ctr     |            |           |             |
    |                 |            |           |             |
    | aes-256-ofb     |            |           |             |
    |                 |            |           |             |
    | aes-256-cfb8    |            |           |             |
    |                 |            |           |             |
    | aes-256-cfb1-py |            |           | slow        |
    +-----------------+------------+-----------+-------------+
    | aes-256-gcm     | 32         | 32        | AEAD        |
    |                 |            |           |             |
    | aes-192-gcm     | 24         | 24        | AEAD        |
    |                 |            |           |             |
    | aes-128-gcm     | 16         | 16        | AEAD        |
    +-----------------+------------+-----------+-------------+
    | camellia-256-cfb| 32         | 16        | 4           |
    |                 |            |           |             |
    | camellia-192-cfb| 24         | 16        | 4           |
    |                 |            |           |             |
    | camellia-128-cfb| 16         | 16        | 4           |
    +-----------------+------------+-----------+-------------+
    | bf-cfb          | 16         | 8         | 1           |
    +-----------------+------------+-----------+-------------+
    | cast5-cfb       | 16         | 8         | 2.5         |
    +-----------------+------------+-----------+-------------+
    | des-cfb         | 8          | 8         | 1.5         |
    +-----------------+------------+-----------+-------------+
    | rc2-cfb-py      | 16         | 8         | 2           |
    +-----------------+------------+-----------+-------------+
    | idea-cfb-py     | 16         | 8         | 2.5         |
    +-----------------+------------+-----------+-------------+
    | seed-cfb-py     | 16         | 16        | 2           |
    +-----------------+------------+-----------+-------------+

  - *pproxy* ciphers have pure python implementations. Program will switch to C cipher if there is C implementation available within pycryptodome_. Otherwise, use pure python cipher.

  - AEAD ciphers use additional payload after each packet. The underlying protocol is different. Specifications: AEAD_.

  - Some pure python ciphers (aes-256-cfb1-py) is quite slow, and is not recommended to use without PyPy speedup. Try install pycryptodome_ and use C version cipher instead.

  - To enable OTA encryption with shadowsocks, add '!' immediately after cipher name.

- netloc

  - It can be "hostname:port" or "/unix_domain_socket". If the hostname is empty, server will listen on all interfaces.

  - Valid netloc: localhost:8080, 0.0.0.0:8123, /tmp/domain_socket, :8123

- localbind

  - It can be "@in" or @ipv4_address or @ipv6_address

  - Valid localbind: @in, @192.168.1.15, @::1

- plugins

  - It can be multiple plugins joined by ",". Supported plugins: plain, origin, http_simple, tls1.2_ticket_auth, verify_simple, verify_deflate

  - Valid plugins: /,tls1.2_ticket_auth,verify_simple

- rules

  - The filename that contains regex rules

- auth

  - The username, colon ':', and the password

URIs can be joined by "__" to indicate tunneling by relay. For example, ss://1.2.3.4:1324__http://4.5.6.7:4321 make remote connection to the first shadowsocks proxy server, and then tunnel to the second http proxy server.

.. _AEAD: http://shadowsocks.org/en/spec/AEAD-Ciphers.html

Examples
--------

- Regex rule

  Define regex file "rules" as follow:

  .. code:: rst

    #google domains
    (?:.+\.)?google.*\.com
    (?:.+\.)?gstatic\.com
    (?:.+\.)?gmail\.com
    (?:.+\.)?ntp\.org
    (?:.+\.)?glpals\.com
    (?:.+\.)?akamai.*\.net
    (?:.+\.)?ggpht\.com
    (?:.+\.)?android\.com
    (?:.+\.)?gvt1\.com
    (?:.+\.)?youtube.*\.com
    (?:.+\.)?ytimg\.com
    (?:.+\.)?goo\.gl
    (?:.+\.)?youtu\.be
    (?:.+\.)?google\..+

  Then start *pproxy*

  .. code:: rst

    $ pproxy -r http://aa.bb.cc.dd:8080?rules -v
    Serving on :8080 by http,socks4,socks5
    http ::1:57768 -> http aa.bb.cc.dd:8080 -> www.googleapis.com:443
    http ::1:57772 -> www.yahoo.com:80
    socks4 ::1:57770 -> http aa.bb.cc.dd:8080 -> www.youtube.com:443

  *pproxy* will serve incoming traffic by http/socks4/socks5 auto-detect protocol, redirect all google traffic to http proxy aa.bb.cc.dd:8080, and visit all other traffic directly from local.

- Use cipher

  Add cipher encryption to make sure data can't be intercepted. Run *pproxy* locally as:

  .. code:: rst

    $ pproxy -i ss://:8888 -r ss://chacha20:cipher_key@aa.bb.cc.dd:12345 -v

  Next, run pproxy.py remotely on server "aa.bb.cc.dd". The base64 encoded string of "chacha20:cipher_key" is also supported:

  .. code:: rst

    $ pproxy -i ss://chacha20:cipher_key@:12345

  The same as:

  .. code:: rst

    $ pproxy -i ss://Y2hhY2hhMjA6Y2lwaGVyX2tleQ==@:12345

  The traffic between local and aa.bb.cc.dd is encrypted by stream cipher Chacha20 with secret key "cipher_key".

- Unix domain socket

  A more complex example:

  .. code:: rst

    $ pproxy -i ss://salsa20!:complex_cipher_key@/tmp/pproxy_socket -r http+ssl://domain1.com:443#username:password

  *pproxy* listen on the unix domain socket "/tmp/pproxy_socket" with cipher "salsa20" and key "complex_cipher_key". OTA packet protocol is enabled by adding ! after cipher name. The traffic is tunneled to remote https proxy with simple http authentication.

- SSL/TLS server

  If you want to listen in SSL/TLS, you must specify ssl certificate and private key files by parameter "--ssl":

  .. code:: rst

    $ pproxy -i http+ssl://0.0.0.0:443 -i http://0.0.0.0:80 --ssl server.crt,server.key --pac /autopac

  *pproxy* listen on both 80 HTTP and 443 HTTPS ports, use the specified SSL/TLS certificate and private key files. The "--pac" enable PAC feature, so you can put "https://yourdomain.com/autopac" path in your device's auto-configure url.

  Simple guide for generating self-signed ssl certificates:

  .. code:: rst

    $ openssl genrsa -des3 -out server.key 1024
    $ openssl req -new -key server.key -out server.csr
    $ cp server.key server.key.org
    $ openssl rsa -in server.key.org -out server.key
    $ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

- SSR plugins

  ShadowsocksR example with plugin "tls1.2_ticket_auth" to emulate common tls traffic:

  .. code:: rst

    $ pproxy -i ssr://chacha20:mypass@0.0.0.0:443/,tls1.2_ticket_auth,verify_simple

- Local bind ip

  If you want to route the traffic by different local bind, use the @localbind URI syntax. For example, server has three ip interfaces: 192.168.1.15, 111.0.0.1, 112.0.0.1. You want to route traffic matched by "rule1" to 111.0.0.2 and traffic matched by "rule2" to 222.0.0.2, and the remaining traffic directly:

  .. code:: rst

    $ pproxy -i ss://:8000/@in -r ss://111.0.0.2:8000/@111.0.0.1?rule1 -r ss://222.0.0.2:8000/@222.0.0.1?rule2

- Redirect protocol

  IPtable NAT redirect example:

  .. code:: rst

    $ sudo iptables -t nat -A OUTPUT -p tcp --dport 80 -j REDIRECT --to-ports 5555
    $ pproxy -i redir://:5555 -r http://remote_http_server:3128 -v

  The above example illustrates how to redirect all local output tcp traffic with destination port 80 to localhost port 5555 listened by **pproxy**, and then tunnel the traffic to remote http proxy.

- Relay tunnel

  Relay tunnel example:

  .. code:: rst

    $ pproxy -r http://server1__ss://server2__socks://server3

  *pproxy* will connect to server1 first, tell server1 connect to server2, and tell server2 connect to server3, and make real traffic by server3.

