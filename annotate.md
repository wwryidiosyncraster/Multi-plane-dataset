# 软件基础

需要安装[Labelme](https://github.com/wkentaro/labelme)，基于PyQt且支持Ubuntu/macOS/Windows多平台。<br>
容器可用Anaconda一类python虚拟环境或docker。<br>

    labelme --labels label.txt
简单使用
## 快捷键
放大/缩小:Ctrl+滚轮 (！原图较高分辨率，加黑边后部分小物体建议放大看)<br>
新建多边形：Ctrl+N<br>
编辑多边形：Ctr+J<br>
撤销：Ctrl+Z<br>
删除多边形：Del<br>
上张/下张：A / D<br>
多个目标选择：Ctrl+点击图片或右侧列表的labels(编辑模式下才可选择)
多个目标移动：Ctrl+移动
建议设置：自动保存，保留上一张的标注结果(Ctrl+p)
# 数据概览
整体数据集包含40个场景，每个场景下存在7~9条视频，视频长度在10秒左右，30FPS。视频中会出现遮挡（特定物体/对象间），旋转，超出视角，视角偏移，尺度变化，动态模糊等因素。
## 视频
通过 Script/seq2vid.py 生成<br>

    python Script\seq2vid.py -r path-to-root-dir -s scene -v video-index
## 文件夹结构
<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="http://lrybbs.top/wp-content/uploads/2021/02/2021020116432151.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">文件夹结构。每个场景文件夹下有instances文件夹和多个图片序列文件夹</div>
</center>
<br>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="http://lrybbs.top/wp-content/uploads/2021/02/2021020116432457.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">instances下文件夹结构。instances文件夹中包含了候选图（展示所有可能出现的实例）和每个实例的细节。</div>
</center>
<br>

## 输出文件夹
按照Labelme默认，设为图片序列文件夹即可。

# 实例标注信息
你需要标注多个场景下的多个视频，视频已经事先处理好为带黑边图片序列。<br>
<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);"
    src="http://lrybbs.top/wp-content/uploads/2021/02/2021020116432681.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">一个场景中所有可能的候选instance</div>
</center>
<br>
在同一场景下，不同图片序列中所含的平面实例是大致相同的，但他们不一定都在序列的第一帧中出现，我们会给出每一个场景下所有可能的候选平面实例，再根据选择策略来在图片序列中选择应该被标注的实例。<br><br>

**对每个图片序列，你需要标注：**
- 每一个实例的位置信息
- 每一个实例的类别信息
- 每一个实例的Id信息


## 位置信息
对于每一个平面实例，用一个四边形框住，保证包含所有属于该平面的像素又具有最小的面积。
<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="http://lrybbs.top/wp-content/uploads/2021/02/2021020116432397.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">单一一个平面实例的位置信息</div>
</center>

**标注顺序** <br>标注时请按照平面的（**左上-右上-右下-左下**）顺时针的进行标注，实例旋转时请保留原本对应点。
<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="http://lrybbs.top/wp-content/uploads/2021/02/2021020116432176.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">平面位置信息标注顺序</div>
</center>

## 类别信息
我们预先给出了一个22类的list (label.txt)，包含室内外场景中我们需要标注的类别。
## Id信息
为图片序列中出现的实例分配一个非负整数(0~N)，根据实例出现的顺序，从0往上分配。
## 选择策略
为了避免主观性，对在图片序列中的每一帧图片中选择实例标注，我们有以下几条细则：
- **What？** 属于每个场景给出的候选实例中并且在当前视频下有完整出现过（出现过四条边），不需要标注用于遮挡的特定物体。
- **When？** (语义)通过刚体形状不变和恒定速度假设，可以确定的推断出物体位置，并且满足(物理)实例四边形存在三条边及以上出现在当前图像中时, 进行标注。当无法准确标注位置信息时（不易推断位置/物体面积太小/物体形状过于细长/物体被遮挡过多），不进行标注。
- **遮挡/超出视角！** 指平面对象间或特定物体造成的遮挡/平面对象超出摄像头外。在能通过简单的（恒定速度模型/刚体平面假设）推断出实例位置时继续标注并保持Id，否则不进行标注，并在下一次该对象出现时分配新的Id。
- **超出视角/尺度变化的判断！** 对于平面四边形，存在3条及以上的边出现，则进行标注，否则不进行标注并在下一次该对象出现时分配新的Id。

# 标注过程
<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="http://cdn.zzcheng.top/annotate.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">每一帧图片的标注方式。其中，object label一栏需要填入平面实例的类别信息，group id需要填入分配给平面实例的Id</div>
</center>
<br>

**Init:** 观看该场景的候选instance和对应多个视频
<br>

**Step1:** 标注视频对应的图片序列。

**Step2:** 标注单帧图片，从候选instance中比对选择应被标注的平面instance。

**Step3:** 用四边形标注位置信息，标注类别信息并分配instance id。

**Step4:** 如果未标完当前图片序列，返回2

**Step5:** 如果未标完当前场景，返回1
<br>

**End:** 打包所有Json文件

<br>

