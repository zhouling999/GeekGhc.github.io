---
layout: post
title: Laravel用户之间相互关注
description: 在一个系统或者论坛 用户之间可以相互关注 类似于Github的followers和following这样的应用场景 在laravel我们也可以去
            实现这样的用户与用户之间的关联
tags:
     Laravel
     Notify
class: post-one
comments: true
poster: /attachments/images/articles/2017-03-20/poster.jpg
---
## 介绍
有关用户之间的相互关注这样的应用场景还是很常见的 每个平台都会有这样类似的需求  就比如**Segmentfault**和我们的**知乎**

当然还有最熟悉的`Github`每个人可以有关注者和粉丝 在我写的社区里也需要用到这样的需求 

现在在我开发的类似知加的问答圈 因为以`laravel`作为后端数据 也同样会应用到这样的功能

索性就谈谈如何在`laravel`中去实现我们用户之间的互相关注

### 建立模型表
这里我们去建立一个中间表 可以想象得到的是这张表里包含了两个用户的`id` 我们可以去创建一个`Model`
```shell
$ php artisan make:model Follow -m
```

创建完我们的表之后 我们去完善下表的字段信息
```php?start_inline=1
Schema::create('follows', function (Blueprint $table) {
    $table->increments('id');
    $table->integer('follower_id')->unsigned()->index();
    $table->integer('followed_id')->unsigned()->index();
    $table->timestamps();
});
```
定义完毕之后去迁移下数据表
```shell
$ php artisan migrate
```

### 定义模型方法
写完我们的数据表 我们是将关注的信息存放在`follows`这个数据表的 因为这是用户与用户之间的关联 
并不是之前的用户与帖子或文章这样的模型关联 其实实现的道理是一样的 
```php?start_inline=1
//用户关注
public function followers()
{
    return $this->belongsToMany(self::class,'follows','follower_id','followed_id')->withTimestamps();
}

//用户的粉丝
public function following()
{
    return $this->belongsToMany(self::class,'follows','followed_id','follower_id')->withTimestamps();
}

//关注用户
public function followThisUser($user)
{
    return $this->followers()->toggle($user);
}
```
因为用户与用户之间也是一种**多对多**的关系 所以我将关注用户的方法写成`followThisUser`

### 定义方法路由
接下来就可以去定义相应的方法路由了 这里为了使用方便我定义了一个控制器
```shell
$ php artisan make:controller FollowController
```

首先我们去定义一下我们的路由
```php?start_inline=1
Route::post('user/follow','FollowersController@follow');
```

如果用户去关注另一个用户的话 只需要去执行`follow`方法 而这个方法也是一个`toggle`式的操作

> 其实整个实现起来就和我们对一篇帖子进行点赞一样 只不过对象变成了用户和一篇帖子

当然如果我们需要拿到一个用户的关注的人和粉丝的话 可以去执行
```php?start_inline=1
$user->followers 
```
以及
```php?start_inline=1
$user->following
```
这样的话我们就可以拿到对应的用户数据信息了