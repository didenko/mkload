As I needed a tunable load generator to play with a metrics system, I did this set of scripts. The `mkload` data flow is:

            ┌  /dev/urandom
            │    ⇩
            │  dd
    writer  │    ⇩
            │    pipe
            │    ⇩
            └  base64
                 ⇩
                 fifo file
                 ⇩
            ┌  cat
            │    ⇩
            │    pipe
    reader  │    ⇩
            │  gzip
            │    ⇩
            └  /dev/null

The chain of components before and after the fifo pipe are `writer` and `reader` respectively and they run a configurable number of instances. Each instance also sleeps for a pseudo-random time - writers up to `3` seconds and readers up to `10` seconds - thus allowing to control a blocking contention on the fifo.

Additionally `mkload` writes its start and stop times as epoch seconds to another specified file.

The `mkload` script never stops and needs to be killed - for which there is the `killer` helper. After being killed, `mkload` takes time to wind down as child processes wake up and die.

`mkload` synopsis is:

    mkload <num_writers> <num_readers> <fifo_path> <times_path>

`killer` synopsis is:

    killer <sleep_seconds> <pid_to_kill>

Here is an example 20 minute run:

    $ mkload 200 50 ./mkload.fifo ./mkload.times & killer 1200 $!
    [1] 2514
    $
    [1]+  Exit 143                mkload 200 50 ./mkload.fifo ./mkload.times
    $ cat mkload.times
    Started:  1518065180
    Finished: 1518066391
    $

PS. The `sleeper` is just a debugging script which prints its PID without buffering before taking a nap.
