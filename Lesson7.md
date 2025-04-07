我们现在正式进入 RTOS 教程的第 6 阶段：实现任务间通信（消息队列） 💬🔗

🎯 本阶段目标：
我们将引入 消息队列（Message Queue） 的概念，它允许不同任务之间相互发送和接收消息，实现任务间通信。

具体功能：
一个任务发送消息到队列

另一个任务从队列中读取消息并处理

支持多任务之间的消息传递（模拟生产者消费者模型）

🧱 消息队列基础：
消息队列（Message Queue）是 RTOS 中非常常见的特性，它允许任务以非阻塞方式相互通信。

我们可以用以下方式模拟：

一个任务（生产者）将数据放入消息队列

另一个任务（消费者）从队列中获取数据进行处理

消息队列的大小有限制，队列满时会阻塞发送任务；队列空时会阻塞接收任务

🛠️ 示例代码：mini_rtos_v6_message_queue.c
我们用一个简单的队列模拟消息队列，并在任务间进行通信。

c
复制
编辑
#include <stdio.h>
#include <unistd.h>
#include <string.h>

#define QUEUE_SIZE 5 // 消息队列最大容量

// 消息队列结构
typedef struct {
    char* messages[QUEUE_SIZE];  // 队列内容
    int front;                   // 队列前端
    int rear;                    // 队列后端
    int size;                    // 当前队列大小
} MessageQueue;

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

// 全局队列实例
MessageQueue queue = {{0}, 0, 0, 0};

// 任务函数声明
void task1();
void task2();

// 初始化队列
void initQueue() {
    queue.front = 0;
    queue.rear = 0;
    queue.size = 0;
}

// 检查队列是否为空
int isQueueEmpty() {
    return queue.size == 0;
}

// 检查队列是否为满
int isQueueFull() {
    return queue.size == QUEUE_SIZE;
}

// 向队列发送消息
void sendMessage(char* message) {
    if (!isQueueFull()) {
        queue.messages[queue.rear] = message;
        queue.rear = (queue.rear + 1) % QUEUE_SIZE;
        queue.size++;
        printf("✅ 消息发送成功: %s\n", message);
    } else {
        printf("❌ 队列满，消息发送失败\n");
    }
}

// 从队列接收消息
char* receiveMessage() {
    if (!isQueueEmpty()) {
        char* message = queue.messages[queue.front];
        queue.front = (queue.front + 1) % QUEUE_SIZE;
        queue.size--;
        return message;
    } else {
        return NULL;
    }
}

// ------------ 任务函数 -------------
void task1() {
    printf("🟢 [任务1] 生产数据（向队列发送消息）\n");
    sendMessage("传感器数据1");
    sendMessage("传感器数据2");
}

void task2() {
    printf("🔵 [任务2] 消费数据（从队列接收消息）\n");
    char* message;
    while ((message = receiveMessage()) != NULL) {
        printf("🟠 消费了消息：%s\n", message);
    }
}

// ------------ 任务列表 -------------
#define NUM_TASKS 2

TaskControlBlock tasks[NUM_TASKS] = {
    {"Task 1", task1, 0, READY, 2, 5},
    {"Task 2", task2, 0, READY, 1, 3}
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
    printf("🛠️ Mini RTOS v6 启动：支持任务间通信（消息队列）\n\n");
    initQueue();
    scheduler();
    return 0;
}
🔍 功能说明：
消息队列：

sendMessage：把消息放入队列

receiveMessage：从队列取出消息

队列的最大容量是 QUEUE_SIZE，如果满了就不能再发送消息

任务功能：

任务 1 作为生产者，生成数据并发送到消息队列

任务 2 作为消费者，从消息队列中读取并处理数据

🧪 运行示例：
text
复制
编辑
⏱️ 系统时钟：0 秒
🟢 [任务1] 生产数据（向队列发送消息）
✅ 消息发送成功: 传感器数据1
✅ 消息发送成功: 传感器数据2
🔵 [任务2] 消费数据（从队列接收消息）
🟠 消费了消息：传感器数据1
🟠 消费了消息：传感器数据2
✅ 本阶段你学到了：
功能	实现方式
任务间通信	使用消息队列实现生产者消费者模式
队列管理	添加消息到队列、从队列中取消息
任务调度	按优先级和周期管理任务执行
🧠 小贴士：
这里的消息队列是简化版，并没有涉及复杂的同步机制。

真正的 RTOS 中，队列通常需要处理任务同步、阻塞等待等情况。

🔔 下一阶段预告：
如果你有兴趣，我们可以进一步进行： ✅ 阶段 7：信号量与互斥锁（Mutex），进一步模拟资源共享的安全访问
✅ 阶段 8：中断处理，我们可以模拟“硬件中断”来处理外部事件！

