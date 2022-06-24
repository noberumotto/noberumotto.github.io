## # 00

我喜欢使用 WPF 是因为它能够很方便地做出漂亮的 Windows GUI 界面，第一次使用我已经不记得是什么时候的事了，应该是好多年前了，但 WPF 并不是我的初恋。

那一年，智能手机还没有像如今这样便宜，身边大多人开始使用智能手机了，我还在使用 `诺基亚直板按键`手机，唯一能接触到的智能设备只有电脑，所以我当时很想在电脑上做出一个软件。

通过百度一番搜寻，我了解到了`易语言`，它把我带进了 Windows 软件的世界，也是带我初尝禁果的对象。可视化的UI编辑界面，所见即所得，中文语言编程，当时这些对我来说简直是强大到爆，那时候我想应该这应该是最厉害的语言了吧。由于时间过去好几年了，我已经不记得用易语言做过什么值得回忆的软件，但还是有。

如果你也曾经在那个时空，我想你应该会知道百度上流传的刷钻软件，什么钻呢，就是QQ上各种订阅服务，像红钻、绿钻、黄钻。在那个年代，在我当时的学校，如果你拥有其中的任意一个钻，那你会成为班上的红人，万众瞩目。我想应该不止我的学校是这样，大多数吧，因为开通一个钻至少需要10块钱，在当时10块钱对于我们学生来说是笔巨款，更别说每个月都要续费了，所以Q钻身份的象征可不是说说而已。回到刷钻软件，接触到易语言和刷钻的时间线几乎是相同的，因为当时的刷钻软件百分之90都是易语言做的，我在逛一些开发论坛的时候就发现了，这些刷钻软件百分之99都是钓鱼用的，界面都是各种钻图标，打着免费的旗号让你输入QQ号和密码，你要是真信并输入点确定了，那你的信息就被 POST 到钓鱼者的服务器里。虽然不愿意承认，但我也上过钩。

由于当时对善恶好坏的认知并不清晰，我也用易语言做了刷钻钓鱼软件，但是并没有传播。这也让我当时接触到了很多新的关于开发的知识，比如了解到了http服务器。做好之后我发现软件在电脑上还没打开就被360软件以病毒对象拦截了，虽然当时有很多免杀教程，但是我对易语言失去兴趣了。被杀软报毒并不是放弃的第一因素，难以做出漂亮界面以及中文编程繁琐的问题也在其中。

于是辗转之下，我了解到了 WinForm，接着就是 WPF。

## # 01

WPF 是 Windows 下当之无愧的完美 GUI 框架，我认为。除了内存占用是硬伤，其他无可挑剔。

无论是数据绑定、XAML布局、动画都是极其方便的，只需要很少的代码就能实现很棒的 UI。但经过长时间的使用以及接触到前端像 VUE 之后后我的想法变了，我希望在 WPF 布局能更简单，或者说跟 VUE 一样。

XAML的语法实在有些重了，我想要是能像 HTML 和 CSS一样就好了，于是就有了这篇博客以及我的第一个实验项目。

为了有代码提示和格式化，我创建了一个 index.vue 文件，把它放到了 WPF 项目中，生成属性暂时用 `资源` 吧，我直接在项目运行时拿到并把它翻译成界面。

```js
<template>
  <div class="app">hello,berumotto!</div>
</template>

<script>
</script>

<style scoped>
.app {
  background: red;
  width: 150px;
  height: 50px;
  color: green;
}
</style>
```

首先是解析出 template ，先用正则表达拿到里面的内容。

```csharp
var template = Regex.Match(text, @"<template>(?<text>[\s\S]*)</template>");
```
拿到内容后就要开始解析标签了，我用到了`状态机`，把内容中的字符一个个切割然后进行处理。首先是要将字符定义好类型，像`<`属于标签打开符号，`"`属于标签属性值的开始符号，`>`属于标签闭合符号。

```csharp
public enum CharType
        {
            /// <summary>
            /// 小于 ＜
            /// </summary>
            Less,
            /// <summary>
            /// 大于 >
            /// </summary>
            Greater,
            /// <summary>
            /// 字符
            /// </summary>
            String,
            /// <summary>
            /// 空格
            /// </summary>
            Space,
            /// <summary>
            /// 等于 =
            /// </summary>
            Equalsign,
            /// <summary>
            /// 双引号 "
            /// </summary>
            Doublequote,
            /// <summary>
            /// 斜杠 /
            /// </summary>
            Slash

        }
```
然后定义好状态，像读到 div 的d开始状态切到`Tag`标签名称状态，读到 class 的c开始状态切换到`Propertie`属性名称状态。
```csharp
public enum State{
            Wait,
            Tag,
            Propertie,
            PropertieValue,
            Input,
            Content
}
```

接下来是创建两个数据结构，一个是 Tag 标签，一个是标签的属性。

