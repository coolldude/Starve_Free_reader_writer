# Classical Solution to Reader Writer Problem
The classical solution to this problem solves it by giving writers exclusive access to the shared 
database while writing to the database. Two of its main variations include the First Reader writer 
problem and the Second Reader writer problem. The first one requires that no reader be kept waiting 
unless a writer has already obtained permission to use the shared object. The second one requires 
that once a writer is ready, that writer perform its write as soon as possible.

# Starve Free Solution
A solution to either problem may result in starvation. In the first case, writers may starve and in
the second case, readers may starve. 
So here I have described a starve free solution for the same which satisfies Mutual Exlusion, Progress
and Bounded Waiting.

## Initialization

In the prosposed starve free solution, I have used three semaphores that is, an extra semaphore apart
from the two used in the classical solution. Also an integer variable to maintain the count of readers is
being maintained.

```rb
int read_cnt = 0;                      //Integer variable keeping track of how many processes are currently reading the object
semaphore rqd_mutex = 1;               //The process holding this semaphore gets the next chance to enter the critical section
semaphore read_cnt_mutex = 1;          //Mutual exclusion semaphore for updating read_cnt variable
semaphore cs_mutex = 1;                //Mutual exclusion semaphore for the writers, also used by the first or last reader 
                                         that enters or exits the critical section. Used to access critical section
```

## Reader's Implementation

```rb
do {   
    wait(rqd_mutex);                  //Waiting for its turn to get executed
    wait(read_cnt_mutex);             //Reader wants to enter C.S
    read_cnt++;                       //No. of readers incremented by 1
    if(read_cnt==1)
       wait(cs_mutex);                //First reader requests access to C.S
    signal(rqd_mutex);                //Releasing mutex so that the next reader or writer can take the token
    signal(read_cnt_mutex);           //Releasing access to read_cnt
      ...
    /* reading is performed */
    /* Critical Section */
      ...
    wait(read_cnt_mutex);             //Requesting access to change read_cnt 
    read_cnt--;                       //Decrement read_cnt by 1
    if(read_cnt==0)
       signal(cs_mutex);              //If readers=0, release access to critical section for next reader or writer
    signal(read_cnt_mutex);           //Releasing access to read_cnt mutex
    
    /* Remainder Section */
} while (true);
```

## Writer's Implementation

```rb
do {
    wait(rqd_mutex);                  //Waiting for its turn to get executed
    wait(cs_mutex);                   //Requests access to critical section
    signal(rqd_mutex);                //Releasing mutex so that the next reader or writer can take the token
      ...
    /* writing is performed */
    /* Critical Section */
      ...
    signal(cs_mutex);                 //Releasing access to critical section for next reader or writer
    
    /* Remainder Section */
} while (true);
```

## Explanation
In the above proposed solution the reader or writer has to first acquire the rqd_mutex in order to enter
the critical section. Also, both of them have equal priority of acquiring it. If a writer comes in
between two readers and even if some readers are present in the critical section, the next reader
if it comes after a writer, the writer would have already acquired the rqd_mutex and thus the reader 
can't acquire it. Now the writer can enter the critical section ,thus readers and writers are now at
equal priority and none would starve. 
The read_cnt__mutex is used to write access to the read_cnt variable. Any reader that enters or
exits the critical section has to acquire this read_cnt_mutex. The cs_mutex is used to gain access 
to the critical section by both readers and the writers. It has to be obtained by every writer
before entering the critical section and signalled every time it exits the section.
