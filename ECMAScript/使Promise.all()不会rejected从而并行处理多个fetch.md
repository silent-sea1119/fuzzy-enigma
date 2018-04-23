
## 问题
在 JavaScript 中，我们可以通过 `Promise.all()` 从而并行执行多个代码片段。然而由于 `Promise.all()` 的自身特性，如果其中某一个 Promise 一旦 rejected，那么 `Promise.all()` 也会理解 rejected 。

``` javascript
const p1 = new Promise((resolve, reject) => resolve('first'));

const p2 = new Promise((resolve, reject) => reject('second was rejected'));

const p3 = new Promise((resolve, reject) => resolve('third'));

Promise.all([p1, p2, p3])
.then(reslove => console.log('Promise.all resolved', reslove))
.catch(error => console.error('Promise.all rejected', error));

// Promise.all rejected second was rejected
```
上面所示代码中，尽管 `p1` 和 `p3` resolved，但是由于 `p2` rejected，所以 `Promise.all()` 结果也是 rejected。这符合 `Promise.all()` 的正常行为预期，但问题在于有时侯，我们需要 `Promise.all()` 中那些  resolved 的 Promise 的值，即使其他 Promise rejected。


## 解决方案
解决方案很简单，就是提前处理可能 rejected 的 Promise。将上面代码中的 `p2` 修改如下：

``` javascript
...

p2 = new Promise((resolve, reject) => reject('second was rejected'))
.catch(err => err);

...

// second was rejected
// Promise.all resolved ['first', 'second was rejected', 'third']
```

## 实际运用：并行处理多个fetch
### 第一层，处理多个fetch
解决了这一问题，我们就可以大胆利用 `Promise.all()` 同时并行处理多个 API 请求了。

一则示例代码如下：
``` javascript
const urls = [
  'https://api.github.com/users/alian',
  'https://api.github.com/users/remy',
  'https://no-such-url',
];

Promise.all(urls.map(url => fetch(url).catch(err => err)))
.then(responses => {
  console.log(responses[0].status); // 200
  console.log(responses[1].status); // 200
  console.log(responses[2]); // TypeError: Failed to fetch
}
```

很好，目前为止，我们成功地使用 `Promise.all()` 并行处理多个 `fetch` ，每个 `fetch` 的 Response 都能够返回，即使某些 `fetch` 出错。


### 第二层，处理多个fetch返回的多个Response
接下来，我们要继续处理 Response Promise的问题，因为 `fetch` 第一次 resolved 之后返回的 Promise 仅仅决议完成的 `Response` 对象，我们还需要继续处理 `Response` 才能获取HTTP响应体中的数据。

这里，我们假定所有HTTP响应返回的数据都是标准的 JSON 对象。

示例代码如下：
``` javascript
const urls = [
  'https://api.github.com/users/alian',
  'https://api.github.com/users/remy',
  'https://no-such-url',
];

Promise.all(urls.map(url => fetch(url).catch(err => err)))
  .then(responses => Promise.all(responses.map(res => 
    res instanceof Error ? res : res.json().catch(err => err))
  ))
  .then(results => {
    console.log(results[0].name); // lyndon
    console.log(results[1].name); // Remy Sharp
    console.log(results[2]); // TypeError: Failed to fetch
  });
```
由以上代码可以看出，我们只需按照之前的思想同样地处理 `fetch` 返回的 `Response` 即可。唯一值得注意的是，由于解析 `Response` 中的 JSON 数据需要使用 `response.json()`，然而考虑到 rejected `Response`，所以在此之前需要先判断一下 `Response` 是否 rejected。

至此，我们成功地结合 `Promise.all()` 和 `fetch` 实现了完整地并行处理多个API的梦想。


### 引入async/await
上面的方案从理论上来说是无可挑剔的，“完美！”，唯一的遗憾就是在写法上有些繁琐，下面我们尝试使用 `async function` 结合 `try/catch` 来重写之前的示例代码.

``` javascript
const parallelRequest = async () => {
  const result = await Promise.all(  
    urls.map(async url => {
	  try {
	    const response = await fetch(url);
	    const responseJson = await response.json();
	      
	    if (!response.ok) {
	      throw new Error(`${response.statusText}`);
	    } 
	    return responseJson;  
	  } catch (error) {  
	    return error;
	  }
	})
  );
};
```

