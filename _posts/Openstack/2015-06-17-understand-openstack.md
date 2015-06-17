---
layout: post
title: 理解Openstack
category: Openstack
tags: 
keywords: 
description: 
---

> 第一篇学习文章，Juno下写的，绝对原创

#目录
  
    
world里面copy出来的，尽量排一下吧，第一次用markdown

- 1.	理解Opensatck	
-     1.1.	设计原则	
-  1.2.	为什么要用消息队列	
-  1.3.	设计模式	
- 2.	使用Opensatck	
-  2.1.	什么是新的
- 2.2.	为什么要用
-  2.3.	应该怎么用
-  2.4.	别人怎么用	
- 2.5.	用了会怎样	
- 3.	虚拟机创建	
- 3.1.	导读	
- 3.2.	虚拟机创建的流程	
- 3.3.	keystone	
- 3.3.1.	定义	
- 3.3.2.	功能	
- 3.3.3.	架构&性能	
- 3.4.	glance	
- 3.4.1.	定义	
- 3.4.2.	修改镜像	
- 3.4.3.	制作镜像	
- 3.4.4.	镜像格式转换	
- 3.5.	nova	
- 3.5.1.	物理机调度	


#1. 理解Openstack

##1.1.	设计原则

opensatck设计原则：

-  ①Scalability and elasticity are our main goals(可扩展性和伸缩性是我们的主要目标)；
- ②Any feature that limits our main goals must be optional(任何影响到可扩展性和伸缩性的功能都必须是可选的)；
- ③Everything should be asynchronous，If you can't do something asynchronously，see#2(所有的环节必须是异步的，如果不能异步实现，参考第②条设计原理)；
- ④All required components must be horizontally scalable(所有的基础组件必须能横向扩展)；
- ⑤Always use shared nothing architecture(SN) or sharding，If you can't share nothing／shard，see#2(始终使用无共享的架构，如果不能实现，参见第②条)；
- ⑥Distribute everything especially logic．Move logic to where state naturally exists(所有的都是分布式的，尤其是逻辑。把逻辑放在状态应该存在的地方)；
- ⑦Accept eventual consistency and use it where it is appropriate(接受最终一致性，并在适合的条件下使用)；
- ⑧Test everything(充足的测试)。


## 1.2.	为什么要用消息队列

异步，解耦

为什么要异步?
参照Opensatck设计原则。

怎么理解消息队列：
http://www.open-open.com/bbs/view/1405088402529
关于消息队列如何解耦消息发送方和接收方，打个可能不太恰当的比喻:
没有消息队列的情况，通信双方好比甲乙两个人面对面扔盘子，乙必须聚精会神的留意甲扔过来的盘子，稍有差池盘子就会落地摔破。消息队列就好比在两人中间放置一个橱柜，甲只需要把盘子逐个有序的放进橱柜，而乙可以去喝杯水或看会儿电视，在他想要盘子的时候来橱柜中拿即可。

##1.3.	设计模式
	composite组件模式
Neutron的众多plugins

	Strategy策略模式
nova-scheduler的物理机调度策略

##1.4.	版本发布
openstack和ubuntu的版本发布同步，半年一版，2014年10月的Juno版为第10个版本。
普遍认为在H版时，Openstack已经趋于稳定。


#2.	使用Opensatck

本章论述传统的IAAS方案过渡到opensatck方案的可行性。
##2.1.	什么是新的
在openstack的世界中，与旧世界（部门现有资源池）在世界观上差异比较显著，主要集中在以下几个方面。

1.	设计准则不同。openstack设计原则要求各个模块独立，充分解耦。既有系统为统一调度。各有利弊，暂不评述。要求采用新的设计原则。
2.	建模不同。openstack是充分抽象的建模方式，而既有系统更加贴近传统的数据中心建模方式、更贴近底层系统。这一处差异影响最大。在传统的数据中心，既有系统能更灵活的适应、开发。而openstack的模型，要实例化到具体的组网方案中，需要做的mapping工作较大。付出这些代价的好处是openstack的模型更加通用，可复用程度高。听起来有点抽象，举个例子：交换机上vlan概念是底层的网络属性，既有系统直接沿用，openstack则抽象为“虚拟网络”和“虚拟子网”两个模型。
3.	开发方式不同。openstack的开发语言python更适合底层IAAS系统的开发，能够更加高效的与操作系统打交道。但需要付出学习成本，好在openstack的客户端SDK提供了java版本。但建议即使是客户端的SDK也使用python版本。
4.	人员要求不同。openstack虽然逐步趋于稳定，但仍然是比较新的技术，并且由于近几年的激进，为自身埋下了诸多弊病。比如说，为了商业利益尽快打开市场，不得不兼容众多厂家，导致了大量的plugins，并且各个plugin并不能保证完全满足设计准则，水准参差不齐。可以说openstack既强大又弱小，强大的是设计，弱小的是实现或案例。在IAAS的世界里，如果说vmware是研究生，openstack才刚刚小学毕业。认清了这一现状，就要求我们的开发人员能够理解openstack的设计，修改openstack的代码，最起码，碰到问题的时候要能够分清是openstack自身的bug还是设计方式使然。
【2015-1-19补充】关于openstack的案例，已经有逐渐涌现，参看：http://www.csdn.net/article/2014-01-14/2818130-2014-OpenStack

