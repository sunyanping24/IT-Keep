<!-- TOC -->

- [Netty中的拆包器](#netty中的拆包器)

<!-- /TOC -->

# Netty中的拆包器
- 固定长度的拆包器 FixedLengthFrameDecoder
基于数据包长度的拆包器 LengthFieldBasedFrameDecoder

- 行拆包器 LineBasedFrameDecoder
每个应用层数据包，都以换行符作为分隔符，进行分割拆分。

- 分隔符拆包器 DelimiterBasedFrameDecoder
每个应用层数据包，都通过自定义的分隔符，进行分割拆分。

- 基于数据包长度的拆包器 LengthFieldBasedFrameDecoder
将应用层数据包的长度，作为接收端应用层数据包的拆分依据。按照应用层数据包的大小，拆包。这个拆包器，有一个要求，就是应用层协议中包含数据包的长度。