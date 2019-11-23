---
description: 如果路上有坑，就要毫不犹豫的跳下去
---

# 一个完整的登陆页

登陆页是一个软件的门面。一个完整的登陆页包含账号密码登陆、验证码登陆、注册及忘记密码四个功能，下面从框架开始一步步完成。

## 搭建环境

### 1. 封装网络请求库

登陆功能需要实现网络请求的功能，引入dio库来进行网络请求类的封装，需要学习的可以参考pub官网的教程，有[中文版](https://github.com/flutterchina/dio/blob/master/README-ZH.md)哦

> dio是一个强大的Dart Http请求库，支持Restful API、FormData、拦截器、请求取消、Cookie管理、文件上传/下载、超时、自定义适配器等...

为了方便使用，Dio提供了一些其它的Restful API, 这些API都是`request`的别名。

### 2. 导入持久存储库

`shared_preferences` 为简单数据提供一个持久的存储。数据被异步地持久化到磁盘。需要注意的一点是，两个平台都不能保证写操作在返回后被持久化到磁盘上，所以这个插件不能用于存储关键数据。

这里主要用于缓存一下token

## 编写widget

如何搭建程序框架就不在这里介绍了，下面主要讲解登陆页面的搭建

#### 给页面加个背景图

```dart
class LoginPage extends StatefulWidget {
  static final String sName = "login";

  @override
  _LoginPageState createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
 
  @override
  void initState() {
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return new Container(
      decoration: BoxDecoration(
        image: DecorationImage(
          image: AssetImage('static/images/login_back.jpeg'),
          fit: BoxFit.cover,
        ),
      ),
    );
  }
}
```

最外围的Container在默认情况下会铺满整个屏幕，使用BoxDecoration对container进行装饰。

这里有个坑需要注意一下，一般情况下我们都会用`Scaffold` 来包装整个页面，因为这个组件提供了一系列非常方便的属性来配置整个页面，但如果我们想要一个背景图，就必须先写一个Container，因为如果使用`Scaffold` ，就会在键盘弹出的时候导致页面重绘，背景图会受到挤压而变形。



```dart
class LoginPage extends StatefulWidget {
  static final String sName = "login";

  @override
  _LoginPageState createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  /// 定义登陆模式：账号密码登陆或者
  String loginMode = 'login';
  /// 定义验证码按钮被点击后的延时
  int vercodeDelay = 0;
  Timer _timer;

  TextEditingController _usernameController = TextEditingController();
  TextEditingController _passwordController = TextEditingController();
  TextEditingController _verCodeController = TextEditingController();

  @override
  void initState() {
    super.initState();
    _usernameController.text = '18515597305';
    _passwordController.text = '123456789';
  }

  @override
  Widget build(BuildContext context) {
    return new Container(
      decoration: BoxDecoration(
        image: DecorationImage(
          image: AssetImage('static/images/login_back.jpeg'),
          fit: BoxFit.cover,
        ),
      ),
      child: Scaffold(
        resizeToAvoidBottomPadding: false, // 就是这里
        backgroundColor: Color.fromARGB(150, 255, 255, 255),
        body: Container(
          padding: EdgeInsets.all(40),
          child: getLoginComp(),
        ),
      ),
    );
  }

  getPasswordComp() {
    return TextField(
      controller: _passwordController,
      decoration: InputDecoration(
        // labelText: "密码",
        hintText: "您的登录密码",
        prefixIcon: Icon(Icons.lock),
        contentPadding: EdgeInsets.fromLTRB(20.0, 10.0, 20.0, 10.0),
        border: OutlineInputBorder(borderRadius: BorderRadius.circular(32.0)),
      ),
      obscureText: true,
    );
  }

  /// 登陆模块
  getLoginComp() {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Text(
          'Welcome',
          style: TextStyle(
            fontSize: 32,
          ),
        ),
        SizedBox(
          height: 32,
        ),
        TextField(
            // autofocus: true,
            controller: _usernameController,
            decoration: InputDecoration(
              // labelText: "用户名",
              hintText: "手机号",
              prefixIcon: Icon(Icons.person),
              contentPadding: EdgeInsets.fromLTRB(20.0, 10.0, 20.0, 10.0),
              border:
                  OutlineInputBorder(borderRadius: BorderRadius.circular(32.0)),
            ),
            keyboardType: TextInputType.number,
            inputFormatters: [
              WhitelistingTextInputFormatter(RegExp("[0-9]")),
            ]),
        SizedBox(
          height: 12,
        ),
        loginMode == 'login' ? getPasswordComp() : getVerificationComp(),
        SizedBox(
          height: 32,
        ),
        SizedBox(
          width: double.infinity,
          height: 40,
          child: FlatButton(
            color: Colors.blue,
            // highlightColor: Colors.transparent,
            colorBrightness: Brightness.dark,
            // splashColor: Colors.grey,
            child: Text(
              "登陆",
              style: TextStyle(fontSize: 20),
            ),
            shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(20.0)),
            onPressed: () {
              onLogin();
            },
          ),
        ),
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: <Widget>[
            FlatButton(
              color: Colors.transparent,
              child: Text(
                loginMode == 'login' ? "验证码登陆" : "账号密码登陆",
                style: TextStyle(fontSize: 16, color: Colors.blue),
              ),
              onPressed: () {
                onChangeLoginMode();
              },
            ),
            FlatButton(
              color: Colors.transparent,
              child: Text(
                "忘记密码",
                style: TextStyle(fontSize: 16, color: Colors.blue),
              ),
              onPressed: () {},
            ),
          ],
        )
      ],
    );
  }

  /// 验证码模块
  getVerificationComp() {
    return Stack(
      children: <Widget>[
        TextField(
          // autofocus: true,
          keyboardType: TextInputType.number,
          controller: _verCodeController,
          decoration: InputDecoration(
            prefixIcon: Icon(Icons.lock),
            contentPadding: EdgeInsets.fromLTRB(20.0, 10.0, 20.0, 10.0),
            border:
                OutlineInputBorder(borderRadius: BorderRadius.circular(32.0)),
          ),
        ),
        Positioned(
          right: 5,
          child: FlatButton(
            disabledColor: Colors.grey[400],
            color: Colors.blue,
            // highlightColor: Colors.transparent,
            colorBrightness: Brightness.dark,
            // splashColor: Colors.grey,
            child: Text(
              vercodeDelay == 0 ? "获取验证码" : vercodeDelay.toString() + 's',
              style: TextStyle(fontSize: 12),
            ),
            shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(20.0)),
            onPressed: vercodeDelay == 0
                ? () {
                    onGetVerification();
                  }
                : null,
          ),
        )
      ],
    );
  }

  /// 获取验证码
  onGetVerification() async {
    if (vercodeDelay != 0) {
      return;
    }

    String phone = '18515597305';
    // String phone = _usernameController.text;
    if (!CommonUtils.isTelphone(phone)) {
      return;
    }
    setState(() {
      vercodeDelay = 60;
      startCountdownTimer();
    });

    // var res = await NetUtils.getVercode(phone);
  }

  /// 变更登陆模式
  onChangeLoginMode() {
    if (loginMode == 'login') {
      this.setState(() {
        loginMode = 'Verification';
      });
    } else {
      this.setState(() {
        loginMode = 'login';
      });
    }
  }

  /// 账号密码登陆
  onLogin() async {
    String phone = _usernameController.text;
    String password = _passwordController.text;
    if (phone == '' || phone.length != 11) {
      return;
    }
    if (password == '' || password.length < 6) {
      return;
    }
    if (loginMode == 'login') {
      Map param = {
        'phone': phone,
        'password': password,
      };
      Map res = await NetUtils.login(param);
      if (res['code'] == 200) {
        Navigator.of(context).pushReplacementNamed("home");
      }
      return;
    }
    var res = await NetUtils.vercodeLogin(password);
    if (res['code'] == 200) {
      Navigator.of(context).pushReplacementNamed("home");
    }
  }

  /// 倒计时
  void startCountdownTimer() {
    const oneSec = const Duration(seconds: 1);

    var callback = (timer) => {
          setState(() {
            if (vercodeDelay < 1) {
              _timer.cancel();
            } else {
              vercodeDelay = vercodeDelay - 1;
            }
          })
        };
    _timer = Timer.periodic(oneSec, callback);
  }

  @override
  void dispose() {
    super.dispose();
    if (_timer != null) {
      _timer.cancel();
    }
  }
}

```

