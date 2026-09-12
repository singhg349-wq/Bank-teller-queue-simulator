# Bank Teller Queue Simulator

A discrete-time simulation of a multi-teller bank service system in Python. Customers wait in a single shared queue, tellers serve them in fixed time cycles, and the simulation tracks how much of each teller's time was spent working versus idle.

## What it does

The simulation runs on a small command-based language rather than a fixed script — each command advances or inspects the system state:

- `call N` — creates `N` tellers, numbered `0` through `N-1`
- `add t1 t2 t3 ...` — adds customers to the back of the queue, each needing the given number of time units of service
- `service N` — advances the simulation by `N` time units: any idle teller picks up the next customer waiting in the queue, then every teller processes up to `N` units against whichever customer they currently have
- `status` — prints each teller's cumulative service time, how many customers are still waiting, and each teller's idle time as a percentage of total elapsed time
- `quit` — ends the simulation and prints a final summary of every teller

## Running

python3 banking.py


The script runs a built-in example scenario (3 tellers, 3 customers, two service cycles) as soon as it's executed — no arguments needed. To try a different scenario, edit the `commands` list at the bottom of the file.

## Example output

Teller: 0 Total service time: 5
Teller: 1 Total service time: 6
Teller: 2 Total service time: 6
Customers waiting in queue: 0
0 idle time: 100.00%
1 idle time: 0.00%
2 idle time: 0.00%
Teller: 0 Total service time: 5
Teller: 1 Total service time: 10
Teller: 2 Total service time: 12
Customers waiting in queue: 0
0 idle time: 100.00%
1 idle time: 50.00%
2 idle time: 0.00%
Simulation ended.
Teller: 0 Total service time: 5
Teller: 1 Total service time: 10
Teller: 2 Total service time: 12


## How it works

This is a classic multi-server, single-queue system: several tellers (servers) pull from one shared FIFO line rather than each having their own queue. Time moves in discrete cycles rather than continuously — each `service N` call represents `N` units of time passing at once, during which every teller either keeps working on their current customer or, if free, grabs the next person in line.

A teller's idle percentage is tracked as accumulated idle time divided by *total elapsed simulation time*, not just the current cycle — so it reflects how idle that teller has been across the whole simulation so far, not just a snapshot.

## Design Notes

- **Manual command parsing**: commands are split into tokens with a hand-written character loop instead of Python's built-in `str.split()`. Functionally identical to `split()` for this input, but it means the parsing logic doesn't depend on a library method doing the work invisibly.

- **No idle gap when picking up a new customer**: within a single `service` call, a teller that just went idle and picked up a new customer from the queue is serviced in that *same* cycle, not the next one. This matters because it means a teller's queue-check and service-work both happen every cycle, so a customer never sits fully served-and-idle for a full extra cycle before their replacement starts.

- **`queue.Queue` instead of a plain list**: `Queue` is a thread-safe structure meant for producer/consumer scenarios across multiple threads, which this single-threaded simulation doesn't strictly need — a plain list or `collections.deque` would work identically here. It was used anyway for the built-in FIFO ordering and the safety of `get_nowait()`/`Empty` handling, which reads clearly even without concurrency.

- **Idle time as a percentage, not a raw count**: dividing accumulated idle time by total elapsed time (rather than reporting raw idle units) keeps the number meaningful and comparable regardless of how long the simulation has been running.

## Author

Gurshmeer Singh
