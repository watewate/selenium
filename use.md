# 代码简要分析

## yml测试用例

```
testinfo:
    - id: test001
      title: 登录
      info: 打开testerhome
testcase:
    - element_info: div.container>ul>li:nth-child(2)
      find_type: css
      operate_type: click
      info: 点击登录
    - element_info: input-lg
      find_type: class_name
      operate_type: send_keys
      msg: lose
      info: 输入用户名
    - element_info: user_password
      find_type: id
      operate_type: send_keys
      msg: XXXX
      info: 输入密码XXXX
    - element_info: div.form-actions
      find_type: css
      operate_type: click
      info: 点击登录
    - element_info: dropdown-avatar
      find_type: class_name
      operate_type: click
      info: 点击图像

check:
    - element_info: //ul[@class='dropdown-menu']/li/a[contains(text(),'lose')]
      find_type: xpath
      check: default_check(默认可以不传，就是简单的查找页面元素)
      info: 查找用户名成功
```
- 关于check下面可以填检查点的类型：
    - CONTRARY = "contrary" # 相反检查点，表示如果检查元素存在就说明失败，如删除后，此元素依然存在
    - CONTRARY_GETVAL = "contrary_getval" # 检查点关键字contrary_getval: 相反值检查点，如果对比成功，说明失败
    - DEFAULT_CHECK = "default_check" # 默认检查点，就是查找页面元素
    - COMPARE = "compare" # 历史数据和实际数据对比

## 某个用例的page层

```
class LoginPage:
    def __init__(self, kwargs):
        _init = {"driver": kwargs["driver"], "test_msg": getYam(kwargs["path"]),
                 "logTest": kwargs["logTest"], "caseName": kwargs["caseName"]}
        self.page = Pages.PagesObjects(_init)

    def operate(self):  # 操作步骤
        self.page.operate()

    def checkPoint(self):  # 检查点
        self.page.checkPoint()
```

- pages再次封装了一层，主要使用了 setupclass+ self.driver.get重定位的方式
  - 避免用例依赖，并不会每个用例重新启动一个session，重连机制也是这样实现的（后续填坑）

```
        if kwargs.get("launch", "0") == "0":  # 若为空， 刷新页面
            self.driver.get(self.driver.current_url)
```

## testcase层调用page层

```
cclass HomeTest(ParametrizedTestCase):
    def testALoginFail(self):
        app = {"logTest": self.logTest, "driver": self.driver, "path": PATH("../Yamls/home/LoginFail.yaml"),
               "caseName": sys._getframe().f_code.co_name}

        page = LoginFailPage(app)
        page.operate()
        page.checkPoint()

    def testBLogin(self):
        app = {"logTest": self.logTest, "driver": self.driver, "path": PATH("../Yamls/home/Login.yaml"),
               "caseName": sys._getframe().f_code.co_name}

        page = LoginPage(app)
        page.operate()
        page.checkPoint()
```

## 封装自己的关键字

- 在BaseOperate中定义自己的关键字
```buildoutcfg

    def operate_by(self, operate, testInfo, logTest):
       ...........
            elements = {
                be.CLICK: lambda: self.click(operate),
                be.GET_VALUE: lambda: self.get_value(operate),
                be.GET_TEXT: lambda: self.get_text(operate),
                be.SEND_KEYS: lambda: self.send_keys(operate),
                be.MOVE_TO_ELEMENT: lambda: self.move_to_element(operate)
            }
```
- 在用例中yaml传入自己的关键字即可,看看下面的operate_type中的鼠标悬停

```buildoutcfg
testcase:
    - element_info: cate_item_108698
      find_type: id
      operate_type: move_to_element
      info: 鼠标悬停到.net分类上
```
