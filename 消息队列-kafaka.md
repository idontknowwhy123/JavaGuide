1 为什么要使用消息队列?<br>

回答:这个问题,咱只答三个最主要的应用场景(不可否认还有其他的，但是只答三个主要的),即以下六个字:<br>

(1)解耦<br>
传统模式:<br>

传统模式的缺点：系统间耦合性太强，如上图所示，系统A在代码中直接调用系统B和系统C的代码，如果将来D系统接入，系统A还需要修改代码，过于麻烦！<br>

中间件模式:<br>
中间件模式的的优点：<br>

将消息写入消息队列，需要消息的系统自己从消息队列中订阅，从而系统A不需要做任何修改。<br>

(2)异步<br>
传统模式:<br>
传统模式的缺点：<br>

一些非必要的业务逻辑以同步的方式运行，太耗费时间。<br>

中间件模式:<br>
中间件模式的的优点：<br>

将消息写入消息队列，非必要的业务逻辑以异步的方式运行，加快响应速度<br>

(3)削峰<br>
传统模式<br>
传统模式的缺点：<br>

并发量大的时候，所有的请求直接怼到数据库，造成数据库连接异常<br>

中间件模式:<br>
中间件模式的的优点：<br>

系统A慢慢的按照数据库能处理的并发量，从消息队列中慢慢拉取消息。在生产中，这个短暂的高峰期积压是允许的。<br>

2 使用了消息队列会有什么缺点?<br>

分析:一个使用了MQ的项目，如果连这个问题都没有考虑过，就把MQ引进去了，那就给自己的项目带来了风险。我们引入一个技术，要对这个技术的弊端有充分的认识，
才能做好预防。要记住，不要给公司挖坑！<br>
回答:回答也很容易，从以下两个个角度来答<br>

系统可用性降低:你想啊，本来其他系统只要运行好好的，那你的系统就是正常的。现在你非要加个消息队列进去，那消息队列挂了，你的系统不是呵呵了。
因此，系统可用性降低

系统复杂性增加:要多考虑很多方面的问题，比如一致性问题、如何保证消息不被重复消费，如何保证保证消息可靠传输。因此，需要考虑的东西更多，系统复杂性增大。

但是，我们该用还是要用的。<br>

4.如何保证消息队列是高可用的<br>


如上图所示，一个典型的Kafka集群中包含若干Producer（可以是web前端产生的Page View，或者是服务器日志，系统CPU、Memory等），
若干broker（Kafka支持水平扩展，一般broker数量越多，集群吞吐率越高），若干Consumer Group，以及一个Zookeeper集群。
Kafka通过Zookeeper管理集群配置，选举leader，以及在Consumer Group发生变化时进行rebalance。
Producer使用push模式将消息发布到broker，Consumer使用pull模式从broker订阅并消费消息。

5 如何保证消息不被重复消费？<br>

分析:这个问题其实换一种问法就是，如何保证消息队列的幂等性?这个问题可以认为是消息队列领域的基本问题。换句话来说，是在考察你的设计能力，
这个问题的回答可以根据具体的业务场景来答，没有固定的答案。<br>

回答:先来说一下为什么会造成重复消费?<br>

其实无论是那种消息队列，造成重复消费原因其实都是类似的。正常情况下，消费者在消费消息时候，消费完毕后，会发送一个确认信息给消息队列，
消息队列就知道该消息被消费了，就会将该消息从消息队列中删除。只是不同的消息队列发送的确认信息形式不同,例如RabbitMQ是发送一个ACK确认消息，
RocketMQ是返回一个CONSUME_SUCCESS成功标志，kafka实际上有个offset的概念，简单说一下(如果还不懂，出门找一个kafka入门到精通教程),
就是每一个消息都有一个offset，kafka消费过消息后，需要提交offset，让消息队列知道自己已经消费过了。那造成重复消费的原因?，就是因为网络传输等等故障，
确认信息没有传送到消息队列，导致消息队列不知道自己已经消费过该消息了，再次将该消息分发给其他的消费者。<br>

