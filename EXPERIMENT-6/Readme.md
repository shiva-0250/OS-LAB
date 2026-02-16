# EXPERIMENT-6
## 6.A.PRODUCER-CONSUMER PROBLEM:
## SOURCE CODE:
## Source code for Semphore intialization:
``` os
#include <stdio.h>
#include <stdlib.h>
#include <sys/ipc.h>
#include <sys/sem.h>

int main() {
    key_t sem_key = ftok("semfile", 75);
    int sem_id = semget(sem_key, 2, 0666 | IPC_CREAT);

    // Set initial values: empty = BUFFER_SIZE, full = 0
    semctl(sem_id, 0, SETVAL, 5); // empty semaphore
    semctl(sem_id, 1, SETVAL, 0); // full semaphore

    printf("Semaphores initialized successfully.\n");
    return 0;
}
```
## Source code for Consumer:
``` os
/* Consumer Program (consumer.c) */
#include <stdio.h>
#include <stdlib.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/sem.h>
#include <unistd.h>

#define BUFFER_SIZE 5

typedef struct {
    int buffer[BUFFER_SIZE];
    int in;
    int out;
} SharedMemory;

void semaphore_wait(int sem_id, int sem_num) {
    struct sembuf sem_op = {sem_num, -1, 0};
    semop(sem_id, &sem_op, 1);
}

void semaphore_signal(int sem_id, int sem_num) {
    struct sembuf sem_op = {sem_num, 1, 0};
    semop(sem_id, &sem_op, 1);
}

int main() {
    key_t key = ftok("shmfile", 65);
    int shmid = shmget(key, sizeof(SharedMemory), 0666 | IPC_CREAT);
    SharedMemory* shm = (SharedMemory*)shmat(shmid, NULL, 0);

    key_t sem_key = ftok("semfile", 75);
    int sem_id = semget(sem_key, 2, 0666 | IPC_CREAT);

    while (1) {
        semaphore_wait(sem_id, 1);
        int item = shm->buffer[shm->out];
        printf("Consumed: %d\n", item);
        shm->out = (shm->out + 1) % BUFFER_SIZE;
        semaphore_signal(sem_id, 0);
        sleep(1);
    }
    
    shmdt(shm);
    return 0;
}
```
## Source code for Producer:
```os
/* Producer Program (producer.c) */
#include <stdio.h>
#include <stdlib.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/sem.h>
#include <unistd.h>

#define BUFFER_SIZE 5

typedef struct {
    int buffer[BUFFER_SIZE];
    int in;
    int out;
} SharedMemory;

void semaphore_wait(int sem_id, int sem_num) {
    struct sembuf sem_op = {sem_num, -1, 0};
    semop(sem_id, &sem_op, 1);
}

void semaphore_signal(int sem_id, int sem_num) {
    struct sembuf sem_op = {sem_num, 1, 0};
    semop(sem_id, &sem_op, 1);
}

int main() {
    key_t key = ftok("shmfile", 65);
    int shmid = shmget(key, sizeof(SharedMemory), 0666 | IPC_CREAT);
    SharedMemory* shm = (SharedMemory*)shmat(shmid, NULL, 0);

    key_t sem_key = ftok("semfile", 75);
    int sem_id = semget(sem_key, 2, 0666 | IPC_CREAT);
    
    shm->in = 0;
    shm->out = 0;

    while (1) {
        int item;
        printf("Enter item to produce: ");
        scanf("%d", &item);

        semaphore_wait(sem_id, 0);
        shm->buffer[shm->in] = item;
        printf("Produced: %d\n", item);
        shm->in = (shm->in + 1) % BUFFER_SIZE;
        semaphore_signal(sem_id, 1);
    }
    
    shmdt(shm);
    return 0;
}
```
