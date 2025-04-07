我们正式进入 RTOS 教程的第 12 阶段：多核调度（Multi-Core Scheduling）与 CPU 亲和性（CPU Affinity）模拟 ⚙️🧠🧠

🎯 本阶段目标：
我们将模拟一个 多核 RTOS 系统，包括：

多核任务调度：模拟两个“CPU 核心”同时运行任务

任务绑定（CPU Affinity）：指定任务只运行在特定核心

核心之间的任务分发：调度器会根据 affinity 和就绪状态合理分配任务

虽然我们无法在单线程 C 程序中真正启用多核 CPU，但我们可以通过两个线程分别模拟两个核心的行为，达到“仿真”效果。

🧱 多核调度基本概念
概念	含义
多核（Multi-core）	系统拥有多个 CPU 核心可同时运行多个任务
亲和性（Affinity）	限制任务只能运行在某些核心上
负载均衡（Load balancing）	多核调度器会将任务合理分配到不同核心上，防止单个核心负担过重
🛠️ 示例代码：mini_rtos_v12_multicore.c
我们使用 pthread 模拟两个核心，每个“核心”在一个线程中独立运行自己的调度器。

c
复制
编辑
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define NUM_TASKS 4
#define NUM_CORES 2

// ------------ 任务状态与类型 -------------
typedef enum { READY, RUNNING, SLEEPING } TaskState;

typedef struct {
    char* name;
    int affinity;        // 属于哪个核心（0 或 1）
    int delay;
    int period;
    int priority;
    TaskState state;
    int executionCount;
    void (*function)(const char* taskName, int coreID);
} TaskControlBlock;

// ------------ 全局任务表 -------------
TaskControlBlock tasks[NUM_TASKS];

// ------------ 核心模拟（每个核心一个线程） -------------
void* cpuCore(void* arg) {
    int coreID = *(int*)arg;

    while (1) {
        TaskControlBlock* selected = NULL;

        // 寻找本核心能执行的最高优先级 READY 任务
        for (int i = 0; i < NUM_TASKS; i++) {
            TaskControlBlock* t = &tasks[i];
            if (t->affinity == coreID && t->state == READY) {
                if (selected == NULL || t->priority < selected->priority) {
                    selected = t;
                }
            }

            // 减少 delay
            if (t->affinity == coreID && t->state == SLEEPING) {
                t->delay--;
                if (t->delay <= 0) {
                    t->state = READY;
                }
            }
        }

        if (selected) {
            selected->state = RUNNING;
            selected->function(selected->name, coreID);
            selected->executionCount++;
            selected->delay = selected->period;
            selected->state = SLEEPING;
        } else {
            printf("💤 核心 %d 空闲中...\n", coreID);
        }

        sleep(1); // 模拟时钟节拍
    }
    return NULL;
}

// ------------ 任务实现 -------------
void genericTask(const char* taskName, int coreID) {
    printf("▶️ [核心 %d] 执行任务：%s\n", coreID, taskName);
}

// ------------ 初始化任务 -------------
void initTasks() {
    tasks[0] = (TaskControlBlock){"Task A", 0, 0, 3, 1, READY, 0, genericTask};
    tasks[1] = (TaskControlBlock){"Task B", 1, 0, 4, 2, READY, 0, genericTask};
    tasks[2] = (TaskControlBlock){"Task C", 0, 0, 5, 3, READY, 0, genericTask};
    tasks[3] = (TaskControlBlock){"Task D", 1, 0, 6, 1, READY, 0, genericTask};
}

// ------------ 主函数 -------------
int main() {
    printf("🛠️ Mini RTOS v12 启动：多核调度模拟\n\n");

    initTasks();

    pthread_t cores[NUM_CORES];
    int coreIDs[NUM_CORES] = {0, 1};

    // 启动两个核心线程
    for (int i = 0; i < NUM_CORES; i++) {
        pthread_create(&cores[i], NULL, cpuCore, &coreIDs[i]);
    }

    // 等待核心运行（模拟系统不退出）
    for (int i = 0; i < NUM_CORES; i++) {
        pthread_join(cores[i], NULL);
    }

    return 0;
}
🔍 程序结构说明：
模块	功能
TaskControlBlock	记录任务状态、优先级、亲和性等
cpuCore()	模拟一个核心的调度器，每个线程是一个核心
affinity 字段	限制任务在哪个核心上运行
genericTask()	打印执行信息，代表任务工作
pthread_create	启动多个“核心”线程
🧪 示例输出：
text
复制
编辑
▶️ [核心 0] 执行任务：Task A
▶️ [核心 1] 执行任务：Task D
💤 核心 0 空闲中...
▶️ [核心 1] 执行任务：Task B
...
✅ 你掌握了：
概念	技术实现
多核调度	多线程模拟多个 CPU 核
CPU 亲和性	每个任务设置 affinity 决定在哪个核心运行
独立核心调度器	每个线程只调度自己负责的任务
💡 实际 RTOS 中还会包含：
核间通信（IPC）

同步机制（如核间锁）

核心间负载迁移（task stealing）

各核共享资源管理

🔔 下一阶段预告：
我们已经构建了一个从任务、调度器、设备、中断到多核支持的“微型 RTOS 模型”
✅ 最后一阶段（阶段 13）：我可以带你整合这些模块，形成一个完整的嵌入式 RTOS 工程结构，支持模块化编程、任务初始化、调试接口等！
