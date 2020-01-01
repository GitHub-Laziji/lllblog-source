---
title: Java模拟JavaScript生成器函数
date: 2020-01-02 01:23:49
categories: 技术分享
tags:
- Java
- JavaScript
---


js中可以定义生成器函数, 使用`yield`迭代结果, 例如
```js
function* gen() { 
  yield 1;
  yield 2;
  yield 3;
}

let g = gen(); 
g.next();
g.next();
```

本文中尝试在Java环境中实现类似的效果

# 实现
## Generator.java
生成器类
```java
import java.util.concurrent.Semaphore;

public class Generator {

    private Function function;

    private Runnable runnable;
    private Thread thread;

    private Semaphore readLock;
    private Semaphore writeLock;

    private Result result;
    private Object nextValue;
    private boolean done = true;

    public Generator(Function function) {
        this.function = function;
        this.runnable = () -> {
            try {
                this.writeLock.acquire();
                Object value = this.function.run(this);
                this.result = new Result(value, true);
                this.readLock.release();
            } catch (Exception e) {
                e.printStackTrace();
            }
        };
        this.reset();
    }

    public boolean isDone() {
        return done;
    }

    public synchronized void reset() {
        this.close();
        this.readLock = new Semaphore(0);
        this.writeLock = new Semaphore(0);
        this.result = null;
        this.nextValue = null;
        this.done = false;
        this.thread = new Thread(this.runnable);
        this.thread.start();
    }

    public synchronized void close() {
        if (this.thread != null) {
            this.thread.stop();
            this.thread = null;
        }
    }

    public synchronized Result next() throws InterruptedException {
        return this.next(null);
    }

    public synchronized Result next(Object value) throws InterruptedException {
        if (this.done) {
            return null;
        }
        this.nextValue = value;
        this.writeLock.release();
        this.readLock.acquire();
        Result result = this.result;
        this.done = result.isDone();
        return result;
    }

    public Object yield(Object value) throws InterruptedException {
        this.result = new Result(value, false);
        this.readLock.release();
        this.writeLock.acquire();
        return this.nextValue;
    }

    @FunctionalInterface
    public interface Function {
        Object run(Generator context) throws Exception;
    }

    public static class Result {
        private Object value;
        private boolean done;

        private Result(Object value, boolean done) {
            this.value = value;
            this.done = done;
        }

        public Object getValue() {
            return value;
        }

        public boolean isDone() {
            return done;
        }

        @Override
        public String toString() {
            return "Result{" +
                    "value=" + value +
                    ", done=" + done +
                    '}';
        }
    }
}
```


# 测试

## Java
```java
public class Main {

    public static void main(String[] args) throws InterruptedException {
        Generator generator = new Generator(context -> {
            int count = 0;
            for (int i = 0; i < 10; i++) {
                Integer value = (Integer) context.yield(i);
                count += value == null ? 0 : value;
            }
            return count;
        });

        Generator.Result result = null;
        do {
            result = generator.next(result == null ? null : result.getValue());
            System.out.println(result);
        } while (!result.isDone());
    }
}

/* 输出结果
Result{value=0, done=false}
Result{value=1, done=false}
Result{value=2, done=false}
Result{value=3, done=false}
Result{value=4, done=false}
Result{value=5, done=false}
Result{value=6, done=false}
Result{value=7, done=false}
Result{value=8, done=false}
Result{value=9, done=false}
Result{value=45, done=true}
*/
```

## 对应的JavaScript代码

```js
let generator = (function* Generator() {
    let count = 0;
    for (let i = 0; i < 10; i++) {
        count += (yield i) || 0;
    }
    return count;
})();

let result;
do {
    result = generator.next(result && result.value);
    console.log(result);
} while (!result.done)

/* 输出结果
{value: 0, done: false}
{value: 1, done: false}
{value: 2, done: false}
{value: 3, done: false}
{value: 4, done: false}
{value: 5, done: false}
{value: 6, done: false}
{value: 7, done: false}
{value: 8, done: false}
{value: 9, done: false}
{value: 45, done: true}
*/
```