+++
title = '【Laravel】事件，队列'
date = 2025-07-08T15:05:13+08:00
draft = true
comments= true
categories = [
    "Laravel",
    "笔记",
]
image = "100.jpg"

+++

## 一、事件

事件系统允许你在应用中定义事件和监听器，当事件发生时，监听器会自动执行相应的操作。在文章管理系统中，你可以定义一些事件，如文章发布、文章更新、文章删除等。事件系统能够使代码更加模块化，主要用于解耦代码，使得文章发布操作与后续操作分离，便于维护和扩展。默认情况下，事件监听器是同步执行的。

**实现步骤：**

### 1、创建事件类

使用命令创建事件类，例如ArticlePublished。

```bash
php artisan make:event ArticlePublished
```

这将在app/Events目录下生成ArticlePublished.php文件。

```php
namespace App\Events;

use App\Models\Article;
use Illuminate\Foundation\Events\Dispatchable;

class ArticlePublished
{
    use Dispatchable;

    public $article;

    public function __construct(Article $article)
    {
        $this->article = $article;
    }
}
```

### 2、创建监听器类

使用命令创建监听器类，例如SendArticlePublishedNotification。

```bash
php artisan make:listener SendArticlePublishedNotification --event=ArticlePublished
```

这将在app/Listeners目录下生成SendArticlePublishedNotification.php文件。

```php
namespace App\Listeners;

use App\Events\ArticlePublished;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendArticlePublishedNotification implements ShouldQueue
{
    public function handle(ArticlePublished $event)
    {
        // 发送通知逻辑，例如发送邮件、短信等
        // 这里可以调用邮件服务、短信服务等
        // 我以邮件发送为例
        $email = "15060257172@163.com";

        // 发送纯文本邮件
        Mail::raw("文章“{$event->article->title}”发布成功！ -- 事件监听", function ($message) use ($email) {
            $message->to($email)->subject('通知邮件 -- 事件');
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

### 3、注册事件和监听器

在app/Providers/EventServiceProvider中注册事件和监听器的映射关系。

```php
protected $listen = [
    \App\Events\ArticlePublished::class => [
        \App\Listeners\SendArticlePublishedNotification::class,
    ],
];
```

触发事件：在文章发布的控制器方法中触发事件。

```php
use App\Events\ArticlePublished;

public function publish(Article $article)
{
    // 发布文章的逻辑

    // 触发事件
    event(new ArticlePublished($article));
}
```



## 二、队列

队列用于处理耗时任务，如发送邮件、生成报告等。在文章管理系统中，你可以将发送通知、文章审核等任务放入队列中异步处理。主要用于异步处理耗时任务，提高系统的响应速度和用户体验。

**实现步骤：**

### 1、配置队列驱动

在.env文件中配置队列驱动，例如使用Redis。

```env
QUEUE_CONNECTION=redis
```

### 2、创建队列任务类

使用命令创建队列任务类，例如ProcessArticleNotification。

```bash
php artisan make:job ProcessArticleNotification
```

这将在app/Jobs目录下生成ProcessArticleNotification.php文件。

```php
namespace App\Jobs;

use App\Models\Article;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;

class ProcessArticleNotification implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable;

    protected $article;

    public function __construct(Article $article)
    {
        $this->article = $article;
    }

    public function handle()
    {
        // 处理通知逻辑，例如发送邮件、短信等
        $email = "15060257172@163.com";

        // 发送纯文本邮件
        Mail::raw("文章“{$this->article->title}”发布成功！ -- 队列", function ($message) use ($email) {
            $message->to($email)->subject('通知邮件 -- 队列');
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

    public function failed(\Exception $exception)
    {
        // 任务失败时的处理逻辑
        \Log::error('邮件发送失败: ' . $exception->getMessage());
        // 可以在这里发送通知、记录数据库等
    }
}
```

### 3、分发队列任务

在控制器中分发队列任务。

```php
use App\Jobs\ProcessArticleNotification;

public function publish(Article $article)
{
    // 发布文章的逻辑
    $article->update(['status' => 'published']);

    // 分发队列任务
    ProcessArticleNotification::dispatch($article);
}
```

### 4、运行队列监听器

在命令行中运行队列监听器，处理队列任务。

```bash
php artisan queue:work
```



<blockquote class="alert-note">
如果你使用了队列任务或队列化的事件监听器，必须运行 php artisan queue:work 来处理它们。
</blockquote>