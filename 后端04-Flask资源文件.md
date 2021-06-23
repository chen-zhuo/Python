# Flask资源文件

### 动态模板

在一般的 Web 程序里，访问一个地址通常会返回一个包含各类信息的 HTML 页面。因为我们的程序是动态的，页面中的某些信息需要根据不同的情况来进行调整，比如对登录和未登录用户显示不同的信息，所以页面需要在用户访问时根据程序逻辑动态生成。

我们把包含变量和运算逻辑的 HTML 或其他格式的文本叫做**模板**，执行这些变量替换和逻辑计算工作的过程被称为**渲染**，**在Flask中渲染工作由模板渲染引擎Jinja2 来完成**。

##### templates文件夹

按照默认的设置，Flask 会从程序实例所在模块同级目录的 `templates` 文件夹中寻找模板，所以我们要在项目根目录创建这个文件夹。

```
mkdir templates
```

##### 基本语法

Jinja2 的语法和 Python 大致相同，在模板里需要添加特定的**定界符**将 Jinja2 语句和变量标记出来，下面是三种常用的定界符：

```
{{ ... }} 用来标记变量。
{% ... %} 用来标记语句，比如 if 语句，for 语句等。
{# ... #} 用来写注释。
{{ 变量|过滤器 }} 左侧是变量，右侧是过滤器名，类似于Python的len()函数。
```

在社交网站上，每个人都有一个主页，借助 Jinja2 就可以写出一个通用的模板：

```jinja2
<h1>{{ username }}的个人主页</h1>
{% if bio %}
    <p>{{ bio }}</p>  {# 这里的缩进只是为了可读性，不是必须的 #}
{% else %}
    <p>自我介绍为空。</p>
{% endif %}  {# 大部分 Jinja 语句都需要声明关闭 #}
```

##### 模板渲染

我们先在 `templates` 目录下创建一个 `index.html` 文件，作为主页模板。主页需要显示电影条目列表和个人信息，代码如下所示：

```jinja2
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>{{ name }}'s Watchlist</title>
</head>
<body>
    <h2>{{ name }}'s Watchlist</h2>
    {# 使用 length 过滤器获取 movies 变量的长度 #}
    <p>{{ movies|length }} Titles</p>
    <ul>
        {% for movie in movies %}  {# 迭代 movies 变量 #}
        <li>{{ movie.title }} - {{ movie.year }}</li>  {# 等同于 movie['title'] #}
        {% endfor %}  {# 使用 endfor 标签结束 for 语句 #}
    </ul>
</body>
</html>
```

Flask项目中的代码如下：

```python
from flask import Flask, render_template

app = Flask(__name__)

# 虚拟构造数据
name = 'Grey Li'
movies = [
    {'title': 'My Neighbor Totoro', 'year': '1988'},
    {'title': 'Dead Poets Society', 'year': '1989'},
    {'title': 'A Perfect World', 'year': '1993'},
    {'title': 'Leon', 'year': '1994'},
    {'title': 'Mahjong', 'year': '1996'},
    {'title': 'Swallowtail Butterfly', 'year': '1996'},
    {'title': 'King of Comedy', 'year': '1999'},
    {'title': 'Devils on the Doorstep', 'year': '1999'},
    {'title': 'WALL-E', 'year': '2008'},
    {'title': 'The Pork of Music', 'year': '2012'},
]

@app.route('/')
def index():
    # 使用 render_template() 函数可以把模板渲染出来，传入的参数为模板文件名（相对于 templates 根目录的文件路径），这里即index.html。
    # 为了正确渲染模板，还要把模板内部使用的变量通过关键字参数传入这个函数，左边的movies是模板中使用的变量名称，右边的movies则是该变量指向的实际对象。
    return render_template('index.html', name=name, movies=movies)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000, debug=True)
```

?> 这里传入模板的 `name` 是字符串，`movies` 是列表，但能够在模板里使用的不只这两种 Python 数据结构，你也可以传入元组、字典、函数等。