Tag.cs
```csharp
    public struct Tag
    {
        public string Name { get; set; }
        public List<TagProperty> Properties { get; set; }

        public object Content { get; set; }
    }
```
TagProperty.cs
```csharp
    public struct TagProperty
    {
        public string Name { get; set; }

        public string Value { get; set; }
    }
```
接着就是一个简单的状态机
```csharp
            int index = 0;
            string token = "";
            var tag = new Tag();
            tag.Properties = new List<TagProperty>();

            TagProperty proptie = new TagProperty();

            while (index < Text.Length)
            {
                char c = Text[index];
                var cType = GetCharType(c);
                switch (state)
                {
                    case State.Wait:
                        switch (cType)
                        {
                            case CharType.Less:
                                state = State.Tag;
                                break;
                        }
                        break;

                    case State.Tag:
                        switch (cType)
                        {
                            case CharType.String:
                                token += c;
                                break;
                            case CharType.Space:
                                state = State.Propertie;
                                tag.Name = token;
                                token = "";
                                break;
                            case CharType.Greater:

                                break;
                        }
                        break;

                    case State.Propertie:
                        switch (cType)
                        {
                            case CharType.String:
                                token += c;
                                break;
                            case CharType.Equalsign:
                                state = State.PropertieValue;
                                index += 1;
                                proptie = new TagProperty();
                                proptie.Name = token;
                                token = "";
                                break;
                        }
                        break;
                    case State.PropertieValue:
                        switch (cType)
                        {
                            case CharType.Doublequote:
                                proptie.Value = token;
                                tag.Properties.Add(proptie);
                                token = "";
                                state = State.Input;
                                break;
                            case CharType.String:
                                token += c;
                                break;
                        }
                        break;
                    case State.Input:
                        switch (cType)
                        {
                            case CharType.Greater:
                                //  完成标签的开始部分
                                state = State.Content;
                                break;
                        }
                        break;
                    case State.Content:

                        switch (cType)
                        {
                            case CharType.Less:
                                if (Text[index + 1] == '/')
                                {
                                    //  结束标记
                                    tag.Content = token;

                                }
                                break;
                            default:

                                token += c;
                                break;
                        }
                        break;
                }
                index++;
            }

            tags.Add(tag);
```
**由于是初步试验，并没有处理所有情况，只是针对 index.vue 进行处理。**

这样，就成功得到 div 标签，标签的属性 class，class 的值 app，以及 div 的内容 hello,berumotto!。

`css`也是一样，使用同样的方法翻译成可识别的数据结构。

最后就是将这些结构数据转换成 WPF 中的控件

Render.cs
```csharp
    public class Render
    {
        private VirtualizingStackPanel content;
        private List<core.Tags.Tag> tags;
        private List<Core.Style.Style> styles;
        public Render(List<core.Tags.Tag> tags, List<Core.Style.Style> styles)
        {
            content = new VirtualizingStackPanel();

            this.tags = tags;
            this.styles = styles;
        }

        public VirtualizingStackPanel GetContent()
        {
            return content;
        }
        public void Handle()
        {

            foreach (var tag in tags)
            {
                var el = new Div();
                el.Name = tag.Name;
                el.HorizontalAlignment = System.Windows.HorizontalAlignment.Left;
                el.VerticalAlignment = System.Windows.VerticalAlignment.Top;
                HandlePropertie(el, tag.Properties);

                el.Content = new TextBlock()
                {
                    Text = tag.Content.ToString()
                };

                content.Children.Add(el);
            }
        }

        private void HandlePropertie(Div control, List<Tags.TagProperty> propties)
        {
            foreach (var propertie in propties)
            {
                switch (propertie.Name)
                {
                    case "class":
                        SetStyles(control, propertie.Value);
                        break;
                }
            }
        }

        private void SetStyles(Div control, string className)
        {
            Debug.WriteLine("给：" + control.Name + "，设置样式：" + className);

            var style = styles.Where(m => m.Name == className).FirstOrDefault();
            foreach (var p in style!.Properties)
            {
                Debug.WriteLine("给：" + control.Name + "，设置样式：" + className + " -> " + p.Name + " > " + p.Value);

                switch (p.Name)
                {
                    case "background":
                        control.Background = GetBrush(p.Value);
                        break;
                    case "color":
                        control.Foreground = GetBrush(p.Value);
                        break;
                    case "width":
                        control.Width = GetDouble(p.Value);
                        break;
                    case "height":
                        control.Height = GetDouble(p.Value);
                        break;
                }

            }
        }

        private double GetDouble(string value)
        {
            return double.Parse(value.Replace("px", ""));
        }

        private Brush GetBrush(string color)
        {
            if (color.ToLower() == "red")
            {
                return new SolidColorBrush(Colors.Red);
            }
            else if (color.ToLower() == "black")
            {
                return new SolidColorBrush(Colors.Black);
            }
            else if (color.ToLower() == "white")
            {
                return new SolidColorBrush(Colors.White);
            }
            else if (color.ToLower() == "green")
            {
                return new SolidColorBrush(Colors.Green);
            }
            return new SolidColorBrush(Colors.Green);
        }

    }
```

在 Mainwindow.xaml.cs 中运行起来

```csharp
        public MainWindow()
        {
            InitializeComponent();

            var test = Application.GetResourceStream(new Uri("pack://application:,,,/WpfApp3;component/Views/index.vue", UriKind.RelativeOrAbsolute));

            StreamReader reader = new StreamReader(test.Stream);
            string text = reader.ReadToEnd();

            var template = Regex.Match(text, @"<template>(?<text>[\s\S]*)</template>");
            var style = Regex.Match(text, @"<style(.*?)>(?<text>[\s\S]*)</style>");

            var tagBuilder = new core.TagBuilder(template.Groups["text"].Value);
            tagBuilder.Render();

            var styleBuilder = new core.StyleBuilder(style.Groups["text"].Value);
            styleBuilder.Render();

            var tags = tagBuilder.GetTags();
            var styles = styleBuilder.GetStyles();

            var render = new Render(tags, styles);
            render.Handle();

            Content = render.GetContent();
        }
```

![img](d/图片/wpfapp3.jpg)

一个简单的想法和作品完成了，当然如果到了真实项目中肯定不能这样在项目运行时编译 index.vue 文件，性能可想而知的差。后面我想直接翻译成 xaml 文件。

## # 02

如果你对这个项目源码感兴趣，我把它放到了 Github 上。

链接：[https://github.com/noberumotto/WpfApp3](https://github.com/noberumotto/WpfApp3)