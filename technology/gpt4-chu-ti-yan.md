# GPT-4 初体验

### 前言

笔者目前正在学习编译原理，然而 gpt-3.5 不能对相关问题给出正确的解答，但恰巧好友吴给了笔者一个 gpt-4 的 key，借此机会探索一下 gpt-4 的潜能。

### 前置条件

1. 能访问外网的设备；
2. ChatGPT Plus or key。

为了满足条件 1，可通过搜索引擎检索关键词 `clash for windows` 获取相关的解决方案，并搭配相关的飞机场实现。值得一提的是，如果要实现在 terminal 当中访问外网，有以下两种方法。

*   方法一：修改 `~/.bashrc` 或 `~/.zshrc`文件，在此类 shell 配置文件中添加以下环境变量（注意替换 proxy\_ip 与 port 为读者实际使用的 proxy\_ip 与 port）。

    > export http\_proxy=http://proxy\_ip:port
    >
    > export https\_proxy=https://proxy\_ip:port



    <figure><img src="../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

    * 添加环境变量之后，键入 `source ~/.bashrc` 或 `source ~/.zshrc`（取决于你使用的 shell 是 bash 还是 zsh 或者是其它 shell），刷新配置文件，使刚添加的环境变量生效。可使用  `wget google.com` 来测试是否成功访问外网。
* 方法二：使用 `proxychains` 工具。首先 `sudo apt install proxychains` 而后在 `/etc/proxychains.conf` 根据提示修改 proxy\_ip 和 port。修改完成后，对需访问外网的命令添加前缀 proxychains 即可。

为了满足条件2，请通过 [openai](https://openai.com/gpt-4) 官网订阅 `ChatGPT Plus` 或者购买 `key`。若读者购买了  `ChatGPT Plus` 可止步阅读于此，以下介绍的是关于 **key** 的配置与使用。

### 环境配置

请参考 openai 官网的 [Quickstart](https://platform.openai.com/docs/quickstart?context=python) 完成初始的环境配置。

### 初体验

创建一个 .py 文件（以 call.py 为例），示例代码如下。在 terminal 中键入`python3 call.py` 即可。

```python
# call.py

import os
import openai

openai.api_key = os.getenv("OPENAI_API_KEY") # 在环境变量中配置 OPENAI_API_KEY=key
                                             # 将 key 替换成在 openai 中获取到的实际值

def send_message(message):
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": "You are a helpful assistant."},
            *previous_messages,
            {"role": "user", "content": message},
        ]
    )

    assistant_response = response.choices[0].message["content"]

    previous_messages.extend([
        {"role": "user", "content": message},
        {"role": "assistant", "content": assistant_response},
    ])

    return assistant_response

previous_messages = []

while True:
    print("User: \n---------------")

    user_input = ""
    for line in iter(input, ''): # 为解决多行输入的问题，读者需要在确认输入时
                                 # 多按一次 Enter
        user_input += line + "   "
    
    assistant_response = send_message(user_input)
    assistant_response_lines = assistant_response.split('\n')

    print("---------------\nGPT-4: \n")

    for line in assistant_response_lines:
        print(line)
    
    print("---------------\n\n")
```

由于时间关系，笔者仅仅体验了 gpt-4 在数学推理方面的能力，结果非常的 **amazing**！不仅能够解出一元一次方程、一元二次方程、二元一次方程组，甚至能够求解定积分与不定积分！（ps：sinx/x 的不定积分不能用初等函数表示，但 gpt-4 依旧给出了正确的解！）

<figure><img src="../.gitbook/assets/image (6) (1).png" alt=""><figcaption><p>求三次方程的解</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption><p>复杂的不定积分求解</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption><p>无法用初等函数表示的不定积分求解</p></figcaption></figure>
