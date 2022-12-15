---
title: "Advanced Go Concurrency Patterns"
layout: post
date: 2018-07-11 08:31:19 -0700
---

Concurrency in Go is implemented using Goroutines.
Goroutines are functions or methods that run concurrently with other functions or methods. They can be thought of as light weight threads. The cost of creating a Goroutine is tiny when compared to a thread. Hence it's common for Go applications to have thousands of Goroutines running concurrently.

Inorder to communicate between these Goroutines, Go implements channels.
Channels send and receive data until the other side is ready. This allows Goroutines to synchronize without explicit locks or condition variables.

## Synchronized Channels
Channels that don't have a buffer size are called Synchronized Channels.
The write and read functions halt till they are resolved.
For instance-
when we apply a read from a channel :
  ```go
  <-channel
  ```
  The execution would not proceed beyond that line of code till the channel has a value to execute the read function.
Similarly, in case of-
```go
channel <- 4
```
  The execution halts till a reader removes the input.
This allows us to synchronize Goroutines which otherwise work independently.



## Challenges Faced in making Goroutines
The following problems occur commonly if Goroutines are used too cavalierly-

#### Go Routines Dead Lock
It is a state when none of the Go routines are able to proceed. i.e - they are waiting for the other ones to provide a means of completion.
```go
func main() {
     c := make(chan int)
     c <- 42    // write to a channel
     val := <-c // read from a channel
     println(val)
}
```
In the above code a deadlock is formed because the code execution stops at __c <- 42__
and the statement __val := <-c__ is never evaluated.

#### Race conditions in GO
A race condition is when two or more routines have access to the same resource, such as a variable or data structure and attempt to read and write to that resource without any regard to the other routines. This type of code can create the craziest and most random bugs you have ever seen. It usually takes a tremendous amount of logging and luck to find these types of bugs
Race conditions arise if we fire two Go Routines without proper initiation.
```go
import (
    "fmt"
    "sync"
)

var Counter int = 0

func Routine(id int) {
    for count := 0; count < 2; count++ {
        value := Counter
        value++
        Counter = value
    }
}

func main() {
  for routine := 1; routine <= 2; routine++ {
        go Routine(routine)
    }
}
```
In the above code, both Goroutines are racing to get the value _Counter_ , add one to the value and write it back to _Counter_.
In such a scenario, one of the Goroutines might alter the value of _Counter_ after the other reads but before writes to it.
So when the second Routine writes to _Counter_, it does so with the outdated value.

Fortunately, Go provision to detect Race conditions.
It can be detected using the following code-
```shell
go run -race file.go
```

#### Sleep in Go routines
Go routines become unresponsive when in sleep mode. Therefore, if a condition occurs where we have to programmatically close the go routines without closing the program, the Go routines are lost and hog the cpu till there sleep ends. This is could be harmful in cases having long sleep durations. These Go routines would not be cleared for a long while.


Let's look at how to write programs that handle communication, periodic events, and cancellation.
These can be mitigated using proper usage of Select construct.

## Select Construct
The core is Go's select statement: like a switch, but the decision is made based on the ability to communicate.
```go
select {
case xc <- x:
    // sent x on xc
case y := <-yc:
    // received y from yc
}
```
Select statement holds execution till one of it's conditions is satisfied.
This way all conditions can be checked simultaneously without being blocked at a statement.
a default condition can also be added, if it is required to be executed if none of the conditions are satisfied. In that scenario, the select construct won't wait if none of the conditions are satisfied.
```go
select {
case xc <- x:
    // sent x on xc
case y := <-yc:
    // received y from yc
default:
  fmt.Println("No condition satisfied")
}
```
When more than one condition is satisfied, the case selected is at random between the successful ones.

By using the Select Construct we can avoid dead locks as we provide an alternative to the program, if the read or write on the channel cannot proceed.

Race conditions can also be resolved by proper implementation of channels select construct, without having to use Locks for Synchronization.

## Timeout in Goroutine
As noted above, using **_sleep_** in Go could possibly lead to rogue Goroutines. To avoid this we refrain from using sleep() at all.
This is facilitated by the usage of **_Timeouts__**.
**_Timeouts__** coupled with switch construct, provide an alternative to **_sleep_** in the following manner-
```go
select {
    case <-time.After(1 * time.Second):
        fmt.Println("timeout 1")
    }
```
So instead of using **_sleep_**, before we run _case res := <-c1_,
we use Timeout along with Switch Construct.

This allows the Goroutine to be responsive even after adding a delay in it's execution.