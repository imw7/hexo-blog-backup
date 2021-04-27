---
title: Go操作Kafka
tags: Kafka
categories: 常用组件和技巧
abbrlink: go_kafka
date: 2019-08-18 00:00:00
toc: true
---

Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者规模的网站中的所有动作流数据，具有高性能、持久化、多副本备份、横向扩展等特点。本文介绍了如何使用Go语言发送和接收kafka消息。<!--more-->

## sarama

Go语言中连接Kafka使用第三方库[sarama](https://github.com/Shopify/sarama)。

### 下载及安装

```bash
go get github.com/Shopify/sarama
```

### 注意事项

`sarama` v1.20之后的版本加入了`zstd`压缩算法，需要用到cgo，在Windows平台编译时会提示类似如下错误：

```bash
# github.com/DataDog/zstd
exec: "gcc":executable file not found in %PATH%
```

所以在Windows平台请使用v1.19版本的sarama。

## 连接Kafka发送消息

```go
package main

import (
	"fmt"
	"github.com/Shopify/sarama"
)

// 基于sarama第三方库开发的Kafka客户端

func main() {
	config := sarama.NewConfig()
	config.Producer.RequiredAcks = sarama.WaitForAll          // 发送完数据需要leader和follow都确认
	config.Producer.Partitioner = sarama.NewRandomPartitioner // 新选出一个partition
	config.Producer.Return.Successes = true                   // 成功交付的消息将在success channel返回

	// 构造一个消息
	msg := &sarama.ProducerMessage{}
	msg.Topic = "web_log"
	msg.Value = sarama.StringEncoder("this is a test log")
	// 连接Kafka
	client, err := sarama.NewSyncProducer([]string{"127.0.0.1:9092"}, config)
	if err != nil {
		fmt.Println("producer closed, err:", err)
		return
	}
	defer func() { _ = client.Close() }()
	// 发送消息
	pid, offset, err := client.SendMessage(msg)
	if err != nil {
		fmt.Println("send msg failed, err:", err)
		return
	}
	fmt.Printf("pid:%v offset:%v\n", pid, offset)
}
```

## 连接Kafka消费消息

```go
package main

import (
	"fmt"
	"github.com/Shopify/sarama"
)

// kafka consumer

func main() {
	consumer, err := sarama.NewConsumer([]string{"127.0.0.1:9092"}, nil)
	if err != nil {
		fmt.Printf("fail to start consumer, err:%v\n", err)
		return
	}
	partitionList, err := consumer.Partitions("web_log") // 根据topic取到所有的分区
	if err != nil {
		fmt.Printf("fail to get list of partition:err%v\n", err)
		return
	}
	fmt.Println(partitionList)
	for partition := range partitionList { // 遍历所有的分区
		// 针对每个分区创建一个对应的分区消费者
		pc, err := consumer.ConsumePartition("web_log", int32(partition), sarama.OffsetNewest)
		if err != nil {
			fmt.Printf("failed to start consumer for partition %d,err:%v\n", partition, err)
			return
		}
		defer pc.AsyncClose()
		// 异步从每个分区消费信息
		go func(sarama.PartitionConsumer) {
			for msg := range pc.Messages() {
				fmt.Printf("Partition:%d Offset:%d Key:%v Value:%v", msg.Partition, msg.Offset, msg.Key, msg.Value)
			}
		}(pc)
	}
}
```

## LogAgent的工作流程

1.读日志 -- `tailf`第三方库

```go
// tail的用法示例
package main

import (
	"fmt"
	"github.com/hpcloud/tail"
	"time"
)

func main() {
	fileName := "./my.log"
	config := tail.Config{
		Location:  &tail.SeekInfo{Offset: 0, Whence: 2}, // 从文件的哪个位置开始读
		ReOpen:    true,                                 // 重新打开
		MustExist: false,                                // 文件不存在不报错
		Follow:    true,                                 // 是否跟随
		Poll:      true,
	}
	tails, err := tail.TailFile(fileName, config)
	if err != nil {
		fmt.Println("tail file failed, err:", err)
		return
	}
	var (
		msg *tail.Line
		ok  bool
	)
	for {
		msg, ok = <-tails.Lines
		if !ok {
			fmt.Println("tail file close reopen, filename:", tails.Filename)
			time.Sleep(time.Second)
			continue
		}
		fmt.Println("msg:", msg.Text)
	}
}
```

2.往Kafka写日志 -- `sarama`第三方库

```go
// 基于sarama第三方库开发的Kafka客户端
package main

import (
	"fmt"
	"github.com/Shopify/sarama"
)

func main() {
	config := sarama.NewConfig()
	config.Producer.RequiredAcks = sarama.WaitForAll          // 发送完数据需要leader和follow都确认
	config.Producer.Partitioner = sarama.NewRandomPartitioner // 新选出一个partition
	config.Producer.Return.Successes = true                   // 成功交付的消息将在success channel返回

	// 构造一个消息
	msg := &sarama.ProducerMessage{}
	msg.Topic = "web_log"
	msg.Value = sarama.StringEncoder("this is a test log")
	// 连接Kafka
	client, err := sarama.NewSyncProducer([]string{"127.0.0.1:9092"}, config)
	if err != nil {
		fmt.Println("producer closed, err:", err)
		return
	}
	defer func() { _ = client.Close() }()
	// 发送消息
	pid, offset, err := client.SendMessage(msg)
	if err != nil {
		fmt.Println("send msg failed, err:", err)
		return
	}
	fmt.Printf("pid:%v offset:%v\n", pid, offset)
}
```

## Kafka和ZooKeeper

先启动zookeeper：

```bash
$ /usr/local/kafka_2.12-2.5.0/ bin/zookeeper-server-start.sh config/zookeeper.properties 
```

再启动kafka：

```bash
$ /usr/local/kafka_2.12-2.5.0/ bin/kafka-server-start.sh config/server.properties 
```

kafka终端读取数据：

```bash
$ /usr/local/kafka_2.12-2.5.0/ bin/kafka-console-consumer.sh --bootstrap-server=127.0.0.1:9092 --topic=web_log --from-beginning
```