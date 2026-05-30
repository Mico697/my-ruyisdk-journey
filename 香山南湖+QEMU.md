## **快速入门：香山南湖 + QEMU 开发体验**
欢迎开始您的 RuyiSDK + 香山南湖开发之旅！  
本指南专为没有 RISC-V 硬件设备的用户设计，您可以通过 QEMU 模拟器快速体验从开发到运行的完整流程。

---

### **学习目标**：
- 掌握 RuyiSDK 包管理器的安装和基础使用
- 学会搭建 RISC-V 的交叉编译环境
- 理解 RISC-V 程序编译与运行流程
- 通过 QEMU 模拟香山南湖处理器环境，完成项目实践

---

### **环境要求**
- **操作平台**：Linux x86_64（推荐主流开发环境，如 WSL2）
- **安装内容**：
  - RuyiSDK 包管理器
  - gnu-upstream 交叉编译工具链
  - QEMU RISC-V 模拟器（支持香山南湖或替代架构）

---

## **0. 判断 QEMU 是否支持香山南湖架构**

### **步骤 0.1**：查看可用的 QEMU 包
```bash
ruyi list --all | grep qemu
```

### **步骤 0.2**：查看 QEMU 支持的 CPU 类型
创建临时环境并激活：
```bash
ruyi venv -t gnu-upstream -e qemu-user-riscv-upstream generic ~/ruyi-test-env
source ~/ruyi-test-env/bin/ruyi-activate
```


检查支持的 CPU（查找 `nanhu`、`xiangshan`、`c910`）：
```bash
ruyi-qemu -cpu help | grep -i "nanhu\|xiangshan\|c910"
```
<img width="765" height="116" alt="image" src="https://github.com/user-attachments/assets/3d8b2baf-40a7-43f9-876a-b16bb2404c59" />

### **结果判断**
- 如支持 `"xiangshan"` 或 `"nanhu"`：可直接使用 `-cpu xiangshan-nanhu`
- 如支持 `"xuantie-c910"`：可使用 `-cpu xuantie-c910` 作为替代
- 如仅支持 `"rv64"`：
  ```bash
  ruyi update
  ```
  然后重复上述步骤更新 QEMU 包。

---

## **1. 环境搭建与安装**

### **步骤 1.1**：创建虚拟环境
```bash
ruyi venv -t gnu-upstream -e qemu-user-riscv-upstream generic ~/ruyi-nanhu-env
```

### **步骤 1.2**：激活虚拟环境
```bash
source ~/ruyi-nanhu-env/bin/ruyi-activate
```
激活成功后提示符变为：  
`«Ruyi ruyi-nanhu-env» user@host:~$`

<img width="431" height="30" alt="image" src="https://github.com/user-attachments/assets/378d3cfc-45d3-4ec9-aabf-db16ec75974b" />

### **步骤 1.3**：验证工具链和 QEMU
```bash
riscv64-unknown-linux-gnu-gcc --version
ruyi-qemu --version
```
说明：RuyiSDK 提供的 `ruyi-qemu` 自动配置了 sysroot 和动态链接器路径，无需手动指定 `-static`。
<img width="786" height="425" alt="image" src="https://github.com/user-attachments/assets/16bd173a-60c1-4acc-b00d-399937975eb1" />


---

## **2. 项目实践：Hello World**

### **步骤 2.1**：编写示例程序
创建源文件 `hello.c`：
```c
#include <stdio.h>
int main() {
    printf("Hello, World!\n");
    return 0;
}
```

### **步骤 2.2**：交叉编译
```bash
riscv64-unknown-linux-gnu-gcc hello.c -o hello
```

### **步骤 2.3**：验证文件格式
```bash
file hello
```
验证输出应该显示为：
`ELF 64-bit LSB pie executable, UCB RISC-V, RVC, dynamically linked...`

### **步骤 2.4**：在 QEMU 中模拟运行
```bash
ruyi-qemu ./hello
```
预期输出：
```plaintext
Hello, World!
```

---

## **3. 项目实践进阶：Coremark 基准测试**

### **步骤 3.1**：提取 Coremark 源码
```bash
mkdir -p ~/coremark && cd ~/coremark
ruyi extract coremark
```

### **步骤 3.2**：进入源码目录
```bash
cd coremark-1.0.1
```

### **步骤 3.3**：编译程序
```bash
make PORT_DIR=linux64 CC=riscv64-unknown-linux-gnu-gcc link
```

### **步骤 3.4**：在 QEMU 中运行
```bash
ruyi-qemu ./coremark.exe
```
预期输出：  
显示 CoreMark 的基准测试结果，如：
```plaintext
CoreMark 1.0 : 6604.623236 / GCC13.2.0 ...
```
<img width="766" height="650" alt="image" src="https://github.com/user-attachments/assets/8e44af53-7757-47a9-bd09-80f40515492a" />


---

## **4. 常用命令速查**

- **创建开发环境**
  ```bash
  ruyi venv -t gnu-upstream -e qemu-user-riscv-upstream generic ~/ruyi-nanhu-env
  ```

- **激活环境**
  ```bash
  source ~/ruyi-nanhu-env/bin/ruyi-activate
  ```

- **退出环境**
  ```bash
  ruyi-deactivate
  ```

- **更新包列表**
  ```bash
  ruyi update
  ```

- **查看已安装包**
  ```bash
  ruyi list installed
  ```

---

## **5. 常见问题**

### Q1: 激活时报错 "No such file or directory"
**解决方法**：确保使用的是 `ruyi-activate` 而非常规的 `activate`。

### Q2: `riscv64-unknown-linux-gnu-gcc` 找不到
**解决方法**：
```bash
ruyi update
ruyi venv -t gnu-upstream -e qemu-user-riscv-upstream generic ~/new-env
```

### Q3: Coremark 编译报错 "gcc: command not found"
**解决方法**：显式指定交叉编译器：
```bash
make CC=riscv64-unknown-linux-gnu-gcc link
```

---

通过本指南，希望您能够快速完成香山南湖的开发与运行！如果遇到其他问题，欢迎随时交流解决。 😊
