# Container (docker) Tips


[Container Security -- By Liz Rice (OReilly)](https://github.com/XinCai/docker_tips/blob/09d335ce35417d34ca9df6b7fc89a3ad2b04a65c/Container%20Security%20by%20Liz%20Rice%20-%20OReilly%20Apr%202020.pdf "book")

## docker container 安全考虑类别 和 Tips

#### 1.Vulnerable application code

The best way to avoid running containers with known vulunerabilities is to scan images 
扫描 docker images, 持续扫描 scanning process application 是否使用了 out of date packages （考虑使用snyk ， 来持续scan application code package）

#### 2. Badly configured container images
配置 container 的时候注意事项 configuring container to run as the root user

#### 3. Build machine attacks

#### 4. Supply chain attacks 

images stored in a registry, 当用户 pull images 时候，确定 pulled 的image 是 当时 pushed 的images.

#### 5. Badly configured containers

#### 6. Vulnerable hosts
容器运行在host machine上面，确定 hosts上 没有运行 vulnerable code, 好的方法是 确定 minimize the amount of software installed on each host.

#### 7. Exposed secrets
Application code 需要配置 密码 或 证书 一类的来 communicate with other components in a system. 

#### 8. Insecure networking
不安全的网络， container 需要 通信 其他的 containers,  或者 outside world


## 安全边界 （security boundaries）
1. 容器是一个security boundary, 运行在container 内的 代码 不应该访问 容器外部的code or data 
2. 数量越多的 security boundaries，安全性越高， 加强每一层的 boundary, 安全性越高

## 多租户 Multitenancy 

共享计算资源 （sharing machine resources） 被成为 `multitenancy`
在一个多租户的环境，不同的users (或者成为 tenants) 运行他们的 workloads 在共享的硬件（或者计算资源）， 需要有 更强的 boundaries between 他们的 workloads, 来确保之间不会被干扰。

### Virtualization

虚拟机 VM -- 有操作系统的虚拟机，通常来讲，虚拟机之间有强力的隔离效果， 意思是 邻居 （neighbors）大多不会 观察到或者 干扰到其他的 租户 (tenant)在 VM. 
多租户 (multitenancy): 意思是 多个不同的group people 共享一个软件  (share a single instance of same software)， 多租户是指软件架构支持一个实例服务多个用户（Customer），每一个用户被称之为租户（tenant），软件给予租户可以对系统进行部分定制的功能。

```
例子：
操作系统 --- 单租户系统
电子邮件 --- 多租户系统
```

### Container Multitenancy 
容器的多租户
1. 在 kubernetes 世界里，使用 `namespace` 来细分 a cluster of machines 机器集群, 被不同的团队，个人和软件来使用。
2. 使用 RBAC （role based access control）来限制 people 和  components 来访问 kubernetes的 不同namespaces


## 安全原则 （security principal）

#### 1. Least Privilege （最低权限 原则）
最低权限原则， 对个人 或者 一个组成员 给出最低的权限， 不超过其职能范围的访问权限。

#### 2. Defense in Depth (深度防御 原则)
多层面 protection， layers of protection.
一个layer 出现了安全状况， 另一个layer 会保护你的部署。 

#### 3. Reducing the Attack Surface （减少攻击面 原则）

1. Reducing access points by keeping interfaces small and simple where possible
2. Limiting the users and components who can access a service
3. Minimizing the amount of code

#### 4. Limiting the Blast Radius （限制爆炸半径 原则）

分割安全控制 表示 当出现 安全事件时，影响是有限的。容器非常适合遵循这一项原则， 因为将架构划分为多个微小的实例服务 micro service， 容器本身就可以充当安全边界。

#### 5. Segregation of Duties （指责分工）
根据人员的职责 来给于特定的 权限。


## File Permissions (文件权限是 安全的基石)  
在 linux 操作系统下，everything is a file. 

### 理解 setuid 位

通常，在类 Unix 操作系统上，文件和目录的所有权是基于文件创建者的默认 uid（user-id）和 gid（group-id）的。启动一个进程时也是同样的情况：它以启动它的用户的 uid 和 gid 运行，并具有相应的权限。这种行为可以通过使用特殊的权限进行改变。


当使用 setuid （设置用户 ID）位时，之前描述的行为会有所变化，所以当一个可执行文件启动时，它不会以启动它的用户的权限运行，而是以该文件所有者的权限运行。所以，如果在一个可执行文件上设置了 setuid 位，并且该文件由 root 拥有，当一个普通用户启动它时，它将以 root 权限运行。显然，如果 **setuid 位使用不当的话，会带来潜在的安全风险**

Because setuid provides a **dangerous pathway** to privilege escalation,

使用 setuid 权限的可执行文件的例子是 passwd，我们可以使用该程序更改登录密码。我们可以通过使用 ls 命令来验证：
```
ls -l /bin/passwd
-rwsr-xr-x. 1 root root 27768 Feb 11 2017 /bin/passwd
```
如何识别 setuid 位呢？相信您在上面命令的输出已经注意到，setuid 位是用 s 来表示的，代替了可执行位的 x。小写的 s 意味着可执行位已经被设置，否则你会看到一个大写的 S。大写的 S 发生于当设置了 setuid 或 setgid 位、但没有设置可执行位 x 时。它用于提醒用户这个矛盾的设置：如果可执行位未设置，则 setuid 和 setgid 位均不起作用。setuid 位对目录没有影响。

#### Docker `--no-new-privileges` flag on a `docker run` command


## Linux Capabilities 

**什么是 Linux Capabilities**  
为了执行权限检查，Linux 区分两类进程：

**特权进程**(其有效用户标识为 0，也就是超级用户 `root`) 
和
**非特权进程**(其有效用户标识为非零)。 

特权进程绕过所有内核权限检查，而非特权进程则根据进程凭证(通常为有效 UID，有效 GID 和补充组列表)进行完全权限检查。

以常用的 passwd 命令为例，修改用户密码需要具有 root 权限，而普通用户是没有这个权限的。但是实际上普通用户又可以修改自己的密码，这是怎么回事？在 Linux 的权限控制机制中，有一类比较特殊的权限设置，比如 SUID(Set User ID on execution)，不了解 SUID 的同学请参考《Linux 特殊权限 SUID,SGID,SBIT》。因为程序文件 `/bin/passwd` 被设置了 SUID 标识，所以普通用户在执行 passwd 命令时，进程是以 `passwd` 的所有者，也就是 `root` 用户的身份运行，从而修改密码。

SUID 虽然可以解决问题，却带来了安全隐患。当运行设置了 SUID 的命令时，通常只是需要很小一部分的特权，但是 SUID 给了它 `root` 具有的全部权限。因此一旦 被设置了 SUID 的命令出现漏洞，就很容易被利用。也就是说 SUID 机制在增大了系统的安全攻击面。

Linux 引入了 capabilities 机制对 root 权限进行细粒度的控制，实现按需授权，从而减小系统的安全攻击面。本文将介绍 capabilites 机制的基本概念和用法。

在执行特权操作时，如果进程的有效身份不是 `root`，就去检查是否具有该特权操作所对应的 capabilites，并以此决定是否可以进行该特权操作。比如要向进程发送信号(`kill()`)，就得具有 capability `CAP_KILL`；如果设置系统时间，就得具有 capability `CAP_SYS_TIME`。

下面是从 capabilities man page 中摘取的 capabilites 列表：

| capability 名称 |描述|
| ----------- |------|
|CAP_AUDIT_CONTROL |	启用和禁用内核审计 改变审计过滤规则；检索审计状态和过滤规则 |
|CAP_AUDIT_READ	| 允许通过 multicast netlink  套接字读取审计日志|
|CAP_AUDIT_WRITE	|将记录写入内核审计日志 |
|CAP_BLOCK_SUSPEND |	使用可以阻止系统挂起的特性 |
|CAP_CHOWN |	修改文件所有者的权限 |
|CAP_DAC_OVERRIDE	|忽略文件的 DAC 访问限制 |
|CAP_FSETID	|允许设置文件的 setuid 位|
|CAP_IPC_LOCK |	允许锁定共享内存片段|
|CAP_IPC_OWNER |	忽略 IPC 所有权检查|
|CAP_KILL |	允许对不属于自己的进程发送信号|
|CAP_LEASE |	允许修改文件锁的 FL_LEASE 标志|
|CAP_LINUX_IMMUTABLE |	允许修改文件的 IMMUTABLE 和 APPEND 属性标志|
|CAP_MAC_ADMIN |	允许 MAC 配置或状态更改|
|CAP_MAC_OVERRIDE|	覆盖 MAC(Mandatory Access Control)|
|CAP_MKNOD|	允许使用 mknod() 系统调用|
