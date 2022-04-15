# 网页端语音播报的 Vue3 实现

## 业务需求

1. 需求目标：实现网页端自动语音播报
2. 业务逻辑：从后端获取的数据达到一定条件后，前端自动播放一段语音
3. 业务场景：实时监控中心

## 技术实现

### 实现逻辑

1. 前端轮询后台数据（你也可以跟后端商量使用 websocket 或其他方案，这里只做参考模拟）
2. 监听后台数据的变化，比较差异，判断目标条件
3. 使用`HTML5`中的网页语音`API`，[SpeechSynthesisUtterance](https://developer.mozilla.org/zh-CN/docs/Web/API/SpeechSynthesisUtterance)

### 代码开发

**数据轮询**

```javascript
const timer = ref(null); // 定义定时器
const tableLength = ref(0); // 定义响应式条件变量
function getBackendData() {
  // 轮询执行的请求方法
  const params = {
    number: "",
  };
  // getApi为请求api封装的方法
  getApi(params).then((res: any) => {
    tableLength.value = res; // 拿到后台数据
  });
}
// 注意定时器执行的时间，在onMounted钩子中（进入页面开始执行）
onMounted(() => {
  timer.value = setInterval(async () => {
    await getBackendData();
  }, 5000);
});
// 别忘了退出页面时将定时器销毁
onUnmounted(() => {
  clearInterval(timer.value);
});
```

**数据监听**

```javascript
watch(
  tableLength, // 监听变化的值
  (newValue, oldValue) => {
    // 判断目标条件
    if (newValue > oldValue) {
      voiceAction(); // 执行目标方法
    }
  },
  { immediate: true }
);
```

> 注意 watch 接收的第一个参数是 tableLength，而不是 tableLength.value（[watch 的具体使用](https://staging-cn.vuejs.org/api/reactivity-core.html#watch)）。

**语音播报 API 调用**

```javascript
function voiceAction() {
  const broadcast = new SpeechSynthesisUtterance(); // 初始化
  broadcast.text = "美团外卖已自动为您接单，请注意查收";
  broadcast.lang = "zh"; // 音种
  broadcast.volume = 1; // 音量
  broadcast.rate = 1; // 音速
  broadcast.pitch = 1; // 音调
  speechSynthesis.speak(broadcast); // 添加到语音队列中播放
}
```

这个 API 大家可能很少会用到，甚至从未见过。在我们使用一个陌生的 API 之前，请留意官方的提醒，并注意查看浏览器的支持情况

![image-20220414181713046](https://gitee.com/huohuomua/pictures/raw/master/202204141817067.png)

注：关于这个 API 的 lang 属性，你可以在此定义你想要的语种。通过 SpeechSynthesis.getVoices()你可以得到当前设备所有可用声音的列表

![image-20220414182631794](https://gitee.com/huohuomua/pictures/raw/master/202204141826840.png)
