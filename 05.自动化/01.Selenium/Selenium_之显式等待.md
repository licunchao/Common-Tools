# Selenium 之显式等待

> **版本说明**：本文基于 **Selenium 4.x+** (2026年最新标准)。
> **核心模块**：`selenium.webdriver.support.ui.WebDriverWait` 和 `selenium.webdriver.support.expected_conditions` (简称 `EC`)。

## 1. 显式等待的核心架构

显式等待不是“死等”，而是一个**轮询机制 (Polling Mechanism)**。

1. **初始化**：创建一个 `WebDriverWait` 对象，设定最大超时时间（如 10 秒）和轮询频率（默认 0.5 秒）。

2. **执行**：调用 `.until()` 方法，传入一个预期条件（EC）。

3. 轮询：

   - WebDriver 立即执行一次条件检查。
   - 如果条件为 `True` → **立即返回结果**，继续执行后续代码。
   - 如果条件为 `False` → 捕获异常，**休眠**一个轮询周期（如 0.5 秒），然后重试。
   
4. 终止：

   - 成功：条件满足，跳出循环。
- 失败：总耗时超过最大超时时间 → 抛出 `TimeoutException`。

------

## 2. WebDriverWait 类详解

**初始化参数**

```python
from selenium.webdriver.support.ui import WebDriverWait

wait = WebDriverWait(
    driver, 
    timeout=10,           # 最大等待时间 (秒)
    poll_frequency=0.5,   # 轮询间隔 (秒)，默认 0.5
    ignored_exceptions=None # 忽略的异常列表，默认忽略 NoSuchElementException)
```

- **`timeout`**: 必填。超过这个时间还没满足条件就报错。
- **`poll_frequency`**: 选填。两次检查之间的睡眠时间。设得太短会增加 CPU 负载，设得太长会降低响应速度。
- **`ignored_exceptions`**: 选填。在轮询过程中，如果抛出这些异常，会被视为“条件未满足”而继续等待，而不是直接报错。默认已经忽略了 `NoSuchElementException`。

**核心方法**

| 方法                                 | 描述                                           | 返回值                                                 |
| ------------------------------------ | ---------------------------------------------- | ------------------------------------------------------ |
| **`.until(method, message='')`**     | 等待直到条件为 `True`。                        | 返回条件函数的返回值（通常是 WebElement 或 Boolean）。 |
| **`.until_not(method, message='')`** | 等待直到条件为 `False` (即元素消失/状态改变)。 | 返回 `True` 或抛出异常。                               |

------

## 3. Expected Conditions (EC) 全方法字典

这是显式等待的灵魂。所有 EC 方法都位于 `selenium.webdriver.support.expected_conditions` 模块中。

### 3.1 元素状态类 (最常用) ⭐⭐⭐⭐⭐

用于等待页面元素的出现、可见或可交互。

| 方法名                                                    | 含义                   | 成功条件                                    | 返回值             | 典型场景                         |
| --------------------------------------------------------- | ---------------------- | ------------------------------------------- | ------------------ | -------------------------------- |
| **`presence_of_element_located(locator)`**                | `元素存在于 DOM`       | `元素在 HTML 源码中存在`                    | `WebElement`       | `元素刚插入 DOM，但可能不可见`   |
| **`visibility_of_element_located(locator)`**              | `元素可见`             | `元素存在 + `height/width > 0` + 非 hidden` | `WebElement`       | `需要读取文本、截图、确认显示`   |
| **`element_to_be_clickable(locator)`**                    | `元素可点击`           | `元素可见 + `enabled=True`                  | `WebElement`       | **`执行 .click() 前的标准动作`** |
| **`invisibility_of_element_located(locator)`**            | `元素不可见`           | `元素不存在 **或** 存在但不可见`            | `Boolean`          | `等待 Loading 遮罩层消失`        |
| **`presence_of_all_elements_located(locator)`**           | `所有元素存在`         | `列表中所有元素都在 DOM 中`                 | `List[WebElement]` | `等待列表数据加载完成`           |
| **`visibility_of_all_elements_located(locator)`**         | `所有元素可见`         | `列表中所有元素都可见`                      | `List[WebElement]` | `等待整个表格渲染完毕`           |
| **`element_to_be_selected(element)`**                     | `元素被选中`           | `单选框/复选框处于 `selected` 状态`         | `Boolean`          | `验证 Checkbox 是否勾选`         |
| **`element_located_to_be_selected(locator)`**             | `定位到的元素被选中`   | `同上，但使用 locator 定位`                 | `Boolean`          | `同上`                           |
| **`element_selection_state_to_be(element, is_selected)`** | `元素选中状态符合预期` | `当前选中状态 == `is_selected`              | `Boolean`          | `确保取消勾选或确保勾选`         |

