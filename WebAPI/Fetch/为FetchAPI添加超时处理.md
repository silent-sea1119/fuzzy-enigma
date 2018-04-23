


## 问题
浏览器全新的网络请求API —— Fetch，用起来很酷，然而 Fetch API 是为了更好地控制偏底层而生，所以对于一些偏上层的东西是没有什么支持的，例如，请求超时处理。

幸运的是，得益于强大的 Promise，我们可以自己实现一个带超时处理的fetch。



## 解决方案
由于一个 fetch 的结果是一个 Promise，所以要实现超时处理，关键的步骤是我们只需要在超时结束的那个时间点上让 Promise rejected。

示例代码如下：
``` javascript
/**  
 * 为 fetch 添加网络请求超时处理
 * @author lfkid
 * @param {number} [timeout=10000] - 超时时长（单位：ms），默认值为 10000，即10s  
 * @param {Promise} fetchInstance - 要包装的 fetch 请求实例  
 * @returns {Promise<any>}  
 */
const fetchWithTimeoutHandler = (fetchInstance, timeout = 10000) =>
  new Promise((resolve, reject) => {
    const timeoutId = setTimeout(() => reject(new Error('网络请求超时')), timeout);

    fetchInstance.then((res) => {
      clearTimeout(timeoutId);
      resolve(res);
    }, (err) => {
      clearTimeout(timeoutId);
      reject(err);
    }):
  });
```

