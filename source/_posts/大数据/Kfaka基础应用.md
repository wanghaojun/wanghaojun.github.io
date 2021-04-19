---
title: Kafka基础应用
date: 2020-05-08 08:31:03
tags: [大数据]
---

这篇文章主要介绍了基于Kafka环境，利用真实的用户行为日志数据来实现大数据平台的增量数据采集过程。是我正在学习的大数据课程中的一个实验。<!--more-->

本文主要有以下内容：

- 在win10环境下的Kafka的基本环境搭建
- kafka-clients实现简单的生产者和消费者
- 使用maven+spring-boot+spring-data-jpa+spring-kafka 实现大数据平台的增量数据采集

## 1.kafka环境搭建（win10）

这一部分我主要参照了kafka的中文文档：http://kafka.apachecn.org/quickstart.html

下载代码并压缩，下载地址：http://kafka.apachecn.org/downloads.html

下载kafka后就可以参照Kafka的中文文档进行基本的kafka操作了，创建一个topic，然后发送消息，启动consumer来消费消息，设置多代理集群，尝试使用kafka connect来导入/导出数据，官方文档写的非常详细，这里我就不再照搬一遍了。

完成kafka官方文档中的QuickStart之后，相信你对Kafka有了简单的认识和了解，接下来就尝试使用Kafka的API来完成基本的消费者和生产者操作吧。

Kafka有四个核心的API:

