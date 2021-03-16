wait() 与sleep()

wait() 与sleep()

(new Object()).wait()
Thread.sleep()
从上面两个方法的调用就可以看出，
1、sleep是静态方法，wait不是
2、wait是任何一个对象拥有的方法，他会释放锁，sleep是Thread的方法，不会释放锁
3、wait与notify或者notifyAll一起使用，notify或者notifyAll会唤醒等待该obj锁的所有线程。控制线程运行顺序。