`render_template()` 函数在调用时会识别并执行 `index.html` 里所有的 Jinja2 语句，返回渲染好的模板内容。在返回的页面中，变量会被替换为实际的值（包括定界符），语句（及定界符）则会在执行后被移除（注释也会一并移除）。

现在访问 http://localhost:5000/ 看到的程序主页如下图所示：

![QQ截图20210621174822](image/QQ截图20210621174822.png)

### 静态文件

**静态文件（static files）和我们的模板概念相反，指的是内容不需要动态生成的文件。**比如图片、CSS 文件和 JavaScript 脚本等。

##### static文件夹

在 Flask 中，我们需要创建一个 `static` 文件夹来保存静态文件，它应该和程序模块、`templates` 文件夹在同一目录层级，所以我们在项目根目录创建它：

```
mkdir static
```

##### 引入静态文件

**在 HTML 文件里，引入这些静态文件需要给出资源所在的 URL。**为了更加灵活，这些文件的 URL 可以通过 Flask 提供的 `url_for()` 函数来生成。**对于静态文件，需要传入的端点值是 `static`，同时使用 `filename` 参数来传入相对于 static 文件夹的文件路径。**

花括号部分的调用会返回 `/static/foo.jpg`：

```jinja2
<img src="{{ url_for('static', filename='foo.jpg') }}">
```

**在实际的项目开发中会涉及到许多静态文件引用，为了更好的组织同类文件，需要在 `static` 目录下面创建子文件夹**，图片文件都可以放在 `image` 子文件夹当中，同样的，如果你有多个 CSS 文件，也可以创建一个 `css` 文件夹来组织他们。

花括号部分的调用会返回 `/static/image/foo.jpg`：

```jinja2
<img src="{{ url_for('static', filename='image/foo.jpg') }}">

##### 添加image图片

**Favicon（favourite icon） 是显示在标签页和书签栏的网站头像。**你需要准备一个 ICO、PNG 或 GIF 格式的图片，大小一般为 16×16、32×32、48×48 或 64×64 像素。把这个图片放到 `static` 目录下 `image` 子文件夹当中，在 HTML 模板里引入它：

```jinja2
<head>
    ...
    <link rel="icon" href="{{ url_for('static', filename='image/favicon.ico') }}">
</head>
```

保存后刷新页面，即可在浏览器标签页上看到这个图片：

![QQ截图20210622141102](image/QQ截图20210622141102.png)

我们再来给标题前面添加一张图片：

```jinja2
<h2>
    <img alt="Avatar" src="{{ url_for('static', filename='images/avatar.png') }}">
    {{ name }}'s Watchlist
</h2>
```

![QQ截图20210622143231](image/QQ截图20210622143231.png)

##### 添加CSS样式

虽然添加了图片，但页面还是非常简陋，在 `static` 目录下 `css` 子文件夹中创建一个 `style.css` 文件，内容如下：

```css
/* 页面整体 */
body {
    margin: auto;
    max-width: 580px;
    font-size: 14px;
    font-family: Helvetica, Arial, sans-serif;
}

/* 头像 */
.avatar {
    width: 40px;
}

/* 电影列表 */
.movie-list {
    list-style-type: none;
    padding: 0;
    margin-bottom: 10px;
    box-shadow: 0 2px 5px 0 rgba(0, 0, 0, 0.16), 0 2px 10px 0 rgba(0, 0, 0, 0.12);
}

.movie-list li {
    padding: 12px 24px;
    border-bottom: 1px solid #ddd;
}

.movie-list li:last-child {
    border-bottom:none;
}

.movie-list li:hover {
    background-color: #f8f9fa;
}
```

接着在页面的 `<head>` 标签内引入这个 CSS 文件：

```jinja2
<head>
    ...
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}" type="text/css">
</head>
```

最后要为对应的元素设置 `class` 属性值，以便和对应的 CSS 定义关联起来：

```html
<h2>
    <img alt="Avatar" class="avatar" src="{{ url_for('static', filename='images/avatar.png') }}">
    {{ name }}'s Watchlist
</h2>
...
<ul class="movie-list">
    ...
</ul>
```

最终的页面如下图所示：

![QQ截图20210622144717](image/QQ截图20210622144717.png)

