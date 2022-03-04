# Codec Negotiation

### 参考
```shell
# 编解码器协商
https://freeswitch.org/confluence/display/FREESWITCH/Codec+Negotiation
# 编解码器和媒体
https://freeswitch.org/confluence/display/FREESWITCH/Codecs+and+Media
```

### 在vars.xml添加全局有效配置
```xml
<!--  FreeSWITCH 能够将两条腿的不同的编解码器进行匹配；全局有效 -->
<X-PRE-PROCESS cmd="set" data="media_mix_inbound_outbound_codecs=true"/>
```

### 定制化指定编解码
* 1、disable-transcoding

禁用转码, 这是您可以在出站 SIP 配置文件中设置的参数。这将强制为支路 B（出站支路）提议的编解码器与支路 A（入站支路）协商的编解码器相同。要设置它，将以下行添加到所需的 SIP 配置文件：
```xml
<param name="disable-transcoding" value="true"/>
```

* 2、absolute_codec_string

这是您可以在拨号方案中设置的通道变量，通常在桥接之前。 这将强制将编解码器列表提议到支路 B，而不考虑其他任何内容。
```xml
<action application="export" data="nolocal:absolute_codec_string=PCMA,PCMU"/> 
<action application="bridge" data="sofia/gateway/mygateway/mynumber"/>
<!-- 或者 -->
<action application="bridge" data="{absolute_codec_string='PCMA,PCMU'}sofia/gateway/mygateway/mynumber"/>
```
fs_cli api 
```shell
bgapi originate {absolute_codec_string=^^:PCMU:PCMA}sofia/gateway/mygateway/mynumber
```

* 3、codec_string

这是您可以在拨号方案中设置的通道变量，通常在桥接之前。 定义的编解码器列表将覆盖出站配置文件的 outbound-codec-prefs 参数中的一组
```xml
<action application="export" data="nolocal:codec_string=PCMA,PCMU"/>
<action application="bridge" data="sofia/gateway/mygateway/mynumber"/>
<!-- 或者 -->
<action application="bridge" data="{codec_string='PCMA,PCMU'}sofia/gateway/mygateway/mynumber"/>
```
fs_cli api
```shell
bgapi originate {codec_string=^^:PCMU:PCMA}sofia/gateway/mygateway/mynumber
```

* 4、Rewriting SDP

重写SDP，switch_r_sdp:
```xml
<action application="set">
<![CDATA[switch_r_sdp=(sdp here)
 ]]>
</action>
```

