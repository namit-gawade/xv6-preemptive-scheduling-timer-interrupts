# DA-4: Preemptive Scheduling Through Timer Interrupts
## Namit — Part 3: Context Switch Path

### GDB Evidence

Breakpoint:
- `break sched`
- Breakpoint hit at `sched()` in `proc.c:61`

Backtrace:
- `sched()`
- `yield()`
- `trap()`
- `alltraps()`

This confirms the required scheduling path:
`trap -> yield -> sched`

Current CPU process:
- PID: 1
- Name: `initcode`
- State: `RUNNABLE`
- Trap frame address: `0x8dffffb4`

Stored process trap frame:
- `trapno = 0`
- `eip = 0`
- `eflags = 512`
- `esp = 4096`

Saved process `eip`:
- `proc->tf->eip = 0`

Manav's timer-interrupt observations:
- Hit 1: `eip = 2148547636` — `scheduler + 36 in section .text`
- Hit 2: `eip = 2148549857` — `popcli + 65 in section .text`
- Hit 3: `eip = 2148549857` — `popcli + 65 in section .text`

The stored `proc->tf->eip` observed during the `sched` breakpoint was `0`, so it did not match either timer-interrupt `eip`. The inspected process trap frame also had `trapno = 0`, so it should not be treated as the same timer-interrupt trap frame observed in Parts 1 and 2.

## Question 2

Without timer interrupts, the operating system would not be able to forcibly regain control from a process that continuously occupies the CPU. For example, if a process executes `while(1);`, it could keep running indefinitely and prevent other runnable processes from getting CPU time, unless it voluntarily yielded, blocked, or otherwise entered the kernel. Timer interrupts provide the periodic hardware event that allows the kernel to interrupt the running process and enter the scheduling path.

## Question 3

When an interrupt occurs, the processor/kernel saves the interrupted execution state in a trap frame. The saved `eip` identifies the instruction address associated with the interrupted execution, `esp` preserves the stack position, and `eflags` preserves the CPU status/control flags. Together with the other saved registers in the trap frame, this state allows the kernel to restore the process's execution context later so that it can continue from the saved execution state.

## Conclusion

The GDB investigation demonstrated the path from a trap through `yield()` to `sched()`. The inspected CPU had a current process, PID 1 (`initcode`), in the `RUNNABLE` state. The process's stored trap frame could be inspected directly, including its saved `eip`, `esp`, and `eflags`.
