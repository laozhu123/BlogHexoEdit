---
title: java线程池的4种拒绝模式
date: 2018-04-08 10:27:30
categories: "java基础学习"
tags:
     - 源码
     - java
     - 基础
---

### 简介
在线程池的使用中，当任务数量超出之前设定的线程池大小（既：核心线程数+等待队列大小），系统需要采取相应策略来应对新进来的任务。当前java线程池有4种不同的策略来应对这种情况。

### 拒绝策略

#### 1.AbortPolicy
> * In the default {@link ThreadPoolExecutor.AbortPolicy}, the
 * handler throws a runtime {@link RejectedExecutionException} upon
 * rejection.

通过该段注释可以知道，该策略是线程池的**默认策略**。该策略的机制是，通过抛出一个RejectedExecutionException异常来拒绝新添加的任务。


    /**
     * A handler for rejected tasks that throws a
     * {@code RejectedExecutionException}.
     */
    public static class AbortPolicy implements RejectedExecutionHandler {
        /**
         * Creates an {@code AbortPolicy}.
         */
        public AbortPolicy() { }

        /**
         * Always throws RejectedExecutionException.
         *
         * @param r the runnable task requested to be executed
         * @param e the executor attempting to execute this task
         * @throws RejectedExecutionException always
         */
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            throw new RejectedExecutionException("Task " + r.toString() +
                                                 " rejected from " +
                                                 e.toString());
        }
    }

只是简单的抛出一个RejectedExecutionException异常，异常中的信息是任务名和拒绝它的线程池。

#### 2.CallerRunsPolicy
>  * In {@link ThreadPoolExecutor.CallerRunsPolicy}, the thread
 * that invokes {@code execute} itself runs the task. This provides a
 * simple feedback control mechanism that will slow down the rate that
 * new tasks are submitted.

该策略直接使用当前线程来运行任务的run方法，这种方式会降低新任务的执行率。看下相应的代码：

    public static class CallerRunsPolicy implements RejectedExecutionHandler {
        /**
         * Creates a {@code CallerRunsPolicy}.
         */
        public CallerRunsPolicy() { }

        /**
         * Executes task r in the caller's thread, unless the executor
         * has been shut down, in which case the task is discarded.
         *
         * @param r the runnable task requested to be executed
         * @param e the executor attempting to execute this task
         */
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                r.run();
            }
        }
    }

当过多的新任务来时，回调用rejectedExecution方法，该方法直接在任务调用者（也就是执行executor.execute(r)）所在线程中执行任务的run方法。


#### DiscardPolicy

> * In {@link ThreadPoolExecutor.DiscardPolicy}, a task that
 * cannot be executed is simply dropped.

通过翻译可知，该策略只是简单的将新任务抛弃，并不会抛出异常。

    /**
     * A handler for rejected tasks that silently discards the
     * rejected task.
     */
    public static class DiscardPolicy implements RejectedExecutionHandler {
        /**
         * Creates a {@code DiscardPolicy}.
         */
        public DiscardPolicy() { }

        /**
         * Does nothing, which has the effect of discarding task r.
         *
         * @param r the runnable task requested to be executed
         * @param e the executor attempting to execute this task
         */
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        }
    }

rejectedExecution方法是空的，不做任何处理。


#### DiscardOldestPolicy

>  * In {@link ThreadPoolExecutor.DiscardOldestPolicy}, if the
 * executor is not shut down, the task at the head of the work queue
 * is dropped, and then execution is retried (which can fail again,
 * causing this to be repeated.)

该策略会将等待队列中等待最久的一个出队列，而后再次执行execute(r)方法，但是执行execute(r)方法时会有再次失败的可能，因为这里的出队列和进队列不是原子性的，在多线程环境下还是会存在别的线程先执行了execute(r)方法，将相应的任务push进等待队列中。

    public static class DiscardOldestPolicy implements RejectedExecutionHandler {
        /**
         * Creates a {@code DiscardOldestPolicy} for the given executor.
         */
        public DiscardOldestPolicy() { }

        /**
         * Obtains and ignores the next task that the executor
         * would otherwise execute, if one is immediately available,
         * and then retries execution of task r, unless the executor
         * is shut down, in which case task r is instead discarded.
         *
         * @param r the runnable task requested to be executed
         * @param e the executor attempting to execute this task
         */
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                e.getQueue().poll();
                e.execute(r);
            }
        }
    }

