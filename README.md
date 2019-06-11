# Sharemind MPC使用指南

目录
=================
   
   * [Sharemind MPC使用指南](#sharemind-mpc使用指南)
      * [前言](#前言)
      * [获取Sharemind](#获取sharemind)
         * [Sharemind虚拟机](#sharemind虚拟机)
            * [示例工程](#示例工程)
            * [使用QtCreator启动服务器](#使用qtcreator启动服务器)
         * [自己部署Sharemind](#自己部署sharemind)
      * [Sharemind程序开发](#sharemind程序开发)
         * [SecreC的使用](#secrec的使用)
         * [web应用程序](#web应用程序)
            * [启动Sharemind网关](#启动sharemind网关)
            * [SecreC程序处理客户端参数](#secrec程序处理客户端参数)
            * [前端数据处理](#前端数据处理)

## 前言

文中很多内容都参考了[Sharemind文档页](https://docs.sharemind.cyber.ee)，这里提供了从入门到进阶的详细资料。本教程重点关注入门和我的使用经验。

## 获取Sharemind

要获取Shareind的服务有两种方式：

1.在[Sharemind SDK](https://sharemind-sdk.github.io/)页面下载Sharemind虚拟机，其中预装了包含Sharemind各种组件的```Debian```系统，免去了配置环境的麻烦，可以快速投入使用，足够用来进行开发与测试，不需要申请，性能不如后面提到的自己部署的方式。该页面还包含了SecreC的文档。**邮件中有一个内含完整示例工程的虚拟机下载地址，下载这个即可。**

2.从[Sharemind MPC](https://sharemind.cyber.ee/sharemind-mpc/)页面申请学术许可证（一年）、软件试用版或正式版，获取Sharemind的各种组件，自己部署到服务器上，性能优于前者，可用于生产环境。

建议先下载并参考虚拟机内的示例工程，后面会引用到其中的文件。

### Sharemind虚拟机
下载完虚拟机后可以通过```VirtualBox```载入。

#### 示例工程

虚拟机桌面的```Demos```文件夹包含了三个示例工程，分别是SecreC的基本使用、Sharemind服务器的使用以及一个完整web应用，在各个项目文件夹下的```README.md```有使用说明。其中```secrec-demo```工程里包含了很多使用基本的API的```.sc```源代码，值得参考。

#### 使用QtCreator启动服务器
虚拟机内安装的```QtCreator```集成了Sharemind的一些Shell脚本，包括编译SecreC程序、启动Sharemind服务器等，可以从菜单栏的```Tools->External->Sharemind SDK```直接使用。这些脚本文件都在```/usr/local/bin```文件夹中，可以修改或者参考其中的命令。

要执行一个SecreC程序，首先需要编译该文件，之后开启Sharemind服务器，然后就可以运行该程序，这些操作都可以在用```QtCreator```打开该文件后，使用上边所说的菜单栏命令完成。如果要运行web应用，还需要开启用```Node.js```写成的Sharemind网关，具体可以参考该项目的readme。

在开启Sharemind服务器前，需要把邮件内的```.p7b```许可证文件放在```/home/sharemind/Desktop/Demos/sharemind```里三个```server```文件夹中（如果已经有就不用放了）。```server```文件夹下的```server.cfg```保存了该服务器的设置文件，其中最下面的```server address book```配置保存了其他服务器的识别信息，包括服务器地址、名称以及公钥名称。有时服务器会提示因权限不足无法执行SecreC编译出的```.sb```文件，这时需要使用```chmod```来修改```.sb```文件的访问权限。

### 自己部署Sharemind

学术许可证申请成功后，会在邮件内发给你```.p7b```结尾的许可证文件、一个虚拟机以及Sharemind APT仓库的登录凭据，通过该仓库可以下载Sharemind的各种组件。

[Sharemind文档页](https://docs.sharemind.cyber.ee)提供了从安装到部署的详细教程、如何使用SecreC和Rmind以及详细的API文档，非常重要。

安装和部署过程一些要注意的问题，这些都包含在[文档页的安装教程](https://docs.sharemind.cyber.ee/2019.03/installation)中：
- 需要用到邮件中的APT登陆凭据添加软件源，之后才能用```apt-get```进行安装。
- 许可证文件需要放在```/etc/sharemind/server.conf```中```LicenseFile```字段的位置。
- 每个服务器都需要生成公私钥对，其中公钥需要共享给其他服务器，放在配置文件指定的位置。

## Sharemind程序开发

### SecreC的使用

文档页提供了[SecreC的基本使用方法](https://docs.sharemind.cyber.ee/2019.03/development/secrec-tutorial)，SecreC的很多语法与C接近。

SecreC的每个变量都属于公有域或者保护域，在保护域内的变量会在程序执行过程中保持加密，公有域则不会。

```
//引入保护域模块
import shared3p;

//声明保护域
domain pd_shared3p shared3p;

public uint foo;            //声明公有域变量foo
pd_shared3p uint[[1]] bar;  //声明保护域变量bar，[[1]]代表一维数组
uint arr[[2]];              //默认为公有域，[[2]]代表二维数组
```

公有域的值可以隐式转换到保护域，但保护域的值不能隐式转换到到公有域。函数```declassify(x)```可以将一个在保护域的值x转换到公有域，因为一些函数只接受公有域输入，可以临时使用这个函数处理输入。

Sharemind为操作数组和矩阵提供了方便的API，```Demos/secrec-demo/basic/hellosecrec-commented.sc```展示了基本API以及数组操作的使用。
文件```Demos/secrec-demo/database/tabledb_advanced.sc```
提供了操作数据库的各个API的使用方式。

### web应用程序

通过构建Sharemind web应用程序，可以在客户端执行服务器上的SecreC程序并返回结果。虚拟机的```/home/sharemind/Desktop/Demos/web-demo```包含了一个完整的web程序，包括客户端和服务器的代码，可以直接用readme中的命令启动，下面说明一些配置和使用的API。

#### 启动Sharemind网关

Sharemind网关为客户端提供访问服务端SecreC程序的接口，用node.js写成。```gateway1/2/3.cfg```是三个Sharemind网关的配置文件，保存其连接的服务器信息。文件```server/nodejs/gateway.js```是Sharemind网关的启动文件，其中最重要的是```scriptsInfoMap```配置，下面的配置表示客户端通过字符串'verify-result'可以调用到服务器的'app_verify_result.sb'程序：

```
scriptsInfoMap['verify-result'] = {
  name: 'app_verify_result.sb',
  type: 'multi',
  otherServerNames: otherServerNames
};
```

配置完成后，需要先启动Sharemind服务器，然后执行```server/nodejs/run_gateways.sh```脚本。该脚本启动三个Sharemind网关，与三个Sharemind服务器交互。如果无法启动，可以试者把```gateway.js```最后的
```server.listen(gatewayPort, gatewayHostname)```
改为
```server.listen(gatewayPort)```

#### SecreC程序处理客户端参数

在```server/secrec```中包含了服务器端的SecreC代码，可用来参考。在Sharemind中，服务端接收到的客户端参数使用字典结构存储，通过```argument("foo")```可以获取数据字典中```foo```索引对应的值。

同样，服务端传给客户端的数据也使用字典。```publish("bar", foo)```表示传回的数据中```bar```索引对应```foo```的值。

#### 前端数据处理

文件```client/web/js/ext/sharemind-web-client.js```是Sharemind的js代码API库。```client/web/js/app.js```文件展示了如何使用这个库调用服务器的程序并交换数据，修改后可以用在别的应用程序的开发。

```app.js```首先包含了三个网关的地址。文件里的```app.vote```函数是一个典型的调用服务端程序并获取输出的函数，可以按照这个框架编写其他的函数，我们在下面对其进行分析。
```
app.vote = function (candidateId, callback) { 
//输入参数candidateId，callback表示回调函数
    app.log('Voting for candidate #'+candidateId); //控制台输出

    // 函数connectGateways连接网关，输入参数为连接成功后执行的函数
    connectGateways(function(err) {
      if (err) {
        // 出错时执行的代码
        if (callback) callback(err);
        return;
      }

      // 创建公有域的值，类型是8位整数，1代表只有一个元素，
      var pub_value = new pub.Uint8Array(1);
      // 将该值设置为传递进来的参数candidateId的值
      pub_value.set(0, new BigInteger(''+candidateId, 10));
      // 将该值转换为保护域的值，类型为8位整数
      var private_value = new priv.Uint8Array(pub_value);
      
      //传给服务端的参数字典
      var args = {};
      //设置字典的值
      args["candidateId"] = private_value;

      app.log("Saving vote.");
      
      //该函数运行服务端"save-vote"对应的程序，并将args发送作为输入
      //"save-vote"对应哪个程序的设置保存在上面提到的scriptsInfoMap中
      gatewayConnection.runMpcComputation("save-vote", args,
        function(err) {
          //出错时执行
          if (err) {
            app.error(err);
            if (callback) callback(err);
            return;
          }
          app.log("Successfully saved vote.");

          $(window).trigger("allDone");

          // 成功时触发回调函数:
          if (callback) callback();
        });

    });
  };
```

很重要的一点是前端传过去的数据类型必须和后端保存该数据的变量的类型一致。

要使用```app.js```提供的函数，可以参考```client/web```里```results.html```和```vote.html```里的使用方法。

首先要调用```app.init()```进行初始化，然后创建回调函数

```
var clbck = function(err, results){
  ...//处理results
}
```

来指定接收到数据后的操作，其中```results```指定了接收到的结果字典的变量名，可以通过```results["foo"]```的方式访问结果字典。最后将该回调函数和其他参数作为输入来调用```app.js```里的某个函数。
