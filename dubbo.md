# DUBBO

tags： 网络拓扑 TCP连接 设计模式

---

## 1.服务的发布与调用  
  
### 网络拓扑：
![topo](https://raw.githubusercontent.com/beifeng2git/knowledge/master/img/topo.PNG)

### zookeeper保存consumers与providers：
![zk](https://raw.githubusercontent.com/beifeng2git/knowledge/master/img/zk.PNG)

### cloudcash-soa001的引用与负载均衡：
``` xml
<dubbo:reference interface="shop.service.IShopClientService" id="shopClientService" version="1.0.0"/>
<dubbo:reference interface="shop.service.IShopBindService" id="shopBindService" version="1.0.0"/>
```

``` graphLR
A(IShopClientService) --> B{loadbalance}
B --> C[shop-soa001]
B --> D[shop-soa002]

A2(IShopBindService) --> B2{loadbalance}
B2 --> C2[shop-soa001]
B2 --> D2[shop-soa002]
```

## 2.单一长连接中的并发调用

### TCP连接数：
![zk](https://raw.githubusercontent.com/beifeng2git/knowledge/master/img/tcp.PNG)

### 每机器每连接/每服务每连接：
![zk](https://raw.githubusercontent.com/beifeng2git/knowledge/master/img/channel.PNG)

### 服务的并发调用：
``` graphLR
A((Thread1)) --> B(shopClientService.queryByCode)
B --> C(future2313 = channel1.sendMsg)
C --> D(future2313.wait)

A2((Thread2)) --> B2(shopClientService.queryByCode)
B2 --> C2(future5622 = channel1.sendMsg)
C2 --> D2(future5622.wait)
```
### 消息的并发处理：  
``` graphLR
A("channel1.received(msg)") --> B("threadpool.execute(msg)")
B --> C("request2313 = decode(msg)")
C --> D("response2313 = reply(request2313)")
```

### DefaultFuture.java代码分析：
![zk](https://raw.githubusercontent.com/beifeng2git/knowledge/master/img/channel2.PNG)
``` java
public Object get(int timeout) throws RemotingException {
        if (timeout <= 0) {
            timeout = Constants.DEFAULT_TIMEOUT;
        }
        if (!isDone()) {
            long start = System.currentTimeMillis();
            lock.lock();
            try {
                while (!isDone()) {
                    done.await(timeout, TimeUnit.MILLISECONDS);
                    if (isDone() || System.currentTimeMillis() - start > timeout) {
                        break;
                    }
                }
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                lock.unlock();
            }
            if (!isDone()) {
                throw new TimeoutException(sent > 0, channel, getTimeoutMessage(false));
            }
        }
        return returnFromResponse();
    }
    
    public static void received(Channel channel, Response response) {
        try {
            DefaultFuture future = FUTURES.remove(response.getId());
            future.doReceived(response);
        } finally {
            CHANNELS.remove(response.getId());
        }
    }
    
    private void doReceived(Response res) {
        lock.lock();
        try {
            response = res;
            if (done != null) {
                done.signal();
            }
        } finally {
            lock.unlock();
        }
        if (callback != null) {
            invokeCallback(callback);
        }
    }
```

## 讨论与思考
#### 1. 责任链模式，装饰器模式，静态/动态代理模式在（dubbo/servlet/netty/mybatis）中的应用？
#### 2. servlet: void doGet(request, response) 为什么不写成 Resonse doGet(request)? 
#### 3. 体会dubbo接口设计: Result invoke(invocation)

---

# 赋诗一首

## 短歌行 
#### `曹操`
### 对酒当歌，人生几何！
### 譬如朝露，去日苦多。
### 慨当以慷，忧思难忘。
### 何以解忧？唯有杜康。
### 青青子衿，悠悠我心。
### 但为君故，沉吟至今。
### 呦呦鹿鸣，食野之苹。
### 我有嘉宾，鼓瑟吹笙。
### 明明如月，何时可掇？
### 忧从中来，不可断绝。
### 越陌度阡，枉用相存。
### 契阔谈?，心念旧恩。
### 月明星稀，乌鹊南飞。
### 绕树三匝，何枝可依？
### 山不厌高，海不厌深。
### 周公吐哺，天下归心。
  
## 译文：
一边喝酒一边高歌，人生短促日月如梭。好比晨露转瞬即逝，失去的时日实在太多！  
席上歌声激昂慷慨，忧郁长久填满心窝。靠什么来排解忧闷？唯有狂饮方可解脱。  
那穿着青领（周代学士的服装）的学子哟，你们令我朝夕思慕。只是因为您的缘故，让我沉痛吟诵至今。  
阳光下鹿群呦呦欢鸣，悠然自得啃食在绿坡。一旦四方贤才光临舍下，我将奏瑟吹笙宴请嘉宾。  
当空悬挂的皓月哟，什么时候才可以拾到；我久蓄于怀的忧愤哟，突然喷涌而出汇成长河。  
远方宾客踏着田间小路，一个个屈驾前来探望我。彼此久别重逢谈心宴饮，争着将往日的情谊诉说。  
月光明亮星光稀疏，一群寻巢乌鹊向南飞去。绕树飞了三周却没敛翅，哪里才有它们栖身之所？  
高山不辞土石才见巍峨，大海不弃涓流才见壮阔。我愿如周公一般礼贤下士，愿天下的英杰真心归顺与我。  

![binglang](https://raw.githubusercontent.com/beifeng2git/knowledge/master/img/binglang.PNG)













