+++
title = '【Laravel】邮件发送'
date = 2025-05-28T11:08:31+08:00
draft = true
categories = [
    "Laravel",
    "mail",
    "笔记",
]
image = "9ad3bf3c4aadf61bd96eebfc1f7f577e.jpg"

+++

## 1、发送纯文字

### 1.1、.env配置

```php
MAIL_MAILER=smtp
MAIL_HOST=smtp.163.com
MAIL_PORT=465
MAIL_USERNAME={自己的邮箱}@163.com
MAIL_PASSWORD={邮箱密钥}
MAIL_ENCRYPTION=ssl
MAIL_FROM_ADDRESS={自己的邮箱}@163.com
MAIL_FROM_NAME="${APP_NAME}"
```

### 1.2、EmailController.php

```php
<?php

namespace App\Http\Controllers;

use App\Mail\VerificationCodeEmail;
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Str;
use Illuminate\Support\Facades\Session;
class EmailController extends Controller
{
    public function sendMail()
    {
        $email = request()->input('email');

        // 验证邮箱格式
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return response()->json([
                'success' => false,
                'message' => '无效的邮箱地址'
            ]);
        }

        // 生成验证码
        $code = mt_rand(1000, 9999);
        Session::put('email_verification_code', $code);

        // 发送纯文本邮件
        Mail::raw("尊敬的用户，您的验证码为：{$code}，请在5分钟内完成验证。", function ($message) use ($email) {
            $message->to($email)->subject('验证码邮件');
        });

        if (Mail::failures()) {
            return response()->json([
                'success' => false,
                'message' => '邮件发送失败，请重试！'
            ]);
        }

        return response()->json([
            'success' => true,
            'message' => '验证码已发送，请查收！'
        ]);
    }
}
```

### 1.3、web.php

```php
Route::get('/mail', function () {
    return view('email.email');
});

Route::get('/mail/send', [EmailController::class, 'sendMail']);
```

### 1.4、email.blade.php

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>发送验证码</title>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
</head>
<body>
<div style="max-width: 500px; margin: 0 auto; padding: 20px;">
    <h2>发送验证码</h2>
    <form id="verification-form">
        <div style="margin-bottom: 15px;">
            <label for="email">邮箱地址:</label>
            <input type="email" id="email" name="email" placeholder="请输入邮箱地址" required style="width: 100%; padding: 8px; margin-top: 5px;">
        </div>
        <button type="submit" id="send-btn" style="padding: 10px 20px; background-color: #4CAF50; color: white; border: none; cursor: pointer;">发送验证码</button>
    </form>
    <div id="message" style="margin-top: 20px; padding: 10px; border-radius: 5px; display: none;"></div>
</div>

<script>
    $(document).ready(function() {
        $('#verification-form').on('submit', function(e) {
            e.preventDefault();

            // 获取邮箱地址
            const email = $('#email').val();

            // 验证邮箱格式
            const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
            if (!emailRegex.test(email)) {
                $('#message').css('background-color', '#f44336').text('请输入有效的邮箱地址！');
                $('#message').show();
                return;
            }

            // 禁用按钮，防止重复点击
            $('#send-btn').prop('disabled', true).text('发送中...');

            // 发送AJAX请求
            $.ajax({
                url: '/mail/send',
                type: 'GET',
                data: { email: email },
                success: function(response) {
                    if (response.success) {
                        $('#message').css('background-color', '#4CAF50').text('验证码已发送，请查收！');
                    } else {
                        $('#message').css('background-color', '#f44336').text(response.message || '发送失败，请重试！');
                    }
                    $('#message').show();
                },
                error: function() {
                    $('#message').css('background-color', '#f44336').text('网络错误，请重试！');
                    $('#message').show();
                },
                complete: function() {
                    // 恢复按钮状态
                    $('#send-btn').prop('disabled', false).text('发送验证码');
                }
            });
        });

        // 5秒后自动隐藏消息
        const observer = new MutationObserver(function(mutations) {
            mutations.forEach(function(mutation) {
                if (mutation.target.id === 'message') {
                    setTimeout(function() {
                        $('#message').hide();
                    }, 5000);
                }
            });
        });

        observer.observe(document.getElementById('message'), {
            attributes: true,
            childList: true,
            characterData: true
        });
    });
