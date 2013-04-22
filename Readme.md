
# Amazon VPC and EC2 Multiple IP Addresses

Here is a quick guide how to setup multiple IP addresses that point to a single EC2 instance using Amazon VPC.

1. Create a new VPC (must be at least a Small to assign up to 8 IP addresses, 4 per Network Interface).

  > For a list of EC2 instance information, visit <http://www.ec2instances.info/>

  > Ensure that your security groups has ports `443` and `80` opened.  You will also need to have port `22` open for SSH ([at least until you change it](https://github.com/niftylettuce/amazon-ec2-node-stack#ubuntu-security-configuration))

2. Launch a new EC2 instance into the VPC with up to 3 secondary IP addresses.

3. Assign and associate Elastic IP's to the VPC and respective Private IP's.

4. SSH into your service and make the following changes to your network interface:

    ```bash
    sudo vim /etc/network/interfaces
    ```

    ```diff
    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).

    # The loopback network interface
    auto lo
    iface lo inet loopback

    # The primary network interface
    auto eth0
    iface eth0 inet dhcp
    +    address 10.0.0.53
    +    netmask 255.255.255.0
    +    gateway 10.0.0.1
    +    up ip addr add 10.0.0.79/24 dev eth0
    +    up ip addr add 10.0.0.80/24 dev eth0
    +    up ip addr add 10.0.0.81/24 dev eth0
    ```

    > Note that `10.0.0.53` is your Primary Private IP address, and `.79`, `.80`, and `.81` are your other Private IP's which have their own unique Public IP addresses.

    ```bash
    sudo /etc/init.d/networking restart
    ```

5. You now should be able to ping your other private IP addresses.

    ```bash
    # replace with your private IP addresses
    ping 10.0.0.79
    PING 10.0.0.79 (10.0.0.79) 56(84) bytes of data.
    64 bytes from 10.0.0.79: icmp_req=1 ttl=64 time=0.051 ms
    64 bytes from 10.0.0.79: icmp_req=2 ttl=64 time=0.052 ms
    64 bytes from 10.0.0.79: icmp_req=3 ttl=64 time=0.048 ms
    64 bytes from 10.0.0.79: icmp_req=4 ttl=64 time=0.049 ms
    ^C
    --- 10.0.0.79 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 2999ms
    rtt min/avg/max/mdev = 0.048/0.050/0.052/0.001 ms
    ```

6. Edit your host file to add the new Public IP's with their respective hostnames.

    ```bash
    sudo vim /etc/hosts
    ```

    ```diff
    127.0.0.1 localhost

    +# 55.123.456.700
    +10.0.0.79 getprove.com getprove
    +
    +# 55.123.456.800
    +10.0.0.80 teelaunch.com teelaunch
    +
    +# 55.123.456.900
    +10.0.0.81 wakeup.io wakeup

    # The following lines are desirable for IPv6 capable hosts
    ::1 ip6-localhost ip6-loopback
    fe00::0 ip6-localnet
    ff00::0 ip6-mcastprefix
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    ff02::3 ip6-allhosts
    ```

    > Note that you'll need to replace the public IP address with your own respective ones and can rename `getprove`, `teelaunch`, and `wakeup` and their domain names with respective ones of your own.

7. Setup your respective server/application environment:

    **Node and Express**

    ```bash
    mkdir ~/test
    cd ~/test
    vim app.js
    ```

    ```js
    var express = require('express')
    var app = express()
    app.get('/', function(req, res) {
      res.send('hello from getprove http server')
    })
    app.listen(80, 'getprove')
    ```

    ```bash
    sudo /home/ubuntu/opt/node/bin/node app
    ```

    Then you would visit <http://55.123.456.700> or <http://getprove.com> if you had your DNS records setup pointing `getprove.com` to `55.123.456.700`.

    > Similarly you can run other node processes for each of the other servers.  This is useful especially if you want to support SSL on browsers that do not have SNI support (Internet Explorer...) -- so that you can run multiple SSL servers on port 443 from the same server.  An example is given below of HTTPS setup.

    ```js
    var express = require('express')
    var app = express()
    var https = require('https')
    var fs = require('fs')

    var options = {}
    options.key = fs.readFileSync('getprove.key').toString()
    options.cert = fs.readFileSync('getprove.crt').toString()
    options.ca = fs.readFileSync('getprove.ca.crt').toString()

    app.get('/', function(req, res) {
      res.send('hello from getprove https secure server')
    })

    var server = https.createServer(options, app)
    server.listen(443, 'getprove')
    ```

    **Nginx**

    Coming soon.

    **Apache**

    Coming soon.
