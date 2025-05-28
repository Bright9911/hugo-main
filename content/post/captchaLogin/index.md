+++
title = '【Laravel】验证码登录'
date = 2025-05-28T11:18:07+08:00
draft = true
categories = [
    "laravel",
    "captcha",
    "笔记",
]
image = "93314e48ec9513292e4b1e8ca1c73c9d.jpg"

+++

## 1、安装验证码扩展包

首先需要安装mews/captcha验证码扩展包，执行命令：

```bash
composer require mews/captcha
```

## 2、修改登录视图

在（login.blade.php）表单中添加验证码输入框和验证码图片区域：

```html
<!DOCTYPE html>
<html>
<head>
    <title>登录</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/layui-src/dist/css/layui.css">
    <link href="https://unpkg.com/layui@2.9.24/dist/css/layui.css" rel="stylesheet">
    <script src="https://unpkg.com/layui@2.9.24/dist/layui.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    <style>
        /* 自定义样式 */
        body {
            background-color: #f5f5f5;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        .login-container {
            max-width: 400px;
            margin: 80px auto;
            padding: 30px;
            background-color: white;
            border-radius: 10px;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
        }

        .login-header {
            text-align: center;
            margin-bottom: 30px;
        }

        .login-header h1 {
            font-size: 24px;
            color: #333;
            font-weight: 500;
        }

        .layui-form-item {
            margin-bottom: 20px;
            margin-left: -30px;
            margin-right: 30px;
        }

        .layui-input {
            width: 100%;
            padding: 12px 15px;
            border: 1px solid #e0e0e0;
            border-radius: 4px;
            font-size: 14px;
            transition: border-color 0.3s;
        }

        .layui-input:focus {
            border-color: #1e88e5;
            box-shadow: 0 0 5px rgba(30, 136, 229, 0.3);
        }

        .layui-btn {
            width: 100%;
            border: none;
            border-radius: 4px;
            color: white;
            font-size: 14px;
            font-weight: 500;
            cursor: pointer;
        }

        .layui-btn:hover {
            opacity: 0.9;
        }

        .register-link {
            text-align: center;
            margin-top: 20px;
        }

        .register-link a {
            text-decoration: none;
            font-size: 14px;
        }

        .register-link a:hover {
            text-decoration: underline;
        }

        /* 响应式设计 */
        @media (max-width: 480px) {
            .login-container {
                margin: 40px 15px;
            }
        }

        .captcha-img {
            cursor: pointer;
            height: 38px;
            border: 1px solid #e0e0e0;
            border-radius: 4px;
            vertical-align: middle;
        }

    </style>
</head>
<body>
<div class="login-container">
    <div class="login-header">
        <h1>登录</h1>
    </div>
    <form class="layui-form" id="loginForm">
        <div class="layui-form-item">
            <label class="layui-form-label">姓名</label>
            <div class="layui-input-block">
                <input type="text" name="name" required lay-verify="required" placeholder="请输入姓名或邮箱" autocomplete="off" class="layui-input">
            </div>
        </div>
        <div class="layui-form-item">
            <label class="layui-form-label">密码</label>
            <div class="layui-input-block">
                <input type="password" name="password" required lay-verify="required" placeholder="请输入密码" autocomplete="off" class="layui-input">
            </div>
        </div>

        <div class="layui-form-item" style="display: flex">
            <label class="layui-form-label">验证码</label>
            <div style="width: 150px;margin-right: 10px">
                <input type="text" name="captcha" required lay-verify="required" placeholder="验证码" class="layui-input">
            </div>
            <div>
                <img src="{{ captcha_src() }}"
                     alt="点击刷新验证码"
                     class="captcha-img"
                     onclick="this.src='{{ captcha_src() }}?'+Math.random()">
            </div>
        </div>

        <div class="layui-form-item">
            <div class="layui-input-block">
                <button class="layui-btn" lay-submit lay-filter="login">登录</button>
            </div>
        </div>
    </form>
    <div class="register-link">
        <a href="{{ route('register') }}" style="margin-right: 20px">注册新账号</a>
        <a href="/password/reset">忘记密码？</a>
    </div>
</div>

<script>
    layui.use(['form'], function(){
        var form = layui.form;
        var $ = layui.$;

        form.on('submit(login)', function(data){
            login(data.field);
            return false;
        });
    });

    async function login(data) {
        try {
            const response = await axios.post('/login', data);
            if (response.data.code === 200) {
                layui.use('layer', function(){
                    var layer = layui.layer;
                    layer.msg(response.data.message, {icon: 1, time: 1000}, function(){
                        window.location.href = response.data.data.redirectUrl;
                    });
                });
            } else {
                layui.use('layer', function(){
                    var layer = layui.layer;
                    layer.msg(response.data.message, {icon: 2, time: 2000});
                });
            }
        } catch (error) {
            // 处理验证码等验证错误（Laravel默认返回422状态码）
            if (error.response && error.response.status === 422) {
                const errors = error.response.data.errors;
                const firstError = Object.values(errors)[0][0];
                layui.use('layer', function(){
                    var layer = layui.layer;
                    layer.msg(firstError, {icon: 2, time: 2000});
                });
            } else {
                layui.use('layer', function(){
                    var layer = layui.layer;
                    layer.msg('登录失败，请重试', {icon: 2, time: 2000});
                });
            }
            console.error('Login error:', error);
        }
    }
</script>
</body>
</html>
```

## 3、修改登录控制器

在登录验证逻辑（LoginController.php）中添加验证码校验规则：

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    public function __construct()
    {
        $this->middleware('guest')->except('logout');
    }

    public function showLoginForm()
    {
        return view('auth.login');
    }

    public function login(Request $request)
    {
        $request->validate([
            'name'     => 'required|string',
            'password' => 'required|string',
            'captcha'  => 'required|captcha',
        ]);

        // 动态选择认证字段：判断输入是否为邮箱
        $loginInput = $request->input('name');
        $field = filter_var($loginInput, FILTER_VALIDATE_EMAIL) ? 'email' : 'name';

        $credentials = [
            $field    => $loginInput,
            'password'=> $request->input('password'),
        ];

        if (Auth::attempt($credentials)) {
            $request->session()->regenerate();

            // 记录登录成功
            \App\Services\LogService::write(
                'login',
                '用户登录成功',
                ['name' => $request->name]
            );

            return response()->json([
                'code' => 200,
                'message' => '登录成功',
                'data' => [
                    'redirectUrl' => route('home')
                ]
            ]);
        }

        // 记录登录失败
        \App\Services\LogService::write(
            'login_failed',
            '登录失败',
            [
                'name' => $request->name,
                'errors' => '姓名或密码不正确'
            ]
        );

        return response()->json([
            'code' => 400,
            'message' => '姓名或密码不正确'
        ]);
    }

    public function logout(Request $request)
    {

        Auth::logout();
        $request->session()->invalidate();
        $request->session()->regenerateToken();

        // 记录退出登录
        \App\Services\LogService::write(
            'logout',
            '用户退出登录',
            ['user_id' => Auth::id()]
        );

        return response()->json([
            'code' => 200,
            'message' => '退出成功',
            'data' => [
                'redirectUrl' => '/login'
            ]
        ]);
    }
}
```

## 4、验证配置（可选）

如果需要自定义验证码样式（如长度、字符集），可以发布配置文件：

```bash
php artisan vendor:publish --provider="Mews\Captcha\CaptchaServiceProvider"
```

然后修改config/captcha.php中的参数（如length控制验证码位数，width控制图片宽度等）。