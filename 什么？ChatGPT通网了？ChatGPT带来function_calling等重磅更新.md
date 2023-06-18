## 前言


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b50a143faf2f4b7aa8de47fad4e0fa99~tplv-k3u1fbpfcp-watermark.image?)
就在前几天，OpenAI发布了6月重磅更新：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/964ecfe504924ab68ffc9d7cecc5f285~tplv-k3u1fbpfcp-watermark.image?)
总结来说就是：

*   增加了函数调用功能
*   增加了对话上下文长度，增加了16k版本的`gpt-3.5-turbo`模型（之前是4k，做过连续对话的都知道，4k根本不够用啊！！）
*   降低了API的调用价格（`gpt-3.5-turbo`普通版本输入token价格直接85折大甩卖）

这每一条都相当炸裂呀，真是又便宜又好用，简直越用越爽

当然最爽最炸裂的要数函数调用了，可操作性直接拉满。就这么说吧，之前的GPT如果是一匹千里马，现在更新之后就是一匹插上天使翅膀的千里马。

## 爽在哪？

大家用过ChatGPT的都知道，GPT这种通用大语言模型，都是基于巨大知识库训练而来，这种方式也就意味着时效性不好，最新的GPT4模型，知识库版本也是2021年9月之前的，你想让它告诉你最新的一些信息，那是不阔能滴

比如我们想知道实时天气

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/61dec6a43ff24e179b18f961f53f21d6~tplv-k3u1fbpfcp-watermark.image?)
又比如我们想知道实时新闻

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a0d4c68f26845298a19717535d1a6d8~tplv-k3u1fbpfcp-watermark.image?)
这些ChatGPT都会告诉你它搞不定

**巴特！有了函数功能，我们的GPT就可以变成这样↓↓↓↓**

![天气.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c7f4e05a82af4eb887b718877cab9e44~tplv-k3u1fbpfcp-watermark.image?) 

这样 

![热搜.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/611c6901616041f680a7a0939df48c2e~tplv-k3u1fbpfcp-watermark.image?)

还有这样 

![新闻.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a86c696d21274bbebea94d06cc49f186~tplv-k3u1fbpfcp-watermark.image?)

甚至这样 


![仓库.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc73c065ad7d4810a6bdf2a95fe818bd~tplv-k3u1fbpfcp-watermark.image?) 

这样是不是一下就有了很大的想象空间了！想要体验的老铁们可以戳[这里](https://my-ai-project-one.vercel.app/)试一下 

这样一来，我们在使用它强大的自然语言能力交互的同时，又兼具了时效性，我们可以将我们的最新数据交由ChatGPT来生成新的结果，简直泰裤辣


## 如何爽？

### 1.看文档

看到这里，有些好奇的小伙伴就要问了，这东西这么厉害，到底怎么用呢？官方是这样说的：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/633685f687d14c14a59ebe454e41dac0~tplv-k3u1fbpfcp-watermark.image?)
翻译过来就是这样的

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c70cc6ee9664e2d906a342564bdc999~tplv-k3u1fbpfcp-watermark.image?)
我这里也大概给小伙伴们画了一个图，帮助大家理解

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66fde357c12a4746b49c412d1f2d966f~tplv-k3u1fbpfcp-watermark.image?)
到这里，老铁们是否已经理解了函数调用的用法了呢？不理解也没关系，我们接着往下看

### 2.上代码

多说无益，现在我们就按照上面的步骤来实现：

*   **第一步**：根据GPT[函数定义规范](https://platform.openai.com/docs/api-reference/chat/create#chat/create-functions)定义一组我们的应用将会用到的函数，同时实现函数代码。函数可以根据业务需求而定：

```typescript
// 函数定义
const fns = [
    {
      // 函数名称，对应的我们要去实现一个名字叫getHotNews的函数
      name: 'getHotNews',
      // 函数描述，就是大概告诉GPT你这个函数是干什么用的
      description: '获得给定用户所有的github仓库信息',
      // 函数参数，就是我们实现的这个函数需要接收的参数描述，类型是一个json对象，调用的时候GPT会给我们返回一个参数json字符串
      parameters: {
        // 类型
        type: 'object',
        // 参数定义，key是参数名，value是描述，是一个标准的json schema
        properties: {
          // 一个叫user的参数
          user: { type: 'string' },
        },
        // 必填参数
        required: ['user'],
      },
    },
    ...more
  ];
// 实现函数，这里是调用微博热搜接口实时热搜数据
export async function getHotNews() {
  const {
    data: {
      data: { realtime },
    },
  } = await axios.get(`https://weibo.com/ajax/side/hotSearch`);
  return realtime.map((item) => ({
    text: item.word,
    rank: item.raw_hot,
  }));
}
```

*   **第二步**：将用户问题以及函数定义一起发送给模型：

```typescript
// 接口服务函数，接收用户问题msg作为参数，比如这里接收的问题是：今天有什么热点
public async chat_with_fns(msg: string) {
  // 组装用户消息
  const messages: ChatCompletionRequestMessage[] = [
      { role: 'user', content: msg },
    ];
  const {
        data: { choices },
      } = await openai.createChatCompletion({
        model: 'gpt-3.5-turbo-0613',
        messages,
    		// 这里就是我们第一步定义的函数描述fns 
        functions: fns,
        function_call: 'auto',
      });
}
```

*   **第三步**：调用GPT给我们返回的函数(如果命中了我们的某个函数，则可以从choices中拿到函数定义)，并将调用结果再次发送给GPT模型：

```typescript
// 接口服务函数，接收用户问题msg作为参数
public async chat_with_fns(msg: string) {
	...
  const message = choices[0].message;
  if (message.function_call && functions[message.function_call.name]) {
    // GPT会给我们返回一个function_call参数，包含函数的名称和参数json字符串 
    const { name, arguments: argStr } = message.function_call;
    // 将参数值解析出来
    const args = Object.values(JSON.parse(argStr));
    // 调用函数
    const fnCallRes = await functions[name](...args);
    // 奖函数调用结果转成字符串，以function的角色发给GPT
    const {
      data: { choices },
    } = await openai.createChatCompletion({
      // 这里要注意，只有0613模型支持函数调用
      model: 'gpt-3.5-turbo-0613',
      messages: [
        // 用户消息
        { role: 'user', content: msg },
        // 函数调用结果，role是function，名称是函数名称，content则是我们函数调用的结果字符串
        { role: 'function', name, content: JSON.stringify(fnCallRes) },
      ],
    });    
  }
}
```

*   **第四步**：将结果返回给用户

```typescript
// 接口服务函数，接收用户问题msg作为参数
public async chat_with_fns(msg: string) {
	...
  if (message.function_call && functions[message.function_call.name]) {
    ...
  	return choices[0].message.content;
  }
	// 用户问题没有匹配上我们的函数则将第一次调用GPT的结果返回给用户 
	return message.content
}
```

到这里，用户的问题如果命中了我们的热搜函数，那么他就可以获取到实时热搜数据了

![热搜.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0256e8ce94ff45b39275fde28c0b7ac7~tplv-k3u1fbpfcp-watermark.image?)

## 总结

相较于之前版本的GPT，0613版模型通过函数能力的加持，可以帮助我们更加灵活的获取数据，也给我们带来了更大的想象空间。我们可以利用这个功能，来构建更具有时效性的AI应用。

此外，我们的应用边界也将变得更加容易确定，比如没有命中函数的问题，我们则判定为业务无关问题，从而防止GPT侃侃而谈
