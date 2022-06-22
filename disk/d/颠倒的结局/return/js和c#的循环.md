在`w95`中，可以在桌面右键创建文本文档，这个时候需要创建一个文档名字，按照`windows95`操作系统的命名规则，第一个文档命名为`新建文本文档`，第二个`新建文本文档 (2)`...以此类推。

我的思路很简单粗暴，就使用递归去对比桌面上的所有文件名，重合了就加一个数字，否则就是返回这个名字，代码像这样：

```js
const names = ["新建文本文档", "新建文本文档1", "新建文本文档2"];

function getCreateName(name, newName = "", index = 1) {
      if (newName == "") {
        newName = name;
      }
      index++;
      names.forEach((name_) => {
        if (name_ == newName) {
          return getCreateName(name, name + index, index);
        }
      });

      // console.log(newName);
      
      return newName;
    },

console.log(getCreateName("新建文本文档"));
```

运行后我发现每次都是返回`新建文本文档`，而不是我想要的`新建文本文档3`，我就纳闷了，于是在最后一句返回 newName 前加了打印语句看看情况，结果发现循环了3次

*打印日志*
```
新建文本文档3
新建文本文档2
新建文本文档
```

然后我才发现在js中的循环里用return并不能终止下面的代码运行，我有点懵了，我印象中循环里return应该是能终止的啊。

我在c#里又测试了一遍，同样的代码

```csharp
 static string[] names = { "新建文本文档", "新建文本文档1", "新建文本文档2" };

        static void Main(string[] args)
        {
            Console.WriteLine(GetCreateName("新建文本文档"));
            Console.ReadKey();
        }

        static string GetCreateName(string name_, string newName_ = "", int index = 0)
        {
            if (string.IsNullOrEmpty(newName_))
            {
                newName_ = name_;
            }

            index++;

            foreach (string name in names)
            {
                if (name == newName_)
                {
                    return GetCreateName(name_, name_ + index, index);
                }
            }
            return newName_;
        }
```

运行后控制台输出的是：`新建文本文档3`

结果证明js的循环中和c#的循环中使用return是有区别的，js中return不能终止下面的代码，而c#可以。

习惯了c#，写js时并没有想到这个问题，当时折腾得我头大。