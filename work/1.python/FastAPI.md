
**`time.sleep()` 是“让整个厨房停工”**（阻塞整个线程） ，`time.sleep()` 是 **Python 的 C 语言实现**，它**不会释放 GIL（全局解释器锁）**，也不会让出事件循环的控制权
**`asyncio.sleep()` 是“让这个厨师去喝茶，换个厨师继续炒菜”**（主动让出 CPU，让事件循环切换任务），`asyncio.sleep()` 是 **Python 实现的一个协程**，它执行时会**主动调用 `await` 让出 CPU**，把控制权交还给事件循环

使用request库也是一样
``` python
# ❌ 错误：requests 库是同步阻塞的
async def fetch_url():
    response = requests.get("https://api.com")  # 阻塞整个线程
    return response.text

# ✅ 正确：用异步 HTTP 库
async def fetch_url():
    async with aiohttp.ClientSession() as session:
        async with session.get("https://api.com") as resp:
            return await resp.text()
```





路由匹配是“按顺序试穿”，不是“按精准度抢答”，确实看 `@app.get` 后面的链接。但问题的关键不在于“看哪个”，而在于 **“匹配的顺序”**和 **“匹配的规则”**。

当请求到达 FastAPI 时，它**不是**一眼看出哪个最匹配，而是**按代码的顺序，从上到下挨个试**。

你写的代码：
```python
@app.get("/users/me")          # 第 1 个
async def read_user_me(): ...

@app.get("/users/{user_id}")   # 第 2 个
async def read_user(user_id: str): ...
```

请求 `GET /users/me`，FastAPI 看第一个路由 `/users/me`
- 请求路径是 `/users/me`
- 规则是 `/users/me`
- **完全匹配！** ✅ → 直接执行 `read_user_me`，**不再往下看**。
- **结果**：正确返回当前用户。

如果你把顺序写反了（错误示例 ❌）
```python
@app.get("/users/{user_id}")   # 第 1 个（现在在前）
async def read_user(user_id: str): ...

@app.get("/users/me")          # 第 2 个（现在在后）
async def read_user_me(): ...
```

**请求 `GET /users/me` 的执行过程**：**FastAPI 看第一个路由 `/users/{user_id}`
- 请求路径是 `/users/me`
- 规则是 `/users/{user_id}`（`{user_id}` 是**通配符**，能匹配任何东西）
- **匹配成功！** ✅ → FastAPI 认为 `user_id` = `"me"`
- **立即执行** `read_user`，**不再往下看**，永远轮不到 `/users/me` 这个路由。
- **结果**：你想获取当前用户，结果却拿到了 `{"user_id": "me"}`，逻辑错误！

为什么 `{user_id}` 能匹配 `"me"`？
因为 `{user_id}` **不是写死的字符串**，它是一个**路径参数（Path Parameter）**，代表“任何一段文本”。
### 通用的路由设计原则

| 规则 | 说明 |
|------|------|
| **写死的路径优先** | 所有固定字符串的路由（如 `/users/me`）写在通配路由前面 |
| **通配路径放后面** | 所有带参数的路由（如 `/users/{user_id}`）写在后面 |
| **越具体越靠前** | 跟写 if-else 一样，先判断特殊情况，再处理一般情况 |
