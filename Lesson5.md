我们现在进入 RTOS 教程的 第四阶段：实现定时任务 / 周期任务（Periodic Task）功能！⏰🔁

🎯 本阶段目标：
我们要让某些任务每隔固定的时间就自动运行一次，比如：

任务 A 每 5 秒运行一次

任务 B 每 2 秒运行一次

这类功能就是“定时任务 / 周期任务”，在真实 RTOS 中非常常见（比如定时采集传感器数据、周期刷新界面等）。

🧱 我们要新增一个字段：period
字段名	类型	说明
period	int	表示任务周期（每几秒运行一次）
🛠️ 示例代码：mini_rtos_v4_periodic.c
c
复制
编辑
#include <stdio.h>
#include <unistd.h>

typedef enum {
    READY,
    SLEEPING
} TaskState;

typedef void (*TaskFunction)();

typedef struct {
    char* name;
    TaskFunction function;
    int delay;       // 剩余延迟时间
    TaskState state;
    int priority;
    int period;      // ⭐ 新增：任务周期
} TaskControlBlock;

// ------------ 任务函数 -------------
void task1() { printf("[任务1] 定时采集传感器\n"); }
void task2() { printf("[任务2] 控制器定时刷新\n"); }
void task3() { printf("[任务3] 上传日志\n"); }

// ------------ 任务列表 -------------
#define NUM_TASKS 3

TaskControlBlock tasks[NUM_TASKS] = {
    {"Task 1", task1, 0, READY, 2, 5}, // 每 5 秒运行一次
    {"Task 2", task2, 0, READY, 1, 3}, // 每 3 秒运行一次
    {"Task 3", task3, 0, READY, 3, 7}  // 每 7 秒运行一次
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
                    printf("😴 %s 睡眠中（剩余 %d 秒）\n", task->name, task->delay);
                }
            }

            // 每周期时间点：自动设置为 READY
            if (systemTick % task->period == 0) {
                if (task->state == SLEEPING) {
                    printf("🔄 %s 重新唤醒（周期性触发）\n", task->name);
                    task->state = READY;
                    task->delay = 0;
                }
            }
        }

        // 按优先级选择任务执行
        TaskControlBlock* selected = NULL;
        for (int i = 0; i < NUM_TASKS; i++) {
            if (tasks[i].state == READY) {
                if (selected == NULL || tasks[i].priority < selected->priority) {
                    selected = &tasks[i];
                }
            }
        }

        if (selected != NULL) {
            printf("▶️ 执行任务：%s（周期 %d 秒）\n", selected->name, selected->period);
            selected->function();
            selected->delay = selected->period; // 下一轮要等整个周期再执行
            selected->state = SLEEPING;
        } else {
            printf("🕸️ 没有任务准备执行，系统空闲中...\n");
        }

        printf("—— 调度器循环结束 ——\n\n");
        sleep(1);
        systemTick++;
    }
}
 
int main() {
    printf("🛠️ Mini RTOS v4 启动：支持周期任务调度\n\n");
    scheduler();
    return 0;
}
📌 本阶段亮点：
每个任务有自己的周期 period

调度器记录 systemTick，每秒递增

当 systemTick % period == 0，说明到了这个任务该执行的时候

系统按照优先级选择 READY 的任务去运行

📊 运行示例：
text
复制
编辑
⏱️ 系统时钟：3 秒
💤 Task 1 醒来了
🔄 Task 2 重新唤醒（周期性触发）
▶️ 执行任务：Task 2（周期 3 秒）
[任务2] 控制器定时刷新
✅ 本阶段你学到：
能力	实现方法
周期性任务调度	利用系统时钟 systemTick 和 period 判断是否唤醒
按优先级调度	在 READY 任务中选优先级最低的值
延迟休眠管理	delay 倒数控制“睡眠”时间
🚀 下一阶段预告（进阶挑战）：
我们可以进一步做： ✅ 阶段 5：抢占式调度器（高优先级任务可以中断当前运行任务）
✅ 阶段 6：任务通信（消息队列），任务之间传数据
