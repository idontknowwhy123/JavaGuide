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

-