> **💡 Locator 格式**：所有需要 `locator` 的地方，必须传入元组 `(By.ID, "value")`。

------

### 3.2 文本与属性类 ⭐⭐⭐⭐

用于等待动态内容加载或属性变化。

| 方法名                                                   | 含义                 | 成功条件                                                     | 典型场景                                     |
| -------------------------------------------------------- | -------------------- | ------------------------------------------------------------ | -------------------------------------------- |
| **`text_to_be_present_in_element(locator, text)`**       | 元素包含指定文本     | 元素可见 **且** `element.text` 包含 `text`                   | 等待异步加载的提示信息、价格、用户名         |
| **`text_to_be_present_in_element_value(locator, text)`** | Input 值包含指定文本 | 元素存在 **且** `element.get_attribute("value")` 包含 `text` | 等待输入框自动填充的内容                     |
| **`attribute_contains(element, attribute, value)`**      | 属性包含某值         | 元素的指定属性值包含 `value`                                 | 等待 class 变化 (如 `active`), 等待 src 加载 |
| **`attribute_to_be_equal(element, attribute, value)`**   | 属性等于某值         | 元素的指定属性值完全等于 `value`                             | 验证 `disabled` 属性是否移除                 |
| **`style_to_be(element, style_property, value)`**        | 样式属性符合预期     | 元素的 CSS 样式值匹配                                        | 验证颜色变化、位移动画结束                   |

------

### 3.3 弹窗与警告类 ⭐⭐⭐

处理 JavaScript 原生的 `alert`, `confirm`, `prompt`。

| 方法名                    | 含义     | 成功条件                 | 返回值                                          |
| ------------------------- | -------- | ------------------------ | ----------------------------------------------- |
| **`alert_is_present()`**  | 弹窗出现 | 浏览器中存在 Alert       | `Alert` 对象 (可调用 `.accept()`, `.dismiss()`) |
| **`no_alerts_present()`** | 无弹窗   | 浏览器中不存在任何 Alert | `Boolean`                                       |

------

### 3.4 框架与窗口类 ⭐⭐⭐

处理 iframe 切换和多窗口管理。

| 方法名                                                | 含义             | 成功条件                    | 返回值/行为                                           |
| ----------------------------------------------------- | ---------------- | --------------------------- | ----------------------------------------------------- |
| **`frame_to_be_available_and_switch_to_it(locator)`** | Frame 可用并切换 | iframe 存在且可切换         | **自动执行** `driver.switch_to.frame()`，无需手动切换 |
| **`new_window_is_opened(window_handles)`**            | 新窗口已打开     | 当前窗口句柄数量 > 初始数量 | `Boolean`                                             |
| **`number_of_windows_to_be(num)`**                    | 窗口数量达到预期 | 当前打开的窗口数 == `num`   | `Boolean`                                             |

------

### 3.5 高级自定义类 ⭐⭐

当内置条件不够用时，可以使用逻辑组合或自定义。

| 方法名                                    | 含义                       | 用法示例                                               |
| ----------------------------------------- | -------------------------- | ------------------------------------------------------ |
| **`and_(condition_a, condition_b, ...)`** | 所有条件同时满足           | `EC.and_(EC.visibility_of(...), EC.text_to_be(...))`   |
| **`or_(condition_a, condition_b, ...)`**  | 任一条件满足               | `EC.or_(EC.alert_is_present(), EC.visibility_of(...))` |
| **`not_(condition)`**                     | 条件不满足                 | `EC.not_(EC.element_to_be_clickable(...))`             |
| **`any_of(condition_a, ...)`**            | (Selenium 4+) 任意一个满足 | 类似 `or_`                                             |

------

## 4. 实战场景代码库

### 场景 A：标准点击流程 (防遮挡、防禁用)

```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(driver, 10)

# 1. 等待元素可点击 (自动检查可见性和 enabled 状态)
button = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, ".submit-btn")))

# 2. 滚动到视图中心 (防止因位置导致的点击失败)
driver.execute_script("arguments[0].scrollIntoView(true);", button)

# 3. 点击
button.click()
```

### 场景 B：等待动态文本加载 (AJAX)

```python
# 等待 ID 为 'price' 的元素中出现 "$99.00"
wait.until(EC.text_to_be_present_in_element((By.ID, "price"), "$99.00"))

# 获取元素并打印
price_elem = driver.find_element(By.ID, "price")
print(f"当前价格: {price_elem.text}")
```

### 场景 C：等待 Loading 遮罩层消失

很多网站加载时会显示一个 `div` 遮罩，必须等它消失才能操作。

