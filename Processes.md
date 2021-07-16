### 1. fork() - cloning a process
When creating a new process, we would use fork(). The basic usage is as follows:
```
int main(){
    fork();
    printf("helloworld. %d\n", getpid());
}
```
The output might be:
```
helloworld. 28917
helloworld. 28918
```
After a new child process is created, both processes will execute the next instruction following the fork() system call. Therefore, we have to distinguish the parent from the child. This can be done by testing the returned value of fork().
For the parent, it returns PID of the child; for the child process, it returns 0. That's why we can do the following to distinguish the parent and child:
```
#include <stdlib.h>

int main(){
    pid_t pid = fork();
    
    if(pid == -1)
        perror("fork");
    
    if(pid == 0) {
        printf("I am the child process; my pid: %d\n\n", getpid());
    } else {
        printf("I am the parent process; my pid: %d\n", getpid());
    }
}
```

However, simply creating new processes is not cool enough. Can we do something cooler? For instance, how to make one process run another program in a child process? Here comes ```execv*```

### 2. execv* - replace the behavior of the current process with another program
When you call ```exec*```, it does not create a child and run a program in such child process; instead, it runs the program that you specify(by passing params to ```exec*```), but it replaces the current process, meaning that the current process ceases to exist (no longer exists).

Although, it is worth mentioning that ```exec*``` does not change the pid - it remains the same, but the process will do different things.

```exec*``` will only return -1 (as well as setting errno) if error occurs.

Still wondering the meaning of ```exec*``` replacing the current process? Let's take a look at an example:
```
// main program
int main(){
    printf("This line will be executed. My pid: %d\n\n", getpid());
    
    // replace current process with another program
    char *args[] = {"./another_program", NULL};
    int ret = execv(args[0], args); 
    
    printf("execlp failed with retval = %d\n", ret);
    printf("This string will be printed only if execlp failed\n");    
    return 0;
}

// another_program.c
int main(){
    printf("Hi, I am another_program! with pid: %d\n", getpid());
    return 0;
}
```
In this example, the main program is replaced by another_program after printing the first line. another_program only prints a string, and if it succeeds, all printf's after ```execv``` will not be executed. Only when ```execv``` fails will those be printed.


If we look at manpage of ```execv*```, there will be a bunch of functions.

The first letter after ```exec``` will be either v or l.
+ v: (const char *path, char *const argv[])
    + (vector) you pass in the arguments to the program as a vector (or an array)
+ l: (const char *path, const char *arg, ... /* (char  *) NULL */)
    + (list) pass the arguments one by one as a list 

The second letter after ```exec``` will be p
+ p: (path) do path searching to look at the path environment variable
    + e.g. execlp("ls", ..., NULL);
    + e.g. execl("/bin/ls", ..., NULL);
    + p is going to make things more convenient.
+ e: (environment variable) pass a different set of environment variables to the new program
    + if not using e, the environment variables from the original process will be passed to the "new" (replaced) process.

https://www.geeksforgeeks.org/difference-fork-exec/