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

## Design patterns in Microservices architectures and Gilmour
- services vs servers
  - services solve a software problem and a management problem
- communication design patterns
  - Request/Response
   - example with a topic layer to identify endpoints
   - confirmed delivery: response or error
   - HTTP friendly
   - sender handles errors
  - async: signals and slots
   - events -> queue sys -> event processors
    - you can scale the processors horizontally
    - e.g: Kafka, RabbitMQ...
    - patterns: fan-out, broadcast, filtering (wildcards)...
- discovery and load balancing
  - replicated load balancer + health checking
  - Gilmour does Pub/Sub, there is no discovery
- error
  - detection
   - timeout: server or client side
  - handling
   - ignore, publish...
- composition: request piping

## static deadlock detection
- concurrency
  - > don't communicate by sharing memory, share memory by communication
  - channels message passing
  - is complicated, bugs hard to find
- deadlocks
  - `all goroutines are asleep - deadlock`
  - what causes a deadlock in go?
      1. sender sends message, Receiver not ready
      2. sender blocks goroutine and wait until Receiver ready
      3. blocked goroutine goes to sleep
  - avoiding deadlocks
    - make sure sender/receiver are matching and compatible
    - then why are hard to avoid in practice?
  - go detects deadlocks at execution time and panics
  - "local deadlock" -> not all program is stuck
  - how to detect a deadlock without running the program?
- static deadlock detector: github.com/nickng/dingo-hunter
    - models the program gorutines as FSMs
    - builds a graph
    - checks for deadlocks in the graph
- modelling concurrency
  - learn from the masters: Hoare, Milner
  - *Session types* on pi-calculus

## Advanced Patterns with io.ReadWriter
- What can you find in `io`, `bufio`, `ioutil`:
  - `Reader`
  - `Writer`
  - `Copy`
  - `LimitReader`
  - `TeeReader`
  - etc
- composition is so easy
- a stream in go is a buffer that you use to read or write in chunks

### examples
- HTTP chunking. Transparently proxy a chunked HTTP in a stream.
  - `httputil.ChunkedReader`
    - problems: 
      - strips out the chunking data
      - can't validate MD5
  - `httputil.ChunkedReader` + `TeeReader`
- prefixing, split into sections
  - `os.Stdout` is a `Writer`
  - `bufio.NewScanner(input)`, `scanner.SplitFunc`, loop over `scanner.Scan`, use `scanner.Bytes`
  - input is a `Reader`
  - `io.Pipe` syncs a `Reader` and a `Writer`, if one does not "work" then it will **block** (which is handy)

## What every developer should know about logging
### logging 101
  - two levels
    - debug: lot information
    - info: understand application flow, what happens in application
  - logging infrastructure is complex

### why do we write logs
  - we want to know what our application is doing
  - good logging is the key to solve bugs
  - replace lot of third party monitoring services

### logging infrastructure
  - is not an easy task
  - workflow
    - application is logging to stdout
    - stdout is redirected to file
    - log forwarders are moving logs to centralized log server
    - log server allows to browse and analyze application bahaviours
    - its unix style, one simple step at the time
  - applications
    - logstash
    - kibana
    - summologic
    - splunk

### how to structure logs in microservices
  - ```[timestamp][RID(request id)=foobar][RD(request delta)=0.003][LD(local delta)=0.001][service=API][search][results=432]```
  - we should adapt log message structure to our application needs

### golang log libraries
  - package log
  - glog - logging library from google, allows more control on log levels

### tips and tricks
  - profile your logging system
  - log sampling: when your systems are produciong too many logs, collect only every Xth message, statiscally you will get enough logs to understand your app behaviour
  - security: never log user passwords, keys, name (does anyone do it?)
  - dapper: tracing system from google
http://research.google.com/pubs/pub36356.html

## Implementing software machines in Go & C
### virtual machines

* many types of virtual machines: system virtualization, hardware emulation
* focus: abstract virtual machines
* inspired by hardware: discrete componentes, processors, storage, communications
* software machines:
	* stateful: state matters. With no state they don't have identity
	* timely: 
	* scriptable: you need to add complex behaviour, if not, it makes no sense to have a machine in the first place
* abstract virtual machine components:
	* stacks, CPU (registers), CPU cache, Heap, Buffers, Network and persistent store
	* timings in different components: depending on heap, buffer, network, etc. different magnitudes

### Memory

* store data and instructions
	* memory model:
		* Von Neumann
		* Harvard
		* Forth (split model)
		* Hybrid
* other parts (not covered)
	* addressing
	* protection
* creating a heap in Go
	* creating sliceHeaders using slice manipulations
	* public interface: 
		* `Bytes()`
		* `Serialise()`
		* `Overwrite()`
	* usage of unsafe: to manipulate bytes is actually legitimate
* array stack:
	* it's a functional data structure which is actually used

### CPU

* dispatch loops: 
	* reading opcodes from mememory
	* change state machine internal state 
	* switch interpreter
		* tokens are often single bytes
* we need stack in order to implement the CPU
* opcodes have mnemonics
* opcodes are stored in a stack to be execute in the dispatch loop
* direct call threading
	* each instruction is represented by a pointer to a function
	* instruction require a machine word
* indirect threading
	* used to represent local jumps in the program
	* https://en.wikipedia.org/wiki/Threaded_code#Indirect_threading