```python
# 等待 class 为 'loading-mask' 的元素不可见 (消失或隐藏)
wait.until(EC.invisibility_of_element_located((By.CLASS_NAME, "loading-mask")))

# 此时可以安全操作底层元素了
driver.find_element(By.ID, "content").click()
```

### 场景 D：处理 iframe (自动切换)

```python
# 等待 iframe 加载完成，并自动 switch 进去
wait.until(EC.frame_to_be_available_and_switch_to_it((By.ID, "payment-frame")))

# 现在可以直接操作 iframe 内部的元素
pay_btn = wait.until(EC.element_to_be_clickable((By.ID, "pay-now")))
pay_btn.click()

# 重要：操作完后切回主文档
driver.switch_to.default_content()
```

### 场景 E：等待新窗口打开并切换

```python
# 记录当前窗口句柄
original_window = driver.current_window_handle

# 触发打开新窗口的操作
driver.find_element(By.ID, "open-new").click()

# 等待窗口数量变为 2
wait.until(lambda d: len(d.window_handles) == 2)

# 切换到新窗口
for handle in driver.window_handles:
    if handle != original_window:
        driver.switch_to.window(handle)
        break

# 在新窗口中等待内容加载
wait.until(EC.presence_of_element_located((By.TAG_NAME, "body")))
```

------

## 5. 高级技巧：反向等待与自定义条件

### 5.1 使用 `.until_not()` (反向等待)

当你需要等待某个元素**消失**，或者等待某个状态**不再存在**时。

```python
# 等待元素彻底从 DOM 中移除 (不仅仅是隐藏)
wait.until_not(EC.presence_of_element_located((By.ID, "temp-message")))

# 等待元素不再被禁用
wait.until_not(EC.element_to_be_selected((By.ID, "processing-checkbox")))
```

### 5.2 编写自定义 Lambda 条件

如果内置 EC 无法满足需求（例如：等待元素宽度大于 100px，或等待列表长度大于 5），可以使用 Lambda 表达式。

**注意**：Lambda 函数必须接收 `driver` 作为参数。

```python
# 场景：等待列表项数量至少为 5 个
wait.until(lambda d: len(d.find_elements(By.CSS_SELECTOR, ".list-item")) >= 5)

# 场景：等待元素宽度大于 100px (用于等待动画展开)
wait.until(lambda d: int(d.find_element(By.ID, "panel").size['width']) > 100)

# 场景：等待 URL 包含特定字符串
wait.until(lambda d: "success" in d.current_url)
```

------

## 6. 常见报错与调试

### ❌ `TimeoutException`

**含义**：在指定时间内，条件从未满足。
**原因**：

1. 定位器写错了（XPath/CSS 错误）。
2. 元素在 iframe 里，没切换上下文。
3. 条件太苛刻（例如用 `clickable` 但元素其实被透明层遮挡）。
4. 页面真的加载太慢，`timeout` 设短了。

**调试技巧**：

```python
try:
    wait.until(EC.element_to_be_clickable((By.ID, "btn")))
except TimeoutException:
    print("超时了！保存截图...")
    driver.save_screenshot("debug_timeout.png")
    # 打印当前页面源码辅助排查
    # print(driver.page_source) 
    raise
```

### ❌ `TypeError: 'module' object is not callable`

**原因**：导入方式错误。
**错误写法**：

```python
from selenium.webdriver.support import expected_conditions
# 下面这样会报错，因为 expected_conditions 是模块，不是函数
element = wait.until(expected_conditions.element_to_be_clickable(...)) 
```

**正确写法**：

```python
from selenium.webdriver.support import expected_conditions as EC
element = wait.until(EC.element_to_be_clickable(...))
```

### ❌ `Message: expected string or bytes-like object`

**原因**：通常在 `text_to_be_present_in_element` 中，第二个参数传了非字符串类型，或者元素文本获取失败。确保传入的是 `str`。

------

## 📝 总结速查表

| 我想做什么？         | 推荐 EC 方法                                                 |
| -------------------- | ------------------------------------------------------------ |
| **我要点击它**       | `EC.element_to_be_clickable`                                 |
| **我要看它的文字**   | `EC.visibility_of_element_located` 或 `EC.text_to_be_present_in_element` |
| **我要等它消失**     | `EC.invisibility_of_element_located` 或 `wait.until_not(...)` |
| **它在 iframe 里**   | `EC.frame_to_be_available_and_switch_to_it`                  |
| **我要等弹窗**       | `EC.alert_is_present`                                        |
| **我要等列表加载完** | `EC.visibility_of_all_elements_located`                      |
| **内置方法不够用**   | `lambda driver: ...` (自定义)                                |

掌握这些方法，你将能解决 99% 的 Selenium 同步问题，让自动化脚本像真人一样“聪明”地等待！