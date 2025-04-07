我们现在进入 RTOS 教程的第 10 阶段：模拟内存管理机制（Memory Management） 🧠🧱

🎯 本阶段目标：
我们将模拟 RTOS 中常见的内存管理机制，包括：

内存池（Memory Pool）：固定大小的内存块，用于任务或对象的分配与回收

动态内存分配：用 malloc/free 或自定义方法来动态获取/释放内存

我们将通过代码实现一个简化版的内存池，实现任务或对象的内存申请、释放过程。

🧱 内存池的概念（比动态分配更适合 RTOS）：
特性	解释
✅ 快速	不需要像 malloc 那样遍历堆，只是拿一个“空闲块”用
✅ 固定大小	每块内存大小一样，分配效率高
✅ 可控	不会产生内存碎片（fragmentation）
✅ 常用于	通信缓冲区、任务栈空间、对象池管理等
🛠️ 示例代码：mini_rtos_v10_memory_pool.c
我们创建一个“固定块大小的内存池”，用于模拟分配任务所需的内存空间。

c
复制
编辑
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define POOL_SIZE 5        // 总共可用内存块数量
#define BLOCK_SIZE 64      // 每块内存大小（字节）

typedef struct {
    int in_use;            // 0=空闲, 1=已占用
    char data[BLOCK_SIZE]; // 模拟内存内容
} MemoryBlock;

// 定义内存池
MemoryBlock memoryPool[POOL_SIZE];

// 初始化内存池
void initMemoryPool() {
    for (int i = 0; i < POOL_SIZE; i++) {
        memoryPool[i].in_use = 0;
    }
    printf("🛠️ 内存池初始化完成，共 %d 块，每块 %d 字节\n", POOL_SIZE, BLOCK_SIZE);
}

// 分配内存块
void* memAlloc() {
    for (int i = 0; i < POOL_SIZE; i++) {
        if (!memoryPool[i].in_use) {
            memoryPool[i].in_use = 1;
            printf("✅ 分配内存块 #%d\n", i);
            return memoryPool[i].data;
        }
    }
    printf("❌ 内存池已满，无法分配内存\n");
    return NULL;
}

// 释放内存块
void memFree(void* ptr) {
    for (int i = 0; i < POOL_SIZE; i++) {
        if (memoryPool[i].data == ptr) {
            memoryPool[i].in_use = 0;
            printf("🔓 释放内存块 #%d\n", i);
            return;
        }
    }
    printf("⚠️ 错误：尝试释放未分配的内存块\n");
}

// 测试用任务模拟分配和释放内存
void testMemoryManagement() {
    void* ptr1 = memAlloc();
    void* ptr2 = memAlloc();
    void* ptr3 = memAlloc();

    strcpy((char*)ptr1, "任务1数据");
    strcpy((char*)ptr2, "任务2数据");

    printf("📦 任务1内存内容: %s\n", (char*)ptr1);
    printf("📦 任务2内存内容: %s\n", (char*)ptr2);

    memFree(ptr2); // 释放任务2的内存
    memAlloc();    // 再次尝试分配
}

int main() {
    printf("🧠 Mini RTOS v10 启动：模拟内存池管理\n\n");
    initMemoryPool();
    testMemoryManagement();
    return 0;
}
🔍 程序说明：
内存池初始化：

总共有 5 个固定大小的内存块（64 字节）

每块用 in_use 标志判断是否被占用

内存分配（memAlloc）：

遍历内存池，找到空闲块就分配

如果池满，则返回 NULL（模拟内存不足）

内存释放（memFree）：

检查释放的是哪一块，将其标记为未占用

模拟任务使用内存：

任务 1 和 2 使用内存块

任务 2 释放内存后，再次分配给新任务

✅ 你掌握的内容：
功能	实现方式
内存池结构	使用数组管理固定数量的内存块
分配内存	遍历池查找空闲块，标记为占用
释放内存	根据指针找到对应块并释放
安全控制	防止重复释放、释放未分配的块
🧠 延伸思考：
如果你要分配不同大小的数据，可以实现多级内存池（小块池、大块池）

如果任务数量非常大，内存池能有效防止系统崩溃（堆溢出）

🔔 下一阶段预告：
