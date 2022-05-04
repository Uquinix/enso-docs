# Scheduling

## Context

For Enso (and other operating systems), scheduling is quintessential, it needs to be fast, optimized, and responsive. We needed a scheduler that did the least amount of task switches possible. For example, in a system with 4 cpus and 4 tasks, we may have:

| cpu    | 1   | 2   | 3   | 4   |
| ------ | --- | --- | --- | --- |
| tick 1 | A   | B   | C   | D   |
| tick 2 | A   | B   | C   | D   |
| tick 3 | A   | B   | C   | D   |
| tick 4 | A   | B   | C   | D   |
| tick 5 | A   | B   | C   | D   |

(no context switch)

That means that we schedule ONLY when the number of running processes is higher than the number of cpus.

However, if we have 5 processes with 4 cpus we may have:

| cpu    | 1   | 2   | 3   | 4   |
| ------ | --- | --- | --- | --- |
| tick 1 | A   | B   | C   | D   |
| tick 2 | E   | A   | B   | C   |
| tick 3 | D   | E   | A   | B   |
| tick 4 | C   | D   | E   | A   |
| tick 5 | B   | C   | D   | E   |

This is ineffective, we want the least context switches possible. 

Solution:

| cpu    | 1   | 2   | 3   | 4   |
| ------ | --- | --- | --- | --- |
| tick 1 | E   | B   | C   | D   |
| tick 2 | A   | E   | C   | D   |
| tick 3 | A   | B   | E   | D   |
| tick 4 | A   | B   | C   | E   |
| tick 5 | A   | B   | C   | D   |

Here, every process is called 4 times. A B C and D have the least amount of context switches. Only process E changes frequently. This is quite good but not perfect. Certainly this method is effective but it isn't equal between processes.

Solution:

| cpu    | 1   | 2   | 3   | 4   |
| ------ | --- | --- | --- | --- |
| tick 1 | A   | C   | D   | E   |
| tick 2 | A   | B   | D   | E   |
| tick 3 | A   | B   | C   | E   |
| tick 4 | A   | B   | C   | D   |
| tick 5 | E   | B   | C   | D   |
| tick 6 | E   | A   | C   | D   |
| tick 7 | E   | A   | B   | D   |
| tick 8 | E   | A   | B   | C   |
| tick 9 | D   | A   | B   | C   |

This is better, each processes here are on the same level.

## Implementation

In Enso, we have a scheduler state for each task:

```c
struct TaskSchedule
{
    int cpu; /* the cpu that the process uses */
    int tick_start; /* the tick where the process starts executing */
    int tick_end; /* the tick where the process stops executing */
    bool is_currently_executed; /* whether the process has executed */

};
```

But we have some problems: What happens if a process is now runnable? Or furthermore, if we have 2 processes and 4 cpus and 1 turning runnable?

That's why we implemented a Part Two of the scheduler:

- when the number of running processes is lower than the number of cpu
- when the number of running processes is higher than the number of cpu

## When the number of Running Processes is Lower than the number of Cpus

If the number of processes is lower than the number of cpus, we don't need to switch context. However, if we have 1 process that started running and it does not execute code, we need to assign it a cpu - that's why we have 'idle cpu'. A lazy cpu is cpu that don't do anything for the moment. For example if we have 3 processes, 5 cores and 2 idle cpus - we would just handover the task to the lazy cpu.

## When the number of Running Processes is Higher than the number of Cpus

If the number of processes is higher than the number of cpus, we need to switch X process. Were X is equal to:

```c
size_t schedule_count = m_min(cpu_count(), running - cpu_count());
```

(where running is the number of running process)

We can't switch higher than the cpu_count (we can't switch for 6 process using 5 cpu) and we can't switch more processes than there is cpus (we can't switch for 1 processes for 5 cpus).

For each `schedule_count`, we get the process that has run the longest and replace it with the process that has queued the longest. However, we can still have an idle cpu. For example, if a process is deleted or sleeping, so when we get the most waiting processes, we check if there is no idle cpu. If there is one, we put the waiting process in the idle cpu. If there isn't any, replace the longest running process with the longest waiting process.

## CPU Cache

The problem of interrupt is that they are really heavy, they invalidate the cache and they can take a lot of cpu cycles. So we need to use them only if needed. That means that if a cpu doesn't need to switch to another process, we don't send them an interrupt.