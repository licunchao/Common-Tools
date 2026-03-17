# Selenium 元素操作完全指南

本文整理了在使用 `from selenium.webdriver.common.by import By` 获取到元素对象后，所有常用的操作方法。内容涵盖**输入、点击、信息获取、状态判断、高级交互**及**截图**。

------

## 1、输入与文本操作

主要用于表单交互（输入框、文本域等）。

| 方法                       | 说明                                                     | 代码示例                                                     |
| -------------------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| **`.send_keys(\*values)`** | 模拟键盘输入。支持字符串和特殊键（如回车、Ctrl）。       | `element.send_keys("hello")` `element.send_keys(Keys.ENTER)` |
| **`.clear()`**             | 清空输入框内容。**注意**：仅适用于可编辑的文本元素。     | `element.clear()`                                            |
| **`.submit()`**            | 提交表单。若元素在 `<form>` 内，效果等同于点击提交按钮。 | `element.submit()`                                           |

> **💡 技巧：模拟组合键**
>
> ```python
> from selenium.webdriver.common.keys import Keys
> # 全选 (Ctrl+A)
> element.send_keys(Keys.CONTROL, "a")
> # 删除
> element.send_keys(Keys.DELETE)
> ```

------

## 2、鼠标点击操作

用于触发基础的点击事件。

| 方法            | 说明                                                       | 代码示例           |
| --------------- | ---------------------------------------------------------- | ------------------ |
| **`.click()`**  | **最常用**。模拟鼠标左键单击。适用于按钮、链接、复选框等。 | `element.click()`  |
| **`.submit()`** | 提交当前元素所在的表单。                                   | `element.submit()` |

> **⚠️ 注意**：右键、双击、拖拽、悬停等复杂操作需使用 `ActionChains`（见下文）。

------

## 3、获取元素信息

用于断言验证或数据提取。

| 方法/属性                  | 说明                                                       | 返回值类型     | 代码示例                              |
| -------------------------- | ---------------------------------------------------------- | -------------- | ------------------------------------- |
| **`.text`**                | **属性**。获取元素及其子元素的**可见文本**。               | `str`          | `print(element.text)`                 |
| **`.get_attribute(name)`** | 获取 HTML 属性的值（如 `href`, `value`, `class`, `src`）。 | `str` / `None` | `url = element.get_attribute("href")` |
| **`.tag_name`**            | **属性**。获取标签名称（如 `"div"`, `"input"`）。          | `str`          | `print(element.tag_name)`             |
| **`.size`**                | **属性**。获取元素尺寸。                                   | `dict`         | `{'width': 100, 'height': 50}`        |
| **`.location`**            | **属性**。获取元素在页面上的坐标。                         | `dict`         | `{'x': 10, 'y': 20}`                  |

> **💡 常见坑点**：
>
> - 获取 `<input>` 标签里的值，**不能**用 `.text`，必须用 `.get_attribute("value")`。
> - 获取链接地址，必须用 `.get_attribute("href")`。

------

## 4、状态判断

返回布尔值 (`True`/`False`)，常用于 `if` 判断或 `assert` 断言。

| 方法                  | 说明                                                         | 代码示例                        |
| --------------------- | ------------------------------------------------------------ | ------------------------------- |
| **`.is_displayed()`** | 元素是否**可见**（未被 `display: none` 隐藏，且宽高有效）。  | `assert element.is_displayed()` |
| **`.is_enabled()`**   | 元素是否**可用**（未被 `disabled` 属性禁用）。               | `if not btn.is_enabled(): ...`  |
| **`.is_selected()`**  | 元素是否被**选中**。专用于单选框 (Radio) 和复选框 (Checkbox)。 | `assert checkbox.is_selected()` |

------

## 5、高级交互 (ActionChains)

处理右键、双击、拖拽、鼠标悬停等复杂场景。

**初始化：**

```python
from selenium.webdriver.common.action_chains import ActionChains
actions = ActionChains(driver)
```

| 操作类型     | 方法链                                  | 说明                                 |
| ------------ | --------------------------------------- | ------------------------------------ |
| **右键点击** | `actions.context_click(element)`        | 模拟鼠标右键                         |
| **双击**     | `actions.double_click(element)`         | 模拟鼠标左键双击                     |
| **鼠标悬停** | `actions.move_to_element(element)`      | 鼠标移动到元素上方（触发 hover）     |
| **拖拽**     | `actions.drag_and_drop(source, target)` | 将 source 拖到 target                |
| **长按**     | `actions.click_and_hold(element)`       | 按住鼠标左键不松手                   |
| **释放**     | `actions.release()`                     | 松开鼠标左键                         |
| **执行动作** | **`.perform()`**                        | **必须调用**！构建完所有动作后执行。 |

