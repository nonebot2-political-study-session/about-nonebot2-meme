<div align=center><img width="320" height="320" src="https://github.com/nonebot2-political-study-session/nb2PSS-Warehouse/blob/main/nb2.jpg"/></div>


# data_source相关


该文档主要对data_source.py进行基础介绍，简单来说，就是告诉你这个地方填啥，这玩意这么填是为什么、干什么用（这说的够明白了伐）

首先，最主要的位置为
```
memes = [

]
```

petpet将从此处读取所有的Meme类，进行有关头像表情包等一系列操作的反馈

在此，介绍一下Meme

```

@dataclass
class Meme:
    name: str
    func: Callable
    keywords: Tuple[str, ...]
    pattern: str = ""

    def __post_init__(self):
        if not self.pattern:
            self.pattern = "|".join(self.keywords)

```

结合万能表情的写法，来对号入座

```

Meme("universal", universal, ("万能表情", "空白表情")),

name -- "universal"
func -- universal
keywords -- ("万能表情", "空白表情")

```

其中pattern我们基本不管，当然我也不了解（不影响编写，咱就当他不存在吧）

估计各位已经了解了，keywords将成为我们触发该命令的引子，那么接下来我将会从需求程度来说说有关func和name的情况

## func(仅仅只是想要给petpet添加新表情包的同学)

该参数需要与另外一个关键文件function.py进行配合使用，在使用前，请在function.py中先添加对应的函数（自己写的时候先定义了也好）

用于在function.py中查找到对应名称的function(def)进行调用，生成对应图片的函数名称

起名时请尽量与对应表情包的含义相近（老实说公共项目你说提交个名字叫a1的函数名称也不好对不对）

关于function.py的方法编写将在about_function.md中描述


## name(还想再进一步了解petpet的运行的同学)

首先，在data_source.py的最下方有此代码
```
memes = [meme for meme in memes if meme.name not in petpet_config.petpet_disabled_list]
```

其实在init中使用的memes并非完全为我们以上（第一块）所定义的memes，他会先在此处进行处理，排除掉在禁用列表中的表情包，然后返回启用的表情列表

而此处的判断就由name是否在禁用列表（petpet_config.petpet_disabled_list）中来实现

其次，在manger.py中的MemeManger类，name为很重要的一个参数，在启用、禁用等方面会有使用


# 相关人员

编写者 [十年(shinianj)](https://github.com/shinianj)

图片提供 [十年(shinianj)](https://github.com/shinianj)






