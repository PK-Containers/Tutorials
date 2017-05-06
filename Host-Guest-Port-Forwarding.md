http://stackoverflow.com/questions/9537751/virtualbox-port-forward-from-guest-to-host

Virtualbox “port forward” from Guest to Host [closed]
    Here is my setup:

    - Host: Windows XP
    - Guest: Ubuntu 10.04
    - Networking: NAT
    I am setting an Apache web server on the Guest, but I want to be able to do this on the Windows machine:

    - go to the browser, type http://localhost:8000
    Also, I tried to change my networking to bridge and I got a new IP. But when I tried to do http://:8000, it says that it could not connect.

    virtualbox portforwarding
    shareimprove this question
    edited Nov 18 '15 at 12:17

    mob
    334
    asked Mar 2 '12 at 17:32

    Carmen
    97331121
    closed as off topic by Robert Harvey♦ Oct 25 '12 at 19:27

    Questions on Stack Overflow are expected to relate to programming within the scope defined by the community. Consider editing the question or leaving comments for improvement if you believe the question can be reworded to fit within the scope. Read more about reopening questions here.
    If this question can be reworded to fit the rules in the help center, please edit the question.

    1	 	
    I had the same problem. Turned out the guest OS had an active firewall that was blocking port 80. – Nicholas Shanks Jul 2 '13 at 15:32
    83	 	
    As a web developer who uses VirtualBox as part of my daily workflow, disagree with this being marked as off topic. Please consider reopening. – sparecycle Dec 5 '13 at 22:25
    1	 	
    1. Go to the VM 2. ifconfig (get local IP - should be 10.0.2.X) 3. ssh 10.0.2.2 to get to the host machine – Mark Roberts Feb 13 '14 at 21:24

    @deeperDATA It may be in the scope of a web developer's job, but stack overflow isn't meant to encapsulate every part of the job. It's a testament to the complexity of our profession that its requirements span multiple stack exchange sites. – Kevin Lawrence Oct 25 '15 at 3:32
    1	 	
    It took me a long time to get it working. Our problem was the ip binding of the application in the guest system, it binded to machine name, meaning 127.0.1.1 in ubuntu. We changed the binding to 0.0.0.0. Port forwarding settings: Host IP = DNS Host IP, Host Port = 8080, Guest IP = IP of eth0, Guest Port = 8080. – slowy Nov 26 '15 at 14:21
    add a comment
    2 Answers
    active oldest votes
    up vote
    155
    down vote
    Network communication Host -> Guest

    Connect to the Guest and find out the ip address:

    ifconfig 
    example of result (ip address is 10.0.2.15):

    eth0      Link encap:Ethernet  HWaddr 08:00:27:AE:36:99
              inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
    Go to Vbox instance window -> Menu -> Network adapters:

    adapter should be NAT
    click on "port forwarding"
    insert new record (+ icon)
    for host ip enter 127.0.0.1, and for guest ip address you got from prev. step (in my case it is 10.0.2.15)
    in your case port is 8000 - put it on both, but you can change host port if you prefer
    Go to host system and try it in browser:

    http://127.0.0.1:8000
    or your network ip address (find out on the host machine by running: ipconfig).

    Network communication Guest -> Host

    In this case port forwarding is not needed, the communication goes over the LAN back to the host.

    On the host machine - find out your netw ip address:

    ipconfig
    example of result:

    IP Address. . . . . . . . . . . . : 192.168.5.1
    On the guest machine you can communicate directly with the host, e.g. check it with ping:

    # ping 192.168.5.1
    PING 192.168.5.1 (192.168.5.1) 56(84) bytes of data.
    64 bytes from 192.168.5.1: icmp_seq=1 ttl=128 time=2.30 ms
    ...
    shareimprove this answer
    edited Apr 9 '12 at 17:22
    answered Apr 9 '12 at 17:17

    Robert Lujo
    7,44032847
    7	 	
    both port will not be 8000. The host port will be 8000 or whaterver u want, but the guest port should be 80 – Yasin Dec 5 '12 at 10:12
    1	 	
    It took some time, but it worked like charm! :) Thanks buddy! :) – Praveen Kumar Feb 27 '13 at 6:19
    5	 	
    Getting from the VM to the host in this case should be possible by going to the VM and getting ITS IP address (10.0.2.15 as below). To get to the host machine from the VM, the IP is 10.0.2.2 (by convention). – Mark Roberts Feb 13 '14 at 21:25
    4	 	
    If using NAT for the guest... If the service that is running on the host is bound to 127.0.0.1 only, then the guest cannot use the public ip of the host to connect to that service (example service: privoxy). Instead you need to use 10.0.2.2 as mentioned by Mark, or whatever 'route -n' (run on the guest) shows as the default gateway. – desm Mar 6 '14 at 20:20
    1	 	
    In case of CentOS, we may have to disable firewall or edit specific rules in the iptable – Reddy Jun 4 '15 at 4:37 
    show 3 more comments

    up vote
    13
    down vote
    That's not possible. localhost always defaults to the loopback device on the local operating system.
    As your virtual machine runs its own operating system it has its own loopback device which you cannot access from the outside.

    If you want to access it e.g. in a browser, connect to it using the local IP instead:

    http://192.168.180.1:8000
    This is just an example of course, you can find out the actual IP by issuing an ifconfig command on a shell in the guest operating system.

    shareimprove this answer
    edited Mar 2 '12 at 17:42
    answered Mar 2 '12 at 17:37

    Chris
    2,5761932

    I tried this as well, but didn't work for both NAT and bridge. The apache logs in the guest doesn't give any errors and the apache is up and running in the guest. – Carmen Mar 2 '12 at 18:07

    Are you sure that you are using the correct IP? You can verify that by trying to access the address from inside the virtual machine. If that fails, you are either using a wrong IP or your apache is not set up to listen on port 8080. – Chris Mar 2 '12 at 18:09

    Worked for me - thanks! – Matt Frear Jul 1 '13 at 14:39

    Worked for me as well. Before replacing "localhost" with the host OS IP address, I always got "Server refused your key" and "Access denied" without explanation in /var/log/auth.log despite LogLevel DEBUG3 in /etc/ssh/sshd_config – V-R Aug 26 '16 at 14:57

    @Chris does this mean that on the host, with virtual host entries like mysite.localhost will not be accessible to the Guest machine, because I must refer to the Host using an IP address only? – danjah Nov 23 '16 at 23:34