**示例：鼠标悬停菜单并点击子项**

```python
menu = driver.find_element(By.ID, "main-menu")
sub_item = driver.find_element(By.ID, "sub-item")

ActionChains(driver)\
    .move_to_element(menu)\
    .move_to_element(sub_item)\
    .click()\
    .perform()
```

------

## 6、元素截图

Selenium 4+ 支持对单个元素进行截图。

| 方法                     | 说明                                       | 代码示例                           |
| ------------------------ | ------------------------------------------ | ---------------------------------- |
| **`.screenshot(path)`**  | 截取元素区域并保存为文件。                 | `element.screenshot("error.png")`  |
| **`.screenshot_as_png`** | 获取图片二进制数据（用于内存处理或上传）。 | `data = element.screenshot_as_png` |

------

## 7、综合实战代码示例

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.action_chains import ActionChains

# 假设 driver 已初始化
# driver = webdriver.Chrome()

# 1. 定位元素
username_input = driver.find_element(By.ID, "username")
password_input = driver.find_element(By.NAME, "pwd")
login_btn = driver.find_element(By.CSS_SELECTOR, ".btn-login")
remember_cb = driver.find_element(By.ID, "remember")

# 2. 输入操作
username_input.clear()
username_input.send_keys("admin_user")
password_input.send_keys("secure_password", Keys.ENTER) # 输入后直接回车

# 3. 状态判断与选择
if not remember_cb.is_selected():
    remember_cb.click()  # 勾选“记住我”

# 4. 获取信息用于断言
welcome_msg = driver.find_element(By.CLASS_NAME, "welcome-text")
actual_text = welcome_msg.text
print(f"欢迎语: {actual_text}")

# 断言验证
assert "admin_user" in actual_text, "登录失败，欢迎语不包含用户名"

# 5. 获取属性
user_role = welcome_msg.get_attribute("data-role")
print(f"用户角色权限: {user_role}")

# 6. 高级操作示例 (右键点击某个特殊图标)
# icon = driver.find_element(By.ID, "special-icon")
# ActionChains(driver).context_click(icon).perform()

# 7. 元素截图
welcome_msg.screenshot("welcome_check.png")
```

------

## 8、常见问题与解决方案

### ***1. `ElementNotInteractableException`***

- **现象**：代码报错，提示元素不可交互。
- **原因**：元素虽然在 DOM 中，但被其他层（如加载遮罩、弹窗）遮挡，或者元素尚未渲染完成（不可见）。
- 解决：
  - 使用 `WebDriverWait` 等待元素变为 `element_to_be_clickable`。
  - 先关闭遮挡的弹窗。
  - 尝试滚动页面使元素进入视口：`driver.execute_script("arguments[0].scrollIntoView();", element)`。

### *** 2. `.send_keys()` 不生效***

- **现象**：代码执行了，但输入框没内容。

- **原因**：某些前端框架（Vue/React/Angular）监听特定的 JS 事件，Selenium 的原生模拟未触发这些事件。

- 解决：

  - 尝试在 `send_keys` 后模拟回车或 Tab 键。

  - 使用 JS 强制赋值：

    ```python
    driver.execute_script("arguments[0].value = '强制输入的值';", element)
    # 有时还需要触发 input 事件
    driver.execute_script("arguments[0].dispatchEvent(new Event('input'));", element)
    ```

### ***3. `.text` 获取为空***

- **现象**：明明页面上有字，`print(element.text)` 却是空的。
- 原因：
  - 文字是在 `value` 属性里（如 `<input value="text">`）。
  - 文字是动态加载的，还没出来。
  - 元素被 CSS 隐藏了。
- 解决：
  - 改用 `.get_attribute("value")` 或 `.get_attribute("innerHTML")`。
  - 添加显式等待，确保元素可见。

------

> **总结**：掌握 `send_keys`, `click`, `text`, `get_attribute`, `is_displayed` 这五个核心方法，即可覆盖 90% 的自动化测试场景；复杂交互则交给 `ActionChains`。