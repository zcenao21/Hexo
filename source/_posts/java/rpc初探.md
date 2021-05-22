---
title: RPC初探
date: 2021-01-09 21:03:50
tags:
    - Java
    - RPC
categories: 
    - Java
---



# RPC

RPC(Remote Procedure Call)，即远程过程调用，主要用于分布式情景下节点间的调用，可以让远程调用使用起来就像本地调用一样方便。<!--more-->

最简单RPC实现包括如下内容

- 调用者发起调用
- 封装调用，打包方法名和参数，序列化
- 被调用端反序列化，得到方法名参数，使用反射并执行得到结果
- 将结果序列化返回给调用者
- 调用者反序列化得到结果

![rpc](https://github.com/zcenao21/photos-blog/blob/main/rpc-start/rpc.png?raw=true)

# 简单实现

为了更好理解RPC，参考知乎作者分享（见文末）的内容，自己敲了一遍（已上传[github](https://github.com/zcenao21/rpc-learn)）

首先是远程服务部分：

```
public class ProviderApp {
    private Calculator calculator = new CalculatorImpl();
    static final int PORT=9090;

    public static void main(String[] args) throws IOException {
        new ProviderApp().run();
    }

    private void run() throws IOException {
        ServerSocket listener = new ServerSocket(PORT);
        try {
            while (true) {
                log.info("rpc method waiting for call!");
                Socket socket = listener.accept();
                try {
                    // 将请求反序列化
                    ObjectInputStream objectInputStream = new ObjectInputStream(socket.getInputStream());
                    Object object = objectInputStream.readObject();

                    log.info("request is {}", object);

                    // 调用服务
                    int result = 0;
                    if (object instanceof CalculateRpcRequest) {
                        CalculateRpcRequest calculateRpcRequest = (CalculateRpcRequest) object;
                        if ("add".equals(calculateRpcRequest.getMethod())) {
                            log.info("add service called!");
                            result = calculator.add(calculateRpcRequest.getA(), calculateRpcRequest.getB());
                        } else {
                            throw new UnsupportedOperationException();
                        }
                    }

                    // 返回结果
                    ObjectOutputStream objectOutputStream = new ObjectOutputStream(socket.getOutputStream());
                    objectOutputStream.writeObject(new Integer(result));
                    log.info("rpc method call finished!");
                } catch (Exception e) {
                    log.error("fail", e);
                } finally {
                    socket.close();
                }
            }
        } finally {
            listener.close();
        }
    }
}
```

其中计算器接口：

```
public interface Calculator {
    public int add(int a, int b);
}
```

计算器加法实现：

```
public class CalculatorImpl implements Calculator{
    public int add(int a, int b) {
     return a+b;
    }
}
```

封装方法和参数的类：

```
public class CalculateRpcRequest implements Serializable {
    int a;
    int b;

    public CalculateRpcRequest(int a,int b){
        this.a=a;
        this.b=b;
    }
    public String getMethod(){
        return "add";
    }
    public int getA() {
        return a;
    }

    public int getB() {
        return b;
    }
}
```

另外一个是调用端：

```
public class ConsumerApp {
    public static void main(String[] args) {
        Calculator calculator = new CalculatorRemoteImpl();
        int result = calculator.add(6, 2);
        log.info("get sum:"+result);
    }
}
```

其中封装方法和参数以及接收返回结果的过程如下：

```
public class CalculatorRemoteImpl implements Calculator {
    static final int PORT=9090;

    public int add(int a, int b) {
        String address = "127.0.0.1";
        try {
            log.info("now call rpc method!");
            Socket socket = new Socket(address, PORT);

            // 将请求序列化
            CalculateRpcRequest calculateRpcRequest = generateRequest(a, b);
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(socket.getOutputStream());

            // 将请求发给服务提供方
            objectOutputStream.writeObject(calculateRpcRequest);
            log.info("send request to service provider! then wait for result!");

            // 将响应体反序列化
            ObjectInputStream objectInputStream = new ObjectInputStream(socket.getInputStream());
            Object response = objectInputStream.readObject();
            log.info("get result from service provider! response:{}",response);

            if (response instanceof Integer) {
                return (Integer) response;
            } else {
                throw new InternalError();
            }

        } catch (Exception e) {
            log.error("fail", e);
            throw new InternalError();
        }
    }

    private CalculateRpcRequest generateRequest(int a, int b) {
        return new CalculateRpcRequest(a,b);
    }
}
```



在idea首先运行Provider，然后再运行Consumer。

Provider端日志：

```
[INFO ] 2021-01-09 21:00:43 rpc method waiting for call!
[INFO ] 2021-01-09 21:00:53 request is com.will.rpc.CalculateRpcRequest@50134894
[INFO ] 2021-01-09 21:00:53 add service called!
[INFO ] 2021-01-09 21:00:53 rpc method call finished!
[INFO ] 2021-01-09 21:00:53 rpc method waiting for call!
```

Consumer端日志：

```
[INFO ] 2021-01-09 21:00:53 now call rpc method!
[INFO ] 2021-01-09 21:00:53 send request to service provider! then wait for result!
[INFO ] 2021-01-09 21:00:53 get result from service provider! response:8
[INFO ] 2021-01-09 21:00:53 get sum:8
```



> 本文主要参考：https://zhuanlan.zhihu.com/p/36528189

