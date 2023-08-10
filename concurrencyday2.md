
# Hello Hata
## Lesson plan 08-08-2023 - v 0.2

Vscode mix new project_name.
Add in the example modules.
Open an iEX shell as well.

## 1. Modules and Functions

Modules - Think of modules like folders that group related functions.
Functions - Individual operations that can be performed.

## 2. Concurrency in Elixir

### Introduction to the Actor Model

Elixir uses the Actor Model for concurrency. Each actor (process) is an independent unit that maintains its state and communicates with other actors via messages.

### Processes
What is a process? -  Processes in Elixir are lightweight. They're isolated from each other, which means a failure in one process doesn't affect another.
Spawn and Spawn_link - To create new processes.
Process identifiers (PIDs) - A unique ID for each process.

Open the iEX:

    >> pid = spawn(fn -> IO.puts "hi padawan" end)


-> Does this make sense? 
We run asynchronously, the inline function fn -> IO.puts "hi" end

Now run this 

    >> Process.alive?(pid) 

What does it say?

Why does it say this? >> Because the async function was executed, and then ended
Once hi was output, the fn was complete.
The Erlang Runtime Garbage Collecter collects whatever is associated to that PID.

Now, lets introduce a Process.seep of 10seconds and lets repeat the Process.alive?

this way, 
	
    pid = spawn(fn -> Process.sleep(10000) && IO.puts "hi padawan" end

Recheck the pid with Process.alive?(pid)
Alive or not?

**What happens after 10 seconds?**

**What does this mean?** 
No Erlang process, associated to that pid is still executing async.

So, spawn is a way to run something asynchronously.

If we use spawn instead of spawn_link, then if the function called crashes, the caller does not crash
but if it is called with spawn link?
Try it.....

The called and the caller processes are LINKED.
(Under the hood, their supervisors are linked)

Read this:
https://elixir-lang.org/getting-started/processes.html#send-and-receive

Example: Create a simple process using spawn that prints a message.

Go to the iex and type:

    defmodule SimpleProcess do
      def print_message do
        IO.puts("Hello from a spawned process!")
      end
    end

    pid = spawn(SimpleProcess, :print_message, [])

Ok async functions don't always have a sleep, 

They can also "wait for a message from the outside", using the receive statement:

Lets remake the function, paste this in IEX:

    defmodule Foo do
      def receiver() do
        receive do
          v -> IO.inspect v
               receiver()
        end
      end
    end

Now, lets call it with spawn using a function:

    >> pid = spawn(Foo, :receiver, [])

This means : run the function on the Foo module , the function name as an 
atom ( has to be an atom ) and if the func has parameters we put it in the 
third parameter of spawn as list.  In this case the func has two parameters 
then is [].

...You can check the function is running...
Now, lets make the function stop by:

    Send : >> pid, "Hey you"

does this make sense?

After the flow goes into IO.inspect v, the func reaches the end, and the 
process is finished.

Now, lets update the module this way:

    defmodule Foo do
      def receiver() do
        receive do
          v -> IO.inspect v
               receiver()
        end
      end
    end

Start it again with:

    >> pid = spawn(Foo, :receiver, [])

Now, you can send multiple messages to the same living process,
e.g:

    iex(11)> send pid, :foo
    :foo
    :foo
    iex(12)> send pid, :bar
    :bar
    :bar
    iex(13)> send pid, :baz
    :baz
    :baz
    iex(14)>

and Process.alive?(pid) is true

    iex(14)> Process.alive?(pid)
    true

Ways to kill the process?

    iex(15)> Process.delete(pid)
    nil
    iex(16)> Process.alive?(pid)
    true

Its still alive after delete, because Process.delete only sends stop 
signal or message to the process and the func or module does not 
have a receive clause to handle a stop signal.


## 3. Message Passing in Processes

https://samuelmullen.com/articles/elixir-processes-send-and-receive

In Elixir, processes don't share state. They communicate by sending and receiving messages, ensuring data consistency.
Sending and receiving messages between processes
Introduction to recursive processes for continuous message listening

### Example: This is the greeter module.

     defmodule Greeter do
      def start do
        spawn(fn -> loop() end)
      end
    
      def loop do
        receive do
          {:greet, name} ->
            IO.puts("Hello, #{name}!")
            loop()
        end
      end
    end
    
    pid = Greeter.start()
    send pid, {:greet, "Alice"}

1.  **Spawning the Process**: When `Greeter.start()` is called, it spawns a new process running the `loop` function. This allows the process to run independently and concurrently with other processes.
    
2.  **Receiving Messages**: The spawned process enters a waiting state inside the `loop` function, expecting to receive messages that match the pattern `{:greet, name}`.
    
3.  **Handling Messages**: When a message with the correct pattern is sent to the process (e.g., `send pid, {:greet, "Alice"}`), the `receive` block inside the `loop` function matches it, and the corresponding code is executed. In this case, it prints "Hello, Alice!" to the console.
    
4.  **Recursion**: After handling a message, the `loop` function calls itself recursively. This enables the process to continue waiting for and handling additional messages. The recursion ensures that the process stays alive and responsive to further messages.
    
5.  **Concurrency**: Multiple instances of the `Greeter` process can be spawned simultaneously, and they will operate independently of each other. They can all receive and handle messages concurrently.

#### Exercise 1:

-   Modify the Greeter module to handle different types of greetings (e.g., Good Morning, Good Evening) based on messages sent to the process.

## 4. Process Management

Process Links and process dictionaries
-  Linking processes means if one fails, the linked processes can be notified. Process dictionaries are like scratch spaces for processes, but their use is generally discouraged.

-  Example: Linked Process

        defmodule LinkedProcess do
          def start_link do
            spawn_link(fn -> loop() end)
          end
        
          def loop do
            receive do
              :exit -> exit(:normal)
              msg -> IO.puts("Received: #{msg}")
                loop()
            end
          end
        end
    
    pid = LinkedProcess.start_link()
    Process.exit(pid, :kill) # Will kill the linked process


Process state and life cycle
https://ibb.co/Q92h9zT

- How to handle errors and gracefully shut down a process.
Error handling and process termination

### Exercise Question : Linked Process

Consider the above example of a linked process. Modify the `LinkedProcess` module to include a function that sends messages to the loop. Then, write code to spawn the linked process, send it several messages, and finally kill it.

-   **Hint**: You can use the `send/2` function to send messages to a process by its PID.
-   **Goal**: Practice working with linked processes, message passing, and process termination.

## 5. GenServer - Elixir's Client-Server Model

**Introduction to GenServer**
- GenServer abstracts common client-server interactions, helping with state management and handling requests.
- Basically a process with all the convenience functions built-in
<b>Callbacks</b>: init/1, handle_call/3, handle_cast/2

### More on GenServer :
  - https://elixir-lang.org/getting-started/mix-otp/genserver.html

### CheatSheet :

   - Doc :  https://elixir-lang.org/downloads/cheatsheets/gen-server.pdf
 





**Synchronous vs. Asynchronous calls**

### Example 1 : 
Implement a basic GenServer that holds and manipulates a state (e.g., a counter).

    defmodule Counter do
      use GenServer
    
      # Client API
      def start_link(initial_count) do
        GenServer.start_link(__MODULE__, initial_count)
      end
    
      def increment(pid) do
        GenServer.cast(pid, :increment)
      end
    
      def value(pid) do
        GenServer.call(pid, :value)
      end
    
      # Server Callbacks
      def handle_cast(:increment, count) do
        {:noreply, count + 1}
      end
    
      def handle_call(:value, _from, count) do
        {:reply, count, count}
      end
    end

    # Usage
    {:ok, pid} = Counter.start_link(0)
    Counter.increment(pid)
    IO.puts(Counter.value(pid)) # Outputs 1


### Example 2 :

 <b>Elixir School lesson ( Queue example ) :</b>

   - https://elixirschool.com/en/lessons/advanced/otp_concurrency

 ### Example 3 :
   <b>Intro to Genserver (Shopping Cart) : </b>
   - https://www.youtube.com/watch?v=C9iqVCcLbdU&t=60s

### Example 4 :
Scroll to  <b>'Putting it together'</b> -> Work through this example.
- Link : https://samuelmullen.com/articles/elixir-processes-send-and-receive

### Steps for install dictionary : 

1. Command to install dictionary / wamerican

```sh
 sudo apt install wamerican
```

2. Command to count the number of lines in the dictionary

```sh
 wc -l /usr/share/dict/american-english
```
 

## <u>Summary</u>

For this GenServer section upon completion you should be able to do the following : 


<b> Examples </b> 
  - There are 4 examples to complete for this section and they will give you better understanding on GenServer.
## 6. OTP (Open Telecom Platform)

OTP Supervisors
- Supervisors monitor other processes and can restart them if they crash. They ensure the system's resilience.
Strategies for fault tolerance
Child specifications

https://elixirschool.com/en/lessons/advanced/otp_supervisors
https://www.openmymind.net/Elixir-A-Little-Beyond-The-Basics-Part-7-supervisors/

#### TP Supervisors

-   Supervisors are specialized processes that monitor other processes, called child processes.
-   If a child process crashes, the supervisor can restart it, allowing for fault tolerance and resilience.
-   Supervisors work according to different strategies to decide how to restart child processes.

#### Strategies for Fault Tolerance

-   **One for One**: If a child process dies, only that process is restarted.
-   **One for All**: If one child process dies, all other child processes are restarted.
-   **Rest for One**: If a child process dies, the rest of the processes started after it are restarted.

#### Child Specifications

-   Child specifications define how a child process is started, how often it should be restarted, and other settings.

Example: Fault Tolerance
`

    defmodule MyWorker do
      use GenServer
    
      def start_link(_) do
        GenServer.start_link(__MODULE__, 0, name: __MODULE__)
      end
    
      def increment do
        GenServer.call(__MODULE__, :increment)
      end
    
      def handle_call(:increment, _from, state) do
        {:reply, :ok, state + 1}
      end
    end
    
    defmodule MySupervisor do
      use Supervisor
    
      def start_link(_) do
        Supervisor.start_link(__MODULE__, [], name: __MODULE__)
      end
    
      def init(_) do
        children = [
          {MyWorker, []}
        ]
    
        Supervisor.init(children, strategy: :one_for_one)
      end
    end` 

This code snippet illustrates a simple supervisor (`MySupervisor`) that monitors a worker (`MyWorker`). If `MyWorker` crashes, it will be restarted by the supervisor, following the `:one_for_one` strategy.

#### Exercise Question

Create a new module called `WorkerSupervisor`, which will be responsible for supervising two workers: `WorkerA` and `WorkerB`. Both workers should be GenServers that keep track of a number, and they should support increment and get operations.

Implement the following functionalities:

1.  **WorkerA and WorkerB**: Create the GenServer modules for both workers.
2.  **WorkerSupervisor**: Create the supervisor module that uses the `:one_for_all` strategy.
3.  **Test Functionality**: Write a test function that spawns multiple processes to concurrently increment both workers and then prints the final values.

Use the simple supervisor example as a reference.
Ensure that your solution demonstrates the usage of supervisors and their strategies for fault tolerance.

### OTP Applications
-  A way to package and configure functionality.
Application structure and life cycle
OTP Behaviours and GenEvent
- Predefined structures for common use-cases.

https://elixir-lang.org/getting-started/mix-otp/supervisor-and-application.html

## 7. Advanced Concurrency Concepts

**Introduction to Poolboy for connection pooling**
-  Poolboy is used for pooling. Instead of creating a new resource (like a database connection) every time, you reuse existing resources.
Analogy: It’s like having a pool of cars for rent. Instead of buying a new car every time you need one, you just rent it.
https://samuelmullen.com/articles/elixir-poolboy-and-littles-law

## ETS (Erlang Term Storage) tables

https://elixir-lang.org/getting-started/mix-otp/ets.html

## Basics of ETS

Use cases in real-time systems

    :ets.new(:my_table, [:set])
    :ets.insert(:my_table, {:key, "value"})
    :ets.lookup(:my_table, :key) # [{:key, "value"}]


## GenStage for event-driven architectures

## 8. Metaprogramming in Elixir

Macros
Compile-time vs. Run-time
Code generation

## 9. Advanced Elixir Concepts

Umbrella Projects
Specifications and types
Behaviours and Protocols

## 10. Distributed Elixir

Node connections
Distributed tasks
Handling network partitions



