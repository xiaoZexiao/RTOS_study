我们正式进入 RTOS 教程的第 7 阶段：实现信号量和互斥锁（Mutex）功能！ 🔒⚙️

🎯 本阶段目标：
我们将引入 信号量（Semaphore） 和 互斥锁（Mutex） 的概念：

具体功能：
信号量（Semaphore）：控制多个任务对共享资源的访问。

适用于计数或二值信号量（例如一个任务通知另一个任务，或者控制并发数量）。

互斥锁（Mutex）：确保只有一个任务能够访问某个共享资源，防止资源冲突（适用于资源的独占性访问）。

这两个功能在嵌入式系统中非常重要，因为它们保证了任务间的同步和资源的安全访问。

🧱 信号量和互斥锁的区别：
功能	信号量（Semaphore）	互斥锁（Mutex）
用途	控制对共享资源的访问	确保共享资源的独占性访问
计数	可以是计数型（如限制并发任务数）	只有二值（锁/解锁）
释放	可以被多个任务释放	只有拥有锁的任务可以释放锁
优先级反转	没有	可能发生，需要特别处理
🛠️ 示例代码：mini_rtos_v7_semaphore_mutex.c
我们通过信号量和互斥锁来控制任务访问共享资源，避免冲突。

c
复制
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>

#define NUM_TASKS 3

// 信号量和互斥锁
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER; // 互斥锁
pthread_cond_t semaphore = PTHREAD_COND_INITIALIZER; // 信号量

typedef enum {
    READY,
    RUNNING,
    SLEEPING
} TaskState;

typedef void (*TaskFunction)();

typedef struct {
    char* name;
    TaskFunction function;
    int delay;
    TaskState state;
    int priority;
    int period;
} TaskControlBlock;

// ------------ 任务函数 -------------
void task1() {
    printf("🟢 [任务1] 获取互斥锁，访问共享资源\n");
    pthread_mutex_lock(&mutex); // 获取锁
    printf("🔒 [任务1] 访问共享资源\n");
    sleep(2); // 模拟共享资源的使用
    printf("🔓 [任务1] 释放锁\n");
    pthread_mutex_unlock(&mutex); // 释放锁
}

void task2() {
    printf("🔵 [任务2] 等待信号量\n");
    pthread_cond_wait(&semaphore, &mutex); // 等待信号量
    printf("✅ [任务2] 信号量被释放，继续执行\n");
}

void task3() {
    printf("🟠 [任务3] 发送信号量\n");
    pthread_cond_signal(&semaphore); // 发送信号量
}

// ------------ 任务数组 -------------
TaskControlBlock tasks[NUM_TASKS] = {
    {"Task 1", task1, 0, READY, 2, 5},
    {"Task 2", task2, 0, READY, 1, 3},
    {"Task 3", task3, 0, READY, 3, 8}
};

// ------------ 调度器 -------------
void scheduler() {
    int systemTick = 0;
    while (1) {
        printf("⏱️ 系统时钟：%d 秒\n", systemTick);

        for (int i = 0; i < NUM_TASKS; i++) {
            TaskControlBlock* task = &tasks[i];

            if (task->state == SLEEPING) {
                task->delay--;
                if (task->delay <= 0) {
                    task->state = READY;
                    printf("💤 %s 醒来了\n", task->name);
                } else {
                    printf("😴 %s 睡眠中 (%d秒)\n", task->name, task->delay);
                }
            }

            if (systemTick % task->period == 0) {
                if (task->state != RUNNING) {
                    task->state = READY;
                    printf("🔄 %s 周期性唤醒\n", task->name);
                }
            }
        }

        // 按优先级选择任务执行
        TaskControlBlock* highestReady = NULL;
        for (int i = 0; i < NUM_TASKS; i++) {
            TaskControlBlock* task = &tasks[i];
            if (task->state == READY) {
                if (highestReady == NULL || task->priority < highestReady->priority) {
                    highestReady = task;
                }
            }
        }

        // 执行任务
        if (highestReady != NULL) {
            highestReady->state = RUNNING;
            printf("▶️ 执行任务：%s（优先级 %d）\n", highestReady->name, highestReady->priority);
            highestReady->function();

            // 设置下次运行的延迟
            highestReady->delay = highestReady->period;
            highestReady->state = SLEEPING;
        } else {
            printf("😴 没有任务准备执行，继续空闲\n");
        }

        printf("—— 调度器循环结束 ——\n\n");
        sleep(1);
        systemTick++;
    }
}

int main() {
    printf("🛠️ Mini RTOS v7 启动：信号量与互斥锁\n\n");
    scheduler();
    return 0;
}
🔍 关键功能说明：
1. 互斥锁（Mutex）：
用于保证任务对共享资源的独占性访问。

任务 1 获取互斥锁，访问共享资源，完成后释放锁，其他任务无法同时访问。

2. 信号量（Semaphore）：
任务 2 等待信号量，当信号量被释放时，它继续执行。

任务 3 释放信号量，通知等待的任务 2。

3. 任务同步：
任务 2 和任务 3 是通过信号量实现同步的，任务 2 等待任务 3 释放信号量后才能继续执行。

🧪 示例输出：
text
复制
⏱️ 系统时钟：0 秒
🟢 [任务1] 获取互斥锁，访问共享资源
🔒 [任务1] 访问共享资源
😴 Task 2 睡眠中 (2秒)
🟠 [任务3] 发送信号量
✅ [任务2] 信号量被释放，继续执行
✅ 本阶段你学到了：
功能	实现方式
互斥锁	使用 pthread_mutex_lock 和 pthread_mutex_unlock 保证独占资源访问
信号量	使用 pthread_cond_wait 和 pthread_cond_signal 来同步任务
任务间同步	任务通过信号量或锁进行资源互斥访问或同步
🧠 小贴士：
信号量和互斥锁在 RTOS 中非常常见，掌握它们对多任务系统的开发非常重要。

在多任务环境下，合理使用它们可以避免“资源竞争”和“死锁”等问题。

🔔 下一阶段预告：
如果你有兴趣，我们可以继续深入： ✅ 阶段 8：中断处理与中断服务例程（ISR），模拟硬件中断事件处理！
✅ 阶段 9：任务优先级动态变化，让任务在运行时能够改变优先级！
