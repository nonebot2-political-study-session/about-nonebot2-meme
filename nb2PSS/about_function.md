<div align=center><img width="320" height="320" src="https://github.com/nonebot2-political-study-session/nb2PSS-Warehouse/blob/main/nb2.jpg"/></div>


# function相关


该文档主要对function.py进行基础介绍，至少，用简单易懂的普通话告诉你怎么写一个表情出来

本次我会从petpet中提取几个方法当做例子，其中将会分为4类，而难度也在从上而下递增
（有时可能会出现特殊情况，这里是大致的难度排序）
：

* 1.素材静图类（使用PS等抠图技术加上代码进行图层叠加实现）
* 2.素材动图类（在素材静图的基础上生成gif动图）
* 3.源码静图类（不使用素材，在获取的图片上通过代码进行处理实现）
* 4.源码动图类（在源码静图的基础上生成gif动图）

首先，在介绍怎么编写以上几种图片时，需要先给各位介绍一些参数传入的介绍

PS：在本质上，我们将艾特目标和发送图片视为同一事物，接下来的介绍都将以图片为基准

```
img: BuildImage= UserImg()
 (petpet对于单图片获取所一般定义的参数，类型为BuildImage，一般默认参数为UserImg()),

arg 
（petpet对于除指令外的文字所一般定义的参数，一般类型为str，一般默认参数为NoArg()）,

user_imgs: List[BuildImage] = UserImgs(min,max)
（petpet对于多图片处理所一般定义的参数，一般类型为List[BuildImage]，一般默认参数为UserImgs(),max设定为获取图片最大量）,

sender_img: BuildImage = SenderImg()
（petpet对于多图片处理时所获取到的图片不足时，用机器人头像以填补所一般定义的参数，一般类型为BuildImage）,

user: UserInfo= User()
（petpet对于用户信息的获取所一般定义的参数，
一般类型为UserInfo,一般默认参数为User()，UserInfo属性包括：
    qq（qq号） group（群号） name（用户名称） gender（用户性别） img_url（用户头像网址） img(用户头像)

```
接下来是对BuildImage类的介绍

## BuildImge(包含于nonebot_plugin_imageutils中)

作为petpet以及memes中的最核心的类，它是nonebot2封装下的PIL，因此大多数情况，

倘若无法通过直接使用BuildImage中的方法进行处理以达到理想效果，可以尝试进行

PIL输出ByteIO，再由BuildImage进行封装输出

BuildImage只有一个属性image，能够返回
``width``（宽）、``height``（高）、``size``（width,height）、``mode``(图像模式，例如RGB、RGBA)、``draw``（ImageDraw类，我看不出来干嘛的，用不上就不说了吧）
例（设img为一个BuildImage类的变量)：
```
#获取图片宽度(int)

width = img.width

#获取图片高度(int)

height = img.height

#获取图片宽与高(SizeType)

width,height = img.size

#获取图片图像模式（基本不用）(ModeType)

mode = img.mode

```
注：获取图片宽高，此操作在复合类型的表情制作时十分有用，在一般的使用中则偏向少见

接下来是几个比较重要的函数介绍

### BuildImage.new(mode:ModeType,size:SizeType,color:Optional[ColorType])

作为最常见的背景及纯色块创建方式
使用例子（创建一个宽高为500*500的纯白色画布）：
```
img = BuildImage.new('RGB',(500,500),'white')
```
或是
```
img = BuildImage.new('RGB',(500,500),'#FFFFFF')

img = BuildImage.new('RGB',(500,500),(255,255,255))

img = BuildImage.new('RGBA',(500,500),(255,255,255,100))

```

以上方法中有区别的只是关于color参数的修改，color参数在黑白等颜色上比较特殊，允许直接使用'white'和'black'，

其余颜色则使用16进制进行判别，最后一种只是因为使用的图像模式为RGBA，加入了不透明度，因此需要添加一个alpha值

