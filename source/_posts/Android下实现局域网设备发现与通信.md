---
title: Android下实现局域网设备发现与通信
date: 2020-06-03 21:11:54
categories: "Android"
tags:
    - 局域网
    - udp
    - 协议
---

## 概述

在使用Android开发智能设备时，一般会分为用于遥控与管理的Host端，和用于执行个性功能的Slave端，二者可以借助网络或蓝牙等途径实现通信。

协议：计算机与网络设备要相互通信，双方就必须基于相同的方法。比如，如何探测到通信目标、由哪一边先发起通信、使用哪种语言进行通信、怎样结束通信等规则都需要事先确定。不同的硬件、操作系统之间的通信，所有的这一切都需要一种规则。而我们就把这种规则称为协议。

本文介绍的就是网络通信。（类似应用商城里的 电视超人 app）

<!-- more -->

## 前提条件: ip 地址

借助网络通信，就需要知道对方的 ip 地址，而常见的网络环境中的 ip 地址一般都是通过 DHCP 服务动态分配的， 所以事先没法知道对方的 ip。

## 查询对方 ip 地址

为了确认对方的地址， 可以通过向局域网内发送查找设备的广播，收到广播的 Slave 端就知道了 Host 的 ip 地址，在向 Host 端发送应答包后，双方就都知道了对方的 ip。

## 局域网设备的通信

在 Host 端和 Slave 端互相知道 ip 地址后，就可以事先局域网通信了。
局域网通信一般都是通过 TCP 或 UDP 实现。 

功能点|TCP|UDP
-|-|-
可靠性|数据不丢失、无差错、不重复、按序到达|可能会丢包或者顺序错乱
效率|支持一对一|支持一对一、一对多、多对一、多对多

## 遇到的问题

《基于Android热点的局域网UDP广播，部分手机收不到UDP报文的问题》： https://blog.csdn.net/qq_27331979/article/details/50379683

## UDP报文的发送和接收

Slave 端：

```java
class SearchRespThread : Thread() {
    var socket : DatagramSocket? = null
    var openFlag = false

    fun destory() {
        socket?.let {
            it.close()
            socket = null
        }
        openFlag = false
    }

    override fun run() {
        try {
            //指定接收数据包的端口
            socket = DatagramSocket(RemoteConst.DEVICE_SEARCH_PORT)
            val buf = ByteArray(1024)
            val recePacket = DatagramPacket(buf, buf.size)
            openFlag = true
            while (openFlag) {
                socket!!.receive(recePacket)
                //校验数据包是否是搜索包
                if (verifySearchData(recePacket)) {
                    //发送搜索应答包
                    val sendData: ByteArray = packSearchRespData()
                    val sendPack = DatagramPacket(
                        sendData,
                        sendData.size,
                        recePacket.socketAddress
                    )
                    socket!!.send(sendPack)
                }
            }
        } catch (e: IOException) {
            destory()
        }
    }

    /**
        * 生成搜索应答数据
        * 协议：$(1) + packType(1) + sendSeq(4) + dataLen(1) + [data]
        * packType - 报文类型
        * sendSeq - 发送序列
        * dataLen - 数据长度
        * data - 数据内容
        * @return
        */
    private fun packSearchRespData(): ByteArray {
        val data = ByteArray(1024)
        var offset = 0
        data[offset++] = RemoteConst.PACKET_PREFIX.toByte()
        data[offset++] = RemoteConst.PACKET_TYPE_SEARCH_DEVICE_RSP.toByte()

        // 添加UUID数据
        val uuid = getUuidData()
        data[offset++] = uuid.size.toByte()
        System.arraycopy(uuid, 0, data, offset, uuid.size)
        offset += uuid.size
        val retVal = ByteArray(offset)
        System.arraycopy(data, 0, retVal, 0, offset)
        return retVal
    }

    /**
        * 校验搜索数据是否符合协议规范
        * 协议：$(1) + packType(1) + sendSeq(4) + dataLen(1) + [data]
        * packType - 报文类型
        * sendSeq - 发送序列
        * dataLen - 数据长度
        * data - 数据内容
        */
    private fun verifySearchData(pack: DatagramPacket): Boolean {
        if (pack.length < 6) {
            return false
        }
        val data = pack.data
        var offset = pack.offset
        var sendSeq: Int
        if (data[offset++].toInt() != '$'.toInt() || data[offset++].toInt() != RemoteConst.PACKET_TYPE_SEARCH_DEVICE_REQ) {
            return false
        }
        sendSeq = data[offset++].toInt() and 0xFF
        sendSeq = sendSeq or (data[offset++].toInt() shl 8 and 0xFF00)
        sendSeq = sendSeq or (data[offset++].toInt() shl 16 and 0xFF0000)
        sendSeq = sendSeq or (data[offset++].toInt() shl 24 and -0x1000000)
        return !(sendSeq < 1 || sendSeq > RemoteConst.SEARCH_DEVICE_TIMES)
    }

    /**
        * 获取设备uuid
        * @return
        */
    private fun getUuidData(): ByteArray {
        return (Build.PRODUCT + Build.ID).toByteArray()
    }
}
```

