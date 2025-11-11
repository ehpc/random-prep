# Implement processParallel, pass Context to processParallel to cancel after timeout

```go
package main

import (
	"context"
	"fmt"
	"math/rand"
	"sync"
	"time"
)

func processData(v int) int {
	time.Sleep(time.Duration(rand.Intn(10)) * time.Second)
	return v * 2
}

func main() {
	in := make(chan int)
	out := make(chan int)

	go func() {
		for i := range 10 {
			in <- i
		}
		close(in)
	}()

	start := time.Now()
	processParallel(in, out, 5)

	for v := range out {
		fmt.Println("v =", v)
	}
	fmt.Println("main duration:", time.Since(start))
}

func processParallel(in, out chan int, numWorkers int) {
  // Your code here
}
```

Answer:

```go
package main

import (
	"context"
	"fmt"
	"math/rand"
	"sync"
	"time"
)

func processData(v int) int {
	time.Sleep(time.Duration(rand.Intn(10)) * time.Second)
	return v * 2
}

func main() {
	in := make(chan int)
	out := make(chan int)

	go func() {
		for i := range 10 {
			in <- i
		}
		close(in)
	}()

	ctx, cancel := context.WithTimeout(context.Background(), 20*time.Second)
	defer cancel()

	start := time.Now()
	processParallel(ctx, in, out, 5)

	for v := range out {
		fmt.Println("v =", v)
	}
	fmt.Println("main duration:", time.Since(start))
}

func processParallel(ctx context.Context, in <-chan int, out chan<- int, numWorkers int) {
	var wg sync.WaitGroup

	for range numWorkers {
		wg.Go(func() {
			for {
				select {
				case <-ctx.Done():
					return
				case v, ok := <-in:
					if !ok {
						return
					}
					processChan := make(chan int, 1)
					go func() {
						processChan <- processData(v)
					}()
					select {
					case <-ctx.Done():
						return
					case result := <-processChan:
						out <- result
					}
				}
			}
		})
	}

	go func() {
		wg.Wait()
		close(out)
	}()
}
```