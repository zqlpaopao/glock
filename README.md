# glock
分布式锁、可重入

1. 加的锁是自己的，过期时间
2. 续期操作 过期时间的一半时间去加，每次加一半，直到，锁被释放(建议默认)，也可以指定加锁续期周期和时间
3. 释放的是自己的锁
4. 加锁成功回调函数
5. 加锁失败回调函数
6. 释放锁失败重试，默认三次
7. 支持查看当前竞争者
8. 支持短锁

# 设计思路
![glock](https://user-images.githubusercontent.com/43371021/160043878-9616051d-1fac-4b9b-80f3-f371ec8cbf8b.png)

#使用方法
```

gLock := pkg.NewGlock(
		pkg.WithSeizeTag(true),                       //持续争夺还是只是一次
		pkg.WithSeizeCycle(2*time.Second),            //持续争夺还是只是一次
		pkg.WithLockKey("key"),                       //争多的标识
		pkg.WithRedisTimeout(3*time.Second),          //redis的操作超时时间,默认3s
		pkg.WithExpireTime(5),                        //master的超时时间
		pkg.WithRenewalOften(pkg.DefaultRenewalTime), //如果抢到master，续期多长时间,默认expire的一半
		pkg.WithRedisClient(redis),
		pkg.WithLockFailFunc(func(i ...interface{}) { //抢锁失败回调函数
			for _, v := range i {
				fmt.Println("传入的参数", v)
			}
		}),
		pkg.WithLockSuccessFunc(func(i ...interface{}) { //抢锁成功回调函数
			for _, v := range i {
				fmt.Println("传入的参数成功", v)
			}
		}),
	)

	gLock.Lock(i, i1)

	for {

		fmt.Println(gLock.GetMembers())//全部竞争锁的成员

		fmt.Println(gLock.IsMaster())//是否是master
		fmt.Println(gLock.Error()) //获取失败原因

		time.Sleep(5 * time.Second)
	}
	
	gLock.UnLock()

```
