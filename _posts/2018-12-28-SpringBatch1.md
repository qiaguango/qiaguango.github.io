---
layout: post
#标题配置
title:  SpringBatch(一)
#时间配置
date:   2018-12-26 09:27:00 +0800
#大类配置
categories: SpringBatch
#小类配置
tag: 教程
---

* content
{:toc}

# SpringBatch

## SpringBatch介绍

> Spring Batch是一个轻量级，全面的批处理框架，旨在开发对企业系统日常运营至关重要的强大批处理应用程序。Spring Batch构建了人们期望的Spring Framework特性（生产力，基于POJO的开发方法和一般易用性），同时使开发人员可以在必要时轻松访问和利用更高级的企业服务。Spring Batch不是一个调度框架。在商业和开源空间中都有许多好的企业调度程序（例如Quartz，Tivoli，Control-M等）。它旨在与调度程序一起使用，而不是替换调度程序。
Spring Batch提供了可重复使用的功能，这些功能对于处理大量记录至关重要，包括记录/跟踪，事务管理，作业处理统计，作业重启，跳过和资源管理。它还提供更高级的技术服务和功能，通过优化和分区技术实现极高容量和高性能的批处理作业。Spring Batch可用于两种简单的用例（例如将文件读入数据库或运行存储过程）以及复杂的大量用例（例如在数据库之间移动大量数据，转换它等等）上）。大批量批处理作业可以高度可扩展的方式利用该框架来处理大量信息。

## 入门程序
> 通过搭建入门程序认识SpringBatch

### 1.工程下载
> 通过[start.spring.io](http://start.spring.io)或IDE下载SpringBatch工程。

![start.spring.io]({{qiaguango.github.io}}/assets/SpringBatch/image001.png)

### 2.pom.xml

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-batch</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.springframework.batch</groupId>
	<artifactId>spring-batch-test</artifactId>
	<scope>test</scope>
</dependency>
```
### 3.数据库配置
> SpringBatch运行需要配置持久化数据源，否则将运行失败，可以使用内存数据库或者关系型数据库作为项目的持久存储。

application.properties配置信息如下：

```
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/springbatch
spring.datasource.username=springbatch
spring.datasource.password=springbatch
spring.datasource.schema=classpath:/org/springframework/batch/core/schema-mysql.sql
spring.batch.initialize-schema=always
```

需要注意的是**spring.datasource.schema**和**spring.batch.initialize-schema**这两个配置信息，配置以后SpringBatch框架将为我们自动建表，用于相关的操作记录，如图所示：
![]({{qiaguango.github.io}}/assets/SpringBatch/image002.png)

### 4.程序源码

> 需要使用**Configuration**和**EnableBatchProcessing**注解

```
@Configuration
@EnableBatchProcessing
public class JobConfiguration {

    //
    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    //
    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job helloWorldJob(){
        return jobBuilderFactory.get("helloWorldJob").start(step1()).build();
    }

    @Bean
    public Step step1(){

        return stepBuilderFactory.get("step1").tasklet(new Tasklet() {
            @Override
            public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                System.out.println("hello world");
                return RepeatStatus.FINISHED;
            }
        }).build();
    }
}

```
## Job的创建和使用

> SpringBatch中的Job创建有多种方式

### 1.单个任务
示例代码：

```
jobBuilderFactory.get("helloWorldJob").start(step1()).build()
```

### 2.多任务
> 可以使用next()方法完成多任务的链路执行

示例代码：

```
jobBuilderFactory.get("jobDemo").start(step()).next(step1()).next(step2()).build();
```

### 3.基于任务执行状态的Job

> 可以使用on().to().from().end()的方式基于任务的执行状态来完成多任务的链路执行

示例代码：

```
jobBuilderFactory.get("jobDemo")
 .start(step()).on("COMPLETED").to(step2())
 .from(step2()).on("COMPLETED").to(step3())
 .from(step3()).end()
 .build();
```

#### 3.1 stopAndRestart()和fail()

> 可以使用on().to().stopAndRestart()的方式来进行各个step的测试  
可以使用fail来中断任务

示例代码：
```
jobBuilderFactory.get("jobDemo")
 .start(step()).on("COMPLETED").to(step2())
 .from(step2()).on("COMPLETED").stopAndRestart()
