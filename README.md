# Use-Android-to-control-Car-sensor-or-IOT-device
Use Android to control Car sensor or IOT device
@[TOC](目录)



## 前言
现在物联网设备和车联网设备中使用蓝牙通信的场景越来越多，前几年只有高端车才具备使用手机控制车辆的功能，现在已经越来越普及，包括自动解锁、一键寻车、车辆控制、电量检测，OTA无线升级等，还有现在很普及的共享单车，也是通过手机的蓝牙无线控制实现的解锁功能。
这几天韧带做手术比较有时间，又不能到处跑，不然有空肯定是出去嗨呀......    见下图：
![没钱嗨个屁](https://img-blog.csdnimg.cn/20210207194212504.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zNzczNDk4OA==,size_16,color_FFFFFF,t_70#pic_center)
这个实例的 Android Studio 版本4.1.2 , TargetSdkVersion版本是 28，Gradle 版本: 6.5

本文主要包含下面几个内容：
 **1.** 首先展示了我做的一个使用Android控制蓝牙设备的一个实例，模拟的是用户使用手机打开车辆前门和打开后备箱。
 **2.** Android 蓝牙通信的基本介绍和实现步骤


Enjoy ！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210131222518882.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zNzczNDk4OA==,size_16,color_FFFFFF,t_70#pic_center)



## 开发环境
 Android Studio 版本4.1.2     
 TargetSdkVersion: 28   
 CompileSdkVersion 28
 Gradle 版本: 6.5



## 1. 步骤展示
 ### 手机注册蓝牙通信Server端
 首先，车主手机端进入Android 车控APP，在车控蓝牙通信过程中，注册为Server端，这样车控APP就可以主动建立蓝牙连接。
 
*Android 蓝牙通信中会会涉及SPP通用数据传输协议，这个协议是Android 2.0中引入的API，通过Socket的形式实现数据传输及交互，需要区分Server端和 Client 端。*

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210207212855463.gif#pic_center)

 ### 蓝牙设备注册Client端
 蓝牙从设备就是通信中的Client端，我这里用一个有蓝牙功能的Pad 模拟了车载蓝牙模块，这样双机通信的方法比较方(省)便(钱) , 现在市面上用得比较多的是HC-06 蓝牙通信模块，可以和单片机配合使用，有钱有时间的自己试一下。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210207212458398.gif#pic_center)
 ### Android手机端通过蓝牙发送打开前门指令
 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210207220407930.gif#pic_center)

 ### Android手机端通过蓝牙发送打开后备箱指令
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210207221523650.gif#pic_center)


从上面的画面大家可以看到，用户的手机和蓝牙设备是没有通过数据线直接连接的，手机通过蓝牙向Client设备发送了指令，Client设备接收到指令后，执行了打开车门和打开后备箱的动作，将模型中的相应部位变成了红色。


## 2. Android 蓝牙通信的基本介绍和实现步骤

 1. #在Manifest 里申请蓝牙权限
       首先，要在新建项目中的AndroidManifest.xml中声明两个权限：BLUETOOTH权限用于请求连接和传送数据；BLUETOOTH_ADMIN权限用于启动设备、发现或进行蓝牙设置，如果要拥有该权限，必须先拥有BLUETOOTH权限。
   

```kotlin
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>

```
***注意：这里有一个坑，如果你的TargetVersion大于22，就意味着你需要让用户动态申请权限，这个时候需要增加下面的ACCESS_FINE_LOCATION和 ACCESS_COARSE_LOCATION权限。 否则车主的手机会无法搜索到蓝牙设备，我就遇到了这个问题。***

```kotlin
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission-sdk-23 android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```
 2. SSP通用数据传输协议中，Android 蓝牙通信用到的四个类：
 - BluetoothAdapter：代表本地蓝牙适配器，是所有蓝牙交互的入口。使用这个你可以发现其他蓝牙设备，查询已配对的设备列表，使用一个已知的 MAC 地址来实例化一个 BluetoothDevice，以及 创建一个BluetoothServerSocket 来为监听与其他设备的通信
   
  - BluetoothDevice：代表一个远程蓝牙设备，使用这个来请求一个与远程设备的 BluetoothSocket连接，或者查询关于设备名称、地址、类和连接状态等设备信息 。
   
   - BluetoothSocket：代表一个蓝牙 socket 的接口（和 TCP Socket 类似）。这是一个连接点，它允许一个应用与其他蓝牙设备通过 InputStream 和 OutputStream 交换数据。
   
   - BluetoothServerSocket：代表一个开 放的服务 器 socket，它 监听接受 的请求（ 与 TCP ServerSocket 类似）。为了连接两台 Android 设备，一个设备必须使用这个类开启一个服务器 socket。当一个远程蓝牙设备开始一个和该设备的连接请求，BluetoothServerSocket 将会返回一 个已连接的BluetoothSocket，接受该连接。
   
   3. 在蓝牙通信中，无论在Server端，还是在Client端，都需要使用BluetoothAdapter适配器启动蓝牙功能

```kotlin
            mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter() 
            iv_server_original!!.setImageResource(R.drawable.server_background_original)
            if (mBluetoothAdapter == null) {
                Toast.makeText(applicationContext, "bluetooth is no available", Toast.LENGTH_LONG).show() 
                return@OnClickListener
            }
            mBluetoothAdapter!!.enable() 
            if (!mBluetoothAdapter!!.isEnabled) {
                Toast.makeText(applicationContext, "bluetooth function is no available", Toast.LENGTH_LONG).show()
                finish()
                return@OnClickListener
            }
            try {
                serverSocket = mBluetoothAdapter!!.listenUsingRfcommWithServiceRecord("aaa", uuid)
                btListen_Thread = btListenThread(serverSocket, bluetoothServerMessageHandle)
                btListen_Thread!!.start()
                Toast.makeText(applicationContext, "服务器端已经开始运行", Toast.LENGTH_SHORT).show()
            } catch (e: IOException) {
                e.printStackTrace()
            }
        })
```
 mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter()  ： 用于获取蓝牙适配器
 mBluetoothAdapter!!.enable()  ： 允许蓝牙适配器，如果enable 失败，表示当前蓝牙设备不可用
 if (mBluetoothAdapter == null)  //表示当前手机不支持蓝牙
 
 4. 使用SSP协议中的uuid作为参数，使用 `mBluetoothAdapter!!.listenUsingRfcommWithServiceRecord("aaa", uuid)` 方法新建ServerSocket对象。并且通过 `btListenThread(serverSocket, bluetoothServerMessageHandle)` 构造函数创建 蓝牙监听线程。
    下面的bluetoothServerMessageHandle对象用户接受蓝牙指令后，在界面的展示。
    
 

```kotlin
            try {
                serverSocket = mBluetoothAdapter!!.listenUsingRfcommWithServiceRecord("aaa", uuid)
                btListen_Thread = btListenThread(serverSocket, bluetoothServerMessageHandle)
                btListen_Thread!!.start()
                Toast.makeText(applicationContext, "服务器端已经开始运行", Toast.LENGTH_SHORT).show()
            } catch (e: IOException) {
                e.printStackTrace()
            }
```
 - 创建蓝牙监听线程：
    下面的`serverSocket!!.accept()` 函数用于接受客户端的连接，并且产生与此蓝牙连接对应的Socket，如果没有连接成功的话，线程就会停在这里。这样就可以获取蓝牙通信的输出流和输入流`DataOutputStream(BufferedOutputStream(clientSocket.getOutputStream()))`  和 
                 `DataInputStream(BufferedInputStream(clientSocket.getInputStream()))` 
 -   blue_tooth_msg_server_thread 是蓝牙消息服务器线程，用于给界面发送异步消息，这个下面会讲

  

```kotlin
blue_tooth_msg_server_thread = bluetoothMsgServerThread(mmInStream!!, msgHandler)
blue_tooth_msg_server_thread!!.start()
```

    
```kotlin
class btListenThread(serverSocket: BluetoothServerSocket?, msgHandler: Handler?) : Thread() {
    var serverSocket: BluetoothServerSocket? = null //蓝牙socket对象
    var clientSocket: BluetoothSocket? = null //蓝牙socket对象
    private var mmInStream: DataInputStream? = null //输入数据流
    var outStream: DataOutputStream? = null //输入数据流
        private set
    var blue_tooth_msg_server_thread: bluetoothMsgServerThread? = null //蓝牙线程
    private val msgHandler: Handler? = null //Handler对象
    override fun run() {
        while (!interrupted()) {
            try {
                clientSocket = serverSocket!!.accept()
                outStream = DataOutputStream(BufferedOutputStream(clientSocket.getOutputStream())) //获取out数据量对象
                mmInStream = DataInputStream(BufferedInputStream(clientSocket.getInputStream()))
                //创建线程接收
                blue_tooth_msg_server_thread = bluetoothMsgServerThread(mmInStream!!, msgHandler)
                blue_tooth_msg_server_thread!!.start()
                val msg = Message() //创建message对象
                msg.what = 0x1235
                msgHandler!!.sendMessage(msg) //通过msgHandler将消息发送到界面
            } catch (e: IOException) {
                e.printStackTrace()
            }
        }
        val msg = Message() //创建message对象
        msg.what = 0x2000
        msgHandler!!.sendMessage(msg) //通过msgHandler将消息发送到界面
        blue_tooth_msg_server_thread!!.interrupt()
        try {
            outStream!!.close()
            mmInStream!!.close()
            clientSocket!!.close()
        } catch (ex: Exception) {
        }
    }

    init {   //构造函数
        this.msgHandler = msgHandler
        this.serverSocket = serverSocket
    }
}
```


 5.  蓝牙消息服务器线程 `bluetoothMsgServerThread` ：

```kotlin
internal class bluetoothMsgServerThread     //构造函数
(//输入数据流
        private val mmInStream: DataInputStream, //Handler对象
        private val msgHandler: Handler?) : Thread() {
    override fun run() {
        val InBuffer = ByteArray(64) //定义缓冲区
        while (!interrupted()) {
            try {
                val readed = mmInStream.read(InBuffer, 0, 64) //读取输入数据
                val msg = Message() //创建message对象
                msg.what = 0x1234
                msg.obj = InBuffer
                msg.arg1 = readed
                msgHandler!!.sendMessage(msg) //通过msgHandler将消息发送到界面
            } catch (e: IOException) {
                e.printStackTrace()
            }
        }
        val msg = Message() //创建message对象
        msg.what = 0x2001
        msgHandler!!.sendMessage(msg) //通过msgHandler将消息发送到界面
    }
}
```



 6.  发送蓝牙指令
      这一部分直接看备注就好了
      
      ***这里有一个坑，蓝牙指令发送完毕以后，一定要记得将蓝牙输出流 flush mmOutStream.flush掉，这个坑花掉了我一个小时的时间。***

```kotlin
        buttonServerSend = findViewById<View>(R.id.buttonServerSend) as Button
        buttonServerSend!!.setOnClickListener(View.OnClickListener {
            iv_server_original!!.setImageResource(R.drawable.server_status_frontopened)
            val mmOutStream = btListen_Thread!!.outStream
            var buffer: ByteArray? = null
            try {
                buffer = editTextServerSend!!.text.toString().toByteArray(charset("UTF-8"))
            } catch (e: UnsupportedEncodingException) {
                e.printStackTrace()
            }
            //byte[] buffer = editTextServerSend.getText().toString().getBytes();       //定义 要发送的数据内容
            if (buffer!!.size < 1) {
                Toast.makeText(applicationContext, "请输入要发送的内容", Toast.LENGTH_SHORT).show()
                return@OnClickListener
            }
            try {
                buffer = "Car_Control_OpenRearDoor".toByteArray(charset("UTF-8"))
                mmOutStream!!.write(buffer) //发送buffer的内容给蓝牙
                mmOutStream.flush() 
                vib!!.vibrate(100) //震动
            } catch (e: Exception) {
                e.printStackTrace()
            }
        })
        stopServerBtn = findViewById<View>(R.id.stopServerBtn) as Button
        stopServerBtn!!.setOnClickListener {
            iv_server_original!!.setImageResource(R.drawable.server_status_rearopened)
            if (btListen_Thread != null) {
                btListen_Thread!!.interrupt()
                try {
                    serverSocket!!.close()
                } catch (e: IOException) {
                    e.printStackTrace()
                }
            }
        }
    }
```

 7.  Client端搜索附件的蓝牙设备
     - 在Client搜索蓝牙的方法里面，首先是获取蓝牙适配器 `BluetoothAdapter.getDefaultAdapter()，`
通过蓝牙设备器获取绑定的蓝牙设备，并将其保存在 Set 内，如果没有配对的蓝牙设备，则弹出提示即可。
    - NameList用于保存可以配对的蓝牙
    - spinner.adapter 用户下拉展示配对的蓝牙设备
    - 
```kotlin
           iv_car_status!!.setImageResource(R.drawable.car_status_original)
            mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter() //获取蓝牙适配器
            if (mBluetoothAdapter == null) {
                Toast.makeText(applicationContext, "bluetooth is no available", Toast.LENGTH_LONG).show()
                return@OnClickListener
            }
            mBluetoothAdapter!!.enable() //允许蓝牙适配器
            if (!mBluetoothAdapter!!.isEnabled) {
                Toast.makeText(applicationContext, "bluetooth function is no available", Toast.LENGTH_LONG).show()
                finish()
                return@OnClickListener
            }
            val pairedDevices = mBluetoothAdapter!!.bondedDevices //获取安卓系统已经配对的蓝牙设备
            if (pairedDevices.size < 1) {
                Toast.makeText(applicationContext, "没有配对的蓝牙设备，请先配对再使用", Toast.LENGTH_LONG).show()
                return@OnClickListener
            }
            val spinner = findViewById<View>(R.id.spinner1) as Spinner //下拉框控件定义
            for (device in pairedDevices) {
                Namelist.add(device.name)
                Addresslist.add(device.address) //将mac地址加入到下拉框中
            }
            //通过适配器对象 显示下拉框中的内容
            val adapter = ArrayAdapter(applicationContext, android.R.layout.simple_spinner_item, Namelist)
            adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item)
            spinner.adapter = adapter
```

 8. Client 端连接服务器
 - `mBluetoothAdapter!!.getRemoteDevice(address)` 可以用户根据蓝牙地址获取到蓝牙设备
 - `mBluetoothAdapter!!.getRemoteDevice(address)` 方法用于根据UUID 连接到ClientSocket
 - `blue_tooth_msg_thread = bluetoothMsgThread(mmInStream!!, bluetoothMessageHandle)` 用于//创建线程接收 单片机发送给安卓 蓝牙的数据


```kotlin
       iv_car_status!!.setImageResource(R.drawable.car_status_original)
            editTextAll!!.setText("") //清空接收区
            try {
                val spinner = findViewById<View>(R.id.spinner1) as Spinner
                address = Addresslist[spinner.selectedItemPosition]
                device = mBluetoothAdapter!!.getRemoteDevice(address) 
                clientSocket = device.createRfcommSocketToServiceRecord(uuid) 
                clientSocket.connect() //连接蓝牙设备
                mmOutStream = clientSocket.getOutputStream() //获取out数据量对象
                mmInStream = DataInputStream(BufferedInputStream(clientSocket.getInputStream()))
                Toast.makeText(applicationContext, "蓝牙设备连接成功", Toast.LENGTH_SHORT).show()
                vib!!.vibrate(100) //震动
                //set_btn_status(true);			 	 			 	   			 	//设置 继电器、led按钮 允许操作

                //创建线程接收 单片机发送给安卓 蓝牙的数据
                blue_tooth_msg_thread = bluetoothMsgThread(mmInStream!!, bluetoothMessageHandle)
                blue_tooth_msg_thread!!.start()
            } catch (e: Exception) {
                //set_btn_status(false);										    //不运行界面操作
                Toast.makeText(applicationContext, "连接蓝牙设备失败:$e", Toast.LENGTH_SHORT).show()
                e.printStackTrace()
            }
```

 10. 好了，这样Client蓝牙设备就可以通过下面的方法获取Server发送过来的蓝牙车控指令。
 - `mmInStream.read(InBuffer, 0, 64)` 用于读取输入数据
 - `msgHandler.sendMessage(msg)` 用于通过msgHandler将消息发送到界面
 - `msgHandler.sendMessage(msg)` 用于通过msgHandler将消息发送到界面
 - `show_result(msg.obj as ByteArray, msg.arg1)`  用于显示内容到界面上
```kotlin
lass bluetoothMsgThread     //构造函数
(//输入数据流
        private val mmInStream: DataInputStream, //Handler对象
        private val msgHandler: Handler) : Thread() {
    override fun run() {
        val InBuffer = ByteArray(64) //定义缓冲区
        while (!interrupted()) {
            try {
                //mmInStream.readFully(InBuffer, 0, 64); //读取输入数据
                val readed = mmInStream.read(InBuffer, 0, 64) 
                val msg = Message() //创建message对象
                msg.what = 0x1234
                msg.obj = InBuffer
                msg.arg1 = readed
                msgHandler.sendMessage(msg)
            } catch (e: IOException) {
                e.printStackTrace()
                val msg = Message() //创建message对象
                msg.what = 0x3000
                msgHandler.sendMessage(msg) 
                break
            }
        }
    }
```

```kotlin
    var bluetoothMessageHandle: Handler = object : Handler() {
        //定义handler对象
        override fun handleMessage(msg: Message) {
            if (msg.what == 0x1234) {                             //接收what = 0x1234的自定义数据
                show_result(msg.obj as ByteArray, msg.arg1) 
                iv_car_status!!.setImageResource(R.drawable.car_status_frontopen)
            }
            if (msg.what == 0x3000) {                             //接收what = 0x1234的自定义数据
                Toast.makeText(applicationContext, "蓝牙连接断开", Toast.LENGTH_SHORT).show()
                iv_car_status!!.setImageResource(R.drawable.car_status_rearopen)
            }
        }
```

  
## 结语
好了，以上就是Android 手机通过蓝牙通信给车辆网设备发送车控指令的流程。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210208085002475.gif#pic_center)

再上三天班，就是2021年的春节。
放假啦. 

Bye！

*我的GitHub地址：https://github.com/18601949127
CSDN博客：https://blog.csdn.net/weixin_37734988*



![在这里插入图片描述](https://img-blog.csdnimg.cn/20210131223121254.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zNzczNDk4OA==,size_16,color_FFFFFF,t_70#pic_center)