如何解决?这个问题针对业务场景来答分以下几点<br>

  (1)比如，你拿到这个消息做数据库的insert操作。那就容易了，给这个消息做一个唯一主键，那么就算出现重复消费的情况，就会导致主键冲突，
  避免数据库出现脏数据。<br>
  (2)再比如，你拿到这个消息做redis的set的操作，那就容易了，不用解决，因为你无论set几次结果都是一样的，set操作本来就算幂等操作。<br>
  (3)如果上面两种情况还不行，上大招。准备一个第三方介质,来做消费记录。以redis为例，给消息分配一个全局id，只要消费过该消息，
  将<id,message>以K-V形式写入redis。那消费者开始消费前，先去redis中查询有没消费记录即可。<br>
6 如何保证消费的可靠性传输?<br>

分析:我们在使用消息队列的过程中，应该做到消息不能多消费，也不能少消费。如果无法做到可靠性传输，可能给公司带来千万级别的财产损失。同样的，
如果可靠性传输在使用过程中，没有考虑到，这不是给公司挖坑么，你可以拍拍屁股走了，公司损失的钱，谁承担。还是那句话，认真对待每一个项目，
不要给公司挖坑。<br>

回答:其实这个可靠性传输，每种MQ都要从三个角度来分析:生产者弄丢数据、消息队列弄丢数据、消费者弄丢数据<br>
这里先引一张kafka Replication的数据流向图<br>
Producer在发布消息到某个Partition时，先通过ZooKeeper找到该Partition的Leader，然后无论该Topic的Replication Factor为多少（也即该Partition有多少个Replica），Producer只将该消息发送到该Partition的Leader。Leader会将该消息写入其本地Log。每个Follower都从Leader中pull数据。

针对上述情况，得出如下分析<br>
(1)生产者丢数据<br>
在kafka生产中，基本都有一个leader和多个follwer。follwer会去同步leader的信息。因此，为了避免生产者丢数据，做如下两点配置

第一个配置要在producer端设置acks=all。这个配置保证了，follwer同步完成后，才认为消息发送成功。

在producer端设置retries=MAX，一旦写入失败，这无限重试

(2)消息队列丢数据<br>
针对消息队列丢数据的情况，无外乎就是，数据还没同步，leader就挂了，这时zookpeer会将其他的follwer切换为leader,那数据就丢失了。针对这种情况，应该做两个配置。

replication.factor参数，这个值必须大于1，即要求每个partition必须有至少2个副本

min.insync.replicas参数，这个值必须大于1，这个是要求一个leader至少感知到有至少一个follower还跟自己保持联系

这两个配置加上上面生产者的配置联合起来用，基本可确保kafka不丢数据

(3)消费者丢数据<br>
这种情况一般是自动提交了offset，然后你处理程序过程中挂了。kafka以为你处理好了。再强调一次offset是干嘛的
offset：指的是kafka的topic中的每个消费组消费的下标。简单的来说就是一条消息对应一个offset下标，每次消费数据的时候如果提交offset，那么下次消费就会从提交的offset加一那里开始消费。
比如一个topic中有100条数据，我消费了50条并且提交了，那么此时的kafka服务端记录提交的offset就是49(offset从0开始)，那么下次消费的时候offset就从50开始消费。
解决方案也很简单，改成手动提交即可。

ActiveMQ和RocketMQ
大家自行查阅吧

7 如何保证消息的顺序性？<br>

分析:其实并非所有的公司都有这种业务需求，但是还是对这个问题要有所复习。

回答:针对这个问题，通过某种算法，将需要保持先后顺序的消息放到同一个消息队列中(kafka中就是partition,rabbitMq中就是queue)。
然后只用一个消费者去消费该队列。

有的人会问:那如果为了吞吐量，有多个消费者去消费怎么办？

这个问题，没有固定回答的套路。比如我们有一个微博的操作，发微博、写评论、删除微博，这三个异步操作。如果是这样一个业务场景，那只要重试就行。
比如你一个消费者先执行了写评论的操作，但是这时候，微博都还没发，写评论一定是失败的，等一段时间。等另一个消费者，先执行写评论的操作后，再执行，就可以成功。

总之，针对这个问题，我的观点是保证入队有序就行，出队以后的顺序交给消费者自己去保证，没有固定套路

