.. _WS-AT:

WebSocket AT 命令集
===================

:link_to_translation:`en:[English]`

- :ref:`介绍 <cmd-ws-intro>`
- :ref:`AT+WSCFG <cmd-WSCFG>`：配置 WebSocket 参数
- :ref:`AT+WSOPEN <cmd-WSOPEN>`：查询/打开 WebSocket 连接
- :ref:`AT+WSSEND <cmd-WSSEND>`：向 WebSocket 连接发送数据
- :ref:`AT+WSCLOSE <cmd-WSCLOSE>`：关闭 WebSocket 连接

.. _cmd-ws-intro:

介绍
------

.. important::
  默认的 AT 固件不支持此页面下的 AT 命令。如果您需要 {IDF_TARGET_NAME} 支持 WebSocket 命令，请自行 :doc:`编译 ESP-AT 工程 <../Compile_and_Develop/How_to_clone_project_and_compile_it>`，在第五步配置工程里选择：

  - 启用 ``Component config`` -> ``AT`` -> ``AT WebSocket command support``

.. _cmd-WSCFG:

:ref:`AT+WSCFG <WS-AT>`：配置 WebSocket 参数
------------------------------------------------------------

设置命令
^^^^^^^^

**命令：**

::

    AT+WSCFG=<link_id>,<ping_intv_sec>,<ping_timeout_sec>[,<buffer_size>]

**响应：**

::

    OK

或

::

    ERROR

参数
^^^^

- **<link_id>**：WebSocket 连接 ID。范围：[0,2]，即最大支持三个 WebSocket 连接。
- **<ping_intv_sec>**：发送 WebSocket Ping 间隔。单位：秒。范围：[1,7200]。默认值：10，即：每隔 10 秒发送一次 WebSocket Ping 包。
- **<ping_timeout_sec>**：WebSocket Ping 超时。单位：秒。范围：[1,7200]。默认值：120，即：120 秒未收到 WebSocket Pong 包，则关闭连接。
- **<buffer_size>**：WebSocket 缓冲区大小。单位：字节。范围：[1,8192]。默认值：1024。

说明
^^^^
- 此命令应在 :ref:`AT+WSOPEN <cmd-WSOPEN>` 之前配置，否则不会生效。

示例
^^^^

::

    // 配置 link_id 为 0 的 WebSocket 连接的 Ping 发送间隔为 30 秒，超时 60 秒，缓冲区 4096 字节
    AT+WSCFG=0,30,60,4096

.. _cmd-WSOPEN:

:ref:`AT+WSOPEN <WS-AT>`：查询/打开一个 WebSocket 连接
------------------------------------------------------------

查询命令
^^^^^^^^

**命令：**

::

    AT+WSOPEN?

**响应：**

当有连接时，AT 返回：

::

    +WSOPEN:<link_id>,<state>,<"uri">

    OK

当没有连接时，AT 返回：

::

    OK

设置命令
^^^^^^^^

**命令：**

::

    AT+WSOPEN=<link_id>,<"uri">[,<"subprotocol">][,<timeout_ms>][,<"auth">]

**响应：**

::

    +WS_CONNECTED:<link_id>

    OK

或

::

    ERROR

参数
^^^^

- **<link_id>**：WebSocket 连接 ID。范围：[0,2]，即最大支持三个 WebSocket 连接。
- **<state>**：WebSocket 连接的状态。

   - 0：WebSocket 连接已关闭。
   - 1：WebSocket 连接正在重连。
   - 2：已建立 WebSocket 连接。
   - 3：接收 WebSocket Pong 超时或读取连接数据错误，正在等待重连。
   - 4：已收到服务器端 WebSocket 关闭帧，正在发送关闭帧到服务器。

- **<"uri">**：WebSocket 服务器的统一资源标识符。
- **<"subprotocol">**：WebSocket 子协议（参考 `RFC6455 1.9 章节 <https://www.rfc-editor.org/rfc/rfc6455#section-1.9>`_）。
- **<timeout_ms>**：建立 WebSocket 连接的超时时间。单位：毫秒。范围：[0,180000]。默认值：15000。
- **<"auth">**：WebSocket 鉴权（参考 `RFC6455 4.1.12 章节 <https://www.rfc-editor.org/rfc/rfc6455#section-4.1>`_）。

示例
^^^^

::

    // uri 参数来自于 https://www.piesocket.com/websocket-tester
    AT+WSOPEN=0,"wss://demo.piesocket.com/v3/channel_123?api_key=VCXCEuvhGcBDP7XhiJJUDvR1e1D3eiVjgZ9VRiaV&notify_self"

.. _cmd-WSSEND:

:ref:`AT+WSSEND <WS-AT>`：向 WebSocket 连接发送数据
-----------------------------------------------------------------

设置命令
^^^^^^^^

**命令：**

::

    AT+WSSEND=<link_id>,<length>[,<opcode>][,<timeout_ms>]

**响应：**

::

    OK

    >

上述响应表示 AT 已准备好从 AT port 接收数据，此时您可以输入数据，当 AT 接收到的数据长度达到 ``<length>`` 后，数据传输开始。

如果未建立连接或数据传输时连接被断开，返回：

::

    ERROR

如果数据传输成功，返回：

::

    SEND OK

参数
^^^^

- **<link_id>**：WebSocket 连接 ID。范围：[0,2]。
- **<length>**：发送的数据长度。单位：字节。
- **<opcode>**：发送的 WebSocket 帧中的 opcode。范围：[0,0xF]。默认值：1，即 text 帧。请参考 `RFC6455 5.2 章节 <https://www.rfc-editor.org/rfc/rfc6455#section-5.2>`_ 了解更多的 opcode。

   - 0x0：continuation 帧
   - 0x1：text 帧
   - 0x2：binary 帧
   - 0x3 - 0x7：为其它非控制帧保留
   - 0x8：连接关闭帧
   - 0x9：ping 帧
   - 0xA：pong 帧
   - 0xB - 0xF：为其它控制帧保留

- **<timeout_ms>**：发送超时时间。单位：毫秒。范围：[0,60000]。默认值：10000。

.. _cmd-WSCLOSE:

:ref:`AT+WSCLOSE <WS-AT>`：关闭 WebSocket 连接
-----------------------------------------------------

设置命令
^^^^^^^^

**命令：**

::

    AT+WSCLOSE=<link_id>

**响应：**

::

    OK

参数
^^^^

- **<link_id>**：WebSocket 连接 ID。范围：[0,2]。

示例
^^^^

::

    // 关闭 ID 为 0 的 WebSocket 连接
    AT+WSCLOSE=0
