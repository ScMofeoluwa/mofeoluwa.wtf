---
title: "Error Groups or Wait Groups?"
description: "A comparison of ErrorGroups and WaitGroups in Go, highlighting their use for synchronization and error handling in concurrent programming."
publishDate: "29 Mar 2024"
tags: ["golang"]
---

Coughs\*, I first learned about error groups about a week ago during an episode of the Go Time podcast [series](https://open.spotify.com/episode/5rx0SSiRSgu6YNF5IS9F7n?si=dd5e0379453d49e9).
It's been a while since I've written, so I thought I'd share my thoughts on error groups and wait groups, and why I believe error groups are the better choice for my future projects.
Please note that, as of now, error groups are not part of the standard Go library but are included in the Go X subdirectories.
You can install it by running `go get golang.org/x/sync/errgroup`
Here are some of the current reasons why I like error groups:

- **Simplified Synchronisation**: Unlike wait groups, error groups automatically handle the addition and removal of goroutines, eliminating the need for manual calls to `wg.Add(1)` and `wg.Done()`.
- **Error Propagation**: Error groups provide built-in support for error handling, which is not available in wait groups. In the past, I've often resorted to sending errors through channels, which I then read from. Error groups streamline this process by allowing functions that return errors to be directly managed.

Suppose you want to spin up goroutines to check if a number is 1 and return an error if at least one number isn't. Using goroutines, the code might look like this:

```go

func checkOne(n int) error {
	if n != 1 {
		return errors.New("number not one")
	}
	return nil
}

func main() {
	numbers := []int{1, 1, 1, 5}

	var wg sync.WaitGroup
	// create channel to handle errors
	errChan := make(chan error, 1)

	for _, number := range numbers {
		number := number
		wg.Add(1)
		go func() {
			defer wg.Done()
			err := checkOne(number)
			if err != nil {
				// send error through the channel
				errChan <- err
				return
			}
		}()
	}

	go func() {
		wg.Wait()
		// close channel after all goroutines are done
		close(errChan)
	}()

	// read channel to get the error
	err := <-errChan
	if err != nil {
		fmt.Println(err.Error())
	}
}

```

Here, a waitgroup is used to wait for all goroutines to complete, and a channel is created to handle errors. If an error occurs, it's sent through the channel, and the main goroutine waits for all others to finish before reading from the channel to handle the error.

However, using error groups we can have this:

```go

func checkOne(n int) error {
	if n != 1 {
		return errors.New("number not one")
	}
	return nil
}

func main() {
	numbers := []int{1, 1, 1, 5}

	// create new error group
	eg := new(errgroup.Group)
	for _, number := range numbers {
		number := number
		eg.Go(func() error {
			return checkOne(number)
		})
	}
	if err := eg.Wait(); err != nil {
		fmt.Println(err.Error())
	}
}

```

Here, the `errgroup.Group` is used to manage multiple concurrent operations, and if any operation fails, the error group's `Wait` method returns the first error encountered.

My key takeaways:

- Error groups are particularly useful for tasks of the same nature. For tasks that vary, you can still use error groups with channels to capture individual errors.
- They also help with context cancelation using the `WithContext()` method
