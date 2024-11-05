# watchdog
A Watchdog program that monitors critical sections of client applications and restarts if needed


## Overview

The WatchDog project provides a mechanism to ensure that the critical sections of a client's process remain "safe". It does this by monitoring the client process and restarting it if the process terminates unexpectedly.

The project is written in C on Linux environment.

## Main Features

- **Process Monitoring**: Keeps track of a client process and ensures it is running.
- **Automatic Restart**: Revives the client process automatically if it terminates unexpectedly.
- **Heartbeat Mechanism**: Uses signals to check the "health" of both client and Watchdog processes.
- **Configurable Intervals**: Allows configuration of thresholds.
- **Inter-Process Communication**: Utilizes signals and semaphores for synchronization between processes.
- **Thread Support**: Employs multithreading to handle concurrent tasks.

## How It Works

1. **Initialization**: The client process initializes the watchdog mechanism by calling the `MMI` ("Make me immortal") function, providing the check interval, threshold, and the program's arguments.

2. **Watchdog Process Creation**: If a watchdog process is not already running, the client forks a new process that executes the watchdog process. If the watchdog process is running, it syncs to communicate with the existing WD process.

3. **Heartbeat Exchange**: The client and watchdog processes send periodic heartbeat signals (`SIGUSR1`) to each other to confirm they are alive.

4. **Monitoring Loop**: Both processes increment a counter each time they send a heartbeat. If a process fails to receive a heartbeat within the specified interval, it assumes the other has terminated.

5. **Revival Mechanism**: Upon detecting that the other process (either the WD process or the client process) has stopped responding, a process will attempt to revive it using the stored arguments.

6. **Termination**: When the client process is ready to terminate, it calls the `DNR` ("Do not resuscitate") function to gracefully shut down the watchdog and allow the client process to terminate.
   

### Integrate WatchDog into Your Client Application

1. **Starting the Watchdog: initialize the Watchdog in code**
```c
int main(int argc, char *argv[])
{
    // Start the watchdog monitoring
    MMI(interval_in_seconds, repetitions, argv);
    // Your application logic here
    // When critical section is done, stop the watchdog
    DNR();
    return 0;
}

```

- **`interval_in_seconds`**: The time interval for heartbeat checks.
- **`repetitions`**: The number of missed heartbeats allowed before reviving.
  
### Terminating the WatchDog
- Call the `DNR()` function before your application exits to gracefully shut down the watchdog process.
