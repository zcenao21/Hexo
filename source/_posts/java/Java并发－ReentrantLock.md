---
title: ReentrantLock
date: 2021-03-27 10:03:50
tags:
    - Java
    - ReentrantLock
categories: 
    - Java
---





# Synchronized

synchronized是java语言底层实现，我们不需要考虑因为它产生的异常，但是也存在不够灵活，比较重等缺点。<!--more-->

下面使用synchronized实现一个生产消费队列：

```java
import java.util.ArrayList;
import java.util.Random;

public class WaitAndNotify {
    static Random random = new Random();
    public static void main(String[] args) throws InterruptedException {
        Runnable target;
        final TaskQueueTest test = new TaskQueueTest();
        for(int i=0;i<3;i++){
            final int finalI = i;
            Thread producer = new Thread(){
                public void run(){
                    try{
                        while(true){
                            test.putTask(finalI+"");
                            sleep(900+random.nextInt(900));
                        }
                    }catch (Exception e){}
                }
            };
            producer.setDaemon(true);
            producer.start();
        }

        for(int j=0;j<3;j++){
            final int finalJ = j;
            Thread consumer = new Thread(){
                public void run(){
                    try{
                        while(true){
                            test.exeTask(finalJ);
                            sleep(900+random.nextInt(900));
                        }
                    }catch (Exception e){}
                }
            };
            consumer.setDaemon(true);
            consumer.start();
        }

        Thread.sleep(5000);
        while(test.getTasksCnt()!=0){
            Thread.sleep(100);
        }

        System.out.println(System.currentTimeMillis()+": main thread done");
    }
}

class TaskQueueTest{
    private int cnt = 0;
    private ArrayList<String> taskList = new ArrayList<String>();

    public int getTasksCnt(){
        return taskList.size();
    }
    public synchronized void putTask(String taskName){
        System.out.println("User " + taskName + " put task " + cnt);
        taskList.add(cnt++ + "");
        this.notify();
    }

    public synchronized void exeTask(int taskNumber) throws InterruptedException {
        while(taskList.isEmpty()){
            this.wait();
        }
        System.out.println("Consumer " + taskNumber + " consume task " + taskList.get(0));
        taskList.remove(0);
    }
}
```

输出：

```
User 1 put task 0
User 2 put task 1
Consumer 0 consume task 0
Consumer 1 consume task 1
User 0 put task 2
Consumer 2 consume task 2
User 1 put task 3
Consumer 0 consume task 3
User 0 put task 4
Consumer 1 consume task 4
User 2 put task 5
Consumer 2 consume task 5
User 0 put task 6
User 1 put task 7
User 2 put task 8
Consumer 0 consume task 6
Consumer 2 consume task 7
Consumer 1 consume task 8
User 0 put task 9
Consumer 1 consume task 9
User 2 put task 10
Consumer 0 consume task 10
User 1 put task 11
Consumer 2 consume task 11
1616810856277: main thread done
```



# ReentrantLock

ReentrantLock是java语言实现的，相对于synchronized较轻量，且更加灵活。同样实现上面的生产消费模型：

```
import java.util.ArrayList;
import java.util.Random;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class WaitAndNotify2 {
    static Random random = new Random();
    public static void main(String[] args) throws InterruptedException {
        Runnable target;
        final TaskQueueTest2 test = new TaskQueueTest2();
        for(int i=0;i<3;i++){
            final int finalI = i;
            Thread producer = new Thread(){
                public void run(){
                    try{
                        while(true){
                            test.putTask(finalI+"");
                            sleep(900+random.nextInt(900));
                        }
                    }catch (Exception e){}
                }
            };
            producer.setDaemon(true);
            producer.start();
        }

        for(int j=0;j<3;j++){
            final int finalJ = j;
            Thread consumer = new Thread(){
                public void run(){
                    try{
                        while(true){
                            test.exeTask(finalJ);
                            sleep(900+random.nextInt(900));
                        }
                    }catch (Exception e){}
                }
            };
            consumer.setDaemon(true);
            consumer.start();
        }

        Thread.sleep(5000);
        while(test.getTasksCnt()!=0){
            Thread.sleep(100);
        }

        System.out.println(System.currentTimeMillis()+": main thread done");
    }
}

class TaskQueueTest2{
    private int cnt = 0;
    private ArrayList<String> taskList = new ArrayList<String>();
    ReentrantLock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    public int getTasksCnt(){
        return taskList.size();
    }
    public void putTask(String taskName){
        lock.lock();
        try{
            System.out.println("User " + taskName + " put task " + cnt);
            taskList.add(cnt++ + "");
            condition.signal();
        }finally {
            lock.unlock();
        }
    }

    public synchronized void exeTask(int taskNumber) throws InterruptedException {
        lock.lock();
        try{
            while(taskList.isEmpty()){
                condition.await();
            }
            System.out.println("Consumer " + taskNumber + " consume task " + taskList.get(0));
            taskList.remove(0);
        }finally {
            lock.unlock();
        }

    }
}
```

输出：

```
User 0 put task 0
User 1 put task 1
User 2 put task 2
Consumer 0 consume task 0
Consumer 1 consume task 1
Consumer 2 consume task 2
User 1 put task 3
Consumer 0 consume task 3
User 0 put task 4
Consumer 2 consume task 4
User 2 put task 5
Consumer 1 consume task 5
User 1 put task 6
Consumer 0 consume task 6
User 2 put task 7
Consumer 2 consume task 7
User 0 put task 8
Consumer 1 consume task 8
User 1 put task 9
Consumer 0 consume task 9
User 0 put task 10
Consumer 2 consume task 10
User 2 put task 11
Consumer 1 consume task 11
1616812412381: main thread done
```





























