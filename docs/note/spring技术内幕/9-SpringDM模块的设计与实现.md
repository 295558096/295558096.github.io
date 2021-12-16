# Spring DM模块的设计与实现

## Spring DM模块的应用场景

- Spring DM模块是Spring的一个子项目，它的主要作用是为那些想要移植到OSGi平台上的Spring应用提供支持。
- 利用Spring DM可以很便利地在主流的OSGi平台和Spring应用之间搭建一座桥梁。
- 通过使用Spring DM，不仅能够保持原有Spring应用的编程模式，比如IoC、AOP的使用，还可以通过OSGi平台来使用OSGi的特性，加强Spring应用的模块化。

### OSGi

- Open Service Gateway Initiative
- 通常指由OSGi联盟制定的、基于Java语言的服务规范。
- OSGi服务平台用于解决Java环境下运行的模块的动态管理问题。

#### 定义

OSGi定义了bundle，作为一个基本的模块单位，这是在OSGi平台中进行模块化管理的基本单元。

相对于war和jar这样的Java开发中常见的模块定义，bundle的定义更为细致，而且这个bundle的生命周期与Java的模块定义不同，它是指在运行状态的生命周期，这个生命周期是软件模块bundle在部署运行后激活和产生的。在OSGi容器中，通过对bundle生命周期进行管理，提高了对bundle管理的动态性和精细程度，从而可以在这个生命周期的基础上加入各种各样的管理特性。

#### 生命周期

以状态机形式表现的bundle生命周期中可以看到`Installed`、`Resolved`、`Starting`、`Active`、`Stopping`和`Uninstalled`等几个状态。

![00101](9-SpringDM%E6%A8%A1%E5%9D%97%E7%9A%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0-image/00101.jpeg)

## Spring DM的应用过程

略

## Spring DM设计与实现

略