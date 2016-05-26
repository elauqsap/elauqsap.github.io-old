---
layout: post
title: "Building a Go Work Pool and Dispatcher"
date: 2016-05-25 22:41:16 -0500
comments: false
categories: [Development, Go, Tips & Tricks]
---

I recently started working with [Go](https://golang.org/) which is a very opinionated open source programming language from Google and contributors. It is a fantastic language and I rather enjoy how it has helped me develop as a programmer these past few months.

<!--more-->

### Back Story
My new position at work requires me to work with a large data set that I decided to truncate into smaller sets for processing. I wanted to process each batch in parallel but without restricting myself to a single "job type". So that in the future when the code requires a different "job type" I would not have to wrangle multiple work pools. In developing the solution, I found a work around to Go's lack of generics so I could process multiple "job types" via the same work pool.

### Dispatcher
The role of the `Dispatcher` is to initialize the `WorkerPool`, dispatch jobs as they are created, and wait for the go routines to finish before closing out the main thread.

``` go dispatcher.go

package main

import "sync"

// Dispatcher creates workers and dispatches jobs when received
type Dispatcher struct {
	JobQueue   chan Job
	MaxWorkers int
	WaitGroup  *sync.WaitGroup
	// A pool of workers channels that are registered with the dispatcher
	WorkerPool chan chan Job
}

// NewDispatcher creates a dispatcher that is used to create workers
// and dispatch jobs to them
func NewDispatcher(maxWorkers int) *Dispatcher {
	pool := make(chan chan Job, maxWorkers)
	return &Dispatcher{JobQueue: make(chan Job, 1024), MaxWorkers: maxWorkers, WorkerPool: pool, WaitGroup: &sync.WaitGroup{}}
}

// Run creates the workers and dispatches jobs from a JobQueue to each worker
func (d *Dispatcher) Run() {
	// starting n number of workers
	for i := 0; i < d.MaxWorkers; i++ {
		worker := NewWorker(d.WorkerPool, d.WaitGroup)
		worker.Start()
	}

  // start the dispatcher routine
	go d.dispatch()
}

func (d *Dispatcher) dispatch() {
	for {
		select {
		case job := <-d.JobQueue:
			// a job request has been received
			go func(job Job) {
				// try to obtain a worker job channel that is available.
				// this will block until a worker is idle
				jobChannel := <-d.WorkerPool
				// dispatch the job to the worker job channel
				jobChannel <- job
			}(job)
		}
	}
}

```
### Worker
A `Worker` is started by the `Dispatcher`and registers itself to the `WorkerPool`. Once a `Job` has been sent to the `Dispatcher`, it waits for a `Worker` to become ready for processing and hands off the `Job` to the `Worker`. The `Worker` triggers the `process()` method of the `Job`.

``` go worker.go
package main

import (
	"fmt"
	"sync"
)

// Worker represents the worker that executes the job
type Worker struct {
  // A pool of workers channels that are registered with the dispatcher
	WorkerPool chan chan Job
  // A channel for receiving a job that was dispatched
	JobChannel chan Job
  // A channel for receiving a worker termination signal
  // (quits after processing)
  quit       chan bool
  // A WaitGroup to signal the completed processing of a Job
	wg         *sync.WaitGroup
}

// NewWorker creates a new worker that can be registered to a WorkerPool
// and receive jobs
func NewWorker(workerPool chan chan Job, wg *sync.WaitGroup) Worker {
	return Worker{
		WorkerPool: workerPool,
		JobChannel: make(chan Job),
		quit:       make(chan bool),
		wg:         wg}
}

// Start method starts the run loop for the worker, listening for a quit channel in
// case we need to stop it
func (w Worker) Start() {
	go func() {
		for {
			// register the current worker into the worker queue.
			w.WorkerPool <- w.JobChannel

			select {
			case job := <-w.JobChannel:
				job.process()
              // signal to the wait group that a queued job has been processed
              // so the main thread can continue
				w.wg.Done()
			case <-w.quit:
				// we have received a signal to stop
				return
			}
		}
	}()
}
```

### Job
All `Job` types implement a `process()` method. This way we do not need to infer types in the `Worker` thus allowing us to achieve a level of generic types in Go.

``` go jobs.go
package main

// Job interface will be implemented for each task so that they
// may be passed to workers in the pool by the dispatcher
type Job interface {
	process() error
}

type BatchJob struct {
  // define the struct
}

func (b BatchJob) process() error {
  // process data here for batch sets ...
}

type SingleJob struct {
  // define the struct
}

func (s SingleJob) process() error {
  // process data here for a single set ...
}
```

### All Together Now
Here is an example of the above code in action. The `struct` need to be defined and `process()` implemented but it demonstrates the overall concept.

``` go main.go
package main

func main() {
  // Initialize a Dispatcher
  dispatcher := NewDispatcher(4)
  // Start the Dispatcher and create/register the Workers to the WorkerPool
  dispatcher.Run()
  // Queue two jobs for processing
  dispatcher.WaitGroup.Add(2)
  // Two jobs of different structures queued
  dispatcher.JobQueue <- BatchJob{}
  dispatcher.JobQueue <- SingleJob{}
  // Block main thread until processing in go routines completes
  dispatcher.WaitGroup.Wait()
}
```
