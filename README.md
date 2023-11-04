# OpenCV-Unity-To-Build-3DHands

# Introduction
This article will introduce how to use **Python** with **OpenCV** image capture, with powerful **Mediapipe** library to achieve ** gesture detection ** and recognition; The recognition results are synchronized to Unity** * in real time to realize the recognition of the gesture model's moving body structure in Unity
# Demo
[Demo展示](https://hackathon2022.juejin.cn/#/works/detail?unique=WJoYomLPg0JOYs8GazDVrw)：https://hackathon2022.juejin.cn/#/works/detail?unique=WJoYomLPg0JOYs8GazDVrw
![在这里插入图片描述](https://img-blog.csdnimg.cn/6d4646f75ffc43039bc83401150ab20c.png)

[CSDN](https://blog.csdn.net/weixin_50679163?type=edu)： **https://blog.csdn.net/weixin_50679163?type=edu**

Python Install **Mediapipe**
```python
pip install mediapipe==0.8.9.1
```
Or Use **setup.py** to install
[https://github.com/google/mediapipe](https://github.com/google/mediapipe)

# Project environment
**Python			3.7
Mediapipe     0.8.9.1
Numpy		1.21.6
OpenCV-Python 		4.5.5.64
OpenCV-contrib-Python		4.5.5.64**
![在这里插入图片描述](https://img-blog.csdnimg.cn/e49c1c0fa21c4df08ee58e57ce57f8b4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQklHQk9TU3lpZmk=,size_20,color_FFFFFF,t_70,g_se,x_16)

# Gesture motion capture part
**Gesture feature point diagram**
![在这里插入图片描述](https://img-blog.csdnimg.cn/5c947c080dbd49339d64b3c0e988ea41.png)
## Real-time motion capture
The core of this project to achieve real-time motion capture in Unity ** is to communicate with **socket** via local **UDP**
For the data file section, Can view the detail [OpenCV + Mediapipe character combination of motion capture and Unity engine] (https://blog.csdn.net/weixin_50679163/article/details/124658313) to the data file is part of the interpretation and use
![在这里插入图片描述](https://img-blog.csdnimg.cn/af582d78c2244f0a9a02b88f21381529.png)
# Core Code
**Camera Capture：**

```python
import cv2

cap = cv2.VideoCapture(0)       #OpenCV摄像头调用：0=内置摄像头（笔记本）   1=USB摄像头-1  2=USB摄像头-2

while True:
    success, img = cap.read()
    imgRGB = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)       #cv2图像初始化
    cv2.imshow("HandsImage", img)       #CV2窗体
    cv2.waitKey(1)      #关闭窗体
```
**FPS**

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
**Socket：**
Define **Localhost**和**post**
```python
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
serverAddressPort = ("127.0.0.1", 5052)			# 定义IP和端口

# 发送数据
sock.sendto(str.encode(str(data)), serverAddressPort)
```

**Gesture motion capture：**

```python
    if hands:
        # Hand 1
        hand = hands[0]
        lmList = hand["lmList"] 
        for lm in lmList:
            data.extend([lm[0], h - lm[1], lm[2]])
```
## Final Code
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

![在这里插入图片描述](https://img-blog.csdnimg.cn/2005161ae5ce4f318893b0767c6ee6f0.png)
# Unity 
## Model
In Unity, we need to build a model of the character, here we need a **21 Sphere** as the feature point of the ** gesture and **21 Cube** as the stand ** in the middle of the ** gesture

# Unity Code
## UDP.cs
The function of this code ** will receive the data sent by the Socket **
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
### UDP.cs
**NoData**：
![在这里插入图片描述](https://img-blog.csdnimg.cn/c2d297a4901944e6a9e7bf58fff9ca0f.png)
**Socket Date**
![在这里插入图片描述](https://img-blog.csdnimg.cn/441a6a66404e46718fac148268b2cc4c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/5c17b5c93eb440288a0aa2d4d4f76d0e.png)
## Line.cs
Here is the cs file corresponding to each Line to achieve the functions: ** Connect the feature points and the Line together **
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
Here is ** read the gesture data identified above and saved **, ** and loop each sub-data to each Sphere point **, so that the feature point moves with the capture of ** camera **
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