* timings:
	* clock pulse
	* synchronization: provides it across the processor

### Transport triggering

* https://en.wikipedia.org/wiki/Transport_triggered_architecture	
* register machine architecture
* exposes internal buses as components
* operations are side-effects of internal writes

### Random thoughts

* C VM: primitive!
* distances in physical: they matter in computing, analogies with physicist
* it's bad to have things not initialized by default (Rust structs must be initialized all fields by default, Golang not! You want nil pointers? Use Option)
* usefulness of functional data structures
* cactus stack: https://en.wikipedia.org/wiki/Parent_pointer_tree

## Grand Treatise of Modern Instrumentation and Orchestration
See https://prometheus.io

- inspired by Google's monitoring strategy: alert for high-level stuff and allow inspecting low level.
- USE method: **U**tilization, **S**aturation and **E**rror rates. Also, latency.
- RED method: **R**equest count (rate) **E**rror count (rate) and **D**uration
- Prometheus :moneybag: :bike: 
  - expvar-like API and other types
  - you have a server and query prometheus endpoints in services
  - it gives you the typical go VM stats: goroutine number, memmory, GC etc
  - it has a query lang called PromQL
  - it gives you percentiles etc
  - counters, gauges... Counters always go up, Gauges can go down (e.g 1,2,3,4,3,4,2,1)
  - there are exporter tools around (e.g Apache)

## GoBridge and the Go Community: Initiatives and Opportunities
- http://bridgefoundry.org/
- https://golangbridge.org/
- must listen https://changelog.com/gotime-6/
- they need people, you can help!
  - code combat https://codecombat.com/about to help people learn Go. If interested talk with William Kennedy (@goinggodotnet)

## Managing and scaling Real-time Data Pipelines using Go
- works at Riot Games, better known for develop the League Of Legends (LOL) videogame.
- high success: from 7 year old startup to XXX workers and millions of players
- reliability -> monitoring at scale
  - you have to trust your monitoring systems, false alarms do the contrary
- 15+ datacenters, 20K servers, 160k services, 6.5M metrics, 100K values/second, 1TB daily volume
- microservices for the win
  - Diverse tech choices
  - Containers and metal
  - Linux, Mac and Windows
  - In house or as a service
- 2 agents by host.
  - Discovery agent
  - Metrics agent
- Endpoints by host providing data and stats in Json format.

#### The pipeline

