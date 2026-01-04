+++
title = '【Laravel】扫码登录'
date = 2025-05-28T11:22:56+08:00
draft = true
categories = [
    "laravel",
    "笔记",
]
image = "4ab9d960e5111d286f6396a8edee2b46.jpg"

+++

## 使用手机扫码登录系统

### 安装二维码生成库

```bash
composer require simplesoftwareio/simple-qrcode
```

### web.php

```php
// 扫码
// 生成二维码
Route::get('/login/qr', [App\Http\Controllers\Auth\LoginController::class, 'generateQrCode']);
// 扫码绑定
Route::get('/auth/scan/{token}', [App\Http\Controllers\Auth\ScanLoginController::class, 'bindScanLogin']);
// 确认登录
Route::get('/auth/confirm/{token}', [App\Http\Controllers\Auth\LoginController::class, 'confirmLogin']);
// 检查 Token 是否绑定成功
Route::get('/auth/check/{token}', [App\Http\Controllers\Auth\LoginController::class, 'checkToken']);
```

### Controllers/Auth/ScanLoginController.php

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Cache;

class ScanLoginController extends Controller
{
    // 扫码绑定
    public function bindScanLogin($token)
    {
        // 验证Token是否有效
        $scanInfo = Cache::get('scan_login_token_' . $token);
        if (!$scanInfo) {
            \Log::error('扫码绑定失败：Token无效或已过期，Token: ' . $token);
            return '二维码已过期或无效';
        }

        // 如果用户未登录，则提示用户登录
        if (!Auth::check()) {
            \Log::info('扫码绑定：用户未登录，Token: ' . $token);
            return redirect()->guest('/login');
        }

        // 绑定扫码登录
        Cache::put('scan_login_token_' . $token, [
            'user_id' => Auth::user()->id,
            'expired_at' => $scanInfo['expired_at']
        ], $scanInfo['expired_at']);

        \Log::info('扫码绑定成功，Token: ' . $token . '，User ID: ' . Auth::user()->id);
        return '扫码绑定成功，请在设备上确认登录';
    }
}
```

### 在Controllers/Auth/LoginController.php中加上

```php
use SimpleSoftwareIO\QrCode\Facades\QrCode;
use Illuminate\Support\Str;

// 生成二维码
public function generateQrCode()
{
    $token = Str::random(32);
    $expiredAt = now()->addMinutes(5);

    // 存储Token到缓存
    \Cache::put('scan_login_token_' . $token, [
        'expired_at' => $expiredAt
    ], $expiredAt);

    // 记录 Token 到会话中
    session(['scan_login_token' => $token]);

    // 生成二维码（SVG 格式）
    $qrCode = QrCode::size(300)->generate(url('/auth/scan/' . $token));

    // 添加正确的 Data URI 前缀
    $qrcodeDataUri = 'data:image/svg+xml;base64,' . base64_encode($qrCode);

    return view('login.qr', ['qrcode' => $qrcodeDataUri]);
}

// 确认登录
public function confirmLogin($token)
{
    // 验证Token是否有效
    $scanInfo = \Cache::get('scan_login_token_' . $token);
    if (!$scanInfo || !isset($scanInfo['user_id'])) {
        return redirect('/login')->withErrors('登录已过期或无效');
    }

    // 登录用户
    Auth::loginUsingId($scanInfo['user_id']);

    // 清除缓存
    \Cache::forget('scan_login_token_' . $token);

    return redirect('/home');
}

// 检查Token
public function checkToken($token)
{
    $scanInfo = \Cache::get('scan_login_token_' . $token);
    if ($scanInfo && isset($scanInfo['user_id'])) {
        return response()->json(['success' => true]);
    }
    return response()->json(['success' => false]);
}
```

### resources/views/login/qr.blade.php

```html
<!DOCTYPE html>
<html>
<head>
    <title>扫码登录</title>
</head>
<body>
<div style="text-align: center; margin-top: 50px;">
    <h2>扫码登录</h2>
    <img src="{{ $qrcode }}" alt="扫码登录">
    <p>请使用手机扫描二维码完成登录</p>
</div>

<script>
    // 每隔 5 秒检查一次 Token 是否绑定成功
    setInterval(function() {
        var token = "{{ session('scan_login_token') }}";
        fetch("{{ url('http://[本地IP]:8000/auth/check') }}/" + token)
            .then(response => response.json())
            .then(data => {
                if (data.success) {
                    window.location.href = "{{ url('/auth/confirm') }}/" + token;
                }
            });
    }, 5000);
</script>
</body>
</html>
```

### 在config/app.php中

```php
'providers' => [
    // ...
    SimpleSoftwareIO\QrCode\QrCodeServiceProvider::class,
],

'aliases' => [
    // ...
    'QrCode' => SimpleSoftwareIO\QrCode\Facades\QrCode::class,
],
```

### 在.env中

```php
APP_URL=http://[本地IP]:8000
```

### 启动项目

```bash
php artisan serve --host 0.0.0.0 --port 8000

// 在以下路径访问
http://[本地IP]:8000/login/qr
```

