# rabbitMQ-persistence

### 单一模式持久化

#### 1.创建rabbitmq节点.

```
oc run rabbit --image=registry.dataos.io/wfw2046/rabbitmq:3-management --env RABBITMQ_ERLANG_COOKIE='!@KLHSDAHSUDASDLAJSD' --env RABBITMQ_DEFAULT_USER=admin --env RABBITMQ_DEFAULT_PASS=<password> --env HOSTNAME=testrabbit

参数说明:

  RABBITMQ_ERLANG_COOKIE  //集群id
  
  RABBITMQ_DEFAULT_USER  //指定一个默认用户
  
  RABBITMQ_DEFAULT_PASS  //默认用户的密码
  
  HOSTNAME               //必须写，指定存储的路径，如果不写默认路径是pod的名字，因为pod的名字删除后会变，所以存储的路径也会变，就会访问不到之前之久化的东西
```
#### 2.创建持久化卷(略)

#### 3.挂载持久化卷

```
oc eidt dc <dcName>  修改dc

在dc文件中以下位置，添加内容：

spec:
 ...
 template:
  ...
  spec:
   containers:
   - env:
     ...
     terminationMessagePath: /dev/termination-log
     //添加以下三行
     volumeMounts:
        - mountPath: /var/lib/rabbitmq     //固定路径不能更改，否则影响持久化 
          name: rabbittest                 //名字随意，与下面volumes中name保持一致即可
   ...
   terminationGracePeriodSeconds: 30
   //添加以下4行
   volumes:
      - name: rabbittest
        persistentVolumeClaim:
          claimName: rabbit                //持久化卷的名字
          
```

#### 4.创建svc

需要开放5672、15672、25672三个端口的svc 

```
oc expose dc <dcName> --name rabbit --port 5672
oc expose dc <dcName> --name rabbit1 --port 15672
oc expose dc <dcName> --name rabbit2 --port 15672
```
#### 5.获得管理界面，expose route
开放15672服务的链接
```
oc expose svc <svcName>
oc get route
```

通过页面访问连接即可                                  
![image](https://github.com/asiainfoLDP/rabbitMQ-persistence/blob/master/20161024132106.png)


### 测试(在集群内测试)
* 通过页面创建一个test用户
![image](https://github.com/asiainfoLDP/rabbitMQ-persistence/blob/master/20161024132339.png)

* 修改t1中的redentials,connection 为创建的用户,5672的svc的地址

* 执行t1脚本发送消息

* 删除pod

* 等pod重新创建后再执行t2，接收到如下消息即可。 

![image](https://github.com/asiainfoLDP/rabbitMQ-persistence/blob/master/20161024134133.png)