##2.2.	为什么要用

	专业的体现，iaas的代名词。

	未来的行业规范，就像struts统一mvc。

	其实是被逼的。

##2.3.	应该怎么用

![1](/public/img/os_framework.jpg)

 
openstack部署图

- 	Horizon	web页面。可以根据需求修改horizon也可以不使用horizon，开发自己的web页面。
- 	OpensatckProxy	openstack代理。南向使用openstack提供的sdk，北向以接口的形式提供openstack的功能。对于北向对原生openstack接口的调用透传至Controller节点。
- 	Controller	openstack控制节点。部署openstack的各个组件的api服务端，以及keystone。可以集中部署也可分别部署至不同节点。
- 	Neutron	网络节点。提供SDN网络的控制节点。可以使用自带的虚拟化组件如ovs，也可连接至物理网络设备进行配置。
- 	Nova-compute	nova计算节点。hypersior层，连接x86物理机进行虚拟化。
- 	Cinder-volume		块存储。连接块设备
- 	Swift-object	对象存储。
- 	物理层	包括计算、存储、网络等物理资源。

##2.4.	别人怎么用
作为openstack金牌会员，在国内华为是在openstack上投入最多的，有一个产品叫FusionCloud。号称与openstack完全兼容。计算叫FusionCompute，网络叫FusionNetwork，存储叫FusionStorage。太没创意了！
我们出一个产品NeuCloud，计算叫NeuNova，存储叫NeuCinder，网络直接叫Neutron！！

华为用FusionCloud卖硬件的，我们没有，还是直接叫Openstack吧。顶多是分支。

##2.5.	用了会怎样
由于Openstack的不成熟，以及人员技术组成的不匹配，风险及成本较高。
丑话说在前头：

- 	部署复杂。openstack组件众多，组件内部有存储多个子组件或plugin，需要有完整的部署方案满足HA，LB及性能等非空能行需求，还需要对部署后的系统提供监控方案。
建议：有那么多公司只做Openstack集成。我们成立部署专门团队是必要的，Fuel是要调研的。
- 	调研成本高。
建议：成立研究小组。搭建实验环境。每天看文档，做实验，验证再验证。
- 	网络知识还不够。为什么把网络单独列出来，不提存储和计算。不光因为我网络基础差，实在是水太深。
建议：把网络工程师加到组里来，坐旁边。我们一起做实验，一起写代码。
- 	开发周期长。第一次会比较痛。
建议：忍一忍就好了。
- 	测试比较难。测试人员一直很痛，换到openstack就更痛了。
建议：人心不能散，队伍才好带


优点也是有的：

- 	看起来比较专业
- 	学习了杂七杂八的知识，开发都可以当实施用了。
- 	一个项目成了，后面都好办了。解了耦，复用也不在话下了。



#3.	虚拟机创建
##3.1.	导读
本章内容摘自：
别以为真懂Openstack: 虚拟机创建的50个步骤和100个知识点
http://www.cnblogs.com/popsuper1982/p/3927390.html

看了不少相关的文章，这一片篇写的最好的，用虚拟机创建把整个用到的组件都介绍到了，并应用了很多写的很好的文章，这篇能看懂基本上算入门了。
不过排版比较乱，把vlan原理一些底层的知识参杂进来了，下面按照自己的思路和理解来写。原文一定要看，相信看完会由衷的发出一声，原来是这么回事啊！！。

##3.2.	虚拟机创建的流程

 
图画的还行，用其他工具放大了看吧。
看的时候注意分清哪些是步骤，哪些是涉及的知识点，这么画到一起比较乱，看下面这个好多了。

 
虚拟机创建流程图，每一步的描述参考：http://www.cnblogs.com/popsuper1982/p/3800426.html

注：和openstack官方的nova开发者网站描述一致。注意在虚拟机创建开始前，镜像，网络，存储都是实现创建配置好的。

##3.3.	keystone
###3.3.1.	定义
官方的定义：
What is Keystone?
Keystone is the identity service used by OpenStack for authentication (authN) and high-level authorization (authZ). It currently supports token-based authN and user-service authorization. It has recently been rearchitected to allow for expansion to support proxying external services and AuthN/AuthZ mechanisms such as oAuth, SAML and openID in future versions.

###3.3.2.	功能
AAA包括：认证(Authentication)，授权(Authorization)，计帐(Accounting) 
keystone实现了前两个A。并且未来版本中会加入对SAML，openID等的支持。功能更加强大，也更复杂了。朝着vmware大踏步前进啊。个人感觉vmware的AA做的太啰嗦，keystone
还是不学为妙。

