如果你有一个USB连接的pcap文件，并且有很多中断，可能这是一个USB键盘连接。

一个像这样的wireshark过滤器可能会很有用：`usb.transfer_type == 0x01 and frame.len == 35 and !(usb.capdata == 00:00:00:00:00:00:00:00)`

知道以“02”开头的数据是通过shift键按下的可能很重要。

你可以在以下链接中阅读更多信息并找到一些关于如何分析的脚本：

* [https://medium.com/@ali.bawazeeer/kaizen-ctf-2018-reverse-engineer-usb-keystrok-from-pcap-file-2412351679f4](https://medium.com/@ali.bawazeeer/kaizen-ctf-2018-reverse-engineer-usb-keystrok-from-pcap-file-2412351679f4)
* [https://github.com/tanc7/HacktheBox\_Deadly\_Arthropod\_Writeup](https://github.com/tanc7/HacktheBox_Deadly_Arthropod_Writeup)