Host 端：

```java
class DeviceSearcher {

    companion object {
        private val executorService = Executors.newSingleThreadExecutor()
        val uiHandler = Handler(Looper.getMainLooper())

        fun search(onSearchListener: OnSearchListener) {
            executorService.execute(SearchRunnable(onSearchListener))
        }
    }

    interface OnSearchListener {
        fun onSearchStart()
        fun onSearchedOne(device: Device)
        fun onSearchFinish()
    }

    class SearchRunnable(private val onSearchListener: OnSearchListener) : Runnable {
        override fun run() {
            try {
                uiHandler.post { onSearchListener.onSearchStart() }
                val socket = DatagramSocket()
                //设置接收等待时长
                socket.soTimeout = RemoteConst.RECEIVE_TIME_OUT
                val sendData = ByteArray(1024)
                val receData = ByteArray(1024)
                val recePack = DatagramPacket(sendData, receData.size)
                //使用广播形式（目标地址设为255.255.255.255）的udp数据包
                val sendPacket = DatagramPacket(sendData, sendData.size, InetAddress.getByName("192.168.43.255"), RemoteConst.DEVICE_SEARCH_PORT)
                //用于存放已经应答的设备
                val devices = HashMap<String, Device>()
                // 搜索指定次数
                for (index in 0 until RemoteConst.SEARCH_DEVICE_TIMES) {
                    sendPacket.data = packSearchData(index + 1)
                    // 发送udp数据包
                    socket.send(sendPacket)
                    try {
                        //限定搜索设备的最大数量
                        var rspCount = RemoteConst.SEARCH_DEVICE_MAX
                        while (rspCount > 0) {
                            socket.receive(recePack)
                            val device = parseRespData(recePack)
                            device?.let { device ->
                                if (!devices.containsKey(device.ip)) {
                                    //保存新应答的设备
                                    devices[device.ip] = device
                                    uiHandler.post { onSearchListener.onSearchedOne(device) }
                                }
                            }
                            rspCount--
                        }
                    } catch (e: SocketTimeoutException) {
                        e.printStackTrace()
                    }
                }
                socket.close()
                uiHandler.post { onSearchListener.onSearchFinish() }
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }


        /**
         * 生成搜索数据包
         * 格式：$(1) + packType(1) + sendSeq(4) + dataLen(1) + [data]
         * packType - 报文类型
         * sendSeq - 发送序列
         * dataLen - 数据长度
         * data - 数据内容
         * @param seq
         * @return
         */
        private fun packSearchData(seq: Int): ByteArray? {
            val data = ByteArray(6)
            var offset = 0
            data[offset++] = RemoteConst.PACKET_PREFIX.toByte()
            data[offset++] = RemoteConst.PACKET_TYPE_SEARCH_DEVICE_REQ.toByte()
            data[offset++] = seq.toByte()
            data[offset++] = (seq shr 8).toByte()
            data[offset++] = (seq shr 16).toByte()
            data[offset++] = (seq shr 24).toByte()
            return data
        }

        /**
         * 校验和解析应答的数据包
         * @param pack udp数据包
         * @return
         */
        private fun parseRespData(pack: DatagramPacket): Device? {
            if (pack.length < 2) {
                return null
            }
            val data = pack.data
            var offset = pack.offset
            //检验数据包格式是否符合要求
            if (data[offset++].toInt() != RemoteConst.PACKET_PREFIX || data[offset++].toInt() != RemoteConst.PACKET_TYPE_SEARCH_DEVICE_RSP) {
                return null
            }
            val length: Int = data[offset++].toInt()
            val uuid = String(data, offset, length)
            return Device(pack.address.hostAddress, pack.port, uuid, false)
        }
    }
}
```

## Socket 通信

Client 端：

