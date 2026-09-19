请你帮我使用或修改下面这段 Python 代码。

我的目标是：调用一个 AI 修改接口，但服务器可能过载，偶尔会返回 429、500、502、503、504。
我可以接受最多等待 15 分钟，所以请不要一遇到 429 就立刻失败。

请按以下要求处理：

1. 保留 15 分钟总截止时间。
2. 遇到 429 或 5xx 时，不要高频重试，要使用指数退避：
   - 初始等待 2 秒
   - 每次翻倍
   - 最大等待 60 秒
   - 加一点随机抖动
3. 如果响应头里有 Retry-After，优先按 Retry-After 等待。
4. 请求 timeout 可以设置长一点，例如 120 秒，避免 AI 处理太慢时直接断开。
5. 如果这个 API 支持异步任务，例如返回 task_id / job_id / status，请改成：
   - 先提交任务
   - 每 5~15 秒轮询一次结果
   - 总等待时间不超过 15 分钟
6. 不要暴力请求，不要绕过服务端限流，只做温和重试。
7. 请告诉我需要替换哪些内容，例如：
   - url
   - headers
   - payload
   - API Key
   - 请求参数
8. 最后请给我一份可以直接运行的完整代码。

下面是代码：

import time
import random
import requests

def ai_modify_hardcore(url, payload, headers=None):
    print("开始提交修改任务，最多等 15 分钟 (ง •_•)ง")
    deadline = time.time() + 15 * 60
    delay = 2

    while time.time() < deadline:
        remaining = deadline - time.time()
        if remaining <= 0:
            break

        try:
            resp = requests.post(
                url,
                json=payload,
                headers=headers,
                timeout=(10, min(120, remaining))
            )

            if resp.status_code == 200:
                print("修改成功 ✨")
                return resp.json()

            if resp.status_code in [429, 500, 502, 503, 504]:
                retry_after = resp.headers.get("Retry-After")

                if retry_after:
                    wait = float(retry_after)
                else:
                    wait = delay + random.uniform(1, 5)

                wait = min(wait, 60, remaining)
                print(f"服务器 {resp.status_code}，等 {wait:.1f}s 后重试 (¦3[▓▓]")
                time.sleep(wait)

                delay = min(delay * 2, 60)
                continue

            resp.raise_for_status()

        except requests.exceptions.RequestException as e:
            wait = min(delay + random.uniform(1, 5), 60, remaining)
            print(f"请求异常：{e}，等 {wait:.1f}s 后重试 (¦3[▓▓]")
            time.sleep(wait)

            delay = min(delay * 2, 60)

    raise TimeoutError("15 分钟内没有成功")
请根据我的实际 API 帮我补全并解释怎么用。
