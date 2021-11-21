# go-pistachio
Stakes for better Golang development 

### The **SOLID** principles
Objective: design good programs.

- **S**ingle Responsability
  - Coupling & Cohesion
  - package names 
  	- good
		- net/http, os/exec, encoding/json
		- importing packages increases coupling
	- bad: server, private, common, util
  - UNIX tools are small sharp tools that together solve larger tasks. Go stdlib tries to mimic that, we all shall do.
 - **O**pen/closed principle
   - open for extension
   - closed for modification: struct embedding example; an embedded type does'nt know its embedded; you can't change the method of the embedded type from the one that embeds it it.
 - **L**iskov substitution principle
 	- Substitution powered by interfaces. Avoid concrete types if possible.
 	- Small interfaces are more powerful (e.g `io.Reader`)
 	  - a lot of the stdlib types implement Reader
	  - you can change one for the other and it will work
  - **I**nterface segregation principle
  	- Do not force clients to implements non-required methods
  	- Again, simpler/smaller is better.
	- Use the smaller interface possible that will do the work. E.g. need a `File` or you can generalize your method receiving instead a `ReadWriteCloser`? Or maybe you just call to `Write()`and you can change it for `Writer`? 
	- Related tool https://github.com/mvdan/interfacer
  - **D**ependency inversion principle
	- structure of your import graph
		- should be acyclic, otherwise compile error but also it would be a design issue

### Summary
- Interfaces let you apply the **SOLID** principles to Go programs.
- Focus on reuse. Forget frameworks, think about design.

### microservices

- Microservices are terrible. gokit tried to make it less terrible. 2 year old project.
- Described by:
	- size: one dev can manage it 
	- data: implements a single bounded context <- we like this one
	- ops: built and deployed independently (12Factor)
	- Architecture: What microservices could be?: 
	  - monolithic -> microservices
	  - type1: rpc, CRUD-like
	  - type2: stream-oriented, event sourcing, pipelining, queues
	
> **Microservices solve ORGANIZATIONAL problems, but they  create TECHNICAL problems.**

It solves big teams org. If you have less than 5 devs you don't need them.
	
Problems:
  -  testing becomes really hard.
  - distributed transactions: another really hard problem.
  - deploys, monitoring, logging, distributed tracing, security...
  - Sean Treadway (Soundcloud) found around 40 issues with microservices

### go kit
  - is not a framework. It's there for help you.
  - similar to Twitter's Finangle (Scala)
   - philosophy: encourages good design
     - no global state
     - declarative
	 
#### code show
- without go-kit
  - `Service` interface
  - `ServeHTTP()` to implement the `http.Handler` interface
  - return json
  - add logging before returning, could use decorators etc!
  - add instrumentation, use Prometheus
  - result: bussiness logic mixed with tons of other stuff :cry: 
- with go-kit
  - move non-bussiness stuff to middlewares
- `Endpoint` func type
- middlewares (using decorator design pattern)
- you can also decorate your own `Service`
	  
#### transport

- HTTP, you can also use it with the Gorilla mux.
- gRPC https://github.com/go-kit/kit/blob/master/examples/addsvc/transport_grpc.go
- Thrift, others...

#### dist tracing

