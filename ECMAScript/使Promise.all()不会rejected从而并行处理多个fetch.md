---


---

<h2 id="问题">问题</h2>
<p>在 JavaScript 中，我们可以通过 <code>Promise.all()</code> 从而并行执行多个代码片段。然而由于 <code>Promise.all()</code> 的自身特性，如果其中某一个 Promise 一旦 rejected，那么 <code>Promise.all()</code> 也会理解 rejected 。</p>
<pre class=" language-javascript"><code class="prism  language-javascript"><span class="token keyword">const</span> p1 <span class="token operator">=</span> <span class="token keyword">new</span> <span class="token class-name">Promise</span><span class="token punctuation">(</span><span class="token punctuation">(</span>resolve<span class="token punctuation">,</span> reject<span class="token punctuation">)</span> <span class="token operator">=&gt;</span> <span class="token function">resolve</span><span class="token punctuation">(</span><span class="token string">'first'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token keyword">const</span> p2 <span class="token operator">=</span> <span class="token keyword">new</span> <span class="token class-name">Promise</span><span class="token punctuation">(</span><span class="token punctuation">(</span>resolve<span class="token punctuation">,</span> reject<span class="token punctuation">)</span> <span class="token operator">=&gt;</span> <span class="token function">reject</span><span class="token punctuation">(</span><span class="token string">'second was rejected'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token keyword">const</span> p3 <span class="token operator">=</span> <span class="token keyword">new</span> <span class="token class-name">Promise</span><span class="token punctuation">(</span><span class="token punctuation">(</span>resolve<span class="token punctuation">,</span> reject<span class="token punctuation">)</span> <span class="token operator">=&gt;</span> <span class="token function">resolve</span><span class="token punctuation">(</span><span class="token string">'third'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

Promise<span class="token punctuation">.</span><span class="token function">all</span><span class="token punctuation">(</span><span class="token punctuation">[</span>p1<span class="token punctuation">,</span> p2<span class="token punctuation">,</span> p3<span class="token punctuation">]</span><span class="token punctuation">)</span>
<span class="token punctuation">.</span><span class="token function">then</span><span class="token punctuation">(</span>reslove <span class="token operator">=&gt;</span> console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span><span class="token string">'Promise.all resolved'</span><span class="token punctuation">,</span> reslove<span class="token punctuation">)</span><span class="token punctuation">)</span>
<span class="token punctuation">.</span><span class="token keyword">catch</span><span class="token punctuation">(</span>error <span class="token operator">=&gt;</span> console<span class="token punctuation">.</span><span class="token function">error</span><span class="token punctuation">(</span><span class="token string">'Promise.all rejected'</span><span class="token punctuation">,</span> error<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token comment">// Promise.all rejected second was rejected</span>
</code></pre>
<p>上面所示代码中，尽管 <code>p1</code> 和 <code>p3</code> resolved，但是由于 <code>p2</code> rejected，所以 <code>Promise.all()</code> 结果也是 rejected。这符合 <code>Promise.all()</code> 的正常行为预期，但问题在于有时侯，我们需要 <code>Promise.all()</code> 中那些  resolved 的 Promise 的值，即使其他 Promise rejected。</p>
<h2 id="解决方案">解决方案</h2>
<p>解决方案很简单，就是提前处理可能 rejected 的 Promise。将上面代码中的 <code>p2</code> 修改如下：</p>
<pre class=" language-javascript"><code class="prism  language-javascript"><span class="token operator">...</span>

p2 <span class="token operator">=</span> <span class="token keyword">new</span> <span class="token class-name">Promise</span><span class="token punctuation">(</span><span class="token punctuation">(</span>resolve<span class="token punctuation">,</span> reject<span class="token punctuation">)</span> <span class="token operator">=&gt;</span> <span class="token function">reject</span><span class="token punctuation">(</span><span class="token string">'second was rejected'</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
<span class="token punctuation">.</span><span class="token keyword">catch</span><span class="token punctuation">(</span>err <span class="token operator">=&gt;</span> err<span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token operator">...</span>

<span class="token comment">// second was rejected</span>
<span class="token comment">// Promise.all resolved ['first', 'second was rejected', 'third']</span>
</code></pre>
<h2 id="实际运用：并行处理多个fetch">实际运用：并行处理多个fetch</h2>
<h3 id="第一层，处理多个fetch">第一层，处理多个fetch</h3>
<p>解决了这一问题，我们就可以大胆利用 <code>Promise.all()</code> 同时并行处理多个 API 请求了。</p>
<p>一则示例代码如下：</p>
<pre class=" language-javascript"><code class="prism  language-javascript"><span class="token keyword">const</span> urls <span class="token operator">=</span> <span class="token punctuation">[</span>
  <span class="token string">'https://api.github.com/users/alian'</span><span class="token punctuation">,</span>
  <span class="token string">'https://api.github.com/users/remy'</span><span class="token punctuation">,</span>
  <span class="token string">'https://no-such-url'</span><span class="token punctuation">,</span>
<span class="token punctuation">]</span><span class="token punctuation">;</span>

Promise<span class="token punctuation">.</span><span class="token function">all</span><span class="token punctuation">(</span>urls<span class="token punctuation">.</span><span class="token function">map</span><span class="token punctuation">(</span>url <span class="token operator">=&gt;</span> <span class="token function">fetch</span><span class="token punctuation">(</span>url<span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token keyword">catch</span><span class="token punctuation">(</span>err <span class="token operator">=&gt;</span> err<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
<span class="token punctuation">.</span><span class="token function">then</span><span class="token punctuation">(</span>responses <span class="token operator">=&gt;</span> <span class="token punctuation">{</span>
  console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span>responses<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span><span class="token punctuation">.</span>status<span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">// 200</span>
  console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span>responses<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">.</span>status<span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">// 200</span>
  console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span>responses<span class="token punctuation">[</span><span class="token number">2</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">// TypeError: Failed to fetch</span>
<span class="token punctuation">}</span>
</code></pre>
<p>很好，目前为止，我们成功地使用 <code>Promise.all()</code> 并行处理多个 <code>fetch</code> ，每个 <code>fetch</code> 的 Response 都能够返回，即使某些 <code>fetch</code> 出错。</p>
<h3 id="第二层，处理多个fetch返回的多个response">第二层，处理多个fetch返回的多个Response</h3>
<p>接下来，我们要继续处理 Response Promise的问题，因为 <code>fetch</code> 第一次 resolved 之后返回的 Promise 仅仅决议完成的 <code>Response</code> 对象，我们还需要继续处理 <code>Response</code> 才能获取HTTP响应体中的数据。</p>
<p>这里，我们假定所有HTTP响应返回的数据都是标准的 JSON 对象。</p>
<p>示例代码如下：</p>
<pre class=" language-javascript"><code class="prism  language-javascript"><span class="token keyword">const</span> urls <span class="token operator">=</span> <span class="token punctuation">[</span>
  <span class="token string">'https://api.github.com/users/alian'</span><span class="token punctuation">,</span>
  <span class="token string">'https://api.github.com/users/remy'</span><span class="token punctuation">,</span>
  <span class="token string">'https://no-such-url'</span><span class="token punctuation">,</span>
<span class="token punctuation">]</span><span class="token punctuation">;</span>

Promise<span class="token punctuation">.</span><span class="token function">all</span><span class="token punctuation">(</span>urls<span class="token punctuation">.</span><span class="token function">map</span><span class="token punctuation">(</span>url <span class="token operator">=&gt;</span> <span class="token function">fetch</span><span class="token punctuation">(</span>url<span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token keyword">catch</span><span class="token punctuation">(</span>err <span class="token operator">=&gt;</span> err<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
  <span class="token punctuation">.</span><span class="token function">then</span><span class="token punctuation">(</span>responses <span class="token operator">=&gt;</span> Promise<span class="token punctuation">.</span><span class="token function">all</span><span class="token punctuation">(</span>responses<span class="token punctuation">.</span><span class="token function">map</span><span class="token punctuation">(</span>res <span class="token operator">=&gt;</span> 
    res <span class="token keyword">instanceof</span> <span class="token class-name">Error</span> <span class="token operator">?</span> res <span class="token punctuation">:</span> res<span class="token punctuation">.</span><span class="token function">json</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">.</span><span class="token keyword">catch</span><span class="token punctuation">(</span>err <span class="token operator">=&gt;</span> err<span class="token punctuation">)</span><span class="token punctuation">)</span>
  <span class="token punctuation">)</span><span class="token punctuation">)</span>
  <span class="token punctuation">.</span><span class="token function">then</span><span class="token punctuation">(</span>results <span class="token operator">=&gt;</span> <span class="token punctuation">{</span>
    console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span>results<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span><span class="token punctuation">.</span>name<span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">// lyndon</span>
    console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span>results<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">.</span>name<span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">// Remy Sharp</span>
    console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span>results<span class="token punctuation">[</span><span class="token number">2</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">// TypeError: Failed to fetch</span>
  <span class="token punctuation">}</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<p>由以上代码可以看出，我们只需按照之前的思想同样地处理 <code>fetch</code> 返回的 <code>Response</code> 即可。唯一值得注意的是，由于解析 <code>Response</code> 中的 JSON 数据需要使用 <code>response.json()</code>，然而考虑到 rejected <code>Response</code>，所以在此之前需要先判断一下 <code>Response</code> 是否 rejected。</p>
<p>至此，我们成功地结合 <code>Promise.all()</code> 和 <code>fetch</code> 实现了完整地并行处理多个API的梦想。</p>
<h3 id="引入asyncawait">引入async/await</h3>
<p>上面的方案从理论上来说是无可挑剔的，“完美！”，唯一的遗憾就是在写法上有些繁琐，下面我们尝试使用 <code>async function</code> 结合 <code>try/catch</code> 来重写之前的示例代码.</p>
<pre class=" language-javascript"><code class="prism  language-javascript"><span class="token keyword">const</span> parallelRequest <span class="token operator">=</span> <span class="token keyword">async</span> <span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token operator">=&gt;</span> <span class="token punctuation">{</span>
	<span class="token keyword">const</span> result <span class="token operator">=</span> <span class="token keyword">await</span> Promise<span class="token punctuation">.</span><span class="token function">all</span><span class="token punctuation">(</span>  
	  urls<span class="token punctuation">.</span><span class="token function">map</span><span class="token punctuation">(</span><span class="token keyword">async</span> url <span class="token operator">=&gt;</span> <span class="token punctuation">{</span>
	    <span class="token keyword">try</span> <span class="token punctuation">{</span>
	      <span class="token keyword">const</span> response <span class="token operator">=</span> <span class="token keyword">await</span> <span class="token function">fetch</span><span class="token punctuation">(</span>url<span class="token punctuation">)</span><span class="token punctuation">;</span>
	      <span class="token keyword">const</span> responseJson <span class="token operator">=</span> <span class="token keyword">await</span> response<span class="token punctuation">.</span><span class="token function">json</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	      
	      <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token operator">!</span>response<span class="token punctuation">.</span>ok<span class="token punctuation">)</span> <span class="token punctuation">{</span>
	        <span class="token keyword">throw</span> <span class="token keyword">new</span> <span class="token class-name">Error</span><span class="token punctuation">(</span><span class="token template-string"><span class="token string">`</span><span class="token interpolation"><span class="token interpolation-punctuation punctuation">${</span>response<span class="token punctuation">.</span>statusText<span class="token interpolation-punctuation punctuation">}</span></span><span class="token string">`</span></span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	      <span class="token punctuation">}</span> 
	      <span class="token keyword">return</span> responseJson<span class="token punctuation">;</span>  
	   <span class="token punctuation">}</span> <span class="token keyword">catch</span> <span class="token punctuation">(</span><span class="token class-name">error</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>  
	    <span class="token keyword">return</span> error<span class="token punctuation">;</span>
	   <span class="token punctuation">}</span>
	 <span class="token punctuation">}</span><span class="token punctuation">)</span>
	<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre>

