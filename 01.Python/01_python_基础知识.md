### 1. 关键词命名规范

| ***类型***                        | ***命名风格***               | ***示例***              | ***说明***                   |
| --------------------------------- | ---------------------------- | ----------------------- | ---------------------------- |
| **变量 **`(Variable)`             | snake_case (小写 + 下划线)   | `user_name`,            | 清晰描述内容                 |
| **参数** `(Parameter)`            | snake_case (小写 + 下划线)   | `def calc(price, tax):` | 同变量，避免与关键字冲突     |
| **函数/方法**` (Function/Method)` | nake_case (小写 + 下划线)    | `get_user_info()`       | 动词开头，表示动作           |
| **类名**` (Class)`                | PascalCase (大驼峰)          | `UserService`           | 名词开头，每个单词首字母大写 |
| **常量**` (Constant)`             | UPPER_CASE (全大写 + 下划线) | `API_KEY`               | 全局不变的值                 |
| **私有成员**` (Private)`          | _single_leading_underscore   | `_helper_method`        | 暗示“内部使用”，非强制私有   |

### 2. 代码注释约定

| ***标签***     | ***含义*** | ***适用场景***                                     | ***优先级*** |
| :------------- | :--------- | :------------------------------------------------- | :----------- |
| **`TODO`**     | 待办       | 计划要做但还没做的功能，或需要优化的代码。         | 中           |
| **`FIXME`**    | 修复       | 代码已知有 Bug 或错误，需要尽快修复。              | 高           |
| **`HACK`**     | 临时方案   | 代码写得比较“脏”或用了临时变通方法，以后需要重构。 | 低 (需警惕)  |
| **`XXX`**      | 警告/注意  | 代码存在潜在风险，其他人修改时需小心。             | 中/高        |
| **`NOTE`**     | 备注       | 重要的说明信息，解释为什么要这么写。               | 低           |
| **`OPTIMIZE`** | 优化       | 性能或资源使用方面需要改进的地方。                 | 中           |

### 3. `unittest` 断言

| ***方法***                    | ***检查条件***     | ***说明***                |
| ----------------------------- | ------------------ | ------------------------- |
| **`assertEqual(a, b)`**       | `a == b`           | 相等                      |
| **`assertNotEqual(a, b)`**    | `a != b`           | 不相等                    |
| **`assertTrue(x)`**           | `bool(x) is True`  | 为真                      |
| **`assertFalse(x)`**          | `bool(x) is False` | 为假                      |
| **`assertIs(a, b)`**          | `a is b`           | 是同一个对象              |
| **`assertIsNone(x)`**         | `x is None`        | 为空                      |
| **`assertIn(a, b)`**          | `a in b`           | 包含 (常用于字符串或列表) |
| **`assertNotIn(a, b)`**       | `a not in b`       | 不包含                    |
| **`assertGreater(a, b)`**     | `a > b`            | 大于                      |
| **`assertLess(a, b)`**        | `a < b`            | 小于                      |
| **`assertRaises(Exception)`** | 抛出异常           | 验证代码是否抛出特定异常  |

### 4. `pytest` 断言

```python
# pytest 风格
def test_login_success(driver):
    driver.get("https://example.com/login")
    driver.find_element(By.ID, "username").send_keys("admin")
    driver.find_element(By.ID, "submit").click()
    
    # 直接使用 assert，无需 self.
    assert "Dashboard" in driver.title
    assert driver.find_element(By.CLASS_NAME, "welcome-msg").is_displayed()
    
    # 甚至可以写复杂的表达式
    items = driver.find_elements(By.CLASS_NAME, "product")
    assert len(items) > 0
```
