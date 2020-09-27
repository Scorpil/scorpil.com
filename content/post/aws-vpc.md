+++
title = "Demystifying AWS VPC"
description = "A short step-by-step guide to creating a secure custom VPC in AWS"
tags = [
    "aws",
    "vpc",
    "infrastructure",
    "devops",
]
date = 2020-09-20T09:43:31Z
author = "Scorpil"
+++

VPC is the topic that flies under the radar of many Software Developers, despite being present in every AWS account (well, maybe not for accounts created before 2009... But that's unlikely). There are a few reasons for this I can think of:

1. Big companies have Ops/DevOps/SysAdmin/SRE/Security departments that take care of VPC
2. Startups often don't bother tuning it - everything works with default settings after all
3. Networking is complicated and is rarely regarded as fun

Despite those somewhat justified arguments, most developers that deal with AWS would benefit from digging deeper into VPC setup and its configuration. Here are my counterpoints:

1. Understanding VPC will help you communicate with; Ops/DevOps/SysAdmin/SRE/Security department much more efficiently
2. Everything works by default because everything is open (i.e. unsecure) by default
3. Oh but it *is* fun when you dig deep enough

So let's take a careful look into AWS networking from the software engineering perspective. This post requires from the reader a very basic level of familiarity with AWS terminology and networking: you need to understand things like regions and availability zones, and have an idea of what an IP address is.

To make this post a bit less abstract, we will configure a VPC for a common real-world scenario: distributed application running in two availability zones in a single region consisting of *public* EC-2 instances serving a web-application and *private* EC-2 instances hosting a database. You can improve on the architecture by using RDS, adding a loadbalancer, configuring VPC peering with other regions, etc., but we will leave those topics out of the scope of this post - the topic at hand is complicated enough.

### VPC - what is it actually

VPC stands for Virtual Private Cloud, which is a very apt name for a service, unlike some other AWS services I can think of...

- It's **Virtual**, because like most user-facing things in modern clouds it's powered by software and not copper. Hence its flexibility: neither you nor AWS will need to plug in a single RJ45 jack to configure it;
- It's **Private**, because it allows you to carve out your own dedicated space for your cloud infrastructure, walled out and carefully guarded against the dangers of the public internet;
- It's a **Cloud** because, well, it's in **AWS**. Or, actually, the better way to think about it as being *around* AWS.

In other words, the idea of VPC is to give the user a granular set of controls over what network requests are allowed to reach its services from the outside, and how to route the requests inside the cloud infrastructure.

VPC operates on the regional level, meaning that if you want your application to span multiple regions you will need to perform VPC setup for each region separately and configure a VPC peering between them.

„Fresh“ AWS accounts come with the „default“ VPC preconfigured. You can’t really start, for example, an EC2 instance without any VPC at all, so „default“ VPC is where the new instances get launched into if nothing else is configured (hence the „default“ name). This way amazon manages to balance user-friendliness with flexibility, and that’s why launching an EC2 instance on a „fresh“ account works out of the box. The problem is, however, that the default VPC is not tailored in any way to the needs of your application, so it will allow all incoming traffic to all existing instances. Correctly configured Security Groups help somewhat protect your resources from completely exposing their ports to the public, and to be frank they are often completely sufficient to protect a simple app. However, as the infrastructure grows to accommodate more distinct components and services with complicated access patterns between them, you might want to invest in a more delicate access configuration.

When creating a custom VPC, there are two important decisions to make:

- the size of the VPC: e.g. roughly the number of instances and services you plan to **ever** host inside the VPC. You can’t change this value so make it big. Normally, you wouldn’t go lower than a few thousand addresses.
- the private IP address range. Instances inside your VPC will receive IP addresses from this address range. Technically, nothing stops you from choosing any IP range you‘d like; however, if your private IP matches the IP of anything on the public internet you won’t be able to access it from within the VPC. For this reason, internal networks are configured to use the subset of one of the following [reserved](https://en.wikipedia.org/wiki/Reserved_IP_addresses) IP ranges:

  - 10.0.0.0-10.255.255.255 - 16 million private IPs in total, the default choice for cloud hosting.
  - 172.16.0.0 – 172.31.255.255 - almost 1 million IPs
  - 192.168.0.0 – 192.168.255.255 - 65536 IP‘s, commonly seen on home routers

For our VPC we want to have around 4000 IPs available in 10.x.x.x range. We can’t enter those numbers directly into the VPC configuration, we need to translate them into a [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) notation first. CIDR is just a special format to describe an IP sub-range concisely. You can use IP subnet calculators like [cidr.xyz](https://cidr.xyz) (for IPv4) and [cidrv6.xyz](https://cidrv6.xyz) (for IPv6) to get an intuition into how CIDR works. CIDR value for our example is 10.0.0.0/20.

We will also enable IPv6 on our VPC. AWS VPC's use IPv4 as a default addressing format, IPv6 configuration is completely optional. The approach AWS takes to IPv6 routing is quite different than what we used to for IPv4. IPv6 spec defines address range \`fc00::/7\` to be private but it's used much less frequently than 10.x.x.x in IPv4. Ipv6 address space is just so large, that AWS simply allocates /56 public address range from his own address pool to your VPC and calls it a day. For this reason, there are no "private" IPv6 addresses in AWS. IPv6 range allocated to every VPC in AWS contains 2^72 addresses, which will be more than enough for \*any\* VPC ever. Just to put the number into perspective, it's enough to assign an IPv6 address to each grain of sand on the planet. The ability to assign globally unique addresses to each of your resources makes network architecture in many ways simpler, but at the same time, it makes secure routing configuration even more important. We'll discuss the reasons for it in more detail in a subnet section.

As a summary, here's what our initial VPC configuration looks like:

| Parameter  | Value                                     |
|------------|-------------------------------------------|
| Name       | `my-secure-vpc` (can be anything really) |
| CIDR block | `10.0.0.0/20`                               |
| IPv6       | Amazon-provided IPv6 CIDR block (in context of this post let's assume it's `2001:db8::aa00::/56`) |

### Internet Gateway

Internet Gateway is an abstract VPC component that represents internet access. The only purpose of the Internet Gateway is to serve as a source/target in the VPC routing configuration. You can’t really configure it in any meaningful way, apart from attaching it to VPC. Internet Gateway is present from the start in the default VPC, but **it won't be created** for any custom VPC you create. Since we want to be able to access some resources in our VPC from the Internet, we need to create a new Internet Gateway and attach it to our VPC.

### Subnets

The cornerstone of understanding VPC is understanding the concept of subnets. In short, **subnet** is a way of splitting your VPC into small manageable chunks, each of which contains instances and services with a similar type of network access. The „type of network access“ here is up to you to define, but in our case, we want a „private“ subnets for database instances and a „public“ subnet for our webserver. An important detail to remember is that subnets can't span multiple availability zones. In our case we want our application to work in two different availability zones, this means we need "private" and "public" subnet in each of them, 4 subnets in total. Fortunately, it's easy to configure functionally similar subnets by attaching them to the same routing table (more about them in the next section).

Subnets are defined in CIDR notation, which we already discussed in VPC section. AWS makes sure that you will not define two overlapping subnets, because that would break our routing logic. For each subnet, AWS reserves first four and last one IP's for its internal tasks, so it's common to configure the size of the subnet of at least /24. This size makes our IP addresses look "cleaner": we can distinguish subnet based on the value of the second-to-last octet in the IP address. You can play around with a subnet calculator to understand why that's the case. For IPv6, AWS set subnet size to /64 (not configurable).

You can configure the subnet to automatically map a public IPv4 address for all instances launched into it - we will need to do this for our public subnet.

Considering anything above, our 4 new subnets would look something like this:

| Name      | CIDR (v4)   | CIDR (v6)         | AZ | Public IPv4|
|-----------|-------------|-------------------|----|------------|
| public-a  | 10.0.1.0/24 | 2001:db8::aa01/56 |  a | ✓          |
| public-b  | 10.0.2.0/24 | 2001:db8::aa02/56 |  b | ✓          |
| private-a | 10.0.3.0/24 | 2001:db8::aa03/56 |  a |            |
| private-b | 10.0.4.0/24 | 2001:db8::aa04/56 |  b |            |

### Route tables

*Route tables* fill the role of the router for your VPC. They allow you to configure destinations for packets depending on their destination IP.

When you first create a VPC, a single "default" route table is created for you. It contains two rules needed to route private traffic internally within our VPC:

| Destination         | Target |
|---------------------|--------|
| `10.0.0.0/20`       | local  |
| `2001:db8::aa01/64` | local  |

All subnets that we created are by default associated with this route table. It's a good security practice to make defaults as restricted as possible, so we will keep default route table private. For our public subnet we need to make a new table, and add routes to Internet Gateway there:

| Destination         | Target            |
|---------------------|-------------------|
| `10.0.0.0/20`       | local             |
| `2001:db8::aa01/64` | local             |
| `0.0.0.0/0`         | internet gateway  |
| `::/0`              | internet gateway  |

Now we just associate our public subnet with the new route table to make them _really_ public.

### Network Access Control Lists (nACL's)

*nACL's* are an additional level of protection that sits in front of your subnets. You can configure them by adding firewall rules operating on [OSI layer 4](https://en.wikipedia.org/wiki/OSI_model#Layer_4:_Transport_Layer). Default NACL configuration allows all traffic through, and it suits our usecase. If you need to block a particular source IP or destination port in one of your subnets - NACL's would be a suitable tool to fullfill the task. 

### Security Groups

Security groups are the last level of defense that define which ports are open on your instance for TCP and/or UDP traffic, and from which sources to allow traffic in. Keep in mind that you need to configure inbound (targeted to instance) and outbound (coming from instance) rules separately. You should only allow the traffic on ports you know your app needs to operate, from sources you expect the requests to come. You can define the source as a CIDR or a reference to another security group. Let's say our webserver serves HTTP traffic on port 80, and it communicates with database through TCP connection on port 12345. In that case, we need two security groups:

| Name | Direction |Protocol | Port  | Source              |
|------|-----------|---------|-------|---------------------|
| web  | Inbound   | TCP     | 80    | `0.0.0.0/0, ::/0`   |
| db   | Inbound   | TCP     | 12345 | security group: web |

### Instances

At this point everything is ready and we can place new instances in our VPC, we just need to provide a correct VPC, subnet and security group during when starting:

| Instance    | Subnet         | Security Group |
|-------------|----------------|----------------|
| webserver-1 | public-a       | web            |
| webserver-2 | public-b       | web            |
| database-1  | private-a      | db             |
| database-2  | private-b      | db             |

Graphically, the VPC looks more or less like this:
![Graphical representation of the configured VPC](/img/vpc.svg)

### Wrap Up

There are quite a few moving parts in AWS VPC's, but all of them have distinct responsibilities and fit well into the whole picture once you get to know them. If you want some additional challenge, you can:
1. Configure [NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) to allow instances from private subnet initiate connection to the Internet.
2. Set up a [bastion host](https://aws.amazon.com/quickstart/architecture/linux-bastion/) to have an ability to securely SSH into instances in your private subnets.

<small>If you see any mistakes or inconsistencies in this post - consider [opening a PR](https://github.com/scorpil/scorpil.com) to fix them. Thanks.</small>