###3.3.3.	架构&性能
keystone的架构参考：http://docs.openstack.org/developer/keystone/architecture.html
keystone的持久层可以是关系型数据库，文件系统或者LDAP。Juno版中默认使用MairaDB。
官方提供了性能测试结果：https://wiki.openstack.org/wiki/KeystonePerformance
ldap略快于关系型数据库

openstack的设计原则要求所有操作必须是异步的，但keystone是例外的，原因很简单，AA不完成，一切操作都无法进行。

必须注意的是，由于在openstack的架构中，几乎所有操作都必须经过keystone，因此keystone必须有负载均衡和高可用方案。


##3.4.	glance
###3.4.1.	定义
What is a virtual machine image?
A virtual machine image is a single file which contains a virtual disk that has a bootable operating
system installed on it.

openstack镜像应该具备的基本功能：
• Disk partitions and resize root partition on boot (cloud-init)
• No hard-coded MAC address information
• SSH server running
• Disable firewall
• Access instance using ssh public key (cloud-init)
• Process user data and other metadata (cloud-init)
• Paravirtualized Xen support in Linux kernel (Xen hypervisor only with Linux kernel version< 3.0)

###3.4.2.	修改镜像
工具，guestfish
libguestfs提供一组工具，可以在虚拟机不启动的情况下，访问虚拟机的内部文件。
安装guestfish：yum install libguestfs-tools
打开debug：export LIBGUESTFS_DEBUG=1
执行命令：
guestfish --rw -a  centos6.5_x86_64_cloud.qcow2
><fs> run          在Centos7.0上执行run报错，换到rhel6上OK。
><fs> list-filesystems
><fs> mount /dev/vda1 /        挂载根分区后，才能进行其他操作。/dev/vda1是list-filesystems返回的结果。
><fs> edit /etc/sysconfig/modules/8021q.modules     修改文件
><fs> df          接下来的命令类似linux的bshell。

也可以用命令guestmount直接把镜像挂载到当前系统中。
guestmount -a centos6.5_x86_64_cloud.qcow2 -i --rw /mnt/guestfish/
然后可以直接安装rpm到镜像中
rpm -qa --dbpath /mnt/guestfish/var/lib/rpm

###3.4.3.	制作镜像
使用virt-manager
找一台有显示器，安装了图形界面的机器,下面操作都是在图形界面中操作 。
打开终端 ，安装：
yum install qemu-kvm
yum install libvirt
yum install virt-manager
service libvirtd start
执行命令：virt-manager
在virt-manager的图形界面上创建虚拟机即可。
创建完成的虚拟机在路径：/var/lib/libvirt/images/vm01.img
把vm01.img copy出来就行openstack要的镜像。

没有图形界面的场合，需要用qemu-img命令创建裸的虚拟机，然后用virt-install命令安装OS，之后用vnc登录虚拟机进行定制。比较麻烦。

###3.4.4.	镜像格式转换
工具qemu-img convert可以转化的格式：raw, qcow2, VDI (VirtualBox), VMDK (VMware) and VHD (Hyper-V).
qemu-img convert -f vmdk -O raw centos64.vmdk centos64.img
qemu-img convert -f raw -O qcow2 centos64.dsk centos64.qcow2
不指定-f参数，会自动检测原格式

##3.5.	nova
###3.5.1.	物理机调度
输入：http://blog.csdn.net/networm3/article/details/8783667
 
物理机调度是由nova-scheduler进行的。
调度分两部，Filter过滤和Weighting排序
	Filter过滤，有两个输入，镜像和flavor。根据镜像的格式，既hypervisor的类型；根据flaver中指定的cpu,memeory,disk等规格 对物理机进行过滤，选出满足要求的物理机。
	Weighting排序	进行虚机消耗权值的计算。通过指定的权值计算算法，计算在某物理节点上申请这个虚机所必须的消耗cost，物理节点越不适合这个虚机，消耗cost就越大，权值Weight就越大，调度算法会选择权值最小的主机。比如说在一台低性能主机上创建一台功能复杂的高级虚拟机的代价是高的。配置文件中默认的算法是主机的剩余内存越大，权值越低，就越容易被选上。

OpenStack默认支持多种过滤策略，如CoreFilter（CPU数过滤策略）、RamFilter（Ram值选择策略）、AvailabilityZoneFilter（指定集群内主机策略）、JsonFiliter（JSON串指定规则策略）。开发者也可以实现自己的过滤策略。在nova.scheduler.filters包中的过滤器有以下几种：
 
物理机调度使用了设计模式中的Strategy模式（策略模式），UML图如下
 


补充：
在horizon上创建虚拟机时，不能指定物理机，用命令行或者api可以：
nova boot --flavor m1.tiny --image cirros-0.3.3-x86_64 --nic net-id=a412a578-22f7-4dd2-bbd0-fe74cb1a8ffd --security-group default --key-name demo-key --hint OS-EXT-SRV-ATTR:host=compute201  demo-instance3
其中的—hint参数，是传递给nova-scheduler进行调度的参数。

