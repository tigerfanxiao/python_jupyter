### create APP

```python
from flask import Flask
# create app instance
app = Flask(__name__)
# register function as a handler for the root URL
@app.route('/')
def hello():
    # respond the request with string or html page
    return 'hello'

# route with dynamic component
@app.route('/user/<name>')
def user(name):
    return f'<h1>{name}</h1>'

# predefine a dynamic component with type
# flask has types: int, float, path
@app.route('/number/<int:id>')
def number(id):
    sum = 5 + id
    return f'sum is {sum}'
```

### set root

```powershell
$env:FLASK_APP="hello.py"
```

### set debug mode

In debug mode, we don't have to restart the server each time we change the code. We can refresh the page and see the changes.

```powershell
$env:FLASK_ENV="development"
flask run
```

set debug mode each time run the app

```python
if __name__ == '__main__':
    # set debug mode
    app.run(debug=True)

```

## Template

Flask will read template only in the template folder built in root path

### Render template

`index.html`is located in the folder named `templates` in project root path

```python
from flask import render_template

@app.route('/')
def hello():
    return render_template('index.html')

```

### Jinja2

Flask support Jinja2 to parse html file

```html

<link rel="shortcut icon" href="{{ url_for('static', filename='favicon.ico') }}">
```

with 

```python
from flash import url_for
# add link to html file
url_for('static', filename='favicon.ico')
```

# 问题

1. 如果static目录下有多层目录, 这个怎么引用, 已经测试了 url_for('static/img', filename='favicon.ico') 失败, 因为不能使用static/img

   解答: html 中 href = "./static/img/my_picture.png" 这里当前目录猜测是项目目录, 或者是运行app文件的目录

   这是浏览器问题, 我才firebox上测试通过, 在chrome上测试不通过

2. redirect只能转页面, 不能带参数吗?

3. 搞不懂url_for()函数是怎么运作的, 里面到底应该放什么参数, 返回又是什么?

4. 学习和做csv module笔记

5. pip freeze>requirements.txt

6. pip install -r requirement.txt

7. excel自动拖动公式, 选中所有单元格,包括有规律的合并单元格, ctrl + enter, 刷公式

8. excel合并单元格拖动公式技巧, max($A$1, A12) + 1 每个合并单元格递增1

9. 有没有一个识别字体的软件?

10. 

## Static files

Flask will read static files only in the static folder built in root path

```python

```

favicon

A favicon is an icon used by browsers for tabs and bookmarks. normal with extension .ico

MIME type

Browser use MIME type, not file extension to determine how to process a url 

vscode shortcut: ctrl + shift + L , select all matched

