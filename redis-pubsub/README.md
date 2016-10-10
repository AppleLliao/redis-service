# redies
���Spring �� ����/����ϵͳ

ʹ��Spring Data Redis����Redis

1. Redis��Pub/Sub����

Redis�Ķ��ĺͷ�������������ͼ6���������ֱ��ÿ����������˵����

pubsub.png
publish: ��ָ����channel(Ƶ��)����message(��Ϣ)

subscribe:����ָ��channel������һ�ζ��Ķ��

psubscribe:����ָ��pattern(ģʽ������Ƶ������ģʽƥ��)��Ƶ��

unsubscribe:ȡ������channel,����һ��ȡ���������

punsubscribe:ȡ��ָ��pattern�Ķ���

pubsub: ��һ���鿴�����뷢��ϵͳ״̬����ʡ����,����������ͬ��ʽ�����������(����ɲμ���

http://redis.io/commands/pubsub)

��SDR��Spring Data Redis����Ƶ����ӦTopic�࣬Top����һ���ӿ���Channel��Pattern����ʵ���࣬�ֱ���ָ�����Ƶ�Ƶ����ģʽƥ���Ƶ�������ڶ�����Ϣ��Subscription�ӿڶ��塣

2.Redis��Ϣ����������������Ϣ������ע��

��SDR�п��������ַ�ʽ��ʵ����Ϣ����������������һ����ͨ��Redis�������ռ䣬һ���Ƕ���Bean��

������Ҫ�漰��RedisMessageListenerContainer��MessageListenerAdapter��MessageListener�����ࡣ

2.1ʹ��Redis�����ռ�ķ�ʽ����

<bean id="mdpListener" class="secondriver.spring.redis.MyMessageListener" />
  <bean id="mdelegateListener" class="secondriver.spring.redis.DefaultMessageDelegate" />

  <redis:listener-container connection-factory="jedisConnectionFactory">
    <redis:listener ref="mdpListener" topic="spring*" />
    <redis:listener ref="mdelegateListener" method="handleMessage"
      topic="cctv5 cctv6 nbtv hello*" />
  </redis:listener-container>
˵��:

����topic�����Ǿ����channel������Ҳ������Pattern�����Ƶ�������⣩�ÿո�������ɡ�

���ﶨ��������Listner��MyMessageListenerʵ����MessageaListener�ӿ�,DefaultMessageDelegateʵ����MessageDelegate�ӿڡ�

MyMessageListener:

package secondriver.spring.redis;

import org.springframework.data.redis.connection.Message;
import org.springframework.data.redis.connection.MessageListener;

public class MyMessageListener implements MessageListener {

  @Override
  public void onMessage(Message message, byte[] pattern) {
    System.out.println("channel:" + new String(message.getChannel())
        + ",message:" + new String(message.getBody()));
  }
}
MessageDelegate�ӿڣ�

package secondriver.spring.redis;

import java.io.Serializable;
import java.util.Map;

public interface MessageDelegate {

  public void handleMessage(String message);

  public void handleMessage(Map<?, ?> message);

  public void handleMessage(byte[] message);

  public void handleMessage(Serializable message);

  // pass the channel/pattern as well
  public void handleMessage(Serializable message, String channel);
}
DefaultMessageDelegate���ͣ�

package secondriver.spring.redis;

import java.io.Serializable;
import java.util.Map;

public class DefaultMessageDelegate implements MessageDelegate {

  @Override
  public void handleMessage(String message) {
    System.out.println("handleMessage(String message):" + message);
  }

  @Override
  public void handleMessage(Map<?, ?> message) {
    System.out.println("handleMessage(Map<?, ?> message):" + message);
  }

  @Override
  public void handleMessage(byte[] message) {
    System.out.println("handleMessage(byte[] message):"
        + new String(message));
  }

  @Override
  public void handleMessage(Serializable message) {
    System.out.println("handleMessage(Serializable message):"
        + message.toString());
  }

  @Override
  public void handleMessage(Serializable message, String channel) {
    System.out
        .println("handleMessage(Serializable message, String channel):"
            + message.toString() + ", channel:" + channel);
  }
}
���ֶ�����Ϣ�����ķ�ʽ��������Redis���䱻���Ϊһ��message-driven POJOs (MDPs)��MessageListenerAdapterʵ����MessageListener�ӿڣ��������Messageί�и�Ŀ���������Target Listener��DefalutMessageDelegate�ķ�������Message�������ʵ�ת����Ȼ��ͨ�����������÷�����

2.2����Bean�ķ�ʽ����

<!-- Bean Configuration -->
  <bean id="messageListener"
    class="org.springframework.data.redis.listener.adapter.MessageListenerAdapter">
    <constructor-arg>
      <bean class="secondriver.spring.redis.MyMessageListener" />
    </constructor-arg>
  </bean>

  <bean id="redisContainer"
    class="org.springframework.data.redis.listener.RedisMessageListenerContainer">
    <property name="connectionFactory" ref="jedisConnectionFactory" />
    <property name="messageListeners">
      <map>
        <entry key-ref="messageListener">
          <list>
            <bean class="org.springframework.data.redis.listener.ChannelTopic">
              <constructor-arg value="springtv" />
            </bean>
            <bean class="org.springframework.data.redis.listener.PatternTopic">
              <constructor-arg value="hello*" />
            </bean>
            <bean class="org.springframework.data.redis.listener.PatternTopic">
              <constructor-arg value="tv*" />
            </bean>
          </list>
        </entry>
      </map>
    </property>
  </bean>
3.ģ����Ϣ�ķ����ͽ���

// �򵥲���RedisMessageListener
  @Ignore
  @Test
  public void test10() throws InterruptedException {
    RedisMessageListenerContainer rmlc;
    // ctx.getBean(RedisMessageListenerContainer.class);
    rmlc = (RedisMessageListenerContainer) ctx.getBean("redisContainer");
    while (true) {
      if (rmlc.isRunning()) {
        System.out
            .println("RedisMessageListenerContainer is running..");
      }
      Thread.sleep(5000);
    }
  }
���������ǲ��ԣ�ͨ��һ����ѭ�������ֳ���һֱ���У�Ȼ����Redis�����ָ��Ƶ��������Ϣ������϶��ĵ�Ƶ������Ϣ�����ͻ������ӽ��յ�������MyMessageListener�����е�onMessage���������á�

��ͼ��ģ����̣�

pubsubmessage.png
������������Ϣ���ֱ���spring,springtv��hello�� nono�ĸ�Ƶ�������ݶ���bean���������е�Topic���ƣ�ֻ��springtv,hello����ģʽƥ�䣬����Ҳͬ���յ�����������Ϣ��

��������ģ���⣬ʵ����Ӧ�ÿ�������ͨ��JedisConnection��Pub/Sub��صķ�������Redis���񷢲���Ϣ�Ļ���RedisTemplate��convertAndSend������

4.���

SDR�����ڷ�չ�׶ε���Ŀ�����������Ķ�Դ���룬һ�����ھ