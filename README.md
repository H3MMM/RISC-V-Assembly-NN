# RISC-V 汇编语言实现的神经网络推理

[![zread](https://img.shields.io/badge/Ask_Zread-_.svg?style=for-the-badge&color=00b0aa&labelColor=000000&logo=data%3Aimage%2Fsvg%2Bxml%3Bbase64%2CPHN2ZyB3aWR0aD0iMTYiIGhlaWdodD0iMTYiIHZpZXdCb3g9IjAgMCAxNiAxNiIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHBhdGggZD0iTTQuOTYxNTYgMS42MDAxSDIuMjQxNTZDMS44ODgxIDEuNjAwMSAxLjYwMTU2IDEuODg2NjQgMS42MDE1NiAyLjI0MDFWNC45NjAxQzEuNjAxNTYgNS4zMTM1NiAxLjg4ODEgNS42MDAxIDIuMjQxNTYgNS42MDAxSDQuOTYxNTZDNS4zMTUwMiA1LjYwMDEgNS42MDE1NiA1LjMxMzU2IDUuNjAxNTYgNC45NjAxVjIuMjQwMUM1LjYwMTU2IDEuODg2NjQgNS4zMTUwMiAxLjYwMDEgNC45NjE1NiAxLjYwMDFaIiBmaWxsPSIjZmZmIi8%2BCjxwYXRoIGQ9Ik00Ljk2MTU2IDEwLjM5OTlIMi4yNDE1NkMxLjg4ODEgMTAuMzk5OSAxLjYwMTU2IDEwLjY4NjQgMS42MDE1NiAxMS4wMzk5VjEzLjc1OTlDMS42MDE1NiAxNC4xMTM0IDEuODg4MSAxNC4zOTk5IDIuMjQxNTYgMTQuMzk5OUg0Ljk2MTU2QzUuMzE1MDIgMTQuMzk5OSA1LjYwMTU2IDE0LjExMzQgNS42MDE1NiAxMy43NTk5VjExLjAzOTlDNS42MDE1NiAxMC42ODY0IDUuMzE1MDIgMTAuMzk5OSA0Ljk2MTU2IDEwLjM5OTlaIiBmaWxsPSIjZmZmIi8%2BCjxwYXRoIGQ9Ik0xMy43NTg0IDEuNjAwMUgxMS4wMzg0QzEwLjY4NSAxLjYwMDEgMTAuMzk4NCAxLjg4NjY0IDEwLjM5ODQgMi4yNDAxVjQuOTYwMUMxMC4zOTg0IDUuMzEzNTYgMTAuNjg1IDUuNjAwMSAxMS4wMzg0IDUuNjAwMUgxMy43NTg0QzE0LjExMTkgNS42MDAxIDE0LjM5ODQgNS4zMTM1NiAxNC4zOTg0IDQuOTYwMVYyLjI0MDFDMTQuMzk4NCAxLjg4NjY0IDE0LjExMTkgMS42MDAxIDEzLjc1ODQgMS42MDAxWiIgZmlsbD0iI2ZmZiIvPgo8cGF0aCBkPSJNNCAxMkwxMiA0TDQgMTJaIiBmaWxsPSIjZmZmIi8%2BCjxwYXRoIGQ9Ik00IDEyTDEyIDQiIHN0cm9rZT0iI2ZmZiIgc3Ryb2tlLXdpZHRoPSIxLjUiIHN0cm9rZS1saW5lY2FwPSJyb3VuZCIvPgo8L3N2Zz4K&logoColor=ffffff)](https://zread.ai/H3MMM/RISC-V-Assembly-NN)

本项目是加州大学伯克利分校 CS61C 课程（计算机体系结构）的经典项目。其核心目标是使用纯 RISC-V 汇编语言，从零开始实现一个基础的神经网络，并最终能够对经典的 MNIST 手写数字数据集进行分类。

这个项目不仅考验了对 RISC-V 指令集的熟练掌握，更深入地探索了底层计算、内存管理和算法在汇编层面的实现。

## 核心功能

- **基础矩阵/向量运算模块**:
  - `dot`: 实现向量点积。
  - `matmul`: 实现矩阵乘法。
  - `argmax`: 找到向量中最大元素的索引。
  - `relu`: 实现 ReLU (Rectified Linear Unit) 激活函数。
  - `abs`: 实现取绝对值函数。

- **文件 I/O 与内存管理**:
  - `read_matrix`: 从二进制文件中读取矩阵数据到内存。
  - `write_matrix`: 将内存中的矩阵数据写入到二进制文件。
  - 所有函数都严格遵循 RISC-V 调用约定，并进行了审慎的内存管理。

- **神经网络分类器**:
  - `classify`: 整合上述所有模块，构建一个完整的前馈神经网络推理流程，能够加载模型权重（`m0.txt`, `m1.txt`）和输入数据，并输出分类结果。

- **完善的测试框架**:
  - 使用 Python 的 `unittest` 框架搭建自动化测试流水线 ([unittests.py](fa20-proj2-starter/unittests/unittests.py))。
  - 动态生成测试汇编代码，调用 Venus 模拟器 ([tools/venus.jar](fa20-proj2-starter/tools/venus.jar)) 执行并验证结果。
  - 覆盖了从单元函数到集成测试的各个层面。

## 技术亮点与挑战 (Highlights & Challenges)

1.  **RISC-V 汇编编程**:
    所有核心逻辑均使用 RISC-V 汇编语言编写。这意味着没有高级语言的抽象，所有操作都需要直接通过指令完成，包括：
    - **循环与分支**: 手动构建 `for` 和 `if` 等控制流逻辑。
    - **指针算术**: 精确计算内存地址，尤其是在处理二维矩阵时，需要将 `(row, col)` 索引转换为一维内存偏移。
    - **函数调用与栈管理**: 严格遵循 RISC-V 调用约定，手动管理栈指针 (`sp`)，保存和恢复调用者/被调用者保存的寄存器 (`ra`, `s0-s11`)。

2.  **矩阵乘法 (`matmul.s`) 的实现**:
    这是项目中最复杂的部分之一。在汇编层面实现三层嵌套循环的矩阵乘法，对寄存器分配和地址计算提出了极高的要求。为了优化性能，需要仔细规划寄存器的使用，以最小化内存访问（load/store 指令）的次数。

3.  **端到端的推理流程 (`classify.s`)**:
    `classify` 函数是整个项目的集大成者。它需要像胶水一样将各个独立的汇编模块（`read_matrix`, `matmul`, `relu`, `argmax` 等）粘合起来，形成一个完整的工作流。这要求对 RISC-V 的函数调用约定有深刻的理解，以便正确地传递参数（如文件名、矩阵维度、内存地址）和处理返回值。

4.  **二进制文件 I/O**:
    `read_matrix.s` 和 `write_matrix.s` 直接与文件系统交互。这涉及到使用 `ecall`（系统调用）来打开、读取、写入和关闭文件。在汇编中处理文件描述符、缓冲区和错误码是一项复杂的任务，需要精确控制参数寄存器 (`a0-a7`)。

## 项目结构

```
.
├── inputs/              # 测试输入数据 (包括 MNIST 和 simple tests)
├── outputs/             # 运行测试时生成的输出文件
├── src/                 # 你的 RISC-V 汇编代码实现
│   ├── argmax.s
│   ├── classify.s
│   ├── dot.s
│   ├── matmul.s
│   ├── read_matrix.s
│   ├── relu.s
│   └── write_matrix.s
├── tools/
│   ├── convert.py       # 矩阵文件格式转换工具
│   └── venus.jar        # RISC-V 模拟器
└── unittests/
    ├── framework.py     # 课程提供的测试框架 (勿动)
    └── unittests.py     # 用于编写和运行测试用例的 Python 脚本
```

## 如何运行测试

环境要求:
- Python 3
- Java (用于运行 `venus.jar`)

1.  进入 `unittests` 目录:
    ```bash
    cd fa20-proj2-starter/unittests
    ```

2.  运行所有单元测试:
    ```bash
    python3 unittests.py
    ```
    测试框架会自动编译和运行 `src` 目录下的相应汇编文件，并报告每个测试用例的成功或失败。

---