```java
fun connect() {
    mExecutorService.execute(ConnectService())
}

private fun disConnect() {
    mExecutorService.execute(SendService("断开"))
}

private fun send(type: String) {
    mExecutorService.execute(SendService(type))
}

private inner class ConnectService : Runnable {
    override fun run() {
        try {
            val socket = Socket(host, PORT)
            socket.soTimeout = 60000
            printWriter = PrintWriter(BufferedWriter(OutputStreamWriter(
                    socket.getOutputStream(), "UTF-8")), true)
            bufferedReader = BufferedReader(InputStreamReader(socket.getInputStream(), "UTF-8"))
            receiveMsg()
        } catch (e: Exception) {
            isConnected = false
            Log.e("====", "Socket对象获取失败:" + e.message)
            runOnUiThread {
                Toast.makeText(this@MainActivity, "连接失败", Toast.LENGTH_SHORT).show()
            }
        }
    }
}

private inner class SendService(val type: String) : Runnable {

    override fun run() {
        if (!isConnected) {
            runOnUiThread {
                Toast.makeText(this@MainActivity, "未连接", Toast.LENGTH_SHORT).show()
            }
        } else {
            printWriter?.println(type)
        }
    }
}

fun receiveMsg() {
    try {
        while (true) {
            var receiveMsg = bufferedReader?.readLine() ?: ""
            if (receiveMsg.isNotEmpty()) {
                Log.e("====", "receiveMsg: $receiveMsg")
                if (receiveMsg.equals("成功连接服务器")) {
                    isConnected = true
                } else if (receiveMsg.equals("服务端断开连接")) {
                    isConnected = false
                }
                runOnUiThread {
                    btnConnect.setImageDrawable(getDrawable(if (isConnected) R.drawable.power_on else R.drawable.power_off))
                    Toast.makeText(this, if (isConnected) "已连接" else "断开连接", Toast.LENGTH_SHORT).show()
                }
            }
        }
    } catch (e: IOException) {
        e.printStackTrace()
    }
}
```

Server 端：

```java
public class SocketServer {

    private static final int PORT = 9999;
    private static final String TAG = "TAG SERVER";
    private ServerSocket server = null;
    private ExecutorService mExecutorService = null;

    public SocketServer() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    server = new ServerSocket(PORT);
                    mExecutorService = Executors.newCachedThreadPool();
                    Log.d(TAG, "服务器已启动...");
                    Socket client;
                    while (true) {
                        client = server.accept();
                        mExecutorService.execute(new Service(client));
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    static class Service implements Runnable {
        private Socket socket;
        private BufferedReader bufferedReader = null;
        private PrintWriter printWriter = null;

        Service(Socket socket) {
            this.socket = socket;
            try {
                printWriter = new PrintWriter(new BufferedWriter(new OutputStreamWriter(socket.getOutputStream(), "UTF-8")), true);
                bufferedReader = new BufferedReader(new InputStreamReader(
                        socket.getInputStream(), "UTF-8"));
                printWriter.println("成功连接服务器");
                Log.d(TAG, "成功连接服务器");
            } catch (IOException e) {
                e.printStackTrace();
            }

        }

        @Override
        public void run() {
            try {
                while (true) {
                    String receiveMsg;
                    if ((receiveMsg = bufferedReader.readLine()) != null) {
                        Log.d(TAG, "receiveMsg:" + receiveMsg);
                        if (receiveMsg.equals("断开")) {
                            Log.d(TAG, "客户端请求断开连接");
                            printWriter.println("服务端断开连接");
                            bufferedReader.close();
                            socket.close();
                            break;
                        } else {
                            int keyCode = -1;
                            if (receiveMsg.equals("上")) {
                                keyCode = 19;
                            } else if (receiveMsg.equals("下")) {
                                keyCode = 20;
                            } else if (receiveMsg.equals("左")) {
                                keyCode = 21;
                            } else if (receiveMsg.equals("右")) {
                                keyCode = 22;
                            } else if (receiveMsg.equals("确定")) {
                                keyCode = 23;
                            } else if (receiveMsg.equals("返回")) {
                                keyCode = 4;
                            } else if (receiveMsg.equals("提高音量")) {
                                keyCode = 24;
                            } else if (receiveMsg.equals("降低音量")) {
                                keyCode = 25;
                            }
                            Instrumentation mInst = new Instrumentation();
                            mInst.sendKeyDownUpSync(keyCode);
                        }
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```
