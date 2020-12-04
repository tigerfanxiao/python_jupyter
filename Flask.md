# Preparation

## Install Flask

```
pip --version
pip3 --version
pip3 install flask
pip3 list
```

```shell
pip3 freeze > requirements.txt
pip3 install -r requirements.txt

```



## Create virtual env

On windows

```powershell
py -3 -m venv venv # create virtual environment in folder venv
.\venv\Scripts\activate # activate the virtual environment
# or if you use power shell
.\venv\Script\Activate.ps1

```

On linux

```shell
# you may need to install python-env
sudo apt-get install python3-env
sudo apt-get install tree # to view the file structure

# create env
python3 -m venv venv
tree venv -L 3 # view 3 levels file tree
chmod 744 ./ven/bin/active # change permisson to excute
. /venv/bin/activate # activate the environment
```

with anaconda

```javascript
conda create -name my_env python=3.7
```



```python
# create the app python file as server.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return "Hello world"
```

### run flask

On Linux

```shell
export FLASK_APP=hello.py
# make suer you pwd is in project foler
flask run
```

on Windows

```powershell
$env:FLASK_APP = "server.py"
flask run
```

### debug mode

when debug mode is on, no need to stop and restart the server each time you make some changes

```shell
export FLASK_ENV=development
```

### render_template

flask will look for the folder named templates 

```python
@app.route('/')
def hello_world():
    return render_template('index.html') # 
```

### render sitemap

```python
from flask import send_from_directory

@app.route('/robots.txt')
@app.route('/sitemap.xml')
def static_from_root():
    return send_from_directory(app.static_folder, request.path[1:])
```



### static files

flask store the static files in static folder

### favicon

if static has sub folder, we can refer the file like 

```python
url_for('static', filename='subfolder/favicon.ico')
```



```html
<link rel="shortcut icon" href="{{ url_for('static', filename='favicon.ico') }}"
```

### Variable Rules

```python
@app.route('/user/<username>')
def show_user_profile(username):
    return 'User %s' % escape(username)

@app.route('/post/<int:post_id>')
def show_post(post_id):
    return 'Post %d' % post_id

@app.route('/path/<path:subpath>')
def show_subpath(subpath):
    return 'Subpath %s' % escape(subpath)
```

| Data type | comments                                   |      |
| --------- | ------------------------------------------ | ---- |
| string    | (default) accepts any text without a slash |      |
| int       | accepts positive integers                  |      |
| float     | accepts positive floating point values     |      |
| path      | like string but also accepts slashes       |      |
| uuid      | accepts UUID strings                       |      |

### Form

```python
from flask import request
@app.route('/submit_form', methods=['POST', 'GET'])
def submit_form():
    if request.method == 'POST':
        data = request.form.to_dict() # transfer the data to dictionary
        print(data)
        return 'form submitted'
   	else:
        return 'something went wrong, try again'
```

### redirect

```python
from flask import redirect
@app.route('/submit_form', methods=['POST', 'GET'])
def submit_form():
    if request.method == 'POST':
        data = request.form.to_dict() 
        print(data)
        return redirect('/thankyou.html') # redirect to new page
```



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

