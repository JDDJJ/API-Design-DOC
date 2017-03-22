# Token交互方案

在token设计方案中，token是一个用户的标识，令牌。是让服务端知道用户是该用户的一种手段。token是app安装后第一网络请求 getToken（）接口得到并存储在App本地磁盘的，不论是以后登录还是注册，只要token不失效，token是从一而终的。App端不需要知道token的生成机制，也不需要知道是怎么挂勾用户信息的。从而保证了用户信息的安全和接口访问的安全。避免了通过用户id等方式请求接口，若被人反编译破解接口信息，则可以通过替换id来进行访问接口的弊端。
##Header中的key-Value命名
> X-Auth-Token

X：非标准参数的标识 Auth：表示完成的动作 Token：表示该参数是什么 例如[X-Request-Id](https://blog.heroku.com/http_request_id_s_improve_visibility_across_the_application_stack) 首字母大写
#App端Token方案

##Token存储方案 

- Token应该存储到磁盘空间 getToken接口调用时 会更新并存储Token
- Application 开启 被分配存储空间后 需先在Application初始化时声明一个静态全局变量去读取接受本地存储的token 以避免每次需要token都需要读取本地

![](/assets/417541BA-8D08-48D8-BE2D-CF4A68BCBEBD.png)

##Token更新方案
###同步锁概念

![](/assets/14F63AA2-741F-4B29-AE0E-00F390E354BF.png)

多个线程访问同一个资源 会出现取同一张票 

```
package com.pakage.ThreadAndRunnable;  
  
public class Runnable_demo implements Runnable{  
    private int ticket=10;  
    public Runnable_demo(){       
    }  
    @Override  
    public void run() {  
        for(int i=0;i<20;i++){  
                if(this.ticket>0){  
                    //休眠1s秒中，为了使效果更明显，否则可能出不了效果  
                    try {  
                        Thread.sleep(1000);  
                    } catch (Exception e) {  
                        e.printStackTrace();  
                    }  
                    System.out.println(Thread.currentThread().getName()+"号窗口卖出："+this.ticket--+"号票");  
                }  
              
        }  
    }  
      
     public static void main(String args[]){  
         Runnable_demo demo=new Runnable_demo();  
         //基于火车票创建三个窗口  
         new Thread(demo,"a").start();  
         new Thread(demo,"b").start();  
         new Thread(demo,"c").start();  
     }  
      
}  
```

![](/assets/20131012151327046.jpeg)

这种情况我们就需要引入锁的概念 当一个线程处理完 下一个线程才可以进行访问 在java中 实现这个锁的功能就需要用到synchronized这个关键字。

###Token 失效

> Token失效 则需要更新Token 更新时 若更新之前访问大量接口 则会多次更新如何解决 ？

这里我们就需要讲更新token的代码块锁住 以防多线程同时更新。并加入一个token更新完成的时间戳 

在其他线程访问这个代码块时 会先判断当前访问时间距离更新时间多久 比如如果小于1s 说明


![](/assets/34C8C2D8-1769-40C7-B440-1DC4E003CAF1.png)

- 第一打开app或者升级之后进行getToken的调用时 需要判断token是否存在 否则如果存在， 直接获取token 会导致登录的用户token失效
- token失效 需要自动更新token 但不需要自动登录 需要提醒用户长时间不开启，需要自己手动再次登录









