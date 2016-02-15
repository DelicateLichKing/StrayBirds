---
layout: post
title: 使用SurfaceView
category: Android
comments: true
---

##介绍
需要另外线程绘制界面、迅速更新界面和渲染 UI 就要使用到 SurfaceView , surfaceView 是在一个新起的单独线程中可以重新绘制画面，而 View 必须在 UI 的主线程中更新画面。那么在 UI 的主线程中更新画面可能会引发问题，比如你更新画面的时间过长，那么你的主 UI 线程会被你正在画的函数阻塞。那么将无法响应按键，触屏等消息。当使用 surfaceView 由于是在新的
线程中更新画面所以不会阻塞你的 UI 主线程。

##使用

### 继承,重写三个必要的方法

```java
    public class MainActivity extends AppCompatActivity implements SurfaceHolder.Callback{
        
    @Override
    public void surfaceCreated(SurfaceHolder holder) {

    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {

    }
    }
```

* `Create`

```java
    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        new Thread(new drawThread()).start();
        Log.i("sur", "surface is created" + m_iPort);
    }
    ...
     class drawThread implements Runnable {
         @Override
         public void run() {
             holder = mSurfaceView.getHolder();
             holder.setFormat(PixelFormat.TRANSLUCENT);//半透明
             if (-1 == m_iPort)
             {
                 return;
             }
             Surface surface = holder.getSurface();
             if (surface.isValid()) {
                 if (!Player.getInstance().setVideoWindow(m_iPort, 0, holder)) {
                     Log.e("sur", "Player setVideoWindow failed!");
                 }
             }
         }
     }    
```

访问 SurfaceView 的底层图形是通过 SurfaceHolder 接口来实现的，通过 `getHolder()` 方法可以得到这个 SurfaceHolder 对象。通常用一个线程来绘制画布。
#### `GetInstance` 与 `new`
对象的实例化方法，也是比较多的，最常用的方法是直接使用 new ，而这是最普通的，如果要考虑到其它的需要，如单实例模式，层次间调用等等。直接使用 new 就不可以实现好的设计，这时候需要使用间接使用 new ，即很多人使用的 GetInstance 方法。这是一个设计方式的代表，而不仅仅指代一个方法名。
 
1. `new` 的使用:
如 `Object_object=new Object()` ，这时候，就必须要知道有第二个 Object 的存在，而第二个 Object 也常常是在当前的应用程序域中的，可以被直接调用的.
 
2. `GetInstance` 的使用:
在主函数开始时调用，返回一个实例化对象，此对象是 `static` 的，在内存中保留着它的引用，即内存中有一块区域专门用来存放静态方法和变量，可以直接使用，调用多次返回同一个对象。
 
3. 两者区别对照:
大部分类 (非抽象类/接口/屏蔽了 constructor 的类) 都可以用 new ，new 就是通过生产一个新的实例对象，或者在栈上声明一个对象，每部分的调用用的都是一个新的对象。
getInstance 是少部分类才有的一个方法，各自的实现也不同。 getInstance 在单例模式(保证一个类仅有一个实例，并提供一个访问它的全局访问点)的类中常见，用来生成唯一的实例， getInstance 往往是 static 的。
 
(1)对象使用之前通过 getinstance 得到而不需要自己定义，用完之后不需要 delete ；
 
(2) new 一定要生成一个新对象，分配内存； getInstance() 则不一定要再次创建，它可以把一个已存在的引用给你使用，这在效能上优于 new ；
 
(3) new 创建后只能当次使用，而 getInstance() 可以跨栈区域使用，或者远程跨区域使用。所以 getInstance() 通常是创建 `static` 静态实例方法的。

注：出处 http://blog.sina.com.cn/s/blog_6a73f3270102v7ll.html

* `Destroyed`

```java
    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        Log.i("sur", "Player setVideoWindow release!" + m_iPort);
        if (-1 == m_iPort)
        {
            return;
        }
        if (holder.getSurface().isValid()) {
            if (!Player.getInstance().setVideoWindow(m_iPort, 0, null)) {
                Log.e("sur", "Player setVideoWindow failed!");
            }
        }
    }
```
### 初始化 `GUI`

```java
    private boolean initeActivity() {
        findViews();
        mSurfaceView.getHolder().addCallback(this);
        setListeners();
        return true;
    }
```
 surfaceView 是用来显示 surface 所绘制的图像，你可以通过 SurfaceHolder 来访问这个 surface ，而 SurfaceView.getHolder() 方法就是用来返回 SurfaceHolder 对象以便访问 surface ,而 `holder.addCallback(this)` 是因为当前 Activity 实现了一个接口 `SurfaceHolder.Callback` ，所以 this 也是 Callback 的一个对象，在 surface 的各个生命周期 (create change destroy) 中会调用重写的那三个方法。 正因为调用了 `holder.addCallback(this)` ，就将当前重写的三个方法与 surface 关联起来了。
 
