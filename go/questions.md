# What will be the output and why

Snippet 1:

```go
package main

import "fmt"

func addElement(s []int) {
	fmt.Println("before:", s)
	s = append(s, 4)
	s[0] = 0
	fmt.Println("inside:", s)
}

func main() {
	s := []int{1, 2, 3}
	addElement(s)
	fmt.Println("outside:", s)
}

/*
Result:
  before: [1 2 3]
  inside: [0 2 3 4]
  outside: [1 2 3]
*/
```

Snippet 2:

```go
package main

import "fmt"

func addElement(s []int) {
	fmt.Println("before:", s)
	s = append(s, 4)
	s[0] = 0
	fmt.Println("inside:", s)
}

func main() {
	s := make([]int, 3, 10)
	s[0], s[1], s[2] = 1, 2, 3
	addElement(s)
	fmt.Println("outside:", s)
}

/*
Result:
  before: [1 2 3]
  inside: [0 2 3 4]
  outside: [0 2 3]
*/
```

# What are channels and mutexes

Channels in Go are concurrency primitives used to safely communicate 
between goroutines — they let you send and receive typed data, synchronizing 
access implicitly by blocking until both sides are ready.

Mutexes (mutual exclusions) are low-level locks used to protect shared data 
from concurrent access — only one goroutine can hold the lock at a time, 
preventing race conditions.

You’d use a channel instead of a mutex when you want to communicate 
between goroutines — not just protect data.

✅ Use channels when goroutines need to send data or signals to each other 
(e.g., worker pools, pipelines, message passing). Channels make ownership of 
data explicit — whoever receives the value now owns it.

✅ Use mutexes when multiple goroutines share the same data structure and 
you just need to control who can access it at a time.

```go
// Channel example — worker pool

jobs := make(chan int, 5)
results := make(chan int, 5)

for w := 1; w <= 3; w++ {
	go func(id int) {
		for j := range jobs {
			results <- j * 2 // send result back
		}
	}(w)
}

for i := 1; i <= 5; i++ {
	jobs <- i
}
close(jobs)

for i := 0; i < 5; i++ {
	fmt.Println(<-results)
}
```

```go
// Mutex example — shared counter

var mu sync.Mutex
counter := 0

for i := 0; i < 5; i++ {
	go func() {
		mu.Lock()
		counter++
		mu.Unlock()
	}()
}

time.Sleep(time.Second)
fmt.Println("counter:", counter)
```

📨 Channels

✅ Pros:
Safe communication: No need to lock — data ownership moves between goroutines.
Clear design: Makes concurrency flow explicit (who sends, who receives).
Composability: Great for pipelines, fan-in/fan-out, worker pools.

❌ Cons:
Overhead: Passing data through channels is slower than direct access under a mutex.
Complexity: Easy to create deadlocks or goroutine leaks if not closed properly.
Less suited for shared mutable state: Not ideal when many goroutines just need 
to tweak one shared value.

🔒 Mutexes

✅ Pros:
Fast: Minimal overhead — great for performance-critical shared memory.
Simple for shared data: Ideal for protecting maps, counters, caches, etc.
Deterministic: Behavior is often easier to reason about than async channel logic.

❌ Cons:
Error-prone: Forgetting Unlock() → deadlock.
No data transfer semantics: You still have to manage who reads/writes what.
Harder to compose: Doesn’t model communication or back-pressure.

# What happens with these ops to channels

1. Write to a closed channel

```go
ch := make(chan int)
close(ch)
ch <- 1
// panic: send on closed channel
```

2. Read from a closed channel

```go
ch := make(chan int, 1)
ch <- 42
close(ch)
fmt.Println(<-ch) // ✅ 42
fmt.Println(<-ch) // ✅ 0 (zero value of the channel type)
```

3. Write to a nil channel

```go
var ch chan int
ch <- 1
// 😵 Deadlock: goroutine blocks forever.
// A nil channel has no receiver, so send blocks permanently.
```

4. Read from a nil channel

```go
var ch chan int
fmt.Println(<-ch)
// Same deal — deadlock.
// It blocks forever because the channel is never ready.
```

5. Closing a channel twice

```go
close(ch)
close(ch) // panic: close of closed channel
```

6. Sending to nil channel in select

```go
var ch chan int
select {
case ch <- 1:
default:
    fmt.Println("skipped") // always hits default
}
```

7. Unbuffered channel deadlock

```go
ch := make(chan int)
ch <- 1 // deadlocks if no concurrent receiver
```

# What happens with these ops to mutexes

1. Unlocking an unlocked mutex

```go
var mu sync.Mutex
mu.Unlock() // panic: unlock of unlocked mutex
```

2. Deferring Unlock before Lock

```go
mu.Unlock()
defer mu.Lock() // wrong order → panic
```

3. Copying a mutex

```go
type Counter struct { mu sync.Mutex; n int }
c2 := c1 // copies the mutex → undefined behavior
// Mutexes must never be copied.
```