1.kafka拓扑结构<br>
![](https://img-blog.csdn.net/20180902105920995?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTAwMjAwOTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
如上图所示，一个典型的Kafka体系架构包括若干Producer(可以是服务器日志，业务数据，页面前端产生的page view等等)，若干broker(Kafka支持水平扩展，一般broker数量越多，集群吞吐率越高)，若干Consumer (Group)，以及一个Zookeeper集群。Kafka通过Zookeeper管理集群配置，选举leader，以及在consumer group发生变化时进行rebalance。Producer使用push(推)模式将消息发布到broker，Consumer使用pull(拉)模式从broker订阅并消费消息。<br>

Kafka 和传统的消息系统不同在于：<br>
1.Kafka是一个分布式系统，易于向外扩展。<br>
2.它同时为发布和订阅提供高吞吐量。<br>
3.它支持多订阅者，当失败时能自动平衡消费者。<br>
4.消息的持久化。<br>
![](https://img-blog.csdn.net/20180819105605738?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbmdqaWFuYW43MzU3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
名称	解释
Broker	消息中间件处理节点，一个Kafka节点就是一个Broker，一个或者多个Broker可以组成一个Kafka集群。
Topic	主题，Kafka根据Topic对消息进行归类，发布到Kafka集群的每条消息都需要指定一个Topic。
Producer	消费生产者，向Broker发送消息的客户端。
Consumer	消息消费者，从Broker读取消息的客户端。
Consumer Group	每个Consumer属于一个特定的Consumer Group，一条消息可以发送到多个Consumer Group，但是一个Consumer Group中只能有一个Consumer能够消费该消息。
Partition	物理上的概念，一个Topic可以分为多个Partition，每个Partition内部是有序的。
https://blog.csdn.net/u013256816/article/details/71091774
https://blog.csdn.net/wangjianan7357/article/details/81836063

补充1.多线程消费者
https://blog.csdn.net/clypm/article/details/80618036
2.多线程Consumer方案
https://www.cnblogs.com/qizhelongdeyang/p/7355309.html

-

一.kafka生产者<br>
1.生产者发送消息的步骤：<br>
1.1创建ProducerRecord对象，这个对象封装了主题和数据，也可以包含键以及分区<br>
1.2将键以及值通过序列化器进行序列化，<br>
1.3如果没有指定分区，分区器会根据键采用默认散列算法来选择一个分区，如果没有键则采用轮询算法来选择分区<br>
1.4这条记录将被添加到一个记录批次中，由一个线程将该批次的消息发送到broker中<br>
1.5服务器收到消息时会返回一个响应，如果写入kafka成功，则返回一个RecordMetaData对象（包含主题，分区以及偏移量相关信息）<br>
1.6如果写入失败，则会返回一个错误，生产者在收到错误之后会尝试重新发送消息。<br>

2.kafka生产者有三个必选属性<br>
bootstrap.servers:broker的地址清单，地址格式为host:port<br>

key.serializer:该参数指定的类将会对键进行序列化<br>

value.serializer:该参数指定的类将会对值进行序列化<br>

其他主要属性<br>
acks:至少要多少个分区副本收到了消息，生产者才认为消息写人成功了<br>
acks=0:生产者并不关系broken服务器是否成功响应<br>
acks=1:至少首领节点收到了消息才算成功<br>
acks=all：所有节点都收到了消息才算成功<br>

retries:生产者重试发送消息的次数<br>
retry.backoff.ms参数可以改变重试的时间间隔<br>


3.kafka生成者发送消息的三种方式<br>
发送并忘记：把消息发送给broken服务器，并不关心它是否正常到达。<br>

同步发送：发送一个消息，返回一个future对象，调用get()方法进行等待<br>

异步发送：发送一个消息，并指定一个回调函数，服务器在返回响应时调用该函数<br>

4.kafka消息的顺序性<br>
在kafka生产端，可以直接将需要保证消息顺序的消息的key指定为相同的，这样分区器采用默认算法(散列)将会保证这批消息会分配到同一个分区，<br>
而kafka可以保证同一个分区的消息是有序的。如果重试次数不为0，由于网络问题在发送消息的时候也有可能前面的消息由于重试最后排到后面去了，从而导致顺序反过来了，为了保证
消息顺序的严格一致性，我们可以把max.in.flight.requests.per.connection设为1，这样就保证生产者尝试发送第一批消息时，就不会有其它的消息发送给broker，不过这样会
严重影响生产者的吞吐量<br>

5.分区<br>
如果消息的key为null，则采用默认的轮询算法消息发送到各个分区上<br>
如果消息的key不为null，则对key采用散列的算法将消息散列到各个分区上<br>
如果想自定义分区，则可以实现partitioner接口重写partition方法实现自定义分区规则<br>
properties.setProperty(ProducerConfig.PARTITIONER_CLASS_CONFIG, MyPartitioner.class.getName());<br>

二、kafka消费者<br>
1.分区和消费者的数量关系<br>
一个分区只能被一个消费者消费，否则无法保证消息的顺序性<br>
0<消费者数量<分区数量<br>
消费者横向伸缩的最大数量就是一个分区对应一个消费者<br>

2.分区再均衡<br>
再均衡：分区的所有权从一个消费者转移到另一个消费者。（有消费者加入或退出消费者群组时）<br>
在再均衡期间，消费者无法读取消息，将会造成整个群组一小段时间的不可用<br>

消费者会向群组协调器的broker发送心跳来维持他们和群组的从属关系，消费者会在轮询以及提交偏移量时向群组协调器发送心跳，如果发送心跳的时间间隔过长，<br>
群组协调器将认为该消费者已经死亡，将触发一次再均衡<br>

3.分区分配的过程<br>
消费者加入群组时，会向群组协调器broker发出一个joinGroup的请求，第一个加入群组的消费者将会被指定为“群主”，群主从群组协调器那边获取到分区列表，然后按照<br>
指定算法将分区分配给各个消费者，分配完后将会把分配策略返回给群组协调器，群组协调器再将这些信息分配给消费者，这样消费者就知道自己管理的分区信息了。<br>

4.轮询<br>
4.1长期运行的应用程序，他通过长期轮询向kafka请求数据<br>
4.2消费者必须对kafka进行轮询，否则会被认为已经死亡<br>
4.3在退出应用程序之前使用close关闭消费者，网络连接和socket也会关闭，并立即触发一次再均衡<br>

5.消费者线程安全<br>
  1.1个消费者对应一个线程去消费消息<br>
  2.一个消费者多线程消费，为了保证消费信息的顺序，一个线程对应只能消费一个分区的消息<br>
  
 6.kafka消费者三个必选属性（和生产者一样）
 bootstrap.servers:broker的地址清单，地址格式为host:port<br>

 key.serializer:该参数指定的类将会对键进行序列化<br>

 value.serializer:该参数指定的类将会对值进行序列化<br>
 
 其他重要属性
 session.timeout.ms:消费者在指定为session.timeout.ms时间内（默认是3S）没有发送心跳给群组协调器，则被认定为已经死亡，协调器就会触发再均衡
 enable.auto.commit:是否自动提交偏移量，默认为true，为了避免重复消费以及数据丢失，可以将其配置为false,
 partition.assignment:分区分配策略
 1.Range
 1.将主题若干个连续的分区分配给消费者
 2.RoundRobin
 2.将主题的所有分区逐个的分配给消费者

7.提交和偏移量
  提交偏移量是为了保证在触发再均衡时，消费者能够从新分配的分区当中正确的偏移量位置开始读取数据
  
  7.1可能出现的重复消费和数据丢失的问题：
  
     1.先提交偏移量，再消费数据（提交的偏移量大于客户端最后处理的一个消息的偏移量）
     在消费数据时一旦出现再均衡的情况，将会出现数据丢失
     
     解决办法：改为手动提交方式，并且在每次数据消费完之后再去手动提交偏移量
  
     2.先消费数据，再提交偏移量，
     在消费数据或提交偏移量时发生了再均衡的情况，将会出现数据的重复消费
     
     解决办法：1.改为手动提交偏移量
              2.保证消息消费的幂等性，以主题+分区+偏移量作为唯一标识，对消息进行缓存，每消费一个消息都去缓存中去判断消息是否已经消费过了，消费过了则不再消费。
  
  
  7.2提交方式
     1.自动提交：按照指定时间间隔提交偏移量（enable.auto.commit设置为true）
     会出现重复消费消息的情况
     比如是5s提交一次偏移量，在上一次提交完偏移量后的3s发生了再均衡，这时这3s的偏移量就没有提交上去，所以在再均衡后这3s的数据会被重复处理
     
     2.手动提交（enable.auto.commit设置为false）
       2.1同步手动提交：commitSync(),优点：提交失败会有重试机制,缺点：应用程序提交完偏移量需要等待broker的响应
       2.2异步手动提交：commitAsync()，优点：应用程序不需要等待broker的响应，缺点：没有重试提交机制，因为重试可能会导致前一批偏移量的提交把后一批的覆盖掉了
       2.3同步和异步组合手动提交：一般情况下，针对于偶尔出现的提交失败，不进行重试不会什么问题，因为如果是临时问题，后面的提交总会有成功的。但是如果是在关闭消费者或者说是在
       再均衡前的最后一次提交，就要确保能够提交成功。
         所以我们可以采取在正常情况下就采取commitSync进行同步提交，在消费者即将关闭退出时采用commitASync进行同步提交，保证再均衡之前的最后一次的提交总是成功的
          
       
      3.再均衡监听器
       onPartitionRevoked()会在再均衡之前消费者停止读取消息之后调用，在这个方法提交偏移量，下一个接管分区的消费者就知道从哪里开始读取消息了
       提交某个特定的偏移量，而不是批次中最后一个消息的偏移量：消费完一个消息就缓存一个偏移量，再均衡之前就提交这个缓存的偏移量就行
       
       OnPartitionAssigned()会在重新分配分区消费者开始消费消息时调用
       在数据保存在数据库之后，提交偏移量之前出现消费者异常退出，会导致数据库出现重复数据，
       可以将数据库数据和偏移量都写到数据库中，在再均衡之后开始消息数据之前调用consumer.seek（）方法重新定位到知道偏移量然后开始消费
      
    7.3如何退出
       在另外一个线程里面调用consumer.wakeup
       
   三、kafka服务器
      kafka使用zookeeper的临时节点来选举控制器，并在节点加入集群或者退出集群时通知控制器，控制器负责在节点加入集群或离开集群时进行分区首领选举。
      
      3.1复制
      复制是kafka架构的核心，因为它可以保证个别节点失效的时仍能保证kafka的可用性和持久性
      每个主题有很多分区，每个分区又有很多个副本，副本就被保存在broker上。
      
      首领副本：负责处理生产者和消费者请求
      跟随者副本：不处理客户端请求，他们唯一的任务就是从首领副本那边复制消息，保持与首领副本一致的状态，如果某个首领挂掉了，其中一个跟随者副本会被提升为新首领。
      
      如果跟随者副本在10s内没有请求任何消息，或者没有请求到最新的数据，那么它就会被认为是不同步的，那么在首领失效时，它就不可能成为新首领。
      
      3.2客户端如何知道往哪里发送请求
      客户端会定时向服务器发送元数据请求获取并缓存主题和分区首领等信息，然后向目标broker上发送请求信息。
      
   四、可靠的数据传递
      
      4.1kafka本身的可靠性
      4.1.1kafka本身的分区多副本复制机制是保证kafka可靠性的核心，只有当消息数据同步到所有的副本，消费者才可以这条消息
      分区是否是同步副本需要满足：
         1.与zookeeper保持一个活跃的会话(通过心跳机制保证)
         2.在过去10s内从首领那里获取过消息，并且消息是最新的
      
      4.1.2不完全的首领选举
       如果把unclean.leader.election.enable设为true，就是允许不同步的副本成为首领，这样就会面临丢失消息的风险，如果设置为true，就要等待原先的首领重新上线，
       这样就降低了系统的可用性
       
      4.2可靠的生产者
        4.2.1选择acks=all的确认模式，保证生成者必须等待所有的副本都收到消息
        acks=0，生产者不需要等待broker的响应
        acks=1，生产者只需要等待首领节点的响应
            
        4.2.2重试机制保证消息的可靠发送
        
      4.3可靠的消费者
         偏移量保证了消费者总是从正确的位置读取消息，
         kafka消息消费的重复消费和消息数据丢失的处理方法：见7.1
     
   五、kafka性能调优实战
   
       5.1问题,kafka消费消息太慢
          解决策略：采用单个消费者多线程消费模式
               1.消费者接受到消息以后将消息根据分区放到指定的消费线程队列去消费（一个分区消息对应一个消费线程的目的是为了保证消息消费的顺序性）
               2.有一个offset缓存去缓存分区和offsets的对应关系，每当每个消费线程消费完对应分区的消息时都会更新这个缓存，
               3.更新完缓存就会提交偏移量
               4.消息消费时需要通过以主题+分区+偏移量为key，以消息为value进行缓存，然后每次消费时判断消息是否存在，存在则不消费，不存在则缓存，还需要有缓存过期策略
               5.通过再均衡监听器保证消息不会被重复消费。
