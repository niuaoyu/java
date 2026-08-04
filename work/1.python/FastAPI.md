
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

