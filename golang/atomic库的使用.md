# sync/atomic 使用总结

## 原子增加
```
    sum := int64(0)
	wg := sync.WaitGroup{}

	for i := 0; i < 5000; i++ {
		wg.Add(1)
		go func() {
			//sum ++    // 1
			atomic.AddInt64(&sum, 1)  // 2
			wg.Done()
		}()
	}

 	//fmt.Println(sum)  // 3
	wg.Wait()
	fmt.Println(sum)
	
    比较1,2两处值的区别。可以看出使用1的时候sum值不为5000，使用2的时候值为5000(原子操作)。
使用3的时候值也不为5000(有的groutine还没执行完就打印sum的值了)。
```

## 比较和改变值
```
    sum := int64(100)
	wg := sync.WaitGroup{}

	for i:=int64(0); i<=100; i++ {
		wg.Add(1)
		go func() {
			atomic.CompareAndSwapInt64(&sum, int64(100), sum+1)
			wg.Done()
		}()
	}

	wg.Wait()
	fmt.Println(sum)
	
	只有当sum为100的时候改变了一次
```

## 获取值
```
    sum := int64(0)
	wg := sync.WaitGroup{}
	value := int64(99)

	go func() {
		wg.Add(1)
		for {
			if num := atomic.LoadInt64(&sum); num == value {
				fmt.Println(num)
				wg.Done()
			}
		}
	}()

	for i:=int64(0); i<100; i++ {
		wg.Add(1)
		go func() {
			atomic.AddInt64(&sum, 1)
			wg.Done()
		}()
	}

	wg.Wait()
	
	如果一个写操作未完成，有一个读操作就已经发生了，这样读操作使很糟糕的。为了原子的
读取数据可以使用LoadInt64()。最后sum的值是可以加到100，测试原子读出，将value设置为
[1,99]则可以判断是否为原子读取。当然不用原子读取也是有几率num等于value，需多测几次。
```

## 单例
```
type singleton struct {}

var (
    instance    *singleton
    initialized uint32
    mu          sync.Mutex
)

func Instance() *singleton {
    if atomic.LoadUint32(&initialized) == 1 {
        return instance
    }

    mu.Lock()
    defer mu.Unlock()

    if instance == nil {
        defer atomic.StoreUint32(&initialized, 1)
        instance = &singleton{}
    }
    return instance
}

引用Go语言高级编程中的例子
```