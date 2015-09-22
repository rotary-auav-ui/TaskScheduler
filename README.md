Task Scheduler – cooperative multitasking for Arduino microcontrollers
Version 1.51: 2015-09-20
 
 
REQUIREMENT:
A lightweight implementation of the task scheduling supporting:
execution period (n times per second)
number of iterations (n times)
execution of tasks in predefined sequence
dynamic change of the execution parameters for both tasks and execution schedule
power saving via entering IDLE sleep mode if no tasks are scheduled to run
 
 
IDEA:
“Task” is a container concept that links together:
Execution interval
Number of execution iterations
A piece of code performing the task activities (callback function)
 
Tasks are linked into execution chains, which are processed by the “Scheduler” in the order they are linked.
 
Tasks are responsible for supporting cooperative multitasking by being “good neighbors”, i.e., running their callback functions in a non-blocking way and releasing control as soon as possible.
 
“Scheduler” is executing Tasks' callback functions in the order the tasks were added to the chain, from first to last. Scheduler stops and exists after processing the chain once in order to allow other statements in the main code of loop() function to run.  This a “scheduling pass”.
 
If compiled with _TASK_SLEEP_ON_IDLE_RUN enabled, the scheduler will place processor into IDLE sleep mode (for approximately 1 ms, as the timer interrupt will wake it up), after what is determined to be an “idle” pass. An Idle Pass is a pass through the chain when no Tasks were scheduled to run their callback functions. This is done to avoid repetitive empty passes through the chain when no tasks need to be executed. If any of the tasks in the chain always requires immediate execution (aInterval = 0), then there will be no end-of-pass delay.
 
Note: Task Scheduler uses millis() to determine if tasks are ready to be invoked. Therefore, if you put your device to any “deep” sleep mode disabling timer interrupts, the millis() count will be suspended, leading to effective suspension of scheduling. Upon wake up, active tasks need to be re-enabled, which will effectively reset their internal time scheduling variables to the new value of millis(). Time spent in deep sleep mode should be considered “frozen”, i.e., if a task was scheduled to run in 1 second from now, and device was put to sleep for 5 minutes, upon wake up, the task will still be scheduled 1 second from the time of wake up. Executing enable() function on this tasks will make it run as soon as possible. This is a concern only for tasks which are required to run in a truly periodical manner (in absolute time terms).