---
title: Процессы
---

# {{ page.title }}

В Elixir весь код запускается внутри процессов. Процессы изолированы друг от друга, запускаются параллельно и взаимодействуют через отправку сообщений. Процессы - не единственная основа параллельной работы Elixir, но они предоставляют базу для построения распределённых и отказоустойчивых программ.

Процессы Elixir не следует путать с процессами операционной системы. Процессы в Elixir очень легковесны в плане использования памяти и ЦПУ (в отличии от потоков во многих других языках программирования). Поэтому запуск десятков или даже сотен тысяч процессов одновременно не проблема.

В этой главе мы изучим базовые конструкции для порождения новых процессов, а также отправку и приём сообщений между процессами.

## `spawn`

Основной механизм порождения новых процессов - автоматически импортируемая функция `spawn/1`:

```iex
iex> spawn fn -> 1 + 2 end
#PID<0.43.0>
```

`spawn/1` принимает функцию, которую она выполнит в другом процессе.

Обратите внимание, что `spawn/1` возвращает PID (идентификатор процесса). К этому моменту, процесс, который вы породили, скорее всего мёртв. Порождённый процесс выполнит переданную функцию и умрёт после её окончания:

```iex
iex> pid = spawn fn -> 1 + 2 end
#PID<0.44.0>
iex> Process.alive?(pid)
false
```

> Вы скорее всего получите идентификаторы процессов, отличные от представленных в этом руководстве.

Мы можем получить PID текущего процесса, вызвав `self/0`:

```iex
iex> self()
#PID<0.41.0>
iex> Process.alive?(self())
true
```

Процессы становятся намного интереснее, когда мы можем отправлять и получать сообщения.

## `send` и `receive`

Мы можем отправлять сообщения в процесс с помощью `send/2` и принимать их через `recieve/1`

```iex
iex> send self(), {:hello, "world"}
{:hello, "world"}
iex> receive do
...>   {:hello, msg} -> msg
...>   {:world, msg} -> "won't match"
...> end
"world"
```

Когда сообщение отправлено в процесс, оно хранится в почтовом ящике процесса. Блок `recieve/1` проходит через почтовый ящик текущего процесса в поисах сообщения, которое подходит под переданный шаблон. `receive/1` поддерживает ограничительные условия и множественные варианты входа, также, как `case/2`.

Процесс, который отправляет сообщение, не блокируется на `send/2`, он помещает сообщение в почтовый ящик получателя и продолжается. В частности, процесс может отправлять сообщение сам себе.

Если в ящике нет сообщений, подходящих какому-либо из шаблонов, текущий процесс будет ждать, пока не появится подходящее сообщение. Время ожидания также может быть задано:

```iex
iex> receive do
...>   {:hello, msg}  -> msg
...> after
...>   1_000 -> "nothing after 1s"
...> end
"nothing after 1s"
```

Можно установить таймаут 0, если ожидается, что сообщение уже должно быть в ящике.

Давайте объединим всё и отправим сообщение между процессами:

```iex
iex> parent = self()
#PID<0.41.0>
iex> spawn fn -> send(parent, {:hello, self()}) end
#PID<0.48.0>
iex> receive do
...>   {:hello, pid} -> "Got hello from #{inspect pid}"
...> end
"Got hello from #PID<0.48.0>"
```

Функция `inspect/1` используется для конвертации внутреннего представления структур данных в строки, обычно для вывода. Помните, что когда блок `recieve` выполняется, процесс отправитель может быть уже мёртв, т.к. единственная его инструкция - отправка сообщения.

Во время работы в консоли, вы можете найти достаточно полезным хелпер `flush/0`. Он получает и печатает все сообщения из ящика:

```iex
iex> send self(), :hello
:hello
iex> flush()
:hello
:ok
```

## Links

The majority of times we spawn processes in Elixir, we spawn them as linked processes. Before we show an example with `spawn_link/1`, let's see what happens when a process started with `spawn/1` fails:

```iex
iex> spawn fn -> raise "oops" end
#PID<0.58.0>

[error] Process #PID<0.58.00> raised an exception
** (RuntimeError) oops
    :erlang.apply/2
```

It merely logged an error but the parent process is still running. That's because processes are isolated. If we want the failure in one process to propagate to another one, we should link them. This can be done with `spawn_link/1`:

```iex
iex> spawn_link fn -> raise "oops" end
#PID<0.41.0>

** (EXIT from #PID<0.41.0>) an exception was raised:
    ** (RuntimeError) oops
        :erlang.apply/2
```

Because processes are linked, we now see a message saying the parent process, which is the shell process, has received an EXIT signal from another process causing the shell to terminate. IEx detects this situation and starts a new shell session.

