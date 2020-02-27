# 认识NMS与OBC

# 简述

NMS和OBC是社区中通常普遍的叫法, 也就是俗称.  

NMS指的是`net.minecraft.server`包, OBC指的是`org.bukkit.craftbukkit`包.  
在通常情况下, 利用BukkitAPI, 我们可以完成许多常规操作, 这两个包通常不会被调用.  
但是凡事也有特例, 特定情况下, BukkitAPI没有给我们造出合适的API, 我们需要自己调用“内核”部分的底层代码, 没有路, 那就自己铺一条.  

> 情况一  
> 在Minecraft 1.8 版本中, 游戏新增了Title, 允许玩家用命令方块在屏幕中投出大字Title.  
> 这么优秀的功能, 负责BukkitAPI维护工作的SpigotMC社区却没有第一时间给出向玩家发送Title的API.  
> 在1.8.8版本之前, 如果你想写插件发送Title, 你只能自己发送相应的数据包.

*在新版API中, 已经提供了相应的API, 即`Player.sendTitle`方法.*    
*如果真的需要在1.8早期版本发送Title, 一些API插件也是不错的选择,例如:*  
*https://www.mcbbs.net/thread-653804-1-1.html*

> 情况二  
> 曾经有人发过一款插件叫做`SuperBan`.  
> https://www.mcbbs.net/thread-558013-1-1.html  
> 这款插件的功能是欺骗玩家的客户端, 让玩家的客户端以为玩家周围有数以万计的点燃的TNT炸弹. 因此, 玩家的客户端会对TNT爆炸进行相应的计算, 这样的计算会超过玩家客户端JVM的负载限制, 玩家的客户端会因此崩溃. 因此达到了该插件标称的“崩了他的客户端”的效果.    
> 显然, 在服务器的World中, 那位玩家周围实际是没有TNT炸弹的, 这个插件自己单方面的发送了相应的数据包欺骗了这位玩家的客户端. 这样的“欺骗”行为, 如果你正儿八经使用BukkitAPI, 无法编写出来.  
> 所以, 如果需要实现这样的功能, 只能靠插件向该玩家客户端发送TNT爆炸数据包.  

BukkitAPI无法完美地满足我们的所有需求. 例如, 我们有时需要手动发送数据包给玩家客户端.  
这时, 我们就不得不跟底层内容“打交道”.  

对于原版Minecraft服务器而言, 底层就是NMS.  
对于所有BukkitAPI的服务端而言, Bukkit底层就是OBC. 我们称呼的`CraftBukkit`指代的就是OBC.  

**由于OBC在实际开发者使用的机会极为稀少, 而NMS相对较多. 因此本文将着重描述NMS的使用, 重点侧重常见的发包问题和NBT操作问题.**

# 我们经常用到反射

我们经常用到反射！  

如果你需要使用NMS或OBC的代码, 它们都属于`net.minecraft.server`或`org.bukkit.craftbukkit`包. 但是NMS具体的包名随着MC版本的变化而变化.  
例如1.14.3的NMS包名为`net.minecraft.server.v1_14_R1`. OBC同理, 1.14.3的OBC包名为`org.bukkit.craftbukkit.v1_14_R1`.

如果你需要调用NMS或者OBC, 不使用反射, 你只能针对每个MC版本都分别推出一款对应的插件, 例如:
对于1.7.2你需要单独改变包名写一个插件, 1.7.10单独写一个插件, 1.8.1单独一个插件, 1.8.2单独一个插件......以此类推, 一共几个Minecraft版本, 就应该有几个插件.  

那么对于一些跨版本的, 比如一款兼容多个版本服务端的插件而言, 这样维护所消耗的时间成本十分巨大, 这很不现实.  
因此, 通常情况开发插件时, 如果涉及NMS调用, 往往是用反射调用的.

**反射属于Java基础内容, 在本文中不予以赘述.**  
*但是在使用时, 会稍加解释*