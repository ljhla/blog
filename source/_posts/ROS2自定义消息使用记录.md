---
title: ROS2自定义消息使用记录
date: 2023-04-01 23:35:11
tags: 
abbrlink: 2llqu4i
categories: 
thumbnail: https://image.wxydejoy.top/uploads/ROS2%20%E8%87%AA%E5%AE%9A%E4%B9%89%E6%B6%88%E6%81%AF%20%E4%BD%BF%E7%94%A8%E8%AE%B0%E5%BD%95.png
imgonly: true
---


## 起因

最近在做人体姿态估计的项目,需要用到ROS2,但是ROS2的消息类型不够用,所以需要自定义消息类型,这里记录一下自定义消息类型的过程.

使用的工具如下

- ROS2 Humble
- VSCode
- Alphapose

之前跟着官方文档做过一次,但是学会了,但又没学会.这次是真的要用,反而出问题了.

## 自定义消息类型

之前以为每个包里面都可以自定义消息,但是想多了,需要独立的包来存放自定义消息.

### 创建包

```bash
ros2 pkg create --build-type ament_python --node-name people_msgs people_msgs
```

### 创建消息

```bash
# Point.msg
float32 x
float32 y
float32 z
```

```bash
# People.msg 以为人体关键点是26个,所以这里就写26个,但是后面发现出了问题
Point[26] keypoints
```
记住这个26,问题就出在这里.

```bash
# PeopleArray.msg
People[] people
```

### 编译

```bash
colcon build --packages-select people_msgs
```

### 使用

```bash
source install/setup.bash
```

## 使用自定义消息

### 创建包

```bash
ros2 pkg create --build-type ament_python --node-name people_sub people_sub
```

### 编写代码

```python
# people_sub.py
import rclpy
from rclpy.node import Node
from people_msgs.msg import PeopleArray

class PeopleSub(Node):
    def __init__(self):
        super().__init__('people_sub')
        self.sub = self.create_subscription(PeopleArray, 'people', self.people_callback, 10)

    def people_callback(self, msg):
        self.get_logger().info('I heard: %s' % msg)

    def getnode(self): # 假如说这是识别后处理关键点的函数
        peoplearray = PeopleArray()
        for i in peoplelist: # 假如说这是识别后所有人的列表
            people = People()
            # people.keypoints.clear() # 这里是关键,不然会出问题
            for j in i: #每个人身上的关键点
                point = Point()
                point.x = j[0]
                point.y = j[1]
                point.z = j[2]
                people.keypoints.append(point)
            peoplearray.people.append(people)
        return peoplearray
        
                
                
        
        return self

def main(args=None):
    rclpy.init(args=args)
    people_sub = PeopleSub()
    rclpy.spin(people_sub)
    people_sub.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```
### 编译

```bash
colcon build --packages-select people_sub
```

### 使用

```bash
source install/setup.bash
ros2 run people_sub people_sub
```

## 问题

我使用自定义消息进行接收,发现接收到的消息是空的,后来发现是因为我在创建消息的时候,它会默认创建一个空的消息(26个数据都是0),所以我在创建消息的时候,需要先清空消息,然后再添加数据.

## 感悟

果然*实践是检验真理的唯一标准*

这次的问题是单步调试出来的,调试之前处理了python路径的问题,不然无法使用VSCode调试.

### python扩展路径问题

```bash
# 1.通过脚本查看python路径
python -c "import sys; print(sys.path)"
```

设置环境vscode路径

Python扩展 设置 -> Pythoninterpreter Path -> (/usr/bin/python3)换成上一步的路径即可解决

还有个问题我问了下GPT

### ROS2使用自定义Python解释器
要指定ROS 2使用的Python解释器，您可以设置环境变量PYTHON_EXECUTABLE，将其设置为要使用的Python解释器的路径。例如，假设您的自定义Python解释器位于/path/to/my/python，则可以使用以下命令在终端中设置环境变量：


`export PYTHON_EXECUTABLE=/path/to/my/python`

设置环境变量后，ROS 2将使用指定的Python解释器。您可以在启动ROS 2节点时验证Python解释器是否正确设置，例如：

`ros2 run my_package my_node --ros-args --log-level debug`

这将在终端输出ROS 2节点的日志信息，其中包括使用的Python解释器。如果您看到的输出中包含您自定义的Python解释器路径，则表示ROS 2正在使用指定的Python解释器。

注意，设置自定义Python解释器可能会导致ROS 2某些功能无法正常工作，因为ROS 2可能依赖于特定版本的Python和Python库。如果您遇到问题，请尝试使用ROS 2默认的Python解释器，或者确保您的自定义Python解释器与ROS 2所需的Python版本和库兼容。