![](https://i.imgur.com/cWwb99a.jpg)

#### design goals

  - decouple
    - pure interface abstraction. 
    - no datastore details.
    - in order to make changes easily
  - resilient
    - batching
    - draining
    - stateless
    - they receive attacks
    - what if Kafka goes down?
  - HTTP native
    - HTTP + JSON everywhere
  - instrumented
    - Metrics are key. 
  - fast
    - 75K+ values/sec on a laptop

#### go
- channels
  - they put them everywhere
  - visualizing them is hard
  - event collector pipeline: generate events, transform them, send to Kafka
- design patterns
  - rate limiting
    - buffered channel
  - caching to avoid backpressure when readers block you
  - tickers with `time.After()`
  - pipelining 
    - they use an empty struct channel (but should have used `context.Done()`)
    - `sync.WaitGroup` with `Add(1)` in every step of the pipeline and `Wait()` at the end
  - batching
    - good for client APIs
    - take advantage of buffers
      - bottleneck if ticking is very low
      - know your data: for metrics, client draining: if server fails, discard old items

## Building Cloud Native applications with Go
- cloud native evolution
  - build software in virtualized envs
  - https://cncf.io
- arch: 
  - abstractions: containers, clusters, microservices
  - container managers: docker, rkt
- Kubernetes: 
  - http://kubernetes.io/docs/whatisk8s
  - container manager + microservices
  - Manage applications, not machines

#### The flow of Deployment
![](http://i.imgur.com/vOaB1wH.jpg =600x)

#### Kubernetes features

- immutability
  - means robustness
  - containers are **immutable**
  - various immutable server images (deployed using Spinnaker) - http://www.spinnaker.io
  - from [build -> package -> deploy] to [build -> package -> **construct** -> deploy]
    - construct the immutable graph that will be deployed
    - k8s is a declarative way of configuring the construct step
- config
  - in deploy time
    - credentials: pod structure
    - env-based: k/v store
    - no flags or env vars to avoid restarting the app
- deploy
  - with config / secrets
- provides a general resource type system
  - DSL, config based on YAML
  - describes: load balancers, autoscaling, networking, clustering, zones, vm stuff, disk stuff...
    - looks like AWS API without the As a Service part
- nested deployments: example with an app with a backend and a frontend sub-deployments
- packages: based on https://github.com/kubernetes/helm
  - > Helm is a tool for managing Kubernetes charts. Charts are packages of pre-configured Kubernetes resources
  - CHART files (config files) <- Helm <- Repository (S3 as example)
- it uses 
  - etcd for storing and distribute cluster stats and configurations
    - distributed K/V, Raft-based
  - prometheus for monitoring which is integrated with the k8s API 

#### Why golang
- designed for large, distributed, collaborative software projects
- static compilation
- strong alternative to C/C++ for cross platform simple and really functional CLI tools
- low level primitives
  - networking
  - process managing
  - syscalls
- critical mass
  - hipster
  - cool
  - trendy

## Building Mobile SDKs with GoMobile
#### Creating a client SDK in Go
- the horrors of generating cross platform code
  - c/c++: "god please no"
  - xamarin: better but tooling is a pain
  - javascript: uggh
  - go moblie: rather nice
    - go can interact with c
    - compile native binaries
- why do we not write dry code?
  - we do it because dont have the time
- how would we test our client?
  - no sandbox
  - binary protocol
  - tcp sockets

#### Generationg native frameworks with Go Mobile
- protobuf (rpc)
- use ```protoc``` to generate the go code
- create a simple rpc server
- GoMobile could generate a callback to call a native client function
- every in go can run native

#### Integration the generated SDK into an Android app
- live coding
- http://github.com/gokitter <-- source code

## Seven ways to profile Go applications
1. **time**
 - GNU's (`/usr/bin/time` not `time`) `/usr/bin/time -v`
2. **go build** -x + toolexec (the latter prepends the command with the tool specified)
  - `go build -toolexec="/usr/bin/time" cmd/compile/internal/gc`
3. **GODEBUG** env variable
  - stats provided by Go
  - `env GODEBUG=gctrace=1` 
4. **Profiling**
  - tips
    - machine must be idle
    - watch out for power saving and thermal scaling
    - avoid virtual machines. too much noise.
    - don't use OSX < El Capitan
    - try to use dedicated performance test hardware
    - run multiple times (OS has several layers of cache and needs warmup etc)
    
4.1. **pprof**
  - descends from Google Performance Tools suite 
  - CPU profiling
  - memory profiling
    - heap allocations (in contrast to stack allocations which are "free", heap ones are costly)
    - Bill tip: don't use htop & friends
  - block profiling
    - aka backpressure measurement?
    - similar to CPU profile but records the amount of time a goroutine spent waiting shared resources.
  - do not mix profiling types! 
  - microbenchmarks
    - `go test -run=XXX -bench=IndexByte -cpuprofile=/tmp/c.p bytes`
      - XXX trick does the test just to run the benchmarks and not the tests
    - https://github.com/pkg/profile
      - good for profiling whole programs from start to end, you put the code in your program
  - reports can be exported to visual graphs
    - in the example, SVG version, the biggest box was a syscall for reading from the network. He said you can avoid to with buffering. 
5. **perf**
  - linux tool, integrate with toolexec
    - what we use on osx? Maybe [this apple tool](https://developer.apple.com/library/tvos/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide) can help.
  - looks awesome
  - percent on the left is how expensive the op was
  - framepointers, Go >=1.7 at least. otherwise doesn't look very good.
6. **flame graph** from netflix
  - func calls are just displayed once and grouped by size
7. **Go tool trace**
  - goroutines
    -  creation/start/end
    -  blocking/unblocking
  -  network
  -  system calls

## Building your own log-based message queue in Go

Good read https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying

- logs 
  - > append only, totally-ordered sequence of records ordered by time --LinkedIn eng blog
  - very simple but also very fast to read and write in disk
  - strong ordering: not all message queues assure it but logs do
  - DDBBs usually write to logs first before writing to the final data structure they use
- Kafka
  - super performant, battle tested, LinkedIn
  - O(1) reads and writes
    - see https://kafka.apache.org/08/design.html, "Constant Time Suffices"
  - distributed, HA, uses ZooKeeper
  - sometimes is too much for some cases -> **lets build one in Go**
- building blocks 
  - indexes
  - readers
    - file readers on demand implementing `io.Reader`
  - watcher: subscribers with channel
  - scanner: map in a buffered way to iterate over the log records
  - streamer: if you need higher throughput than the scanner
- result: BigLog https://github.com/ninibe/netlog/tree/master/biglog  
- demo
  - API
  - simple demo with curl client against the HTTP API
  - pub/sub example

## Real-Time Go
- GC-based langs don't have good reputation in RT systems
- talk is about building a RT bidding system with Go
- RT definition
  - something executed very fast? nop
  - > system... finite and specified period
  - aka **deadline**
- types
  - hard: e.g patience analysis in medic systems
    - if deadline is missed => :bomb:
  - form: occasional deadline misses are tolerable although the result is not useful
  - soft: usefulness degrades with bigger deadline misses
    - e.g 3D FPS game lag (PWNT :-)
    - cam control systems in rooms
- RT bidding
  - automated 2nd price auctions
  - 100-120ms
  - Nash equilibrium, game theory algos
- Travel Audience (Amadeus company)
  - itermediary between publishers and users
  - they do RT bidding
  - segment users based on travel destinations
- "The Problem"
  - intersect user interest and available offers 
- iterations
  - C lang, 2K QPS + memcached
    - never worked really well
  - go version
    - some deadlines missed
    - solution: set an internal deadline of 75ms
    - use `time.After`, also `context.CancelFunc`
  - storage:
    - first memcached
    - then MongoDB
      - https://aphyr.com/posts/322-jepsen-mongodb-stale-reads
      - external DDBB adds net latency
    - then Redis

This below also collects common comments made during reviews of Go code and some best practices collected from different sources.

You can view this as a supplement to [Effective Go](https://golang.org/doc/effective_go.html).

## Gofmt

Run [gofmt](https://golang.org/cmd/gofmt/) on your code to automatically fix the majority of mechanical style issues. Almost all Go code in the wild uses `gofmt`. The rest of this document addresses non-mechanical style points.

An alternative is to use [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports), a superset of `gofmt` which additionally adds (and removes) import lines as necessary.

As a developer, we often forget to run `go fmt` hence the best way to avoid pushing your code with a bad format is to configure your IDE to run `go fmt` whenever the code is saved.

## Go vet

Run `go vet` on your code to check if any suspicious piece of code before every single commit. Vet will examine your code and reports suspicious constructs, such as Printf calls whose arguments do not align with the format string. Vet uses heuristics that do not guarantee all reports are genuine problems, but it can find errors not caught by the compilers. This is very useful as we - developers some time make silly mistakes when we are in rush :)

Run`go vet` for all of sub-packages of your code by following command:

`go vet ./...`

## Comment Sentences

See <https://golang.org/doc/effective_go.html#commentary>. Comments documenting declarations should be full sentences, even if that seems a little redundant. This approach makes them format well when extracted into godoc documentation. Comments should begin with the name of the thing being described and end in a period:

```go
// Request represents a request to run a command.
type Request struct { ...

// Encode writes the JSON encoding of req to w.
func Encode(w io.Writer, req *Request) { ...
```

## Contexts

Values of the context.Context type carry security credentials, tracing information, deadlines, and cancellation signals across API and process boundaries. Go programs pass Contexts explicitly along the entire function call chain from incoming RPCs and HTTP requests to outgoing requests.

Most functions that use a Context should accept it as their first parameter:

```go
func F(ctx context.Context, /* other arguments */) {}
```

A function that is never request-specific may use context.Background(), but err on the side of passing a Context even if you think you don't need to. The default case is to pass a Context; only use context.Background() directly if you have a good reason why the alternative is a mistake.

Don't add a Context member to a struct type; instead add a ctx parameter to each method on that type that needs to pass it along. The one exception is for methods whose signature must match an interface in the standard library or in a third party library.

Don't create custom Context types or use interfaces other than Context in function signatures.

If you have application data to pass around, put it in a parameter, in the receiver, in globals, or, if it truly belongs there, in a Context value.

Contexts are immutable, so it's fine to pass the same ctx to multiple calls that share the same deadline, cancellation signal, credentials, parent trace, etc.

Read more about context at:

1. [Context isn’t for cancellation](https://dave.cheney.net/2017/08/20/context-isnt-for-cancellation)
2. [Context is for cancelation](https://dave.cheney.net/2017/01/26/context-is-for-cancelation)

## Copying

To avoid unexpected aliasing, be careful when copying a struct from another package. For example, the bytes.Buffer type contains a `[]byte` slice and, as an optimization for small strings, a small byte array to which the slice may refer. If you copy a `Buffer`, the slice in the copy may alias the array in the original, causing subsequent method calls to have surprising effects.

In general, do not copy a value of type `T` if its methods are associated with the pointer type, `*T`.

## Declaring Empty Slices

When declaring an empty slice, prefer

```go
var t []string
```

over

```go
t := []string{}
```

The former declares a nil slice value, while the latter is non-nil but zero-length. They are functionally equivalent—their `len` and `cap` are both zero—but the nil slice is the preferred style.

Note that there are limited circumstances where a non-nil but zero-length slice is preferred, such as when encoding JSON objects (a `nil` slice encodes to `null`, while `[]string{}` encodes to the JSON array `[]`).

When designing interfaces, avoid making a distinction between a nil slice and a non-nil, zero-length slice, as this can lead to subtle programming errors.

For more discussion about nil in Go see Francesc Campoy's talk [Understanding Nil](https://www.youtube.com/watch?v=ynoY2xz-F8s).

## Crypto Rand

Do not use package `math/rand` to generate keys, even throwaway ones. Unseeded, the generator is completely predictable. Seeded with `time.Nanoseconds()`, there are just a few bits of entropy. Instead, use `crypto/rand`'s Reader, and if you need text, print to hexadecimal or base64:

```go
import (
    "crypto/rand"
    // "encoding/base64"
    // "encoding/hex"
    "fmt"
)

func Key() string {
    buf := make([]byte, 16)
    _, err := rand.Read(buf)
    if err != nil {
        panic(err)  // out of randomness, should never happen
    }
    return fmt.Sprintf("%x", buf)
    // or hex.EncodeToString(buf)
    // or base64.StdEncoding.EncodeToString(buf)
}
```

## Doc Comments

All top-level, exported names should have doc comments, as should non-trivial unexported type or function declarations. See <https://golang.org/doc/effective_go.html#commentary> for more information about commentary conventions.

## Don't Panic

See <https://golang.org/doc/effective_go.html#errors>. Don't use panic for normal error handling. Use error and multiple return values.

## Error Strings

Error strings should not be capitalized (unless beginning with proper nouns or acronyms) or end with punctuation, since they are usually printed following other context. That is, use `fmt.Errorf("something bad")` not `fmt.Errorf("Something bad")`, so that `log.Printf("Reading %s: %v", filename, err)` formats without a spurious capital letter mid-message. This does not apply to logging, which is implicitly line-oriented and not combined inside other messages.

## Examples

When adding a new package, include examples of intended usage: a runnable Example, or a simple test demonstrating a complete call sequence.

Read more about [testable Example() functions](https://blog.golang.org/examples).

## Goroutine Lifetimes

When you spawn goroutines, make it clear when - or whether - they exit.

Goroutines can leak by blocking on channel sends or receives: the garbage collector will not terminate a goroutine even if the channels it is blocked on are unreachable.

Even when goroutines do not leak, leaving them in-flight when they are no longer needed can cause other subtle and hard-to-diagnose problems. Sends on closed channels panic. Modifying still-in-use inputs "after the result isn't needed" can still lead to data races. And leaving goroutines in-flight for arbitrarily long can lead to unpredictable memory usage.

Try to keep concurrent code simple enough that goroutine lifetimes are obvious. If that just isn't feasible, document when and why the goroutines exit.

## Handle Errors

Do not discard errors using `_` variables. If a function returns an error, check it to make sure the function succeeded. Handle the error, return it, or, in truly exceptional situations, panic.

When checking an error, don't check for a specific text in error, check its type instead. Don't write:
```go
if strings.Contains(err.Error(), "timeout"){
    // do something
}
```

Instead, write:
```go
if nerr, ok := err.(net.Error); ok && nerr.Timeout() {
    // do something
}
if err != nil {
    log.Fatal(err)
}
```

If you are using Go 1.13 afterward, better look at https://blog.golang.org/go1.13-errors. It offers a very nice way to wrap, check error.

Read more about error at:
1. [Effective Go - Errors](https://golang.org/doc/effective_go.html#errors)
1. [Error handling and go](https://blog.golang.org/error-handling-and-go)
2. [Errors are values](https://blog.golang.org/errors-are-values)

## Imports

Avoid renaming imports except to avoid a name collision; good package names should not require renaming. In the event of collision, prefer to rename the most local or project-specific import.

Imports are organized in groups, with blank lines between them. The standard library packages are always in the first group.

```go
package main

import (
	"fmt"
	"hash/adler32"
	"os"

	"appengine/foo"
	"appengine/user"

        "github.com/foo/bar"
	"rsc.io/goversion/version"
)
```

[goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) will do this for you.

## In-Band Errors

In C and similar languages, it's common for functions to return values like -1 or null to signal errors or missing results:

```go
// Lookup returns the value for key or "" if there is no mapping for key.
func Lookup(key string) string

// Failing to check a for an in-band error value can lead to bugs:
Parse(Lookup(key))  // returns "parse failure for value" instead of "no value for key"
```

Go's support for multiple return values provides a better solution. Instead of requiring clients to check for an in-band error value, a function should return an additional value to indicate whether its other return values are valid. This return value may be an error, or a boolean when no explanation is needed. It should be the final return value.

```go
// Lookup returns the value for key or ok=false if there is no mapping for key.
func Lookup(key string) (value string, ok bool)
```

This prevents the caller from using the result incorrectly:

```go
Parse(Lookup(key))  // compile-time error
```

And encourages more robust and readable code:

```go
value, ok := Lookup(key)
if !ok  {
    return fmt.Errorf("no value for %q", key)
}
return Parse(value)
```

This rule applies to exported functions but is also useful for unexported functions.

Return values like nil, "", 0, and -1 are fine when they are valid results for a function, that is, when the caller need not handle them differently from other values.

Some standard library functions, like those in package "strings", return in-band error values. This greatly simplifies string-manipulation code at the cost of requiring more diligence from the programmer. In general, Go code should return additional values for errors.

## Indent Error Flow

Try to keep the normal code path at a minimal indentation, and indent the error handling, dealing with it first. This improves the readability of the code by permitting visually scanning the normal path quickly. For instance, don't write:

```go
if err != nil {
	// error handling
} else {
	// normal code
}
```

Instead, write:

```go
if err != nil {
	// error handling
	return // or continue, etc.
}
// normal code
```

If the `if` statement has an initialization statement, such as:

```go
if x, err := f(); err != nil {
	// error handling
	return
} else {
	// use x
}
```

then this may require moving the short variable declaration to its own line:

```go
x, err := f()
if err != nil {
	// error handling
	return
}
// use x
```

## Initialisms

Words in names that are initialisms or acronyms (e.g. "URL" or "NATO") have a consistent case. For example, "URL" should appear as "URL" or "url" (as in "urlPony", or "URLPony"), never as "Url". As an example: ServeHTTP not ServeHttp. For identifiers with multiple initialized "words", use for example "xmlHTTPRequest" or "XMLHTTPRequest".

This rule also applies to "ID" when it is short for "Identity Document" (which is pretty much all cases when it's not the "id" as in "ego", "superego"), so write "appID" instead of "appId".

Code generated by the protocol buffer compiler is exempt from this rule. Human-written code is held to a higher standard than machine-written code.

## Interfaces

Go interfaces generally belong in the package that uses values of the interface type, not the package that implements those values. The implementing package should return concrete (usually pointer or struct) types: that way, new methods can be added to implementations without requiring extensive refactoring.

Do not define interfaces on the implementor side of an API "for mocking"; instead, design the API so that it can be tested using the public API of the real implementation.

Do not define interfaces before they are used: without a realistic example of usage, it is too difficult to see whether an interface is even necessary, let alone what methods it ought to contain.

```go
package consumer  // consumer.go

type Thinger interface { Thing() bool }

func Foo(t Thinger) string { … }
package consumer // consumer_test.go

type fakeThinger struct{ … }
func (t fakeThinger) Thing() bool { … }
…
if Foo(fakeThinger{…}) == "x" { … }
// DO NOT DO IT!!!
package producer

type Thinger interface { Thing() bool }

type defaultThinger struct{ … }
func (t defaultThinger) Thing() bool { … }

func NewThinger() Thinger { return defaultThinger{ … } }
```

Instead return a concrete type and let the consumer mock the producer implementation.

```go
package producer

type Thinger struct{ … }
func (t Thinger) Thing() bool { … }

func NewThinger() Thinger { return Thinger{ … } }
```

## Line Length

There is no rigid line length limit in Go code, but avoid uncomfortably long lines. Similarly, don't add line breaks to keep lines short when they are more readable long--for example, if they are repetitive.

Most of the time when people wrap lines "unnaturally" (in the middle of function calls or function declarations, more or less, say, though some exceptions are around), the wrapping would be unnecessary if they had a reasonable number of parameters and reasonably short variable names. Long lines seem to go with long names, and getting rid of the long names helps a lot.

In other words, break lines because of the semantics of what you're writing (as a general rule) and not because of the length of the line. If you find that this produces lines that are too long, then change the names or the semantics and you'll probably get a good result.

This is, actually, exactly the same advice about how long a function should be. There's no rule "never have a function more than N lines long", but there is definitely such a thing as too long of a function, and of too stuttery tiny functions, and the solution is to change where the function boundaries are, not to start counting lines.

## Mixed Caps

See <https://golang.org/doc/effective_go.html#mixed-caps>. This applies even when it breaks conventions in other languages. For example an unexported constant is `maxLength` not `MaxLength` or `MAX_LENGTH`.

Also see [Initialisms](https://github.com/golang/go/wiki/CodeReviewComments#initialisms).

## Named Result Parameters

Consider what it will look like in godoc. Named result parameters like:

```go
func (n *Node) Parent1() (node *Node)
func (n *Node) Parent2() (node *Node, err error)
```

will stutter in godoc; better to use:

```go
func (n *Node) Parent1() *Node
func (n *Node) Parent2() (*Node, error)
```

On the other hand, if a function returns two or three parameters of the same type, or if the meaning of a result isn't clear from context, adding names may be useful in some contexts. Don't name result parameters just to avoid declaring a var inside the function; that trades off a minor implementation brevity at the cost of unnecessary API verbosity.

```go
func (f *Foo) Location() (float64, float64, error)
```

is less clear than:

```go
// Location returns f's latitude and longitude.
// Negative values mean south and west, respectively.
func (f *Foo) Location() (lat, long float64, err error)
```

Naked returns are okay if the function is a handful of lines. Once it's a medium sized function, be explicit with your return values. Corollary: it's not worth it to name result parameters just because it enables you to use named returns. Clarity of docs is always more important than saving a line or two in your function.

Finally, in some cases you need to name a result parameter in order to change it in a deferred closure. That is always OK.

## Naked Returns

See [Named Result Parameters](https://github.com/golang/go/wiki/CodeReviewComments#named-result-parameters).

## Package Comments

Package comments, like all comments to be presented by godoc, must appear adjacent to the package clause, with no blank line.

```go
// Package math provides basic constants and mathematical functions.
package math
/*
Package template implements data-driven templates for generating textual
output such as HTML.
....
*/
package template
```

For "package main" comments, other styles of comment are fine after the binary name (and it may be capitalized if it comes first), For example, for a `package main` in the directory `seedgen` you could write:

```go
// Binary seedgen ...
package main
```

or

```go
// Command seedgen ...
package main
```

or

```go
// Program seedgen ...
package main
```

or

```go
// The seedgen command ...
package main
```

or

```go
// The seedgen program ...
package main
```

or

```go
// Seedgen ..
package main
```

These are examples, and sensible variants of these are acceptable.

Note that starting the sentence with a lower-case word is not among the acceptable options for package comments, as these are publicly-visible and should be written in proper English, including capitalizing the first word of the sentence. When the binary name is the first word, capitalizing it is required even though it does not strictly match the spelling of the command-line invocation.

See <https://golang.org/doc/effective_go.html#commentary> for more information about commentary conventions.

## Package Names

All references to names in your package will be done using the package name, so you can omit that name from the identifiers. For example, if you are in package chubby, you don't need type ChubbyFile, which clients will write as `chubby.ChubbyFile`. Instead, name the type `File`, which clients will write as `chubby.File`. Avoid meaningless package names like util, common, misc, api, types, and interfaces. See <http://golang.org/doc/effective_go.html#package-names> and<http://blog.golang.org/package-names> for more.

## Pass Values

Don't pass pointers as function arguments just to save a few bytes. If a function refers to its argument `x` only as `*x` throughout, then the argument shouldn't be a pointer. Common instances of this include passing a pointer to a string (`*string`) or a pointer to an interface value (`*io.Reader`). In both cases the value itself is a fixed size and can be passed directly. This advice does not apply to large structs, or even small structs that might grow.

## Receiver Names

The name of a method's receiver should be a reflection of its identity; often a one or two letter abbreviation of its type suffices (such as "c" or "cl" for "Client"). Don't use generic names such as "me", "this" or "self", identifiers typical of object-oriented languages that gives the method a special meaning. In Go, the receiver of a method is just another parameter and therefore, should be named accordingly. The name need not be as descriptive as that of a method argument, as its role is obvious and serves no documentary purpose. It can be very short as it will appear on almost every line of every method of the type; familiarity admits brevity. Be consistent, too: if you call the receiver "c" in one method, don't call it "cl" in another.

## Receiver Type

Choosing whether to use a value or pointer receiver on methods can be difficult, especially to new Go programmers. If in doubt, use a pointer, but there are times when a value receiver makes sense, usually for reasons of efficiency, such as for small unchanging structs or values of basic type. Some useful guidelines:

- If the receiver is a map, func or chan, don't use a pointer to them. If the receiver is a slice and the method doesn't reslice or reallocate the slice, don't use a pointer to it.
- If the method needs to mutate the receiver, the receiver must be a pointer.
- If the receiver is a struct that contains a sync.Mutex or similar synchronizing field, the receiver must be a pointer to avoid copying.
- If the receiver is a large struct or array, a pointer receiver is more efficient. How large is large? Assume it's equivalent to passing all its elements as arguments to the method. If that feels too large, it's also too large for the receiver.
- Can function or methods, either concurrently or when called from this method, be mutating the receiver? A value type creates a copy of the receiver when the method is invoked, so outside updates will not be applied to this receiver. If changes must be visible in the original receiver, the receiver must be a pointer.
- If the receiver is a struct, array or slice and any of its elements is a pointer to something that might be mutating, prefer a pointer receiver, as it will make the intention more clear to the reader.
- If the receiver is a small array or struct that is naturally a value type (for instance, something like the time.Time type), with no mutable fields and no pointers, or is just a simple basic type such as int or string, a value receiver makes sense. A value receiver can reduce the amount of garbage that can be generated; if a value is passed to a value method, an on-stack copy can be used instead of allocating on the heap. (The compiler tries to be smart about avoiding this allocation, but it can't always succeed.) Don't choose a value receiver type for this reason without profiling first.
- Finally, when in doubt, use a pointer receiver.

## Synchronous Functions

Prefer synchronous functions - functions which return their results directly or finish any callbacks or channel ops before returning - over asynchronous ones.

Synchronous functions keep goroutines localized within a call, making it easier to reason about their lifetimes and avoid leaks and data races. They're also easier to test: the caller can pass an input and check the output without the need for polling or synchronization.

If callers need more concurrency, they can add it easily by calling the function from a separate goroutine. But it is quite difficult - sometimes impossible - to remove unnecessary concurrency at the caller side.

## Useful Test Failures

Tests should fail with helpful messages saying what was wrong, with what inputs, what was actually got, and what was expected. It may be tempting to write a bunch of assertFoo helpers, but be sure your helpers produce useful error messages. Assume that the person debugging your failing test is not you, and is not your team. A typical Go test fails like:

```go
if got != tt.want {
	t.Errorf("Foo(%q) = %d; want %d", tt.in, got, tt.want) // or Fatalf, if test can't test anything more past this point
}
```

Note that the order here is actual != expected, and the message uses that order too. Some test frameworks encourage writing these backwards: 0 != x, "expected 0, got x", and so on. Go does not.

If that seems like a lot of typing, you may want to write a [table-driven test](https://github.com/golang/go/wiki/TableDrivenTests).

Another common technique to disambiguate failing tests when using a test helper with different input is to wrap each caller with a different TestFoo function, so the test fails with that name:

```go
func TestSingleValue(t *testing.T) { testHelper(t, []int{80}) }
func TestNoValues(t *testing.T)    { testHelper(t, []int{}) }
```

In any case, the onus is on you to fail with a helpful message to whoever's debugging your code in the future.

## Variable Names

Variable names in Go should be short rather than long. This is especially true for local variables with limited scope. Prefer `c` to `lineCount`. Prefer `i` to `sliceIndex`.

The basic rule: the further from its declaration that a name is used, the more descriptive the name must be. For a method receiver, one or two letters is sufficient. Common variables such as loop indices and readers can be a single letter (`i`, `r`). More unusual things and global variables need more descriptive names.

## Use Makefile for common commands

Use a Makefile to collect common commands that every developer in your projects need to execute every day. You can include fundamental commands like `go fmt`, `go vet`, `go test` or building docker images there. 

Here is an example:

```shell
SHELL:=/bin/bash
PROJECT_NAME=myapp
GO_BUILD_ENV=CGO_ENABLED=0 GOOS=linux GOARCH=amd64
GO_FILES=$(shell go list ./... | grep -v /vendor/)

BUILD_VERSION=$(shell cat VERSION)
BUILD_TAG=$(BUILD_VERSION)
DOCKER_IMAGE=$(PROJECT_NAME):$(BUILD_TAG)

.SILENT:

all: fmt vet install test

build:
	$(GO_BUILD_ENV) go build -v -o $(PROJECT_NAME).bin .

install:
	$(GO_BUILD_ENV) go install

vet:
	go vet $(GO_FILES)

fmt:
	go fmt $(GO_FILES)

test:
	go test $(GO_FILES) -cover

integration_test:
	go test -tags=integration $(GO_FILES)

docker: build
	docker build -t $(DOCKER_IMAGE) .;\
        rm -f $(PROJECT_NAME).bin 2> /dev/null; \
```

Once the Makefile is created, everyone just need to type simple command like: `make all`, `make build` or `make docker`.

## Don't use default http.Client & http.Server in production

Don't use default http.Client in production code. The `http.DefaultClient` is very convenient as you can very easily do HTTP request by `http.Get(url)` or `http.DefaultClient.Get(url)` without the need to initialize the client yourself but it should not be used for production. 

The reason is the default client is created with Timeout is 0, which means `no timeout`. This means when the target server has outage or something like that, the connection from the client will be hang forever and will cause the user request (if one is associated) to be hang as well.

The best way to avoid the above problem is to define a custom http.Client with timeout and reuse it in your application:

```go
var c = &http.Client{
  Timeout: time.Second * 10,
}
res, err := c.Get(url)
```

Or you can control more over the transport by creating a custom one:

```go
var ts = &http.Transport{
  Dial: (&net.Dialer{
    Timeout: 5 * time.Second,
  }).Dial,
  TLSHandshakeTimeout: 5 * time.Second,
}
var c = &http.Client{
  Timeout: time.Second * 10,
  Transport: ts,
}
res, err := c.Get(url)
```

The same is applied for `http.Server`, create a custom server for using in production instead of using the default one:

```go
server := http.Server{
	Addr:         *addr,
	ReadTimeout:  30 * time.Second,
	WriteTimeout: 30 * time.Second,
}
if err := server.ListenAndServe(); err != nil {
	log.Panicf("main: listening on %s failed: %v", server.Addr, err)
}
```

Read more about this at:

1. [Don't use default http client](https://medium.com/@nate510/don-t-use-go-s-default-http-client-4804cb19f779)
2. [The complete guide to golang http timeout](https://blog.cloudflare.com/the-complete-guide-to-golang-net-http-timeouts/)

## Add context to errors

Wrapping errors with a custom message provides context as it gets propagated up the stack. Make sure the root error is still accessible somehow for type checking.

Don't write:

```go
file, err := os.Open("foo.txt")
if err != nil {
	return err
}
```

Instead, write:

```go
import "github.com/pkg/errors"

// ...

file, err := os.Open("foo.txt")
if err != nil {
	return errors.Wrap(err, "failed to open foo.txt")
}
```

If you are using Go 1.13 afterward, better look at https://blog.golang.org/go1.13-errors. It offers a very nice way to wrap, check error.
## Avoid global variables

Global variables make testing and readability hard. It also makes everyone in the same package can access it even they don't need it and should not able to access to it. The best practice is to set it as a dependency of a struct and use dependency injection to inject it whenever you need it (often in main.go)

So, don't write:

```go
var db *sql.DB

func main() {
	db = // ...
	http.HandleFunc("/drop", DropHandler)
	// ...
}

func DropHandler(w http.ResponseWriter, r *http.Request) {
	db.Exec("DROP DATABASE prod")
}
```

Instead, use dependency injection to inject it into the struct:

```go
func main() {
	db := // ...
	handlers := Handlers{DB: db}
	http.HandleFunc("/drop", handlers.DropHandler)
	// ...
}

type Handlers struct {
	DB *sql.DB
}

func (h *Handlers) DropHandler(w http.ResponseWriter, r *http.Request) {
	h.DB.Exec("DROP DATABASE prod")
}
```

## Project Structure/Layout

It's always a good idea to structure your code to make it readable and extensible.
If you're writing a library/framework, source code of the Go SDK is a good place to look at. But if you are writing an application which composed of different modules, it's recommended to look at [go-project-layout]( https://github.com/golang-standards/project-layout). A sample implementation of the layout can be found at the [robusta-project](https://github.com/pthethanh/robusta).

For beginner: A simple approach to check if you have a good code, good design is to write unit test for your code. Most beginner write their code and cannot even write unit test for their code because of complexity of the code/design, especially when external dependencies such as database, mail, message queue, ....  are needed.



## References

1. [Effective Go](https://golang.org/doc/effective_go.html)
2. [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
3. [Best practices for a new go developer](https://blog.rubylearning.com/best-practices-for-a-new-go-developer-8660384302fc)
4. [Error handling in Go](https://blog.golang.org/error-handling-and-go)
5. [Errors are values](https://blog.golang.org/errors-are-values)
6. [Slice tricks](https://github.com/golang/go/wiki/SliceTricks)
7. [Table driven testing](https://github.com/golang/go/wiki/TableDrivenTests)
8. [HTTP services best practices - Mat Ryer](https://medium.com/statuscode/how-i-write-go-http-services-after-seven-years-37c208122831)
9. [Go best practices - Peter Bourgon](https://peter.bourgon.org/go-best-practices-2016/)
10. [Twelve Go Best Practices -Francesc Campoy Flores](https://talks.golang.org/2013/bestpractices.slide#1)
11. [Don’t use Go’s default HTTP client in production](https://medium.com/@nate510/don-t-use-go-s-default-http-client-4804cb19f779)
12. [Go project layout](https://medium.com/golang-learn/go-project-layout-e5213cdcfaa2)
13. [Building APIs - Mat Ryer](https://go-talks.appspot.com/github.com/matryer/golanguk/building-apis.slide)
14. [Context isn't for cancellation - Dave Cheney](https://dave.cheney.net/2017/08/20/context-isnt-for-cancellation)
15. [The package level logger anti pattern - Dave Cheney](https://dave.cheney.net/2017/01/23/the-package-level-logger-anti-pattern)

