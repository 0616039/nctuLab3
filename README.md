# Route Configuration

This repository is a lab for NCTU course "Introduction to Computer Networks 2018".

---
## Abstract

In this lab, we are going to write a Python program with Ryu SDN framework to build a simple software-defined network and compare the different between two forwarding rules.

---
## Objectives

1. Learn how to build a simple software-defined networking with Ryu SDN framework
2. Learn how to add forwarding rule into each OpenFlow switch

---
## Execution


> * How to run your program?  
    1.在第一個terminal輸入sudo mn --custom topo.py --topo topo --link tc --controller remote  
    2.在第二個terminal輸入sudo ryu-manager SimpleController.py --observe-links或是sudo ryu-manager controller.py --observe-links        
> * What is the meaning of the executing command (both Mininet and Ryu controller)?  
    mn : mininet
    --custom topo.py : 透過 -–custom 參數指定 py 檔  
    --topo topo : 根據 python 中最後一行的名稱  
    --link tc : 使用者可以用連線進行設定  
    --controller remote : 設置controller, remote = 外部controller控制  
    ryu-manager : 啟動ryu  
    --observe-links : 自動下發LLDP
> * Show the screenshot of using iPerf command in Mininet (both `SimpleController.py` and `controller.py`)
   ![SimpleController.png](https://github.com/nctucn/lab3-0616039/blob/master/SimpleController.png)
   ![controller.png](https://github.com/nctucn/lab3-0616039/blob/master/controller.png)
---
## Description

### Tasks



1. Environment Setup  
   1.登入 root  
   2.clone 進自己的 github  
   3.輸入sudo service openvswitch-switch start  
   4.再輸入sudo mn 試試看  
2. Example of Ryu SDN  
   1.先開一個terminal,輸入 cd /root/Route_Configuration/src/  
   2. 跑 SimpleTopo.py with Mininet  
   3.開另外一個terminal,跑 SimpleController.py
   4.跑 the SimpleController.py with Ryu managerur,輸入 sudo ryu-manager SimpleController.py --observe-links  
   5.先exit原本的terminal,在退出後來的terminal  
3. Mininet Topology  
   1.輸入 cp SimpleTopo.py topo.py  
   2.在第一個terminal跑topo.py with mininet  
   3.再在第二個terminal跑 SimpleController.py with Ryu manager  
4. Ryu Controller  
   1.根據圖,在controller.py加上連接,有兩條路徑，分別是h1->s1->s3->h2跟h2->s3->s2->s1->h1  
5. Measurement  
   1.在第一個terminal輸入 sudo mn --custom topo.py --topo topo --link tc --controller remote 跑topo.py  
   2.在第二個terminal輸入 sudo ryu-manager SimpleController.py --observe-links 跑SimpleController.py  
   3.接著再mininet後面輸入  h1 iperf -s -u -i 1 –p 5566 > ./out/result1 &再輸入 h2 iperf -c 10.0.0.1 -u –i 1 –p 5566  
   4.會跑出結果，之後再exit第一個terminal,結束第二個terminal  
   5.在第一個terminal輸入 sudo mn --custom topo.py --topo topo --link tc --controller remote 跑topo.py  
   6.在第二個terminal輸入 sudo ryu-manager controller.py --observe-links 跑controller.py  
   7.接著再mininet後面輸入  h1 iperf -s -u -i 1 –p 5566 > ./out/result1 &再輸入 h2 iperf -c 10.0.0.1 -u –i 1 –p 5566   
   8.會跑出結果，之後再exit第一個terminal,結束第二個terminal   
### Discussion


1. Describe the difference between packet-in and packet-out in detail.  
   packet-in:對於接收到的封包進行轉送到 Controller 的動作  
   packet-out:對於接收到來自 Controller 的封包轉送到指定的連接埠    
2. What is “table-miss” in SDN?  
   一個packet在一個Flow Table中沒有發現能夠匹配成功的Flow Entry
3. Why is "`(app_manager.RyuApp)`" adding after the declaration of class in `controller.py`?  
   要撰寫一支 Ryu 應用程式，你只需要將你的應用程式類別繼承自 RyuApp 即可  
4. Explain the following code in `controller.py`.
    ```python
    @set_ev_cls(ofp_event.EventOFPPacketIn, CONFIG_DISPATCHER)
    ```
    負責 Packet-In 事件 在 Switch 與 Ryu 完成交握的狀況下執行 也代表著，只要更換參數，就能管理 OpenFlow 所提供的事件，也因此可以將管理邏輯，由事件     帶入整體管理環境中。
    接下來，我們看看事件內部的運作該怎麼規劃。首先，介紹透過事件提取出來的參數：
    ev.msg：代表 Packet-In 傳入的資訊（包含 switch、in_port_number等...） msg.datapath：在 OpenFlow 中，datapath 代表的就是 Switch               datapath.ofproto、datapath.ofproto_parser：取出此 Switch 中，使用的 OpenFlow 協定及 Switch 與 Ryu 之間的溝通管道。 接下來，介紹執行的部       分。在接收到 Packet-In 的事件時，我們規劃的動作（Action）是將此封包進行 Flooding，也就是傳往除了 in_port 外的所有 port 上。我們透過             ofp_parser建立此動作：
    actions = [datapath.ofproto_parser.OFPActionOutput(out_port)] 這裡使用到的OFPActionOutput，會依給定的參數而將封包傳往指定 port 上。例如：     ofproto_parser.OFPActionOutput(1)，則代表將封包傳往 port 1。因此在這裡我們使用out_port，規劃封包轉送到除了 in_port 以外的所有 port 上。  
5. What is the meaning of “datapath” in `controller.py`?    
   OpenFlow 交換器以及 Flow table 的操作都是透過 Datapath 類別的實體來進行。 在一般的情況下，會由事件傳遞給事件管理的訊息中取得  
6. Why need to set "`ip_proto=17`" in the flow entry?  
   使用UDP  
7. Compare the differences between the iPerf results of `SimpleController.py` and `controller.py` in detail.  
    接收到的ACK比例不同  
8. Which forwarding rule is better? Why?  
    controller.py is better  
    因為它的loss比較少  
    
---
## References
* **my refrerence**  
    * [ryu](https://osrg.github.io/ryu-book/zh_tw/html/switching_hub.html)  
    * [table miss](https://www.cnblogs.com/CasonChan/p/4620652.html)  
* **Ryu SDN**  
    * [Ryubook Documentation](https://osrg.github.io/ryu-book/en/html/)
    * [Ryubook [PDF]](https://osrg.github.io/ryu-book/en/Ryubook.pdf)
    * [Ryu 4.30 Documentation](https://github.com/mininet/mininet/wiki/Introduction-to-Mininet)
    * [Ryu Controller Tutorial](http://sdnhub.org/tutorials/ryu/)
    * [OpenFlow 1.3 Switch Specification](https://www.opennetworking.org/wp-content/uploads/2014/10/openflow-spec-v1.3.0.pdf)
    * [Ryubook 說明文件](https://osrg.github.io/ryu-book/zh_tw/html/)
    * [GitHub - Ryu Controller 教學專案](https://github.com/OSE-Lab/Learning-SDN/blob/master/Controller/Ryu/README.md)
    * [Ryu SDN 指南 – Pengfei Ni](https://feisky.gitbooks.io/sdn/sdn/ryu.html)
    * [OpenFlow 通訊協定](https://osrg.github.io/ryu-book/zh_tw/html/openflow_protocol.html)
* **Python**
    * [Python 2.7.15 Standard Library](https://docs.python.org/2/library/index.html)
    * [Python Tutorial - Tutorialspoint](https://www.tutorialspoint.com/python/)
* **Others**
    * [Cheat Sheet of Markdown Syntax](https://www.markdownguide.org/cheat-sheet)
    * [Vim Tutorial – Tutorialspoint](https://www.tutorialspoint.com/vim/index.htm)
    * [鳥哥的 Linux 私房菜 – 第九章、vim 程式編輯器](http://linux.vbird.org/linux_basic/0310vi.php)

---
## Contributors


* [0616039](https://github.com/0616039)
* [David Lu](https://github.com/yungshenglu)

---
## License

GNU GENERAL PUBLIC LICENSE Version 3
