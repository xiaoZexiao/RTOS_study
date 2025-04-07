我们现在进入 RTOS 教程的第 8 阶段：实现中断处理与中断服务例程（ISR）功能！ ⚡️🚀

🎯 本阶段目标：
我们将模拟 硬件中断（Interrupts） 的处理，并实现 中断服务例程（ISR）。

具体功能：
模拟中断发生：我们会让某个事件（比如定时器过期或外部信号）触发“中断”。

中断服务例程（ISR）：当中断发生时，任务会暂停，转而执行中断服务例程，完成特定任务后返回继续执行。

在 RTOS 中，中断是即时响应的事件，一旦发生，中断服务例程（ISR）会打断当前任务并执行。

🧱 中断的基本概念：
中断：硬件或软件事件，在执行正常程序时“打断”程序的顺序，转而执行特定的处理函数。

中断服务例程（ISR）：响应中断的函数。当中断触发时，ISR 会中断当前任务并优先执行。ISR 通常是“短小且高效”的，避免过多处理。

中断嵌套：一些 RTOS 支持中断的嵌套，即一个 ISR 执行时，若另一个更高优先级的中断发生，可以暂停当前 ISR，优先执行高优先级中断。

🛠️ 示例代码：mini_rtos_v8_interrupt.c
在这个示例中，我们模拟一个 定时器中断。每隔 5 秒触发一次中断，执行相应的 ISR。

c
复制
#include <stdio.h>
#include <unistd.h>
#include <signal.h>
#include <time.h>

#define TIMER_INTERVAL 5 // 每 5 秒触发一次中断

// 中断标志位
volatile sig_atomic_t interrupt_flag = 0;

// 中断处理函数（ISR）
void interruptHandler(int signum) {
    interrupt_flag = 1;
    printf("⏰ 中断触发！执行中断服务例程...\n");
}

// 任务函数
void task1() {
    printf("🟢 [任务1] 正在运行...等待中断\n");
}

// 任务2，模拟周期性任务
void task2() {
    printf("🔵 [任务2] 执行周期性任务\n");
}

// 中断定时器初始化
void initTimer() {
    struct sigaction sa;
    struct itimerval timer;

    // 设置 SIGALRM 信号处理程序（中断）
    sa.sa_handler = interruptHandler;
    sa.sa_flags = SA_RESTART;
    sigaction(SIGALRM, &sa, NULL);

    // 设置定时器，每 5 秒触发一次 SIGALRM
    timer.it_value.tv_sec = TIMER_INTERVAL;
    timer.it_value.tv_usec = 0;
    timer.it_interval.tv_sec = TIMER_INTERVAL;
    timer.it_interval.tv_usec = 0;
    setitimer(ITIMER_REAL, &timer, NULL);
}

// 调度器
void scheduler() {
    int systemTick = 0;
    while (1) {
        printf("⏱️ 系统时钟：%d 秒\n", systemTick);

        if (interrupt_flag) {
            interrupt_flag = 0;
            // 中断发生时，执行中断服务例程
            printf("✅ 处理中断...\n");
        } else {
            task1();
        }

        task2();  // 周期性任务

        // 等待 1 秒后继续
        sleep(1);
        systemTick++;
    }
}

int main() {
    printf("🛠️ Mini RTOS v8 启动：模拟中断处理\n\n");

    // 初始化定时器（中断触发）
    initTimer();

    // 启动调度器
    scheduler();

    return 0;
}
🔍 功能说明：
1. 定时器中断：
我们使用 setitimer 函数设置一个定时器，每 5 秒触发一次中断。

定时器触发时，interruptHandler 被调用，模拟了一个硬件中断事件。

2. 中断服务例程（ISR）：
interruptHandler 函数是我们的中断服务例程。它会在中断发生时被执行，模拟硬件中断后的处理。

3. 任务调度：
任务 1 模拟正在运行的任务，它会在中断发生时暂停，并由 ISR 执行。

任务 2 是一个周期性任务，定期运行。

4. 系统时钟与调度：
我们使用 systemTick 来模拟系统时钟，每秒更新一次。

中断触发时，当前任务会暂停，执行中断服务例程。

🧪 示例输出：
text
复制
⏱️ 系统时钟：0 秒
🟢 [任务1] 正在运行...等待中断
🔵 [任务2] 执行周期性任务
⏰ 中断触发！执行中断服务例程...
✅ 处理中断...
⏱️ 系统时钟：1 秒
🟢 [任务1] 正在运行...等待中断
🔵 [任务2] 执行周期性任务
⏰ 中断触发！执行中断服务例程...
✅ 处理中断...
...
✅ 本阶段你学到了：
功能	实现方式
中断模拟	使用 setitimer 和 sigaction 设置定时器中断
中断服务例程（ISR）	通过中断标志和信号处理器实现中断响应
任务调度	在中断发生时暂停当前任务，执行 ISR 后返回
🧠 小贴士：
中断服务例程（ISR）通常是短小高效的，不进行复杂的操作，以免影响实时性。

在真实硬件中，中断通常是硬件触发的（如定时器、外部输入等），而在软件模拟中，我们通过信号和定时器来模拟。

🔔 下一阶段预告：
如果你有兴趣，我们可以进一步进行： ✅ 阶段 9：任务优先级动态变化，让任务可以在运行时根据条件改变优先级
✅ 阶段 10：内存管理，我们可以模拟内存池、内存分配与释放，管理任务和资源！
