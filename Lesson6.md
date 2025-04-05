我们正式进入 RTOS 教程的第 5 阶段：实现“抢占式调度器”功能（Preemptive Scheduling）⚡️🧠

🎯 本阶段目标：
我们要让 RTOS 支持抢占式调度：

如果正在运行的任务优先级较低

高优先级任务突然变成 READY（比如刚醒来）

系统会立即打断当前任务，先执行高优先级任务！

✅ 这是真正 RTOS 中的关键特性：实时响应、高优先级抢占低优先级

🧱 核心思路：
我们需要：

一个全局变量表示“当前正在运行的任务”

在每个时钟 tick 时检查是否有更高优先级的任务 READY

如果有：✅ 抢占当前任务，切换到更高优先级任务

🛠️ 示例代码：mini_rtos_v5_preemptive.c
c
复制
编辑
#include <stdio.h>
#include <unistd.h>

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

#define NUM_TASKS 3

// ------------ 任务函数 -------------
void task1() { printf("🟢 [任务1] 采集传感器\n"); }
void task2() { printf("🔵 [任务2] 控制电机\n"); }
void task3() { printf("🟠 [任务3] 上传数据\n"); }

// ------------ 任务数组 -------------
TaskControlBlock tasks[NUM_TASKS] = {
    {"Task 1", task1, 0, READY, 2, 4},
    {"Task 2", task2, 0, READY, 1, 6},
    {"Task 3", task3, 0, READY, 3, 8}
};

TaskControlBlock* currentTask = NULL; // 当前正在执行的任务

// ------------ 抢占式调度器 -------------
void scheduler() {
    int systemTick = 0;

    while (1) {
        printf("⏱️ 系统时钟：%d 秒\n", systemTick);

        // 更新所有任务状态（睡眠计时 + 周期性唤醒）
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

        // 抢占逻辑：寻找优先级最高的 READY 任务
        TaskControlBlock* highestReady = NULL;
        for (int i = 0; i < NUM_TASKS; i++) {
            TaskControlBlock* task = &tasks[i];
            if (task->state == READY) {
                if (highestReady == NULL || task->priority < highestReady->priority) {
                    highestReady = task;
                }
            }
        }

        // 如果当前没有任务，或者有更高优先级任务，则抢占
        if (highestReady != NULL &&
            (currentTask == NULL || highestReady->priority < currentTask->priority)) {
            
            if (currentTask != NULL && currentTask->state == RUNNING) {
                currentTask->state = READY; // 当前任务被打断，回到 READY
                printf("⚠️ 抢占：%s 被中断！\n", currentTask->name);
            }

            // 切换到高优先级任务
            currentTask = highestReady;
            currentTask->state = RUNNING;
            printf("▶️ 执行：%s（优先级 %d）\n", currentTask->name, currentTask->priority);
            currentTask->function();

            // 设置下次运行的延迟
            currentTask->delay = currentTask->period;
            currentTask->state = SLEEPING;
        } else {
            printf("😴 无更高优先级任务，继续空闲\n");
        }

        printf("—— 调度器循环结束 ——\n\n");
        sleep(1);
        systemTick++;
    }
}

int main() {
    printf("🛠️ Mini RTOS v5 启动：抢占式优先级调度\n\n");
    scheduler();
    return 0;
}
🔍 抢占场景示意：
text
复制
编辑
▶️ 执行：Task 3（优先级 3）
🟠 [任务3] 上传数据
💤 Task 2 醒来了
⚠️ 抢占：Task 3 被中断！
▶️ 执行：Task 2（优先级 1）
🔵 [任务2] 控制电机
✅ 本阶段你掌握了：
功能	实现方式
抢占调度	比较当前运行任务和 READY 中的最高优先级任务
中断任务	若发现更高优先级任务，当前任务切回 READY
高优先级立即执行	系统优先运行优先级更高任务，响应更快
🧠 小贴士：
虽然我们这里没有真的“打断中途正在执行的任务”——因为在用户态我们没法中断函数执行——但这个逻辑已经模拟了 抢占式调度器 的核心工作方式。

🔔 下一阶段预告：
✅ 阶段 6：任务间通信（消息队列）
我们可以让一个任务“发送消息”，另一个任务“接收并处理”，模拟信号、命令、共享缓冲区。