### BuildImage.copy()

创建一个BuildImage类的副本，其用途在于让你别瞎动源变量，副本随你玩
例 (设img为一个BuildImage类变量,创建一个img副本）：

```
img1 = img.copy()
```

### BuildImage.resize(size: SizeType,resample: ResampleType = Image.ANTIALIAS,keep_ratio: bool = False,inside: bool = False,direction: DirectionType = "center",bg_color: Optional[ColorType] = None,**kwargs)

该方法用于调整图像尺寸，参数很多是吧（笑）

*``size``：期望图片大小
*``keep_ratio``: 是否保持长宽比，默认为 `False`
*``inside``: `keep_ratio` 为 `True` 时，若 `inside` 为 `True`，则调整图片大小至包含于期望尺寸，不足部分设为指定颜色；若 `inside` 为 `False`，则调整图片大小至包含期望尺寸，超出部分裁剪
*``direction``: 调整图片大小时图片的方位；默认为居中
可选参数：``center``,``north``,``south``,``west``,``east``,``northwest``,``northeast``,``southwest``,``southeast``
*``bg_color``: 不足部分设置的颜色

其必要参数仅仅为size，其余参数需要设定时自行设置
例（将一个名为img的 500 * 500 的BulidImage图像更改为 1000 * 800 的BulidImage图像）：
```
img = img.resize((1000,800))
```
倘若添加 keep_ratio = True ，则会变为 800 * 800 的图像

如果keep_ratio = True时，同时设置inside = True，则会生成一个 1000 * 800 的图片，其中 800 * 800 的原图片会在正中间，而四周将会被黑色所覆盖（也算是某种意义上的描边处理了）

### BuildImage.resize_canvas(size: SizeType,direction: DirectionType = "center",bg_color: Optional[ColorType] = None,)
调整“画布”大小，超出部分裁剪，不足部分设为指定颜色，相信很多玩PS或者绘画的同学都懂画布大小的情况吧

* ``size``: 期望图片大小
* ``direction``: 调整图片大小时图片的方位；默认为居中，可选参数参考resize
* ``bg_color``: 不足部分设置的颜色

例（将一个名为img的 500 * 500 的BulidImage图像更改为 1000 * 800 画布的BulidImage图像）：
```
img = img.resize_canvas((1000,800))
```
### BuildImage.resize_width(width: int, **kwargs)

调整图片宽度，不改变长宽比
例（将一个名为img的 500 * 500 的BulidImage图像通过以下代码更改大小）：
```
img = img.resize_width(1000)
```
返回一张内容与img相同的 1000 * 1000 的BulidImage图像
### BuildImage.resize_height(height: int, **kwargs)

调整图片高度，不改变长宽比
例（将一个名为img的 500 * 500 的BulidImage图像通过以下代码更改大小）：
```
img = img.resize_height(1000)
```
返回一张内容与img相同的 1000 * 1000 的BulidImage图像

### BuildImage.rotate(angle: float,resample: ResampleType = Image.BICUBIC,expand: bool = False,**kwargs)

旋转图片
* ``angle``: 旋转角度，逆时针为正方向，采用角度制

例（将一个名为img的 500 * 500 的BulidImage逆时针旋转90度）：
```
#使用math库更加准确
import math
img = img.rotate(math.pi/2)
```

### BuildImage.square()

将图片裁剪为方形

例（将一个名为img的 500 * 600 的BulidImage裁剪为方形）：
```
img = img.square()
```

返回 500 * 500 的BulidImage图像

### BuildImage.circle()

将图片裁剪为圆形

例（将一个名为img的 500 * 600 的BulidImage裁剪为圆形）：
```
img = img.circle()
```

返回 500 * 500 的圆形内容BulidImage图像

### BuildImage.circle_corner(r: float)

将图片裁剪为圆角矩形
* ``r``: 圆角半径

例（将一个名为img的 500 * 600 的BulidImage裁剪为圆角半径为5的矩形）：
```
img = img.circle_corner(5)
```

返回 500 * 600 的圆角半径为5的矩形BulidImage图像

### BuildImage.crop(box: BoxType)

截取图片一部分
* ``box``: 其参数为截取位置的左上角xy（为前两位参数值），右下角xy（后两位参数值）,他们的类型都为int,在参数计算时若不用整除请记得添加int()取整

例（截取一个名为img的 500 * 600 的BulidImage的中心位置的一块 100 * 100 的图片）：
```
img = img.crop((int(img.width/2-50),int(img.height/2-50),int(img.width/2+50),int(img.height/2+50)))
```

返回 100 * 100 内容为img中央位置的BulidImage图像

### BuildImage.convert(mode: ModeType, **kwargs)

更改图片图像模式

* ``mode``: 修改后的图像模式

例（将一个名为img的 500 * 600 的‘RGB’模式BulidImage变为‘RGBA’模式）：
```
img = img.convert('RGBA')
```

返回 500 * 600 内容为img图像模式为'RGBA'的BulidImage图像

### BuildImage.paste(img: Union[IMG, "BuildImage"],pos: PosTypeInt = (0, 0),alpha: bool = False,below: bool = False,)
粘贴图片,最常用的方法
* ``img``: 待粘贴的图片
* ``pos``: 粘贴位置,带粘贴图片的左上角的xy，为int
* ``alpha``: 图片背景是否为透明
* ``below``: 是否粘贴到底层
例（将一个名为img1的 100 * 100 的BulidImage粘贴到另一个名为img2的 500 * 600 的BulidImage的中央并存在img3中：
```
img3 = img2.paste(img1,(int(img2.width\2-img1.width\2),int(img2.height\2-img1.height\2)))
```

返回 500 * 600 内容为img2中央为img1的BulidImage图像
注：不能将图片左上角坐标写为负数
### BuildImage.draw_text(xy: XYType,text: str,max_fontsize: int = 30,min_fontsize: int = 12,allow_wrap: bool = False,style: FontStyle = "normal",weight: FontWeight = "normal",fill: ColorType = "black",spacing: int = 4,halign: HAlignType = "center",valign: VAlignType = "center",lines_align: HAlignType = "left",stroke_ratio: float = 0,stroke_fill: Optional[ColorType] = None,font_fallback: bool = True,fontname: str = "",fallback_fonts: List[str] = [],)
噔噔咚！吓死了吧！参数多的和鬼一样的，还很重要哦，这是粘贴文字在图片上的操作
* ``xy``: 文字区域，顺序依次为 左上xy，右下xy，类型为float
* ``text``: 文字，支持多行
* ``max_fontsize``: 允许的最大字体大小
* ``min_fontsize``: 允许的最小字体大小
* ``allow_wrap``: 是否允许折行
* ``style``: 字体样式，默认为 "normal"，其余可选“italic”，"oblique"
* ``weight``: 字体粗细，默认为 "normal"，其余可选"ultralight", "light", "bold", "ultrabold", "heavy"
* ``fill``: 文字颜色
* ``spacing``: 多行文字间距
* ``halign``: 横向对齐方式，默认为居中
* ``valign``: 纵向对齐方式，默认为居中
* ``lines_align``: 多行文字对齐方式，默认为靠左
* ``stroke_ratio``: 文字描边的比例，即 描边宽度 / 字体大小
* ``stroke_fill``: 描边颜色
* ``font_fallback``: 是否使用后备字体，默认为 `True`
* ``fontname``: 指定首选字体
* ``fallback_fonts``: 指定备选字体

其实没啥好怕的啦，必需参数也只是``xy``，``text``，当然想要达到更好的效果，那当然要传入更多参数了
例（在一个名为img的 500 * 600 的BulidImage的左上角粘贴字号大小为25，字体为IPix(一个中文像素字体)的文字内容为'wq佬牛逼'的图片：
```
img = img.draw_text((0,0,150,30),text = 'wq佬牛逼',fontname = 'IPix.tff')
```
返回 500 * 600 内容为img左上角带有像素文字'wq佬牛逼'的BulidImage图像

注：若首选字体不在resources中，将会跳过使用备用字体，在resources中的配置将会在下方有关资源配置时提到
又注：字体的字号和数量会影响到文字区域的大小，文字区域大小必须能够包含所有文字，若无法预知文字区域究竟为多少，可以转用Text2Image预先生成文字图片，再使用图片粘贴

#输出相关函数
## bulid_image中
### save_jpg（bg_color: ColorType = "white"）
保存为jpg类型
* ``bg_color``: 由 png 转为 jpg 时的背景颜色，默认为白色

返回BytesIO类型
### save_png（）
保存为png类型

返回BytesIO类型
## utils中
### save_gif(frames: List[IMG], duration: float)
保存gif动图
* ``frames``:图片列表，存储gif动图的每一帧图片
* ``duration``:每帧之间的时间间隔，单位为秒
  
  具体用途，动图例子中将会解释

  返回BytesIO类型
### make_jpg_or_gif(img: BuildImage, func: Maker, keep_transparency: bool = False)
制作动图或静图
* ``img``: 输入图片
* ``func``: 图片处理函数，输入img，返回处理后的图片
* ``keep_transparency``: 传入gif时，是否保留该gif的透明度

具体用途，例子中将会解释

返回BytesIO类型
### make_gif_or_combined_gif(img: BuildImage,maker: GifMaker,frame_num: int,duration: float,frame_align: FrameAlignPolicy = FrameAlignPolicy.no_extend,input_based: bool = False,keep_transparency: bool = False,)
* ``img``: 输入图片，如头像
* ``maker``: 图片处理函数生成，传入第几帧，返回对应的图片处理函数
* ``frame_num``: 目标gif的帧数
* ``duration``: 相邻帧之间的时间间隔，单位为秒
* ``frame_align``: 要叠加的gif长度大于基准gif时，gif长度对齐方式
* ``input_based``: 是否以输入gif为基准合成gif，默认为`False`，即以目标gif为基准
* ``keep_transparency``: 传入gif时，是否保留该gif的透明度
  
  因为使用次数不多，仅仅只是列入此处

  返回BytesIO类型

# 资源配置
## 为了给petpet提供pr的人士
在github从petpet中fork出来的自己仓库的resources文件夹中的image文件夹中添加以新Meme的name为名的文件夹，将图片资源从0开始编号；在fonts中添加入对应需要的tff文件
```
--nonebot_plugin_petpet
--resources
    --fonts
        --Ipix.tff
        ...
    --images
        --Meme.name
            --0.png
            --1.png
            ...
```
## 为了自用圈子petpet的人士
在本地文件夹bot目录中的data文件夹的petpet中的resources文件夹中的image文件夹中添加以新Meme的name为名的文件夹，将图片资源从0开始编号；在fonts中添加入对应需要的tff文件
```
--data
    --petpet
        --fonts
            --Ipix.tff
            ...
        --images
            --Meme.name
                --0.png
                --1.png
                ...
```
# 例子说明
## 1.素材静图类
```
Meme("rip_angrily", rip_angrily, ("怒撕",))

def rip_angrily(img: BuildImage = UserImg(), arg=NoArg()):

    #将从UserImg()获取的图片进行RGBA转换，切方并且进行大小调整
    img = img.convert("RGBA").square().resize((105, 105))

    #使用load_image从资源库中读取资源并赋值到frame
    frame = load_image("rip_angrily/0.png")

    #粘贴旋转后的img并粘贴于素材frame之下
    frame.paste(img.rotate(-24, expand=True), (18, 170), below=True)

    #同上
    frame.paste(img.rotate(24, expand=True), (163, 65), below=True)

    #返回对应静图
    return frame.save_jpg()
```
## 2.素材动图类
```
Meme("pat", pat, ("拍",))

def pat(img: BuildImage = UserImg(), arg=NoArg()):

    #将从UserImg()获取的图片进行RGBA转换，切方
    img = img.convert("RGBA").square()

    #创建坐标列表
    locs = [(11, 73, 106, 100), (8, 79, 112, 96)]

    #创建图像列表
    img_frames: List[IMG] = []

    #进行gif逐张处理（10为事先确认素材张数）
    for i in range(10):

        #使用load_image从资源库中读取资源并赋值到frame
        frame = load_image(f"pat/{i}.png")

        #进行左上角xy和右下角xy赋值
        x, y, w, h = locs[1] if i == 2 else locs[0]

        #进行图片粘贴
        frame.paste(img.resize((w, h)), (x, y), below=True)

        #将此次处理好的图片放置于图像列表中
        img_frames.append(frame.image)
    
    #预先设定图片播放顺序
    seq = [0, 1, 2, 3, 1, 2, 3, 0, 1, 2, 3, 0, 0, 1, 2, 3, 0, 0, 0, 0, 4, 5, 5, 5, 6, 7, 8, 9]

    #对图片进行重新排列
    frames = [img_frames[n] for n in seq]

    #返回gif动图，时间间隔为0.085
    return save_gif(frames, 0.085)
```
## 3.源码静图类
```
Meme("cyan", cyan, ("群青",))

def cyan(img: BuildImage = UserImg(), arg=NoArg()):

    #确定替换颜色
    color = (78, 114, 184)

    #将从UserImg()获取的图片进行RGB转换，切方并调整大小，最后添加颜色滤镜(colr_mask)
    frame = img.convert("RGB").square().resize((500, 500)).color_mask(color)

    #在图片上添加文字
    frame.draw_text(
        (400, 40, 480, 280),
        "群\n青",
        max_fontsize=80,
        weight="bold",
        fill="white",
        stroke_ratio=0.04,
        stroke_fill=color,
    ).draw_text(
        (200, 270, 480, 350),
        "YOASOBI",
        halign="right",
        max_fontsize=40,
        fill="white",
        stroke_ratio=0.06,
        stroke_fill=color,
    )

    #返回图片
    return frame.save_jpg()
```
## 4.源码动图类
```
#数学不好，这个就自己看看吧（真不愧是wq佬）

Meme("wave", wave, ("波纹",))

def wave(img: BuildImage = UserImg(), arg=NoArg()):

    #获取图片，进行比较，确定图像宽度
    img_w = min(max(img.width, 360), 720)
    period = img_w / 6
    amp = img_w / 60

    #设定gif张数
    frame_num = 8
    phase = 0

    #创建sin函数
    sin = lambda x: amp * math.sin(2 * math.pi / period * (x + phase)) / 2

    #创建制作函数的制作函数……（套娃呢）
    def maker(i: int) -> Maker:
        def make(img: BuildImage) -> BuildImage:
            img = img.resize_width(img_w)
            img_h = img.height
            frame = img.copy()
            for i in range(img_w):
                for j in range(img_h):
                    dx = int(sin(i) * (img_h - j) / img_h)
                    dy = int(sin(j) * j / img_h)
                    if 0 <= i + dx < img_w and 0 <= j + dy < img_h:
                        frame.image.putpixel(
                            (i, j), img.image.getpixel((i + dx, j + dy))
                        )

            frame = frame.resize_canvas((int(img_w - amp), int(img_h - amp)))
            nonlocal phase
            phase += period / frame_num
            return frame

        return make

    return make_gif_or_combined_gif(
        img, maker, frame_num, 0.01, FrameAlignPolicy.extend_loop
    )
```
# 相关人员

编写者 [十年(shinianj)](https://github.com/shinianj)

图片提供 [十年(shinianj)](https://github.com/shinianj)






