### 一句话描述try catch



**不要用 try catch 包裹 Promise , Promise 很强大，不用担心异常会往上抛！我们只需要给 Promise 增加 Promise#catch 就 OK 了**



**总结 ： 能被 try catch 捕捉到的异常，必须是在报错的时候，线程执行已经进入 try catch 代码块，且处在 try catch 里面，这个时候才能被捕捉到。**



#### try catch 分为三个阶段

##### 1、 之前： 

	类似  语法错误，未执行就报错，不会走到try catch ![1588944957846](C:\Users\ADMINI~1\AppData\Local\Temp\1588944957846.png)



##### 2、 之中： 

代码报错的时候，线程执行处于 try catch 之中，则能捕捉到异常。

	![1588945035335](C:\Users\ADMINI~1\AppData\Local\Temp\1588945035335.png)上述报错的时机，都是代码执行进入了 try catch ，执行 d() 方法的时候，线程执行处在 try 里面，所以能捕捉到。



##### 3、 之后：

代码报错的时候，线程已经执行完 try catch，这种不能捕捉到异常。



	setTimeout 里面报错，实际上是 100ms 之后执行的代码报错，此时代码块 try catch 已经执行完成，111 都已经被执行了，故无法捕捉异常。

如下： 

![1588945098041](C:\Users\ADMINI~1\AppData\Local\Temp\1588945098041.png)





![1588945119302](C:\Users\ADMINI~1\AppData\Local\Temp\1588945119302.png)

	方法定义在 try catch 代码块里面，但是执行方法在 try catch 外，在执行 d 方法的时候报错，此时 try catch 已经执行完成，111 都已经被执行了，故而无法捕捉异常