</script>
</body>
</html>
```



## 2、发送模板

### 2.1、创建邮件模板

在 resources/views/emails 目录下创建一个新的视图文件，例如 verification.blade.php。这个文件将作为邮件的模板。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>验证码邮件</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
        }
        .container {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
            background-color: #fff;
            border-radius: 5px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        .header {
            background-color: #4CAF50;
            color: white;
            padding: 10px;
            border-radius: 5px 5px 0 0;
            text-align: center;
        }
        .content {
            padding: 20px;
        }
        .footer {
            text-align: center;
            padding: 10px;
            color: #777;
            font-size: 12px;
        }
        .code {
            font-size: 24px;
            font-weight: bold;
            color: #4CAF50;
            text-align: center;
            padding: 10px;
            margin: 20px 0;
            background-color: #f9f9f9;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>验证码邮件</h1>
        </div>
        <div class="content">
            <p>尊敬的用户，您的验证码为：</p>
            <div class="code">{{ $code }}</div>
            <p>请在5分钟内完成验证。</p>
        </div>
        <div class="footer">
            <p>© 2023 验证码系统。保留所有权利。</p>
        </div>
    </div>
</body>
</html>
```

### 2.2、创建邮件类

使用 Artisan 命令创建一个邮件类：

```bash
php artisan make:mail VerificationMail
```

### 2.3、更新邮件类

打开 app/Mail/VerificationMail.php 文件，并更新其内容：

```php
<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class VerificationMail extends Mailable
{
    use Queueable, SerializesModels;

    public $code;

    /**
     * Create a new message instance.
     *
     * @param  string  $code
     * @return void
     */
    public function __construct($code)
    {
        $this->code = $code;
    }

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.verification')
                    ->subject('验证码邮件');
    }
}
```

### 2.4、更新控制器

在 MailController 中使用 VerificationMail 类来发送邮件：

```php
use App\Mail\VerificationMail;
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\Session;

public function sendMail()
{
    try {
        $email = request()->input('email');

        // 验证邮箱格式
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return response()->json([
                'success' => false,
                'message' => '无效的邮箱地址'
            ]);
        }

        // 生成验证码
        $code = mt_rand(1000, 9999);
        Session::put('email_verification_code', $code);

        // 发送邮件
        Mail::to($email)->send(new VerificationMail($code));

        if (Mail::failures()) {
            return response()->json([
                'success' => false,
                'message' => '邮件发送失败，请重试！'
            ]);
        }

        return response()->json([
            'success' => true,
            'message' => '验证码已发送，请查收！'
        ]);
    } catch (\Exception $e) {
        return response()->json([
            'success' => false,
            'message' => '服务器错误：' . $e->getMessage()
        ]);
    }
}
```



## 3、恶搞页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>恭喜您中奖了！</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
        }
        .container {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
            background-color: #fff;
            border-radius: 5px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        .header {
            background-color: #FFD700;
            color: white;
            padding: 10px;
            border-radius: 5px 5px 0 0;
            text-align: center;
        }
        .content {
            padding: 20px;
        }
        .footer {
            text-align: center;
            padding: 10px;
            color: #777;
            font-size: 12px;
        }
        .prize {
            font-size: 24px;
            font-weight: bold;
            color: #FFD700;
            text-align: center;
            padding: 10px;
            margin: 20px 0;
            background-color: #f9f9f9;
            border-radius: 5px;
        }
    </style>
</head>
<body>
<div class="container">
    <div class="header">
        <h1>恭喜您中奖了！</h1>
    </div>
    <div class="content">
        <p>尊敬的用户，您已被选为本月的幸运用户！</p>
        <div class="prize">恭喜您获得了一款神秘游戏：好玩的贪吃蛇！</div>
        <p>请点击下方链接领取您的大奖：</p>
        <p><a href="https://github.com/JIEJOE-Visual/snakeball" style="color: #FFD700; text-decoration: none; padding: 10px 20px; background-color: #333; border-radius: 5px; display: inline-block;">立即领取</a></p>
        <p>注意：此链接将在24小时内有效，请尽快领取！</p>
    </div>
    <div class="footer">
        <p>© 2025 幸运抽奖系统。保留所有权利。</p>
    </div>
</div>
</body>
</html>
```