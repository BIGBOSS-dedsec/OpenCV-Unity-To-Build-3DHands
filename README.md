# OpenCV-Unity-To-Build-3DHands

# 前言
本篇文章将介绍如何使用**Python**利用**OpenCV**图像捕捉，配合强大的**Mediapipe**库来实现**手势动作检测**与识别；将识别结果实时同步至**Unity**中，实现手势模型在Unity中运动身体结构识别
# Demo演示
[Demo展示](https://hackathon2022.juejin.cn/#/works/detail?unique=WJoYomLPg0JOYs8GazDVrw)：https://hackathon2022.juejin.cn/#/works/detail?unique=WJoYomLPg0JOYs8GazDVrw
![在这里插入图片描述](https://img-blog.csdnimg.cn/6d4646f75ffc43039bc83401150ab20c.png)
本篇文章所用的技术会整理后开源，后续可以持续关注：

[GitHub](https://github.com/BIGBOSS-dedsec)：**https://github.com/BIGBOSS-dedsec**

[CSDN](https://blog.csdn.net/weixin_50679163?type=edu)： **https://blog.csdn.net/weixin_50679163?type=edu**

同时本篇文章实现的技术参加了**稀土掘金2022编程挑战赛-游戏赛道**

[作品展示](https://hackathon2022.juejin.cn/#/works/detail?unique=WJoYomLPg0JOYs8GazDVrw)：https://hackathon2022.juejin.cn/#/works/detail?unique=WJoYomLPg0JOYs8GazDVrw
![alt](https://img-blog.csdnimg.cn/fe53f98d73984d669c9501c222ede030.png)
# 认识Mediapipe
项目的实现，核心是强大的**Mediapipe** ，它是**google**的一个**开源**项目：
| 功能            | 详细                   |
| ------------- | -------------------- |
| 人脸检测 FaceMesh | 从图像/视频中重建出人脸的3D Mesh |
| 人像分离          | 从图像/视频中把人分离出来        |
| 手势跟踪          | 21个关键点的3D坐标          |
| 人体3D识别        | 33个关键点的3D坐标          |
| 物体颜色识别        | 可以把头发检测出来，并图上颜色      |

[Mediapipe Dev](https://mediapipe.dev/)
![在这里插入图片描述](https://img-blog.csdnimg.cn/634b06ee5f7d4295a388e651e77f2b22.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQklHQk9TU3lpZmk=,size_20,color_FFFFFF,t_70,g_se,x_16)
以上是**Mediapipe**的几个常用功能   ，*这几个功能我们会在后续一一讲解实现*
Python安装**Mediapipe**
```python
pip install mediapipe==0.8.9.1
```
也可以用 **setup.py** 安装
[https://github.com/google/mediapipe](https://github.com/google/mediapipe)

# 项目环境
**Python			3.7
Mediapipe     0.8.9.1
Numpy		1.21.6
OpenCV-Python 		4.5.5.64
OpenCV-contrib-Python		4.5.5.64**
![在这里插入图片描述](https://img-blog.csdnimg.cn/e49c1c0fa21c4df08ee58e57ce57f8b4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQklHQk9TU3lpZmk=,size_20,color_FFFFFF,t_70,g_se,x_16)
*实测也支持Python3.8-3.9*
# 手势动作捕捉部分
**手势特征点图**
![在这里插入图片描述](https://img-blog.csdnimg.cn/5c947c080dbd49339d64b3c0e988ea41.png)
## 实时动作捕捉
本项目在Unity中实现**实时动作捕捉的核心**是通过本地**UDP**与**socket**  进行通信
关于数据文件部分，详细可以查看[OpenCV+Mediapipe人物动作捕捉与Unity引擎的结合](https://blog.csdn.net/weixin_50679163/article/details/124658313)中对数据文件部分的讲解和使用
![在这里插入图片描述](https://img-blog.csdnimg.cn/af582d78c2244f0a9a02b88f21381529.png)
# 核心代码
**摄像头捕捉部分：**

```python
import cv2

cap = cv2.VideoCapture(0)       #OpenCV摄像头调用：0=内置摄像头（笔记本）   1=USB摄像头-1  2=USB摄像头-2

while True:
    success, img = cap.read()
    imgRGB = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)       #cv2图像初始化
    cv2.imshow("HandsImage", img)       #CV2窗体
    cv2.waitKey(1)      #关闭窗体
```
**视频帧率计算**

```python
import time

#帧率时间计算
pTime = 0
cTime = 0

while True
cTime = time.time()
    fps = 1 / (cTime - pTime)
    pTime = cTime

    cv2.putText(img, str(int(fps)), (10, 70), cv2.FONT_HERSHEY_PLAIN, 3,
                (255, 0, 255), 3)       #FPS的字号，颜色等设置
```
**Socket通信：**
定义**Localhost**和**post**端口地址
```python
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
serverAddressPort = ("127.0.0.1", 5052)			# 定义IP和端口

# 发送数据
sock.sendto(str.encode(str(data)), serverAddressPort)
```

**手势动作捕捉：**

```python
    if hands:
        # Hand 1
        hand = hands[0]
        lmList = hand["lmList"] 
        for lm in lmList:
            data.extend([lm[0], h - lm[1], lm[2]])
```
## 完整代码
### Hands.py
```python
from cvzone.HandTrackingModule import HandDetector
import cv2
import socket

cap = cv2.VideoCapture(0)
cap.set(3, 1280)
cap.set(4, 720)
success, img = cap.read()
h, w, _ = img.shape
detector = HandDetector(detectionCon=0.8, maxHands=2)

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
serverAddressPort = ("127.0.0.1", 5052)

while True:  
    success, img = cap.read()   
    hands, img = detector.findHands(img) 
    data = []

    if hands:
        # Hand 1
        hand = hands[0]
        lmList = hand["lmList"]  
        for lm in lmList:
            data.extend([lm[0], h - lm[1], lm[2]])

        sock.sendto(str.encode(str(data)), serverAddressPort)

    cv2.imshow("Image", img)
    cv2.waitKey(1)
```
### py代码效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/2005161ae5ce4f318893b0767c6ee6f0.png)
# Unity 部分
## 建模
在Unity中，我们需要搭建一个人物的模型，这里需要一个**21个Sphere**作为**手势的特征点**和**21个Cube**作为**中间的支架**
具体文件目录如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/0f5e064941df492e8c261971a1e79711.png)
**Line的编号对应手势模型特征点**
![在这里插入图片描述](https://img-blog.csdnimg.cn/5c947c080dbd49339d64b3c0e988ea41.png)
# Unity代码
## UDP.cs
本代码功能**将Socket发送的数据进行接收**
```csharp
using UnityEngine;
using System;
using System.Text;
using System.Net;
using System.Net.Sockets;
using System.Threading;

public class UDPReceive : MonoBehaviour
{

    Thread receiveThread;
    UdpClient client; 
    public int port = 5052;
    public bool startRecieving = true;
    public bool printToConsole = false;
    public string data;


    public void Start()
    {

        receiveThread = new Thread(
            new ThreadStart(ReceiveData));
        receiveThread.IsBackground = true;
        receiveThread.Start();
    }
    
    private void ReceiveData()
    {

        client = new UdpClient(port);
        while (startRecieving)
        {
            try
            {
                IPEndPoint anyIP = new IPEndPoint(IPAddress.Any, 0);
                byte[] dataByte = client.Receive(ref anyIP);
                data = Encoding.UTF8.GetString(dataByte);

                if (printToConsole) { print(data); }
            }
            catch (Exception err)
            {
                print(err.ToString());
            }
        }
    }
}
```
### UDP.cs接收效果图
**无数据接收**下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/c2d297a4901944e6a9e7bf58fff9ca0f.png)
**数据通信接收下**
![在这里插入图片描述](https://img-blog.csdnimg.cn/441a6a66404e46718fac148268b2cc4c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5c17b5c93eb440288a0aa2d4d4f76d0e.png)
## Line.cs
这里是每个Line对应cs文件，实现功能：**使特征点和Line连接在一起**
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class LineCode : MonoBehaviour
{

    LineRenderer lineRenderer;

    public Transform origin;
    public Transform destination;

    void Start()
    {
        lineRenderer = GetComponent<LineRenderer>();
        lineRenderer.startWidth = 0.1f;
        lineRenderer.endWidth = 0.1f;
    }
// 连接两个点
    void Update()
    {
        lineRenderer.SetPosition(0, origin.position);
        lineRenderer.SetPosition(1, destination.position);
    }
}
```
## Hands.cs
这里是**读取上文识别并保存的手势动作数据**，**并将每个子数据循环遍历到每个Sphere点**，使特征点随着**摄像头的捕捉**进行运动
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class HandTracking : MonoBehaviour
{
    public UDPReceive udpReceive;
    public GameObject[] handPoints;
    void Start()
    {
        
    }
    void Update()
    {
        string data = udpReceive.data;
        
        data = data.Remove(0, 1);
        data = data.Remove(data.Length - 1, 1);
        print(data);
        string[] points = data.Split(',');
        print(points[0]);
        
        for (int i = 0; i < 21; i++)
        {

            float x = 7-float.Parse(points[i * 3]) / 100;
            float y = float.Parse(points[i * 3 + 1]) / 100;
            float z = float.Parse(points[i * 3 + 2]) / 100;

            handPoints[i].transform.localPosition = new Vector3(x, y, z);

        }

    }
}

```
### 最终实现效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/f26208cf46094cbdb3f914602b75daad.png)

**Good Luck，Have Fun and Happy Coding！！！**