Linking can also be done manually by calling `Process.link/1`. We recommend that you take a look at [the `Process` module](https://hexdocs.pm/elixir/Process.html) for other functionality provided by processes.

Processes and links play an important role when building fault-tolerant systems. Elixir processes are isolated and don't share anything by default. Therefore, a failure in a process will never crash or corrupt the state of another process. Links, however, allow processes to establish a relationship in a case of failures. We often link our processes to supervisors which will detect when a process dies and start a new process in its place.

While other languages would require us to catch/handle exceptions, in Elixir we are actually fine with letting processes fail because we expect supervisors to properly restart our systems. "Failing fast" is a common philosophy when writing Elixir software!

`spawn/1` and `spawn_link/1` are the basic primitives for creating processes in Elixir. Although we have used them exclusively so far, most of the time we are going to use abstractions that build on top of them. Let's see the most common one, called tasks.

## Tasks

Tasks build on top of the spawn functions to provide better error reports and introspection:

```iex
iex(1)> Task.start fn -> raise "oops" end
{:ok, #PID<0.55.0>}

15:22:33.046 [error] Task #PID<0.55.0> started from #PID<0.53.0> terminating
** (RuntimeError) oops
    (elixir) lib/task/supervised.ex:74: Task.Supervised.do_apply/2
    (stdlib) proc_lib.erl:239: :proc_lib.init_p_do_apply/3
Function: #Function<20.90072148/0 in :erl_eval.expr/5>
    Args: []
```

Instead of `spawn/1` and `spawn_link/1`, we use `Task.start/1` and `Task.start_link/1` which return `{:ok, pid}` rather than just the PID. This is what enables tasks to be used in supervision trees. Furthermore, `Task` provides convenience functions, like `Task.async/1` and `Task.await/1`, and functionality to ease distribution.

We will explore those functionalities in the ***Mix and OTP guide***, for now it is enough to remember to use `Task` to get better error reports.

## State

We haven't talked about state so far in this guide. If you are building an application that requires state, for example, to keep your application configuration, or you need to parse a file and keep it in memory, where would you store it?

Processes are the most common answer to this question. We can write processes that loop infinitely, maintain state, and send and receive messages. As an example, let's write a module that starts new processes that work as a key-value store in a file named `kv.exs`:

```elixir
defmodule KV do
  def start_link do
    Task.start_link(fn -> loop(%{}) end)
  end

  defp loop(map) do
    receive do
      {:get, key, caller} ->
        send caller, Map.get(map, key)
        loop(map)
      {:put, key, value} ->
        loop(Map.put(map, key, value))
    end
  end
end
```

Note that the `start_link` function starts a new process that runs the `loop/1` function, starting with an empty map. The `loop/1` function then waits for messages and performs the appropriate action for each message. In the case of a `:get` message, it sends a message back to the caller and calls `loop/1` again, to wait for a new message. While the `:put` message actually invokes `loop/1` with a new version of the map, with the given `key` and `value` stored.

Let's give it a try by running `iex kv.exs`:

```iex
iex> {:ok, pid} = KV.start_link
{:ok, #PID<0.62.0>}
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush()
nil
:ok
```

At first, the process map has no keys, so sending a `:get` message and then flushing the current process inbox returns `nil`. Let's send a `:put` message and try it again:

```iex
iex> send pid, {:put, :hello, :world}
{:put, :hello, :world}
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush()
:world
:ok
```

Notice how the process is keeping a state and we can get and update this state by sending the process messages. In fact, any process that knows the `pid` above will be able to send it messages and manipulate the state.

It is also possible to register the `pid`, giving it a name, and allowing everyone that knows the name to send it messages:

```iex
iex> Process.register(pid, :kv)
true
iex> send :kv, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush()
:world
:ok
```

Using processes to maintain state and name registration are very common patterns in Elixir applications. However, most of the time, we won't implement those patterns manually as above, but by using one of the many abstractions that ship with Elixir. For example, Elixir provides [agents](https://hexdocs.pm/elixir/Agent.html), which are simple abstractions around state:

```iex
iex> {:ok, pid} = Agent.start_link(fn -> %{} end)
{:ok, #PID<0.72.0>}
iex> Agent.update(pid, fn map -> Map.put(map, :hello, :world) end)
:ok
iex> Agent.get(pid, fn map -> Map.get(map, :hello) end)
:world
```

A `:name` option could also be given to `Agent.start_link/2` and it would be automatically registered. Besides agents, Elixir provides an API for building generic servers (called `GenServer`), tasks, and more, all powered by processes underneath. Those, along with supervision trees, will be explored with more detail in the ***Mix and OTP guide*** which will build a complete Elixir application from start to finish.

For now, let's move on and explore the world of I/O in Elixir.