- Zipkin (Twitter's open source distributed tracing system)
- middleware to move trace ID from HTTP header to Context.

![](https://i.imgur.com/QnEYGkv.png =512x300)
		
## Idiomatic Go Tricks
> Mat Ryer - http://matryer.com

:four: :star: :star: :star: :star:

- Idiomatic:
  - adjective: using, containing, denoting expressions.
  - talk natural go language
- Go code usually has the same appearance
  - returns a tuple, {interface, error}
  - line of sight
  - quickly scan to see what it does

### tips

  - make happy return that last statement if possible
  - next time you write else, consider flipping the logic
  - single method interfaces 
    - type func to implement the interface
  - log blocks. Prettify them adding `------` separators.
  - return teardown functions. teardown functions basically cleanup the called method garbage.
  - good timing. Basically talked about deferring stop methods on timers. Not serious business here.
  - use type assertion in order to execute one or another logic in your program
  - mocking:
    - simple mocks. Create a mock that implements  only the methods you need from the interface you want to mock.
    - third party structs. If they don't provide an interface, just create it.
  - retrying. Do not bloat the `Try` function with arguments
  - empty struct implementations.
    - `struct{}` to group methods together
  - semaphores in goroutines.
    - buffered channels. How many concurrent goroutines you want. Special dedication to Mr **William Kennedy** who encouraged avoid use them at yesterday's workshop.
  - debug logs with a build tag

### Summary
How to become a native speaker:

- read the std lib
- write obvious code
- do not surprise your users
- simplicity

### how to build go code
- monorepo
	- cross-cutting concern changes in a single commit
	- they have custom lint tools that will open source soon to solve issues :smile:
	- quick security patches appliance
- pull requests + code review
- HTTP/JSON -> gRPC
	- schemas + codegen rule. Alternative http://swagger.io/specification/
- service discovery
	- 50% in the audience using Consul, some etcd and little ZooKeeper (he says it's a nightmare)
		- Consul understands regions (in fact, they have 11) for e.g if your run in a cloud env
	- he says SD is one if the biggest wins of Microservices arch
	- Consul scales for real: they have 10K nodes.
	- DSN SRV vs API
- metrics: Prometheus cluster
	- they mix it with Consul to gather metrics for new services
- deploy
	- commit to master generates 
	  - builds
	  - tests
	  	- monorepo makes them run integration tests between microservices which code is in the same repo
	  - docker images. 
	    - created on every build.
	- feature flagging
	  - no multiple branches, no multiple versions
	- incremental rollout with Chef
	  - no blue green deployments.
- monitoring
    - they use graphite + prometheus + grafana (he encourage the use of this last one)
    	- because prometheus is pull-oriented they don't crash a push system like they did with http://opentsdb.net/
- structured logging
    - ![](http://i.imgur.com/Y8B9Ksw.jpg =512x)
    - breaking logs down into something readable. JSON formatted log as example.
    - rsyslog + elasticsearch + kibana
    - alternatives: logdna, loggly, splunk
	- logging and metrics will converge
- dist tracing
	- still sucks
	- global transaction ID + an ID per microservice => complicated
	- Zipkin => he says **it sucks** (storage and UI) Instead use opentracing.
		- go and do a good one in go
- uptime monitoring
    - via Service Discovery.
    - Nagios in every instance.

## Advanced testing concepts for Go 1.7
- Go 1.7 is out (╯°. °）╯︵ ┻┻
> The testing package now supports the definition of tests with subtests and benchmarks with sub-benchmarks. This support makes it easy to write table-driven benchmarks and to create hierarchical tests. It also provides a way to share common setup and tear-down code. See the package documentation for details. -- from 1.7 release notes
- tests
- benchmarks
	- pre 1.7: table-driven tests
	  - pros: less duplication code, less time 
	  - problem: benchmarking non-friendly, you had to create funcs and call them from the tests
	- 1.7: table-driven benchmarks 
- error messages
	- 1.7. They added message formatting. `t.Errorf`
- error/fatal/skip
	- pre 1.7: first error stops the execution
	- 1.7: multiple errors reported with the subtest that throws it
- test/benchmark isolation: how to execute just a subtest that fails 
	- pre 1.7: you do it manually commenting etc
	- 1.7: 
	  - `go test --run={test_name}/"{regex}"`
	  - every slash separates a regex.
	  - Examples:
	    - `go test --run=TestTime/"in Europe"`
	    - `go test --run=TestTime/12:[0-9]`
- test names are unique, if you don't pass a name it will generate a sequence
- SetUp and TearDown: usually frameworks emulate XUnit
  - pre 1.7: 
    ```go
    func TestFoo(b * testing.B) {
      //common set-up code

      // test

      // common tear-down code
    }
    ```
  - 1.7: They encourage us to use the following approach:
  - in the Run func start with setup and defer the teardown.
- parallelism
  - how to separate parallel tests that you don't want to run together? (maybe they allocate similar stuff etc)
   - `tc := tc` inside a concurrent loop trick :-| hope go vet detects it soon :-|++
    - otherwise the loop beeing concurrent the next concurrent exec of the loop would overwrite tc and things would go nasty
  - teardown in parallel tests
   - sync parallel tests in a group putting them together inside a func runned by `r.Run()` and after that call put the teardown func 

## A beginners guide to Context
- problems
  - each new request spawns it's own goroutine
  - goroutines don't have any "thread local" state
  - you have to care about cancellation
- solution: `Context` package
  - request scoped data
  - cancellation, deadlines & timeouts
  - it is safe for concurrent use
  - must be the first parameter, is a best practice
 
> 1.7 includes context in the core standard library

### derived contexts
  - package provides derive new Context values from existing ones
  - `Background()`: the top level context for incoming request
  - `WithCancel()`: return a copy of the parent with a new Done channel, this is closed when the cancel function is fire
  - `WithTimeout()`: returns a context with the deadline
  - ...
  
### more

- you can store whatever type you want in the context (we actually now that storing our type is not recommended at all)
  - examples:
   - `WithValues()`: add user session to the context
   - `WithTimeout()`: request to obtain the resource from an api
- In 1.7: `client.Do(request.WithContext(ctx))`
- `ctxthttp` helper package
  - context-aware HTTP requests.

## Go from Dev to Prod
- logging: structured with useful info
- building: vendor your deps but not if you write a lib
- testing: do blackbox testing using a different package name for your test (e.g instead of `demo` use `demo_test`) so you don't have access to unexported symbols from your test.
- examples: use example files as documentation.
- profiling: 
  - use benchmarks
  - you can add benchmark results as a CI step, so if something goes slow than a threshold make the build fail
 - containers: use Scratch or Alpine
 - tracing: 
   - client: go-kit's opentracing, x/net/trace
   - server: https://github.com/tracer/tracer