```

## Flow的创建和使用

> 在SpringBatch中可以使用flow可以包含多个step，在job执行时可以通过调用flow来完成任务的执行

示例代码：
```
@Configuration
@EnableBatchProcessing
public class FlowDemo {
    @Autowired
    private JobBuilderFactory jobBuilderFactory;
    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Bean
    public Step flowDemoStep1(){
        return stepBuilderFactory.get("flowDemoStep1").tasklet(new Tasklet() {
            @Override
            public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                System.out.println("flowDemoStep1");
                return RepeatStatus.FINISHED;
            }
        }).build();
    }

    @Bean
    public Step flowDemoStep2(){
        return stepBuilderFactory.get("flowDemoStep2").tasklet(new Tasklet() {
            @Override
            public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                System.out.println("flowDemoStep2");
                return RepeatStatus.FINISHED;
            }
        }).build();
    }

    @Bean
    public Step flowDemoStep3(){
        return stepBuilderFactory.get("flowDemoStep3").tasklet(new Tasklet() {
            @Override
            public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                System.out.println("flowDemoStep3");
                return RepeatStatus.FINISHED;
            }
        }).build();
    }

    @Bean
    public Flow flowDemoFlow(){
        return new FlowBuilder<Flow>("flowDemoFlow")
                .start(flowDemoStep1())
                .next(flowDemoStep2())
                .build();
    }

    @Bean
    public Job flowDemoJob(){
        return jobBuilderFactory.get("flowDemojob")
                .start(flowDemoFlow())
                .next(flowDemoStep3())
                .end()
                .build();
    }
}

```

## Split的创建和使用
> 可以使用split用于多线程任务的执行

示例代码：
```
@Configuration
@EnableBatchProcessing
public class JobSplit {
    @Autowired
    private JobBuilderFactory jobBuilderFactory;
    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Bean
    public Step jobSplitStep1(){
        return stepBuilderFactory.get("jobSplitStep1")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                        System.out.println("jobSplitStep1");
                        return RepeatStatus.FINISHED;
                    }
                }).build();
    }

    @Bean
    public Step jobSplitStep2(){
        return stepBuilderFactory.get("jobSplitStep2")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                        System.out.println("jobSplitStep2");
                        return RepeatStatus.FINISHED;
                    }
                }).build();
    }

    @Bean
    public Step jobSplitStep3(){
        return stepBuilderFactory.get("jobSplitStep3")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                        System.out.println("jobSplitStep3");
                        return RepeatStatus.FINISHED;
                    }
                }).build();
    }


    @Bean
    public Flow jobSplitFlow1(){
        return new FlowBuilder<Flow>("jobSplitFlow1")
                .start(jobSplitStep1())
                .build();
    }

    @Bean
    public Flow jobSplitFlow2(){
        return new FlowBuilder<Flow>("jobSplitFlow2")
                .start(jobSplitStep2())
                .next(jobSplitStep3())
                .build();
    }

    @Bean
    public Job jobSplitDemo(){
        return jobBuilderFactory.get("jobSplitDemo")
                .start(jobSplitFlow1())
                .split(new SimpleAsyncTaskExecutor())
                .add(jobSplitFlow2())
                .end()
                .build();
    }
}
```

## 决策器的使用
> 可以使用JobExecutionDecider接口来完成决策器的实现，任务根据决策器的返回结果来决定任务执行顺序

### MyDecider 示例代码
```
public class MyDecider implements JobExecutionDecider {

    private int count;
    @Override
    public FlowExecutionStatus decide(JobExecution jobExecution, StepExecution stepExecution) {
        count++;
        if(count%2 == 0){
            return new FlowExecutionStatus("even");
        }else{
            return new FlowExecutionStatus("odd");
        }

    }
}
```

### DeciderDemo 示例代码
```
@Configuration
@EnableBatchProcessing
public class DeciderDemo {
    @Autowired
    private JobBuilderFactory jobBuilderFactory;
    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Bean
    public Step deciderDemoStep1(){
        return stepBuilderFactory.get("deciderDemoStep1")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                        System.out.println("deciderDemoStep1");
                        return RepeatStatus.FINISHED;
                    }
                }).build();
    }

    @Bean
    public Step deciderDemoStep2(){
        return stepBuilderFactory.get("deciderDemoStep2")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                        System.out.println("even");
                        return RepeatStatus.FINISHED;
                    }
                }).build();
    }

    @Bean
    public Step deciderDemoStep3(){
        return stepBuilderFactory.get("deciderDemoStep3")
                .tasklet(new Tasklet() {
                    @Override
                    public RepeatStatus execute(StepContribution stepContribution, ChunkContext chunkContext) throws Exception {
                        System.out.println("odd");
                        return RepeatStatus.FINISHED;
                    }
                }).build();
    }

    @Bean
    public JobExecutionDecider myDecider(){
        return new MyDecider();
    }

    @Bean
    public Job deciderDemoJob(){
        return jobBuilderFactory.get("deciderDemoJob")
                .start(deciderDemoStep1())
                .next(myDecider())
                .from(myDecider()).on("even").to(deciderDemoStep2())
                .from(myDecider()).on("odd").to(deciderDemoStep3())
                .from(deciderDemoStep3()).on("*").to(myDecider())
                .end()
                .build();
    }
}
```
### 执行结果
```
2018-12-29 10:25:47.925  INFO 9774 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [deciderDemoStep1]
deciderDemoStep1
2018-12-29 10:25:47.998  INFO 9774 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [deciderDemoStep3]
odd
2018-12-29 10:25:48.041  INFO 9774 --- [           main] o.s.batch.core.job.SimpleStepHandler     : Executing step: [deciderDemoStep2]
even
```


