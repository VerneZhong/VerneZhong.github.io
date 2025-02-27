---
title: CompletableFuture 异步编程API列表
date: 2020-08-07 16:08:10
categories: 
- Java笔记
tags:
- Java8异步编程
---
![](https://github.com/user-attachments/assets/61e0d341-8bc7-4716-acbb-ebfb7dfdf08b)

<!-- more -->

#### CompletableFuture异步编程API

##### 静态工厂方法：

- runAsync(Runnable runnable)：使用ForkJoinPool.commonPool()作为它的线程池执行异步代码。

- runAsync(Runnable runnable, Executor executor)：使用指定的thread pool执行异步代码。

- supplyAsync(Supplier<U> supplier)：使用ForkJoinPool.commonPool()作为它的线程池执行异步代码，异步操作有返回值。


- supplyAsync(Supplier<U> supplier, Executor executor)：使用指定的thread pool执行异步代码，异步操作有返回值


- complete(T t)：完成异步执行，并返回future的结果


- completeExceptionally(Throwable ex)：异步执行不正常的结束，抛出异步执行的错误堆栈


##### 转换(类似于流里的map)：

- thenApply(Function<? super T,? extends U> fn)：接受一个Function<? super T,? extends U>参数用来转换CompletableFuture

- thenApplyAsync(Function<? super T,? extends U> fn)：接受一个Function<? super T,? extends U>参数用来转换CompletableFuture，使用ForkJoinPool


- thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)：接受一个Function<? super T,? extends U>参数用来转换CompletableFuture，使用指定的线程池


##### flatMap：

- thenCompose(Function<? super T, ? extends CompletionStage<U>> fn)：在异步操作完成的时候对异步操作的结果进行一些操作，并且仍然返回CompletableFuture类型。

- thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn)：在异步操作完成的时候对异步操作的结果进行一些操作，并且仍然返回CompletableFuture类型。使用ForkJoinPool。


- thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn,Executor executor)：在异步操作完成的时候对异步操作的结果进行一些操作，并且仍然返回CompletableFuture类型。使用指定的线程池。


##### 组合：

- thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)：当两个CompletableFuture都正常完成后，执行提供的fn，用它来组合另外一个CompletableFuture的结果。

- thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)：当两个CompletableFuture都正常完成后，执行提供的fn，用它来组合另外一个CompletableFuture的结果。使用ForkJoinPool。
  thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn, Executor executor)：当两个CompletableFuture都正常完成后，执行提供的fn，用它来组合另外一个CompletableFuture的结果。使用指定的线程池。

- thenAcceptBoth(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action)：当两个CompletableFuture都正常完成后，执行提供的action，用它来组合另外一个CompletableFuture的结果。


- thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action)：当两个CompletableFuture都正常完成后，执行提供的action，用它来组合另外一个CompletableFuture的结果。使用ForkJoinPool。


- thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action, Executor executor)：当两个CompletableFuture都正常完成后，执行提供的action，用它来组合另外一个CompletableFuture的结果。使用指定的线程池。


##### 计算结果完成时的处理：

- whenComplete(BiConsumer<? super T,? super Throwable> action)：当CompletableFuture完成计算结果时对结果进行处理，或者当CompletableFuture产生异常的时候对异常进行处理。

- whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)：当CompletableFuture完成计算结果时对结果进行处理，或者当CompletableFuture产生异常的时候对异常进行处理。使用ForkJoinPool。


- whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)：当CompletableFuture完成计算结果时对结果进行处理，或者当CompletableFuture产生异常的时候对异常进行处理。使用指定的线程池。


- handle(BiFunction<? super T, Throwable, ? extends U> fn)：当CompletableFuture完成计算结果或者抛出异常的时候，执行提供的fn


- handleAsync(BiFunction<? super T, Throwable, ? extends U> fn)：当CompletableFuture完成计算结果或者抛出异常的时候，执行提供的fn，使用ForkJoinPool。


- handleAsync(BiFunction<? super T, Throwable, ? extends U> fn, Executor executor)：当CompletableFuture完成计算结果或者抛出异常的时候，执行提供的fn，使用指定的线程池。


- thenAccept(Consumer<? super T> action)：当CompletableFuture完成计算结果，只对结果执行Action，而不返回新的计算值


- thenAcceptAsync(Consumer<? super T> action)：当CompletableFuture完成计算结果，只对结果执行Action，而不返回新的计算值，使用ForkJoinPool。


- thenAcceptAsync(Consumer<? super T> action, Executor executor)：当CompletableFuture完成计算结果，只对结果执行Action，而不返回新的计算值


- acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action)：当任意一个CompletableFuture完成的时候，action这个消费者就会被执行。


- acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action)：当任意一个CompletableFuture完成的时候，action这个消费者就会被执行。使用ForkJoinPool


- acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action, Executor executor)：当任意一个CompletableFuture完成的时候，action这个消费者就会被执行。使用指定的线程池


- applyToEither(CompletionStage<? extends T> other, Function<? super T,U> fn)：当任意一个CompletableFuture完成的时候，fn会被执行，它的返回值会当作新的CompletableFuture<U>的计算结果。


- applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T,U> fn)：当任意一个CompletableFuture完成的时候，fn会被执行，它的返回值会当作新的CompletableFuture<U>的计算结果。使用ForkJoinPool


- applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T,U> fn, Executor executor)：当任意一个CompletableFuture完成的时候，fn会被执行，它的返回值会当作新的CompletableFuture<U>的计算结果。使用指定的线程池


- allOf(CompletableFuture<?>... cfs)：在所有Future对象完成后结束，并返回一个future。


- anyOf(CompletableFuture<?>... cfs)：在任何一个Future对象结束后结束，并返回一个future。


##### CompletableFuture异常处理：

- exceptionally(Function fn)：只有当CompletableFuture抛出异常的时候，才会触发这个exceptionally的计算，调用function计算值。

