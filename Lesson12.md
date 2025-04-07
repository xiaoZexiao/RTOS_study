我们现在进入 RTOS 教程的第 11 阶段：模拟硬件接口与设备管理机制 🛠️📟

🎯 本阶段目标：
我们将实现一个简化版设备驱动模型，模拟 RTOS 中如何：

注册设备（device register）

管理设备状态（开关、读写）

让任务与硬件交互，如访问传感器、写入串口

🧱 RTOS 中的设备管理机制（简化模型）
概念	含义
设备（Device）	一个抽象对象，代表硬件（如传感器、LED、串口）
驱动（Driver）	控制设备的程序，提供 open / read / write / close 接口
设备管理器（Device Manager）	系统统一管理所有设备的注册、调度、状态维护
💡 我们将模拟：
2 个设备：LED 灯、温度传感器

每个设备都可以 open、read、write

任务可以调用这些设备接口进行交互

🛠️ 示例代码：mini_rtos_v11_device.c
c
复制
编辑
#include <stdio.h>
#include <string.h>

#define MAX_DEVICES 5

// ------------ 设备接口结构体（相当于驱动函数表）-------------
typedef struct {
    char name[32];
    void (*open)();
    void (*read)();
    void (*write)(const char* data);
    void (*close)();
} Device;

// ------------ 全局设备列表（模拟设备管理器） -------------
Device deviceTable[MAX_DEVICES];
int deviceCount = 0;

// ------------ 设备操作实现（LED） -------------
void led_open() { printf("🔌 LED 已打开\n"); }
void led_read() { printf("🔦 LED 当前状态：关闭\n"); }
void led_write(const char* data) {
    printf("💡 LED 设置为：%s\n", strcmp(data, "on") == 0 ? "开启" : "关闭");
}
void led_close() { printf("❌ LED 已关闭\n"); }

// ------------ 设备操作实现（温度传感器） -------------
void temp_open() { printf("🌡️ 温度传感器已开启\n"); }
void temp_read() { printf("🌡️ 当前温度：25.6°C\n"); }
void temp_write(const char* data) {
    printf("⚠️ 传感器是只读的，不能写入：%s\n", data);
}
void temp_close() { printf("🌡️ 传感器已关闭\n"); }

// ------------ 注册设备到设备表 -------------
void registerDevice(const char* name, void (*open)(), void (*read)(), void (*write)(const char*), void (*close)()) {
    if (deviceCount < MAX_DEVICES) {
        strcpy(deviceTable[deviceCount].name, name);
        deviceTable[deviceCount].open = open;
        deviceTable[deviceCount].read = read;
        deviceTable[deviceCount].write = write;
        deviceTable[deviceCount].close = close;
        printf("✅ 设备注册成功：%s\n", name);
        deviceCount++;
    } else {
        printf("❌ 设备表已满，无法注册新设备\n");
    }
}

// ------------ 根据名称查找设备 -------------
Device* getDevice(const char* name) {
    for (int i = 0; i < deviceCount; i++) {
        if (strcmp(deviceTable[i].name, name) == 0) {
            return &deviceTable[i];
        }
    }
    return NULL;
}

// ------------ 任务函数模拟访问设备 -------------
void task_led_control() {
    Device* led = getDevice("led");
    if (led) {
        led->open();
        led->write("on");
        led->read();
        led->write("off");
        led->close();
    }
}

void task_temp_check() {
    Device* sensor = getDevice("temp_sensor");
    if (sensor) {
        sensor->open();
        sensor->read();
        sensor->write("try_to_write");  // 故意测试写入
        sensor->close();
    }
}

// ------------ 主函数 -------------
int main() {
    printf("🛠️ Mini RTOS v11 启动：设备管理模拟\n\n");

    // 注册两个设备
    registerDevice("led", led_open, led_read, led_write, led_close);
    registerDevice("temp_sensor", temp_open, temp_read, temp_write, temp_close);

    // 模拟任务访问设备
    printf("\n🔁 任务 1 开始操作 LED...\n");
    task_led_control();

    printf("\n🔁 任务 2 开始读取传感器...\n");
    task_temp_check();

    return 0;
}
🔍 功能说明：
1. 设备注册
所有设备通过 registerDevice() 注册到全局设备表

类似真实 RTOS 的设备管理器或驱动加载流程

2. 设备接口
每个设备实现自己的 open / read / write / close

任务通过设备名获取设备指针并调用这些操作

3. 任务使用设备
task_led_control() 操作 LED，控制开关

task_temp_check() 模拟读取传感器，测试只读特性

✅ 你掌握了什么：
功能	技术实现
设备抽象	通过结构体封装驱动函数接口
驱动注册	使用全局设备表，模拟设备管理
任务访问设备	任务通过设备名调用标准接口
💡 延伸思考：
想象你做了一个嵌入式系统，有多个传感器、显示屏、舵机，每个都可以注册为设备

RTOS 会统一调度和管理这些设备，保证设备不被多个任务同时访问（结合信号量/互斥锁）

🔔 下一阶段预告：
如果你想继续深入，我们可以挑战：

✅ 阶段 12：多核调度（Multi-core）与 CPU Affinity 模拟
✅ 阶段 13：构建一个完整小型嵌入式系统工程模板
