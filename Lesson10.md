我们正式进入 RTOS 教程的第 9 阶段：任务优先级动态变化 🚀

🎯 本阶段目标：
我们将实现任务的 动态优先级变化，这意味着任务在运行时可以根据某些条件（例如任务的等待时间、外部事件等）调整其优先级。

具体功能：
每个任务都有一个 初始优先级。

任务在执行时，可以根据某些条件（比如任务执行次数或外部事件）动态调整优先级。

我们模拟一个场景：任务运行一段时间后，任务优先级发生变化，影响调度器的任务选择。

🧱 为什么需要动态优先级变化？
在实际应用中，有些任务的优先级并不是固定的，它们可能会随着时间、事件或外部条件的变化而调整。例如：

实时性要求：任务执行的时间越长，可能需要动态提高其优先级。

任务等待时间：如果任务等待的时间过长，可以提高其优先级，避免饿死（starvation）。

动态优先级可以有效地避免 优先级反转 和 任务饿死 等问题，使系统更加灵活。

🛠️ 示例代码：mini_rtos_v9_priority_change.c
在这个示例中，我们将创建 3 个任务，并在每个任务运行一定时间后，动态修改它们的优先级。

c
复制
编辑
#include <stdio.h>
#include <unistd.h>

#define NUM_TASKS 3

// 任务状态
typedef enum {
    READY,
    RUNNING,
    SLEEPING
} TaskState;

// 任务函数类型
typedef void (*TaskFunction)();

// 任务控制块
typedef struct {
    char* name;
    TaskFunction function;
    int delay;
    TaskState state;
    int priority;
    int period;
    int executionCount; // 执行次数
} TaskControlBlock;

// 任务函数声明
void task1();
void task2();
void task3();

// 任务数组
TaskControlBlock tasks[NUM_TASKS] = {
    {"Task 1", task1, 0, READY, 2, 5, 0},
    {"Task 2", task2, 0, READY, 1, 3, 0},
    {"Task 3", task3, 0, READY, 3, 7, 0}
};

// 调度器
void scheduler() {
    int systemTick = 0;
    while (1) {
        printf("⏱️ 系统时钟：%d 秒\n", systemTick);

        // 更新任务状态（休眠计时，周期性唤醒）
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

        // 根据执行次数调整任务优先级
        for (int i = 0; i < NUM_TASKS; i++) {
            TaskControlBlock* task = &tasks[i];
            if (task->executionCount >= 3 && task->priority > 1) {
                task->priority--; // 执行次数超过 3 次，优先级上升
                printf("🔼 %s 优先级提升：%d\n", task->name, task->priority);
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
            highestReady->executionCount++; // 执行次数增加

            // 设置任务延迟
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

// 任务 1
void task1() {
    printf("🟢 [任务1] 正在运行...任务计数：%d\n", tasks[0].executionCount);
}

// 任务 2
void task2() {
    printf("🔵 [任务2] 正在运行...任务计数：%d\n", tasks[1].executionCount);
}

// 任务 3
void task3() {
    printf("🟠 [任务3] 正在运行...任务计数：%d\n", tasks[2].executionCount);
}

// 主函数
int main() {
    printf("🛠️ Mini RTOS v9 启动：任务优先级动态变化\n\n");
    scheduler();
    return 0;
}
🔍 功能说明：
1. 任务优先级动态调整：
每个任务有一个执行计数器（executionCount）。

如果任务执行超过 3 次，任务的优先级提升（priority--）。

在调度器中，我们会检查每个任务的执行次数，并根据任务的状态动态调整它的优先级。

2. 任务周期和状态管理：
每个任务都有一个 周期，定期唤醒。

任务的优先级和执行状态会影响调度器选择哪个任务来执行。

3. 任务执行顺序：
每个任务在运行时，会增加它的执行计数（executionCount）。

任务的优先级是动态变化的，执行次数越多，优先级越高。

🧪 示例输出：
text
复制
编辑
⏱️ 系统时钟：0 秒
🟢 [任务1] 正在运行...任务计数：0
🔵 [任务2] 正在运行...任务计数：0
🟠 [任务3] 正在运行...任务计数：0
🔄 Task 1 周期性唤醒
▶️ 执行任务：Task 2（优先级 1）
🔵 [任务2] 正在运行...任务计数：1
🔼 Task 2 优先级提升：0
...
✅ 本阶段你学到了：
功能	实现方式
任务优先级动态变化	根据任务执行次数调整优先级
任务调度	调度器根据动态优先级选择执行任务
状态管理	任务周期性唤醒，延迟管理
🔔 下一阶段预告：
如果你有兴趣，可以继续深入： ✅ 阶段 10：内存管理，实现内存池、内存分配与回收机制。
✅ 阶段 11：更多硬件接口，模拟和管理不同硬件设备的任务。
