---
title: golang 并发 
date: 2021-08-01 13:15:50 
tags:
    - golang 
categories:
    - golang 
img: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/cover/co62.jpg
tinyImg: https://cdn.jsdelivr.net/gh/GoldArowana/static_source@main/images/tiny/cover/co62.jpg
---

## 互斥锁Mutex功能

### 不用锁会怎么样

```go
import (
	"fmt"
	"sync"
)

// go run -race main.go
func main() {
	var count = 0
	var wg sync.WaitGroup

	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()

			for j := 0; j < 100000; j++ {
				count++
			}
		}()
	}

	wg.Wait()
	fmt.Println(count)
}

```

### 用mutex解决并发计数问题

```go
import (
	"fmt"
	"sync"
)

func main() {
	var count = 0
	var mu sync.Mutex
	var wg sync.WaitGroup

	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()

			for j := 0; j < 100000; j++ {
				mu.Lock()
				count++
				mu.Unlock()
			}
		}()
	}

	wg.Wait()
	fmt.Println(count)
}
```

### count和mutex可以封装到一起, 而且一般mutex会定义在要锁的字段的上面：

```go
import (
	"fmt"
	"sync"
)

type Counter struct {
	sync.Mutex
	Count int64
}

func (c *Counter) Incr() {
	c.Lock()
	c.Count++
	c.Unlock()
}

var Incr10k = func(counter *Counter, wg *sync.WaitGroup) {
	defer wg.Done()

	for j := 0; j < 100000; j++ {
		counter.Incr()
	}
}

func main() {
	var counter Counter
	var wg sync.WaitGroup
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go Incr10k(&counter, &wg)
	}

	wg.Wait()
	fmt.Println(counter.Count)
}

```

## Mutex实现


## 参考资料

[sync.mutex 源代码分析](https://colobu.com/2018/12/18/dive-into-sync-mutex/)
