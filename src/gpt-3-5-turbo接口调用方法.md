date: 2023-03-02 15:12:08

tags: gpt python

在3月1日刚发布的gpt-3.5-turbo，学习一下它的调用方法

+ 以post方法调用
```python
import requests

url = 'https://api.openai.com/v1/chat/completions'

Question = input("Question:")

data = {"model": "gpt-3.5-turbo","messages": [{"role": "user", "content": Question}]}

headers = {
    'Authorization': 'Bearer sk-' #后面输入自己的api-key
}

response = requests.post(url, json=data, headers=headers)

if response.status_code == 200:
    x = response.json()
    print(x['choices'][0]['message']['content'])
else:
    print("error:",response.status_code)
```

启动程序之后输入问题然后发送请求。

* 以python openai模块调用
```python
import openai
openai.api_key="sk-"
message=[]
while(True):
    Q = input("user:")
    message.append({"role":"user","content":Q})
    com=openai.ChatCompletion.create(model="gpt-3.5-turbo",messages=message)
    message.append({"role":"assistant","content":com.choices[0].message.content})
    print("assistant:",com.choices[0].message.content)
```
跟上面的代码功能有所不同的是，这个可以联系上下文，类似网页上与chatgpt对话的过程。
用户的角色设置为user，ai的角色设置为assistant。
目前不管是直接post请求还是用python的openai模块，都不需要连接外网，平时使用较为方便。
## 程序大小问题
在实际测试后发现：直接用post的版本，生成exe的大小在6M左右。而使用openai模块的版本，因为要打包openai模块，大小在70M左右。所以还是建议用post的版本
以下程序也可实现联系上下文
```python
import requests

url = 'https://api.openai.com/v1/chat/completions'

headers = {
    'Authorization': 'Bearer sk-'
}

message=[]

while(True):
    Q = input("user:")
    message.append({"role":"user","content":Q})
    data = {"model": "gpt-3.5-turbo","messages": message}
    response = requests.post(url, json=data, headers=headers)
    if response.status_code == 200:
        x = response.json()
        print("assistant:",x['choices'][0]['message']['content'])
        message.append({"role":"assistant","content":x['choices'][0]['message']['content']})
    else:
        print("error:",response.status_code)
        break
```