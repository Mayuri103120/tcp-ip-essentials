## Exercises with IP address and subnet mask

In these exercises, we will configure IP addresses and subnet masks of the hosts on the single segment network in various ways, and observe the effect of each configuration as hosts try to reach one another over the network segment.

Students who complete these exercises often come away believing that two hosts on the same network segment can reach one another only if their IP addresses and subnet masks are configured so that they are in the same subnet. But, this is **not correct**! I will try to address this misconception in advance.

For a message to be transmitted on a network segment, the sending host needs to know which network interface (assuming it has more than one interface) to send the message from. To identify the interface that the packet should be sent from, the sending host

* looks at its routing table,
* identifies the rule in the routing table that applies to the destination IP address in the packet,
* and sends the packet using the network interface specified by that rule.

(You may have thought a routing table is only relevant when there are routers connecting multiple networks; but routing tables are also used at end hosts to determine how to send packets, even on a single segment!) 

If there is no rule in the routing table that applies to the destination IP address in the packet, a "Network unreachable" is returned and nothing is sent on the network segment.

When will there be a rule that applies to the destination address in the packet?

* When you modify the routing table to add such a rule.
* When a network interface is configured with an IP address and subnet mask, a rule is automatically added to the routing table that applies to all destination addresses in the same subnet.

In these *Exercises with IP address and subnet mask*, a Host A can send to Host B if Host B's IP address is within the range of addresses in Host A's subnet. This is because of that automatically added rule. If Host B's IP address is not within the range of addresses in Host A's subnet, then there will be no automatically added rule that applies to messages to Host B. Under these circumstances, either (1) a rule must be added, or (2) the packet will not be sent, and a "Network unreachable" error is returned.

In summary, the correct understanding should be: two hosts on the same network segment can reach one another only if

* **either** their IP addresses and subnet masks are configured so that they are in the same subnet,
* **or** a rule has been added to their routing tables to describe which interface to use to communicate with one another.

**otherwise**, the sender will observe a "Network unreachable" error.

### Remove the default route

Before you can work on the exercises in this section, you will have to complete some extra setup steps, in which you manipulate the routing table on the remote hosts.

**Note**: If you make a mistake in these setup steps, you may lose your connection to the remote host. Rebooting the host should restore connectivity, in case this happens. To reboot the hosts in your topology, visit the slice page in the GENI Portal, and click on the "Restart" button. Wait a few minutes for your hosts to come back up before you try to connect again.
 
The aim of the next exercise is to learn what happens when you try to send IP packets to a network that your host doesn't know how to reach.

We are going to trigger a “network is unreachable” error message. This message occurs when there is no route in the host's routing table that describes how to reach a particular destination. 

Currently, however, there is a “default gateway” rule in the routing table that describes how to route *all* traffic whose destination address is not specifically given by any other rule. When there is a "default gateway" rule, we will never observe a "network is unreachable" message.  

However, if we just remove the default gateway rule, we'll lose access to the remote host over SSH, since the SSH connection between you and the remote host is routed using that default gateway rule. 
To make this exercise work without losing our SSH connection, we need to replace the default rule with one specific to the IP address we are using to connect. Then we'll be able to observe the "destination unreachable" message AND maintain our SSH connection.

I will show you how to do this with an example - to do it yourself, you'll have to substitute the relevant IP addresses and port numbers for _your_ connection in the commands below.

You will repeat the steps below for all four hosts in your topology.


First, use

```
route -n
```

on the remote host to see the current routing table. For example:

```
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.16.0.1      0.0.0.0         UG    0      0        0 eth0
10.10.0.0       0.0.0.0         255.255.255.0   U     0      0        0 eth1
172.16.0.0      0.0.0.0         255.240.0.0     U     0      0        0 eth0
```

Make a note of the default gateway - here, 172.16.0.1.

Next, find out what network you’re connecting from by running

```
netstat -n  | grep <PORT>
```

where in the command above, you substitute the port number that you use to SSH into this node (which you get from the GENI Portal). For example (with port 26203 as the SSH port), I might run


```
netstat -n  | grep 26203
```

and see the following output:

```

tcp        0     36 172.17.3.4:26203        216.165.95.188:34488    ESTABLISHED
tcp        0      0 172.17.3.4:26203        216.165.95.188:34490    ESTABLISHED
tcp        0      0 172.17.3.4:26203        216.165.95.188:34324    ESTABLISHED
```

From this output, you can find out your own public IP address (as observed by the remote host). (Here, my public IP address is 216.165.95.188.)

Now, add a route to the routing table that is *specific* to the network that you connect from:

```
sudo route add -net <FIRST OCTET OF YOUR IP>.<SECOND OCTET OF YOUR IP>.0.0/16 gw <DEFAULT GW YOU OBSERVED BEFORE>
```

for example:

```
sudo route add -net 216.165.0.0/16 gw 172.16.0.1
```

Once you have done so, you can delete the default gateway route without losing your SSH connection:

```
sudo route del default gw <DEFAULT GW>
```

e.g.

```
sudo route del default gw 172.16.0.1
```

Check your routing table with

```
route -n
```

and make sure there is no default gateway rule (no rule with 0.0.0.0 as the destination). If your routing table looks good, you can continue!

Remember to repeat this step on all of the hosts in the topology.

**IMPORTANT NOTE**: As a result of the steps above, you may get the following error message whenever you use `sudo` for the remainder of this lab exercise:

```
sudo: unable to resolve host
```

This error is *not* a cause for concern, and you can safely ignore it.


### Exercise - network unreachable

First, we will see what happens when we try to reach a host at an address for which we do not have a routing rule.


On "romeo", run

```
arp -i eth1 -n
```

and 


```
route -n
```

to see the current ARP table and routing table. Save these for your lab report. Are there any entries in either table that apply to the address 10.10.10.100?

Then, run


```
sudo tcpdump -i eth1 -w $(hostname -s)-net-unreachable.pcap
```

on "romeo". While `tcpdump` is running, open a second terminal to "romeo" and in that terminal, use `ping` to send an ICMP echo request to IP address 10.10.10.100:


```
ping -c 1 10.10.10.100
```

Save the `ping` output. Use Ctrl+C to stop the `tcpdump`, then use 

```
tcpdump -enX -r $(hostname -s)-net-unreachable.pcap
```

to play back the summary in the terminal.


Next, add a rule to the routing table on "romeo" that applies to 10.10.10.100:

```
sudo route add -host 10.10.10.100 dev eth1
```

and then use 

```
route -n
```

to see the new routing table. Save this for your lab report. 

Then, run


```
sudo tcpdump -i eth1 -w $(hostname -s)-host-unreachable.pcap
```

on "romeo". While `tcpdump` is running, open a second terminal to "romeo" and in that terminal, use `ping` to send an ICMP echo request to IP address 10.10.10.100:


```
ping -c 1 10.10.10.100
```

Save the `ping` output. Use Ctrl+C to stop the `tcpdump`, then use 

```
tcpdump -enX -r $(hostname -s)-host-unreachable.pcap
```

to play back the summary in the terminal.

Use `scp` to transfer both packet captures to your laptop for further analysis using Wireshark.

Also delete the routing table rule you added earlier, with

```
sudo route del -host 10.10.10.100 dev eth1
```



**Lab report**: Can you see any ICMP echo request sent on the network? Why? Explain what happened using the `route -n` output and the `ping` output in each case.


**Lab report**: Why does "romeo" not send any ARP request in the first part of this exercise, but does send ARP requests in the second part?



### Exercise - experiments with subnet masks

For this experiment, you will need *two* terminals on *each* host in your topology. Set up your terminal application accordingly.

Configure the (experiment) interface on each host, according to the following table:



| Host          | IP address    | Subnet mask     |
| ------------- | ------------- |-----------------|
| romeo         | 10.10.0.100   | 255.255.255.240 |
| juliet        | 10.10.0.101   | 255.255.255.0   |
| hamlet        | 10.10.0.102   | 255.255.255.0   |
| ophelia       | 10.10.0.120   | 255.255.255.240 |


To change the IP address and/or netmask of a given interface on our hosts, use the syntax

```
sudo ifconfig INTERFACE IP-ADDRESS netmask NETMASK
```

substituting appropriate values for `INTERFACE` name, `IP-ADDRESS`, and `NETMASK`.


Run 


```
route -n
```

on each host, and save the routing tables. 


We will run this exercise in four parts:


#### Part 1

On each host, run

```
sudo tcpdump -en -i eth1
```

From "romeo", ping "hamlet" (10.10.0.102) or "juliet" (10.10.0.101):


Observe what happens. Stop your `tcpdump`, and save the `tcpdump` output for all the hosts.

#### Part 2

On each host, run

```
sudo tcpdump -en -i eth1
```

From "ophelia", ping "hamlet" (10.10.0.102) or "juliet" (10.10.0.101). Note the output in the `ping` window.

Observe what happens. Stop your `tcpdump`, and save the `tcpdump` output for all the hosts.

#### Part 3

On each host, run

```
sudo tcpdump -en -i eth1
```

From "hamlet" or "juliet", ping "romeo" (10.10.0.100)

Observe what happens. Stop your `tcpdump`, and save the `tcpdump` output for all the hosts.

#### Part 4

On each host, run

```
sudo tcpdump -en -i eth1
```

From "hamlet" or "juliet", ping "ophelia" (10.10.0.120).

Observe what happens. Stop your `tcpdump`, and save the `tcpdump` output for all the hosts.




When you are finished running this exercise, you can restore the default gateway route on all hosts with

```
sudo route add default gw <DEFAULT GW>
```

using the default gateway you noted at the beginning of this section, e.g.

```
sudo route add default gw 172.16.0.1
```

**Lab report**: In the routing table for each host, show the rule that applies to traffic that is sent within the same subnet. (This rule is added to the routing table automatically when you configure the IP address and netmask on the network interface.) 


**Lab report**: Use bitwise analysis to answer the following questions. Show how the netmask was applied in each case.

* What is the range of IP addresses (i.e. smallest IP address and largest IP address) that is in the same subnet as "romeo"?
* What is the range of IP addresses (i.e. smallest IP address and largest IP address) that is in the same subnet as "juliet"?
* What is the range of IP addresses (i.e. smallest IP address and largest IP address) that is in the same subnet as "hamlet"?
* What is the range of IP addresses (i.e. smallest IP address and largest IP address) that is in the same subnet as "ophelia"?

**Lab report**: Explain why the other hosts cannot exchange ICMP echo messages with "ophelia", while they can with "romeo", which has the same subnet mask as "ophelia". Use your answer to the previous question to support this explanation.

