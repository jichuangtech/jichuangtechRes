## 							Android内核开发

#### 一：系统分区

1. 概念：

   **ROM**, **RAM**的区别

   - 随机存取存储器（random access memory，RAM）又称作“[随机存储器](http://baike.baidu.com/view/151093.htm)”，是与[CPU](http://baike.baidu.com/view/2089.htm)直接交换数据的[内部存储器](http://baike.baidu.com/view/2397718.htm)，也叫[主存](http://baike.baidu.com/view/313837.htm)(内存)。它可以随时读写，而且速度很快，通常作为[操作系统](http://baike.baidu.com/view/880.htm)或其他正在运行中的程序的临时数据存储媒介。当电源关闭时RAM不能保留[数据](http://baike.baidu.com/view/38752.htm)。如果需要保存数据，就必须把它们写入一个长期的存储设备中（例如[硬盘](http://baike.baidu.com/view/4480.htm)）。RAM和[ROM](http://baike.baidu.com/view/15546.htm)相比，两者的最大区别是RAM在[断电](http://baike.baidu.com/view/631268.htm)以后保存在上面的数据会自动消失，而[ROM](http://baike.baidu.com/subview/15546/11756847.htm)不会自动消失，可以长时间断电保存

   - 读存储器，英文简称ROM。ROM所存数据，一般是装入整机前事先写好的，整机工作过程中只能读出，而不像随机[存储器](http://baike.baidu.com/view/87697.htm)那样能快速地、方便地加以改写。ROM所存数据稳定，断电后所存数据也不会改变。从电脑来说一般比较好理解，RAM就是我们平时所说的运行内存，它的确是随时可读写的。因为CPU处理的数据都是以运行内存为中介的。断电后信息是不保存的。那么对于ROM来说，是不是就是硬盘呢？不是说ROM只可以读吗？硬盘却是可以修改的。的确，必须明确一点，RAM与ROM都是内存，而硬盘是外存，所以ROM不等于硬盘。计算机中的ROM主要是用来存储一些系统信息，或者启动程序BIOS程序，这些都是非常重要的，只可以读一般不能修改，断电也不会消失。

     那么对于手机来说呢？其实很多困惑都来自于手机厂商的宣传信息的误导。因为一般手机厂商都会说有多少G的RAM,多少G的ROM;在手机里面，RAM就是跟电脑一样的运行内存一样；而ROM就不一样了，你想想看，如果只用来存储一些系统信息和开机引导程序，需要几个G的容量?其实手机的ROM就跟硬盘挂上钩了，手机中的ROM有一部分用来存储系统信息，还有一些装机软件，剩余的大部分容量都是就是拿来作为硬盘用的，**可读可写**；

2. 分类

   - **System**分区: 就是我们刷ROM的分区

     这里是挂载到/system目录下的分区。这里有 /system/bin 和 /system/sbin 保存很多系统命令。它是由编译出来的system.img来烧入。

     相当于你电脑的C盘，用来放系统。这个分区基本包含了整个安卓操作系统，除了内核（kerne）和ramdisk。包括安卓用户界面、和所有预装的系统应用程序。擦除这个分区，会删除整个安卓系统。你可以通过进入Recovery程序或者bootloader程序中，安装一个新ROM，也就是新安卓系统

   - **Data**分区:   分区就是我们装APK的分区

   - **Catch**分区：是缓存分区

   - **SDCard**分区：就是挂载的SD卡

   - **recovery**分区 ：recovery 分区即恢复分区，在正常分区被破坏后，仍可以进入这一分区进行备份和恢复.我的理解是这个分区保存一个简单的OS或底层软件，在Android的内核被破坏后可以用bootloader从这个分区引导进行操作。

           这个分区可以认为是一个boot分区的替代品，可以是你的手机进入Recovery程序，进行高级恢复或安卓系统维护工作

   - **boot**分区：

     - 这个分区负责手机开机。它包括内核和内存。如果没有这个分区，设备将无法启动。仅在非常需要的情况下，可以通过recovery擦除这个分区。并且，一旦擦除该分区，在安装一个包含/boot分区的新系统前，设备不能够被启动

     - 一般的嵌入式Linux的设备中.bootloader,内核，根文件系统被分为三个不同分区。在[Android](http://lib.csdn.net/base/15)做得比较复杂，从这个手机分区和来看，这里boot分区是把内核和ramdisk file的根文件系统打包在一起了，是编译生成boot.img来烧录的。   

       如果没有这个分区，手机通常无法启动到安卓系统。只有必要的时候，才去通过Recovery软件擦除（format）这个分区，一旦擦除，设备只有再重新安装一个新的boot分区，可以通过安装一个包含boot分区的ROM来实现，否则无法启动安卓系统；

   - **userdata**分区：

     它将挂载到 /data 目录下, 它是由编译出来的userdata.img来烧入。

     这个分区也叫用户数据区，包含了用户的数据：联系人、短信、设置、用户安装的程序。擦除这个分区，本质上等同于手机恢复出厂设置，也就是手机系统第一次启动时的状态，或者是最后一次安装官方或第三方ROM后的状态。在Recovery程序中进行的“data/factory reset ”操作就是在擦除这个分区；

   - **cached**分区：它将挂载到 /cache 目录下。这个分区是安卓系统缓存区，保存系统最常访问的数据和应用程序。擦除这个分区，不会影响个人数据，只是删除了这个分区中已经保存的缓存内容，缓存内容会在后续手机使用过程中重新自动生成

3. 镜像文件烧写

   - fastboot工具

   - 在uboot命令下：

     通常在u-boot启动过程中，会有3秒的停留，在串口终端敲击回车中断u-boot启动内核的过程，这时就可以输入u-boot支持的各种命令与板子交互了；

#### 二：系统启动流程

1. bootloader：它通常由处理器的片上**ROM中的引导代码**和**u-boot**两部分组成

#### 三：编译系统

1. [理解 Android Build 系统](https://www.ibm.com/developerworks/cn/opensource/os-cn-android-build/)

   - 配置 (/device/xxxx/prodcut/AndroidProducts.mk)

     下面的属性在 lunch() 命令中被定义

     - 要构建的Android风格：TARGET_PRODUCT 
     - 选择哪些模块进行安装： TARGET_BUILD_VARIANT 
     - 所构建的目标

   - 编译目标类型

     - BUILD_PACKAGE（既可以编apk，也可以编资源包文件，但是需要指定LOCAL_EXPORT_PACKAGE_RESOURCES:=true)
     - BUILD_JAVA_LIBRARY（java共享库）
     - **BUILD_STATIC_JAVA_LIBRARY**（java静态库），例如 *.jar 这类文件
     - BUILD_EXECUTABLE（执行文件）
     - BUILD_SHARED_LIBRARY（native共享库），例如：so库
     - **BUILD_STATIC_LIBRARY**（native静态库）

   - 编译目标类型的几个区别
     - 简单的说，jar包其实就是一个zip格式的压缩包，那么：
       1. BUILD_JAVA_LIBRARY编译出来的jar包，里面是DEX格式的文件，如果用户想用这个jar包放到Eclipse来做Android APP的开发，Eclipse是不认识这种格式的文件的，通常会报错：Conversion to Dalvik format failed with error 1；
       2. 而BUILD_STATIC_JAVA_LIBRARY编译出来的jar包，里面每个java文件对应的class文件都单独存在，顾名思义，每个java文件里面用到的变量都被静态编译到了class内部，这种格式的jar包可以在Eclipse里面导入并正常使用，但是可能存在一定的兼容性隐患，这个另外讨论。

   - 属性解释
     - LOCAL_MODULE_CLASS来指定模块的类型，标识了所编译模块最后放置的位置，如果不指定，不会放到系统中，之后放在最后的obj目录下的对应目录中

       - 编译jar包：LOCAL_MODULE_CLASS := JAVA_LIBRAYIES

         放在表示放于**/system/jar**目录下

       - 定义动态库文件目标：LOCAL_MODULE_CLASS := SHARED_LIBRAYIES

         放在**/system/lib**下

       - 编译可执行文件：LOCAL_MODULE_CLASS := EXECUTABLES

         放在**/system/bin**

       - 编译App：LOCAL_MODULE_CLASS := APPS

         放在**/system/app**下面

       - 编译etc下面的文件: LOCAL_MODULE_CLASS := ETC

          放在**ETC**目录

     - PRODUCT_PACKAGES：用来指定哪些子模块中的 apk,so,等内容可以被编译到 system.img 中

2. 在系统中添加内容

   - 添加新产品

   - 添加新模块

     - apk，so，jar（无源码）

       ```makefile
       include $(BUILD_PREBUILT)
       ```

       ```makefile
       No private recovery resources for TARGET_DEVICE a16v2w11
       make: Entering directory `/var/aosp'
       target Prebuilt APK: TestApkPrebuilt (out/target/product/a16v2w11/obj/APPS/TestApkPrebuilt_intermediates/TestApkPrebuilt)
       Install: out/target/product/a16v2w11/system/app/TestApkPrebuilt
       make: Leaving directory `/var/aosp'
       ```

     - apk（带有源码）

       ````makefile
       include $(BUILD_PACKAGE)
       ````

       ```makefile
       make: Entering directory `/var/aosp'
       Install: out/target/product/a16v2w11/system/app/ZXICMainControl.apk
       make: Leaving directory `/var/aosp'
       ```

     - java 静态库

       ```makefile
       include $(BUILD_STATIC_JAVA_LIBRARY)
       ```

       ```makefile
       make: Entering directory `/var/aosp'
       target Java: testJar (out/target/common/obj/JAVA_LIBRARIES/testJar_intermediates/classes)
       Copying: out/target/common/obj/JAVA_LIBRARIES/testJar_intermediates/classes-jarjar.jar
       Copying: out/target/common/obj/JAVA_LIBRARIES/testJar_intermediates/emma_out/lib/classes-jarjar.jar
       Copying: out/target/common/obj/JAVA_LIBRARIES/testJar_intermediates/classes.jar
       target Static Jar: testJar (out/target/common/obj/JAVA_LIBRARIES/testJar_intermediates/javalib.jar)
       make: Leaving directory `/var/aosp'
       ```

       使用**out/target/common/obj/JAVA_LIBRARIES/testJar_intermediates/classes.jar**

     - so动态库

       ```makefile
       include $(BUILD_SHARED_LIBRARY)
       ```

       ```makefile
       make: Entering directory `/var/aosp'
       Import includes file: out/target/product/a16v2w11/obj/SHARED_LIBRARIES/testSo_intermediates/import_includes
       target thumb C: testSo <= vendor/starnet/external/testSo/main.c
       target SharedLib: testSo (out/target/product/a16v2w11/obj/SHARED_LIBRARIES/testSo_intermediates/LINKED/testSo.so)
       target Symbolic: testSo (out/target/product/a16v2w11/symbols/system/lib/testSo.so)
       Export includes file: vendor/starnet/external/testSo/Android.mk -- out/target/product/a16v2w11/obj/SHARED_LIBRARIES/testSo_intermediates/export_includes
       target Strip: testSo (out/target/product/a16v2w11/obj/lib/testSo.so)
       Install: out/target/product/a16v2w11/system/lib/testSo.so
       make: Leaving directory `/var/aosp'
       ```

     - c/c++可执行程序

       ```makefile
       include $(BUILD_EXECUTABLE)
       ```

       ```makefile
       make: Entering directory `/var/aosp'
       Import includes file: out/target/product/a16v2w11/obj/EXECUTABLES/testExe_intermediates/import_includes
       target thumb C: testExe <= vendor/starnet/external/testExe/main.c
       target Executable: testExe (out/target/product/a16v2w11/obj/EXECUTABLES/testExe_intermediates/LINKED/testExe)
       target Symbolic: testExe (out/target/product/a16v2w11/symbols/system/bin/testExe)
       Export includes file: vendor/starnet/external/testExe/Android.mk -- out/target/product/a16v2w11/obj/EXECUTABLES/testExe_intermediates/export_includes
       target Strip: testExe (out/target/product/a16v2w11/obj/EXECUTABLES/testExe_intermediates/testExe)
       Install: out/target/product/a16v2w11/system/bin/testExe
       make: Leaving directory `/var/aosp'
       ```

       如果像重复生成该可执行程序，则需要删除**out/target/product/a16v2w11/obj/EXECUTABLES/testExe_intermediates**这个目录

3. 系统签名

   - [编译release版本签名系统](https://www.cnblogs.com/leaven/p/3860583.html)

#### 四：层次结构

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fuixdu87xvj31c80t4wq2.jpg)

![](https://ws1.sinaimg.cn/large/0069RVTdgy1fuo4z6sytwj31hs0c442e.jpg)

1. 应用程序层：Android App，以Apk形式运行着；

    该层提供一些核心应用程序包，例如电子邮件、短信、日历、地图、浏览器和联系人管理等。同时，开发者可以利用[Java](http://lib.csdn.net/base/17)语言设计和编写属于自己的应用程序，而这些程序与那些核心应用程序彼此平等、友好共处；

2. 系统框架层--系统服务&SDK接口

   - 该层是Android应用开发的基础，开发人员大部分情况是在和她打交道。应用程序框架层包括活动管理器、窗口管理器、内容提供者、视图系统、包管理器、电话管理器、资源管理器、位置管理器、通知管理器和XMPP服务十个部分。在Android平台上，开发人员可以完全访问核心应用程序所使用的API框架。并且，任何一个应用程序都可以发布自身的功能模块，而其他应用程序则可以使用这些已发布的功能模块。基于这样的重用机制，用户就可以方便地替换平台本身的各种应用程序组件。
   - 各个系统服务对外提供API接口，形成SDK供应用程序层调用，因此 
   - 疑问🤔️
     - init 进程启动的初始化java程序以什么形式存在系统中， bin ?
   - 参考文章：
     - [框架系统服务运行原理](https://blog.csdn.net/geyunfei_/article/details/78851024)

3. Android的硬件抽象层：可以将HAL层看作是图中硬件库装载器，不同的硬件类型定义了不同的头文件，这些头文件用于定义硬件库.so文件的API接口：

   - [HAL解析一](https://www.cnblogs.com/microliang/p/3424311.html)
   - [HAL解析二](https://www.cnblogs.com/microliang/p/3427322.html)
   - [HAL解析三](https://www.cnblogs.com/microliang/p/3436147.html)

4. 驱动层

   驱动广义的意思：驱使硬件进行动作的程度代码；

5. 硬件

   - 参考文章:
     - [Android 从硬件到应用：一步一步向上爬 1 -- 从零编写底层硬件驱动程序](https://blog.csdn.net/wu20093346/article/details/41871429)

#### 五：内部结构

1. Manifest文件

   - [自定义权限-<permission>](https://blog.csdn.net/weixin_37077539/article/details/56279789)

2. 跨进程通信

   - Binder
   - ashmen

3. 日志系统

   - 内核日志：dmesg 命令。日志通常保存在 /var/log 目录下。但是由于系统重启时就会丢失，所以Android增加了一个驱动程序并注册成为一个基于内存的控制台，其内容可以通过 **/proc/last_kmsg** 来访问


#### 五：开机动画

1. 一个文件路径在BootAnimation.cpp的开头有定义:

   ```c
   //这个应该是OEM厂商自己定制
   define OEM_BOOTANIMATION_FILE "/oem/media/bootanimation.zip"   
   //正常情况下的Android原始开机动画
   #define SYSTEM_BOOTANIMATION_FILE "/system/media/bootanimation.zip"   
   //加密手机的开机动画
   #define SYSTEM_ENCRYPTED_BOOTANIMATION_FILE "/system/media/bootanimation-encrypted.zip" 
   ```


#### 六：特有名词

1. 内存 DDR：*DDR内存*全称是*DDR* SDRAM(Double Data Rate SDRAM，双倍速率SD**RAM**）；

2. Dalvik：Dalvik是Google公司自己设计用于Android平台的虚拟机；

3. Zygote：它是Android系统最重要的进程，所有后续的Android应用程序都是由它fork出来的；

4. android固件：固化的软件，英文为firmware，它是把某个系统程序写入到特定的[硬件系统](https://www.baidu.com/s?wd=%E7%A1%AC%E4%BB%B6%E7%B3%BB%E7%BB%9F&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)中的flashROM。 　　[手机固件](https://www.baidu.com/s?wd=%E6%89%8B%E6%9C%BA%E5%9B%BA%E4%BB%B6&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)相当于手机的系统，刷新固件就相当于刷系统。不同的手机对应不同的固件，在刷固件前应该充分了解当前固件和所刷固件的优点缺点和兼容性， 并做好充分的准备

5. 

    

#### 七：参考

1. 罗生阳：
   1. https://blog.csdn.net/Luoshengyang/

2.编译系统

https://www.ibm.com/developerworks/cn/opensource/os-cn-android-build/

