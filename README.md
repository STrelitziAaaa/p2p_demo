- 种子配置文件.torrent
- 启动client程序
  - 发送上线通知给tracker
    - 请求至少包括: client拥有的file_list
    - server记录ip:port 在 online_list,更新该ip:port拥有的file_list
      - 定时询问client是否在线
- 上传
  - client生成.torrent
  - .torrent至少包括:
    - file_name
    - file_id(全部文件md5+timestamp+rand,保证唯一性)
    - 文件分块,hash校验值(md5,或其他摘要算法),包括分块的hash和整体的hash
      - 每一块大小固定比如512bytes,末尾可能<512
    - tracker server 的 ip:port
  - client发送上传请求给server
    - 请求至少包括: file_id
    - 可能面临的情况: file_id重复 (由file_id的复杂性,基本不考虑)
  - server记录该资源,维护一个该资源id和相应拥有该资源user_ip的字典

- 下载
  - client根据.torrent向tracker server请求下载
    - 请求至少包括: file_id
  - server返回拥有该资源的online_peer_ip__port_list
    - 必须是返回拥有完整资源的peer_ip
  - client拿着这个list向其他peer请求
    - 请求至少包括: 希望得到的资源分块的序号
    - 至少面临的情况:
      - 资源已删除
      - 用户已下线
  - client每下载完一个piece,按照.torrent提供的各块hash码校验
    - 通过seek定位分块数据
    - 若校验失败,需要重新请求
  - 当完整下载完后,校验整体,然后通知服务器已拥有该资源,服务器记录更新
    - msg_type = 4, 请求至少包括 file_id
- 登出:
  - 下线请求
  - 服务器更新online状态
---
- 协议:
- 定义固定大小的头(单一个content_length或struct.pack),头中含有体的长度,再读取变长的体
```c
client与server的交互:
header : int content_length
body: (json)
  可能的键值:
  int protocol_version
  string transport_layer = "TCP"
  int msg_type(0上线 1上传 2下载 3下线 4更新)
  int file_id if msg_type == 1 or 2 or 4
  int[] file_list if msg_type == 0
```
```
client与client的交互:
header : int content_length
body: (json)
  int protocol_version
  string transport_layer = "TCP"
  int msg_type(2 下载)
  int file_id
  int file_piece_seqnum
  file_piece_content(二进制)
  ...
```