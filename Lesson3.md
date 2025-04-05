太棒了！我们现在正式进入你自己动手构建 RTOS 调度器的 第二阶段：实现任务延迟和任务状态管理（睡眠 / 唤醒） 💡💻

🚀 阶段 2 目标：
我们要在上一阶段的基础上，实现一个小型调度器，它能：

✅ 1. 管理多个任务
✅ 2. 每个任务都有“延迟时间”
✅ 3. 调度器检查哪些任务已经醒了，再执行
✅ 4. 支持任务“运行一会 → 休息几秒 → 再次运行”

🧱 新知识点：任务控制块（TCB）扩展
我们现在要给每个任务加点“属性”：

属性名	说明
name	任务名字
function	任务的函数
delay	延迟时间（剩下几秒钟才能再次执行）
state	当前状态（就绪 Ready / 睡眠 Sleeping）
🛠️ 完整示例代码：mini_rtos_v2.c
c
复制
编辑
#include <stdio.h>
#include <unistd.h> // 用于 sleep()
#include <string.h> // 用于字符串处理

// 定义任务状态枚举
typedef enum {
    READY,
    SLEEPING
} TaskState;

// 定义函数指针类型（任务函数）
typedef void (*TaskFunction)();

// 任务控制块 TCB
typedef struct {
    char* name;
    TaskFunction function;
    int delay;       // 剩余延迟秒数
    TaskState state; // 当前任务状态
} TaskControlBlock;

// ------------ 任务函数 -------------
void task1() {
    printf("[任务1] 检查传感器\n");
}

void task2() {
    printf("[任务2] 控制电机\n");
}

void task3() {
    printf("[任务3] 发送数据\n");
}

// ------------ 任务列表 -------------
#define NUM_TASKS 3

TaskControlBlock tasks[NUM_TASKS] = {
    {"Task 1", task1, 0, READY},
    {"Task 2", task2, 0, READY},
    {"Task 3", task3, 0, READY}
};

// ------------ 调度器 -------------
void scheduler() {
    while (1) {
        for (int i = 0; i < NUM_TASKS; i++) {
            TaskControlBlock* task = &tasks[i];

            if (task->state == READY) {
                printf("▶️ 正在运行：%s\n", task->name);
                task->function();

                // 设置延迟时间
                task->delay = 3;         // 休息 3 秒
                task->state = SLEEPING;  // 设置为睡眠
            }
            else if (task->state == SLEEPING) {
                task->delay--; // 每轮调度器循环1秒，减少延迟
                if (task->delay <= 0) {
                    task->state = READY; // 延迟结束，重新就绪
                    printf("💤 %s 已经醒来，准备执行\n", task->name);
                } else {
                    printf("😴 %s 还在睡眠，剩余 %d 秒\n", task->name, task->delay);
                }
            }
        }

        printf("——— 调度器循环结束，等待 1 秒 ———\n\n");
        sleep(1); // 模拟系统“时钟”
    }
}

// ------------ 主函数 -------------
int main() {
    printf("🛠️ Mini RTOS v2 启动：支持延迟和任务状态管理\n\n");
    scheduler(); // 启动调度器
    return 0;
}
🎯 程序运行流程（举例）：
所有任务都是 READY 状态，都会执行一遍

每个任务执行完，设置为 SLEEPING，延迟 3 秒

每秒减少 delay，直到 0

delay 到了就“醒来”，变成 READY，再次执行

✅ 你可以尝试做的实验：
修改任务函数，打印不同内容

设置不同任务的延迟时间（比如 task1 休息 2 秒，task2 5 秒）

观察调度器如何轮流执行和跳过“睡觉”的任务

🧩 下一阶段预告（如果你继续学）：
我们可以继续做：

✅ 支持任务“优先级调度”（Priority Scheduling）

✅ 添加“任务创建 / 删除”

✅ 支持“定时任务 / 周期任务”

✅ 模拟抢占式调度（更接近真实 RTOS）
