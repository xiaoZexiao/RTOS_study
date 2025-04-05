我们现在进入 RTOS 教程的 第三阶段：添加“任务优先级调度”功能！ 🚀

🎯 本阶段目标：
在上一阶段的基础上，我们加入“优先级（Priority）”的概念：

每个任务都有一个优先级（数字越小，优先级越高）

调度器优先执行高优先级、状态为 READY 的任务

如果多个任务 READY，就选优先级最高的那一个运行

这就是非抢占式优先级调度（non-preemptive priority scheduling）！

🧱 新的 TCB（任务控制块）结构
我们会添加一个新字段：priority

c
复制
编辑
typedef struct {
    char* name;
    TaskFunction function;
    int delay;
    TaskState state;
    int priority; // ⭐ 新增：优先级（越小越高）
} TaskControlBlock;
🛠️ 完整示例代码：mini_rtos_v3_priority.c
c
复制
编辑
#include <stdio.h>
#include <unistd.h>

// ------------ 类型定义 -------------
typedef enum {
    READY,
    SLEEPING
} TaskState;

typedef void (*TaskFunction)();

typedef struct {
    char* name;
    TaskFunction function;
    int delay;
    TaskState state;
    int priority;  // ⭐ 新增字段
} TaskControlBlock;

// ------------ 任务函数 -------------
void task1() { printf("[任务1] 读取传感器\n"); }
void task2() { printf("[任务2] 控制电机\n"); }
void task3() { printf("[任务3] 发送数据\n"); }

// ------------ 任务列表（含优先级）-------------
#define NUM_TASKS 3

TaskControlBlock tasks[NUM_TASKS] = {
    {"Task 1", task1, 0, READY, 2}, // ⭐ 优先级 2
    {"Task 2", task2, 0, READY, 1}, // ⭐ 优先级 1（最高）
    {"Task 3", task3, 0, READY, 3}  // ⭐ 优先级 3（最低）
};

// ------------ 调度器：优先级调度 -------------
void scheduler() {
    while (1) {
        TaskControlBlock* selectedTask = NULL;

        // 选择优先级最高的 READY 任务
        for (int i = 0; i < NUM_TASKS; i++) {
            TaskControlBlock* task = &tasks[i];

            if (task->state == SLEEPING) {
                task->delay--;
                if (task->delay <= 0) {
                    task->state = READY;
                    printf("💤 %s 醒来了！\n", task->name);
                } else {
                    printf("😴 %s 睡眠中 (%d秒)\n", task->name, task->delay);
                }
            }

            if (task->state == READY) {
                if (selectedTask == NULL || task->priority < selectedTask->priority) {
                    selectedTask = task;
                }
            }
        }

        // 运行选中的任务
        if (selectedTask != NULL) {
            printf("▶️ 正在运行：%s（优先级 %d）\n", selectedTask->name, selectedTask->priority);
            selectedTask->function();
            selectedTask->delay = 3;
            selectedTask->state = SLEEPING;
        } else {
            printf("😴 当前无任务就绪，空闲中...\n");
        }

        printf("——— 调度器循环结束，等待 1 秒 ———\n\n");
        sleep(1);
    }
}

// ------------ 主函数 -------------
int main() {
    printf("🛠️ Mini RTOS v3 启动：支持优先级调度\n\n");
    scheduler();
    return 0;
}
🔍 运行逻辑说明：
每轮调度器循环：

先检查所有任务的延迟，如果 delay==0 就唤醒

然后在所有 READY 状态的任务中，找出优先级最高的那个（数字最小）

只运行那个任务，并设置它为 SLEEPING

每秒循环一次，模拟系统时钟

🧪 示例输出（你可能会看到）：
text
复制
编辑
▶️ 正在运行：Task 2（优先级 1）
[任务2] 控制电机
😴 Task 2 睡眠中 (2秒)
😴 Task 1 睡眠中 (1秒)
▶️ 正在运行：Task 1（优先级 2）
[任务1] 读取传感器
...
✅ 本阶段你学到了：
功能	技术
支持优先级调度	遍历任务，找出 READY 中优先级最高的
延迟任务 + 唤醒	每秒减少 delay，0 后变回 READY
动态调度	不再固定执行顺序，按优先级执行
🔔 下一阶段预告（你继续的话）：
✅ 阶段 4：实现定时任务 / 周期任务（比如每5秒自动运行一次）
✅ 阶段 5：模拟抢占式调度器（让高优先级任务中断低优先级任务）