- The [Producer API](http://kafka.apachecn.org/documentation.html#producerapi) 允许一个应用程序发布一串流式的数据到一个或者多个Kafka topic。
- The [Consumer API](http://kafka.apachecn.org/documentation.html#consumerapi) 允许一个应用程序订阅一个或多个 topic ，并且对发布给他们的流式数据进行处理。
- The [Streams API](http://kafka.apachecn.org/documentation/streams) 允许一个应用程序作为一个*流处理器*，消费一个或者多个topic产生的输入流，然后生产一个输出流到一个或多个topic中去，在输入输出流中进行有效的转换。
- The [Connector API](http://kafka.apachecn.org/documentation.html#connect) 允许构建并运行可重用的生产者或者消费者，将Kafka topics连接到已存在的应用程序或者数据系统。比如，连接到一个关系型数据库，捕捉表（table）的所有变更内容。

## 2.实现简单的生产者消费者

这一阶段，我们将使用Kafka的java客户端来使用Kafka的API实现简单的生产者和消费者。

首先，要启动zookeeper和kafka环境，具体过程在上一节的kafka的中文文档中有介绍。

接下来直接上代码：

需要在pom.xml中添加以下依赖，注意，我是用的kafka-clients的版本是2.2.0。

```xml
    <dependencies>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>2.2.0</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>1.7.25</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
```

生产者代码，ProducerTest.java:

```java
package producer;

import org.apache.kafka.clients.producer.*;
import org.apache.kafka.common.serialization.StringSerializer;
import java.util.Properties;

public class ProducerTest implements Runnable{

    private final KafkaProducer<String,String> producer;
    private final String topic;

    private ProducerTest(String topicName) {
        Properties properties = new Properties();
        properties.put("bootstrap.servers", "localhost:9092");
        properties.put("acks","all");
        properties.put("retries",0);
        properties.put("batch.size",16384);
        properties.put("linger.ms","1");
        properties.put("buffer.memory",33554432);
        properties.put("key.serializer", StringSerializer.class.getName());
        properties.put("value.serializer", StringSerializer.class.getName());
        this.producer = new KafkaProducer<String, String>(properties);
        this.topic = topicName;
    }

    @Override
    public void run() {
        int no = 1;
        try {
            for (;;){
                String msg = "this is " + no + "   messages";
                    ProducerRecord<String,String> record = new ProducerRecord<String, String>(topic,Integer.toString(no),msg);
                    producer.send(record);
                    if (no % 100 == 0){
                        System.out.println(msg);
                    }
                    if (no == 1000){
                        System.out.println("finish");
                        break;
                    }
                    no ++;
                }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            producer.close();
        }
    }

    public static void main(String[] args) {
        ProducerTest producerTest = new ProducerTest("testtopic");
        Thread thread = new Thread(producerTest);
        thread.start();
    }
}

```

参数说明：

- bootstrap.servers： broker的地址
- acks：指定必须要有多少个分区的副本接收到该消息，服务端才会向生产者发送响应，可选值为：0,1,2，…，all
- retries：生产者向kafka发送消息可能会发生错误，有的是临时性的错误，比如网络突然阻塞了一会儿，有的不是临时的错误，比如“消息太大了”，对于出现的临时错误，可以通过重试机制来重新发送
- batch.size：生产者在发送消息时，可以将即将发往同一个分区的消息放在一个批次里，然后将这个批次整体进行发送，这样可以节约网络带宽，提升性能。该参数就是用来规约一个批次的大小的。但是生产者并不是说要等到一个批次装满之后，才会发送，不是这样的，有时候半满，甚至只有一个消息的时候，也可能会发送，具体怎么选择，我们不知道，但是不是说非要等装满才发。因此，如果把该参数调的比较大的话，是不会造成消息发送延迟的，但是会占用比较大的内存。但是如果设置的太小，会造成消息发送次数增加，会有额外的IO开销。
- linger.ms：生产者在发送一个批次之前，可以适当的等一小会，这样可以让更多的消息加入到该批次。这样会造成延时增加，但是降低了IO开销，增加了吞吐量
- buffer.memory：生产者的内存缓冲区大小。如果生产者发送消息的速度 > 消息发送到kafka的速度，那么消息就会在缓冲区堆积，导致缓冲区不足。这个时候，send()方法要么阻塞，要么抛出异常。
- key.serializer：关键字的序列化方式
- value.serializer：消息值的序列化方式

消费者代码，ConsumerTest.java:

```java
package consumer;

import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.serialization.StringDeserializer;
import java.time.Duration;
import java.util.Arrays;
import java.util.Properties;

public class ConsumerTest implements Runnable{

    private final KafkaConsumer<String,String> consumer;
    private ConsumerRecords<String,String> msgList;
    private String topic;
    private static final String groupId = "groupA";

    public ConsumerTest(String topicName) {
        Properties properties = new Properties();
        properties.put("bootstrap.servers","localhost:9092");
        properties.put("group.id",groupId);
        properties.put("enable.auto.commit","true");
        properties.put("auto.commit.interval.ms", "1000");
        properties.put("session.timeout.ms","30000");
        properties.put("max.poll.records", 1000);
        properties.put("key.deserializer", StringDeserializer.class.getName());
        properties.put("value.deserializer", StringDeserializer.class.getName());
        this.consumer = new KafkaConsumer<String, String>(properties);
        this.topic = topicName;
        this.consumer.subscribe(Arrays.asList(topicName));

    }

    @Override
    public void run() {
        System.out.println("--------开始消费------------");
        try {
            while (true){
                msgList = consumer.poll(Duration.ofMillis(100));
                if (msgList !=null && msgList.count()>0){
//                    System.out.println("11111111111111");
                    int no = 1;
                    for (ConsumerRecord<String,String> record : msgList){
                        if (no % 100 == 0){
                            System.out.println(no+"===receive key:"+record.key()+"===value:"+record.value());
                        }
                        if (no == msgList.count()){
                            break;
                        }
                        no ++;
                    }
                }
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            consumer.close();
        }
    }
    
    public static void main(String[] args) {
        ConsumerTest consumerTest = new ConsumerTest("testtopic");
        Thread thread = new Thread(consumerTest);
        thread.start();
    }
    
}
```

参数介绍：

- bootstrap.servers ：broker集群地址
- group.id ：消费者隶属的消费者组名称
- enable.auto.commit ：boolean 类型，配置是否开启自动提交消费位移的功能，默认开启
- auto.commit.interval.ms ：当enbale.auto.commit参数设置为 true 时才生效，表示开启自动提交消费位移功能时自动提交消费位移的时间间隔
- session.timeout.ms ： 响应超时时间
- max.poll.records ： 这个参数用来配置 Consumer 在一次拉取请求中拉取的最大消息数，默认值为500（条）。如果消息的大小都比较小，则可以适当调大这个参数值来提升一定的消费速度。
- key.deserializer：关键字的反序列化方式
- value.deserializer：消息值的序列化方式

## 3.spring-boot+Kafka

由于我之前接触过spring-boot，无意之中发现kafka已经可以整合到spring中了，考虑到spring-data-jpa操作数据库也比较方便快捷，于是打算以maven+spring-boot+spring-data-jpa+spring-kafka来实现整个实验，开始学习使用spring-kafka。

全部源码可以从github上获取：https://github.com/wanghaojun/kafka_homework2

数据集获取：http://insis.bjtu.edu.cn/file/Datasets_1.zip

项目开发过程：

1) 设计存储结构化日志数据的MySQL关系表；

由于在项目中使用了spring-data-jpa，可以自动构建mysql表，所以这里只展示项目建立的model：

```java
package com.wanghaojun.springkafka.model;

import javax.persistence.*;
import java.util.Date;

@Entity
@Table(name = "record")
public class Record {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String userId;

    private String articalId;

    private String type;

    private String time;
    // 新增属性，描述记录产自哪个生产者
    private String producerName;

    public Record() {
    }

    public String getProducerName() {
        return producerName;
    }

    public void setProducerName(String producerName) {
        this.producerName = producerName;
    }

    public int getId() {
        return id;
    }

    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    public String getArticalId() {
        return articalId;
    }

    public void setArticalId(String articalId) {
        this.articalId = articalId;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public String getTime() {
        return time;
    }

    public void setTime(String time) {
        this.time = time;
    }
}

```

生产的Table：

![](http://img.wanghaojun.cn//img/20200422102844.png)



2) 根据原始数据和创建的Kafka话题，设计并实现三种不同类型的模拟日志抓取的Kafka生产者；

（1）首先实现数据集处理的组件：

```java
package com.wanghaojun.springkafka.component;

import com.wanghaojun.springkafka.dto.RecordDto;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.io.*;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Comparator;

/**
 * 数据集组件类
 */
@Component
public class DataSet {

    @Value("${file.path}")
    public String filePath;

    @Value("${file.sort-path}")
    public String sortPath;
    

    /**
     * 文件读取方法
     * @param num 读取行数
     * @return ArrayList<String>
     */
    public ArrayList<String> readFromFile(int num)  {
        File file = new File(filePath);
        ArrayList<String> text = new ArrayList<>();
        String line;
        try {
            BufferedReader in = new BufferedReader(new FileReader(file));
            if (num==-1){
                while ((line = in.readLine())!= null){
                    text.add(line);
                }
            }else {
                while ((line = in.readLine())!= null && num!=0){
                    text.add(line);
                    num--;
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return text;
    }


    /**
     * 按照时间顺序排序
     */
    public void sort(){
        ArrayList<String> text = readFromFile(-1);
        text.sort((s, t1) -> {
            String s1 = s.split("\\001")[3];
            String s2 = t1.split("\\001")[3];
            return s1.compareTo(s2);
        });
        File file = new File(sortPath);
        Writer writer = null;

        try {
            writer = new FileWriter(file);
            for (String line: text){
                writer.write(line+"\n");
            }
        } catch (IOException e){
            e.printStackTrace();
        } finally {
            if (writer != null) {
                try {
                    writer.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        System.out.println("sort and write to file finish!!!");
    }

    /**
     * 将spring转为RecordDto类的对象
     * @param arrayList 字符数组
     * @return ArrayList<RecordDto>
     */
    public ArrayList<RecordDto> toRecordDto(ArrayList<String> arrayList) {
        ArrayList<RecordDto> recordDtos = new ArrayList<>();
        for (String a:arrayList){
            String[] line = a.split("\\001");
            RecordDto recordDto = new RecordDto();
            recordDto.setUserId(line[0]);
            recordDto.setType(line[1]);
            recordDto.setArticalId(line[2]);
            recordDto.setTime(line[3]);
            recordDtos.add(recordDto);
        }
        return recordDtos;
    }

    /**
     * 将日期转为时间戳
     * @param date 日期
     * @return 时间戳
     */
    public Long dateToTimeStamp(String date)  {
        try {
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            return (simpleDateFormat.parse(date).getTime()/1000);
        }catch (Exception e){
            e.printStackTrace();
        }
        return Long.valueOf("0");
    }
}

```

（2）构建一个Test 对数据集进行排序：

```java
@SpringBootTest
class DatasetTest {

    @Autowired
    private DataSet dataSet;
    
    @Test
    void sortTest(){
        dataSet.sort();

    }
}
```

（3）构建生产者的组件类：

```java
package com.wanghaojun.springkafka.component;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;

import java.util.ArrayList;

@Component
public class Producer {

    @Autowired
    private DataSet dataSet;

    private final String topic = "Record";

    @Autowired
    private KafkaTemplate<String, String> template;

    /**
     * 小时级生产者 一小时的数据发送一次
     * @param num 发送次数 ，-1为发送数据集中的所有内容
     * @param time 消息跨越时间，比如小时级生产者每次发送一个小时的数据
     * @param name 生产者名称
     */
    public void sendMsgSec(int num,int time,String name){
        ArrayList<String> text = dataSet.readFromFile(num*500);
        // 发送记录数
        int i=0;
        // 发送次数
        int c=0;
        while ((i < num) && (c < text.size()) ){
            Long temp =dataSet.dateToTimeStamp(text.get(c).split("\001")[3]) + time;
            StringBuilder sendMsg = new StringBuilder();
            while ( (c<text.size()) &&  (temp >= dataSet.dateToTimeStamp(text.get(c).split("\001")[3])) ){
                sendMsg.append(text.get(c)).append("\n");
                c++;
            }
            i++;
//            System.out.println(sendMsg.toString()+name);
            template.send(topic,sendMsg.toString()+name);
            try{
                Thread.sleep(5000);
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }

    /**
     * 秒级生产者 一条记录发送一次
     * @param num 发送次数
     * @param name 生产者名称
     */
    public void sendMsgLine(int num,String name)  {
        ArrayList<String> text = dataSet.readFromFile(num);
        int i=0;
        while (i < text.size()){
            template.send(topic, text.get(i) + "\n" + name);
            try{
                Thread.sleep(1000);
            }catch (Exception e){
                e.printStackTrace();
            }
            i++;
        }
    }
}


```

（4）构建controller部分，在访问某个API时触发生产者，开始生产数据：

```java
package com.wanghaojun.springkafka.controller;


import com.wanghaojun.springkafka.component.Customer;
import com.wanghaojun.springkafka.component.Producer;
import com.wanghaojun.springkafka.model.Record;
import com.wanghaojun.springkafka.repository.RecordRepository;
import net.bytebuddy.asm.Advice;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class KafkaController {

    @Autowired
    private RecordRepository recordRepository;

    @Autowired
    private Producer producer;


    @Autowired
    private Customer customer;

    /**
     * 查看当前数据库中所有数据
     * @return Iterable<Record> Json
     */
    @GetMapping("/")
    public Iterable<Record> getRecords(){
        return recordRepository.findAll();
    }

    /**
     * 查看当前数据库中数据数目
     * @return 数据数目
     */
    @GetMapping("/count")
    public Object count(){
        return recordRepository.count();
    }

    /**
     * 秒级生产者1
     * @param num 产生数据，-1为全部
     * @return Object
     */
    @GetMapping("/produce1")
    public Object produce1(int num){
        producer.sendMsgLine(num,"producer1");
        return new String("produce "+num+" message");
    }

    /**
     * 小时级生产者，每次生产1小时数据
     * @param num 发送次数
     * @return Object
     */
    @GetMapping("/producehour")
    public Object produceHour(int num){
        producer.sendMsgSec(num,3600,"producehour");
        return new String("produce "+num+" message");
    }

    /**
     * 秒级生产者2
     * @param num 产生数据，-1为全部
     * @return Object
     */
    @GetMapping("/produce2")
    public Object produce2(int num){
        producer.sendMsgLine(num,"producer2");
        return new String("produce "+num+" message");
    }

}

```

3) 根据Kafka中存储日志数据的话题中的数据量情况，设计并实现日志解析与结构化数据存储的Kafka消费者；

消费者代码实现：

```java
package com.wanghaojun.springkafka.component;

import com.wanghaojun.springkafka.dto.RecordDto;
import com.wanghaojun.springkafka.model.Record;
import com.wanghaojun.springkafka.repository.RecordRepository;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.Iterator;

@Component
public class Customer {


    @Autowired
    private RecordRepository recordRepository;

    /**
     * 监听Mytopic主题
     * @param cr 收到的消息
     */
    @KafkaListener(topics = "MyTopic")
    public void listen(ConsumerRecord<?, ?> cr) {
//       System.out.println("receive a message");
        this.custome(cr);
    }

    /**
     * 消费数据 把数据存入服务器端的mysql数据库
     * @param cr  待消费 的数据
     */
    private void custome(ConsumerRecord<?, ?> cr){
        ArrayList<Record> records = new ArrayList<Record>();
        String  value  = (String) cr.value();
        String[] lines = value.split("\n");
        String name = lines[lines.length-1];
        int l = lines.length -1 ;
//        System.out.println(lines.length);
        for (int i = 0 ;i<l ; i++){
            Record record = new Record();
            record.setUserId(lines[i].split("\\001")[0]);
            record.setType(lines[i].split("\\001")[1]);
            record.setArticalId(lines[i].split("\\001")[2]);
            record.setTime(lines[i].split("\\001")[3]);
            record.setProducerName(name);
            records.add(record);
        }
        recordRepository.saveAll(records);
    }

}

```

项目的依赖文件pom.xml和spring-boot配置文件application.yml可以去GitHub的代码仓库查看。

## 参考文献

http://kafka.apachecn.org/intro.html

https://blog.csdn.net/Shannon076/article/details/81259652