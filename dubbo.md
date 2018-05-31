# DUBBO

tags�� �������� TCP���� ���ģʽ

---

## 1.����ķ��������  
  
### �������ˣ�
![topo](https://raw.githubusercontent.com/beifeng2git/knowledge/master/img/topo.PNG)

### zookeeper����consumers��providers��
![zk](https://raw.githubusercontent.com/beifeng2git/knowledge/master/img/zk.PNG)

### cloudcash-soa001�������븺�ؾ��⣺
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

## 2.��һ�������еĲ�������

### TCP��������
![zk](https://raw.githubusercontent.com/beifeng2git/knowledge/master/img/tcp.PNG)

### ÿ����ÿ����/ÿ����ÿ���ӣ�
![zk](https://raw.githubusercontent.com/beifeng2git/knowledge/master/img/channel.PNG)

### ����Ĳ������ã�
``` graphLR
A((Thread1)) --> B(shopClientService.queryByCode)
B --> C(future2313 = channel1.sendMsg)
C --> D(future2313.wait)

A2((Thread2)) --> B2(shopClientService.queryByCode)
B2 --> C2(future5622 = channel1.sendMsg)
C2 --> D2(future5622.wait)
```
### ��Ϣ�Ĳ�������  
``` graphLR
A("channel1.received(msg)") --> B("threadpool.execute(msg)")
B --> C("request2313 = decode(msg)")
C --> D("response2313 = reply(request2313)")
```

### DefaultFuture.java���������
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

## ������˼��
#### 1. ������ģʽ��װ����ģʽ����̬/��̬����ģʽ�ڣ�dubbo/servlet/netty/mybatis���е�Ӧ�ã�
#### 2. servlet: void doGet(request, response) Ϊʲô��д�� Resonse doGet(request)? 
#### 3. ���dubbo�ӿ����: Result invoke(invocation)

---

# ��ʫһ��

## �̸��� 
#### `�ܲ�`
### �ԾƵ��裬�������Σ�
### Ʃ�糯¶��ȥ�տ�ࡣ
### �����Կ�����˼������
### ���Խ��ǣ�Ψ�жſ���
### �������ƣ��������ġ�
### ��Ϊ���ʣ���������
### ����¹����ʳҰ֮ƻ��
### ���мα�����ɪ���ϡ�
### �������£���ʱ�ɶޣ�
### �Ǵ����������ɶϾ���
### Խİ���䣬������档
### ����̸?������ɶ���
### ������ϡ����ȵ�Ϸɡ�
### �������ѣ���֦������
### ɽ����ߣ��������
### �ܹ��²������¹��ġ�
  
## ���ģ�
һ�ߺȾ�һ�߸߸裬�����̴��������󡣺ñȳ�¶ת˲���ţ�ʧȥ��ʱ��ʵ��̫�࣡  
ϯ�ϸ����������������������������ѡ���ʲô���Ž����ƣ�Ψ�п������ɽ��ѡ�  
�Ǵ������죨�ܴ�ѧʿ�ķ�װ����ѧ��Ӵ���������ҳ�Ϧ˼Ľ��ֻ����Ϊ����Ե�ʣ����ҳ�ʹ��������  
������¹Ⱥ���ϻ�������Ȼ�Եÿ�ʳ�����¡�һ���ķ��ͲŹ������£��ҽ���ɪ��������α���  
�������ҵ����Ӵ��ʲôʱ��ſ���ʰ�����Ҿ����ڻ����Ƿ�Ӵ��ͻȻ��ӿ������ɳ��ӡ�  
Զ������̤�����С·��һ��������ǰ��̽���ҡ��˴˾ñ��ط�̸�����������Ž����յ�������˵��  
�¹������ǹ�ϡ�裬һȺѰ����ȵ���Ϸ�ȥ��������������ȴû���ᣬ���������������֮����  
��ɽ������ʯ�ż�Ρ�룬�󺣲�������ż�׳������Ը���ܹ�һ��������ʿ��Ը���µ�Ӣ�����Ĺ�˳���ҡ�  

![binglang](https://raw.githubusercontent.com/beifeng2git/knowledge/master/img/binglang.PNG)













