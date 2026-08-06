# 命令行编译方法

APT 的 CDK 工程无需打开 CDK 界面，可命令行编译。

> ⚠️ **所有路径、工程名、配置名都用客户实际工程的值，不要照抄本文档里的示例名。**
> 工程目录 = 客户当前所在目录（或问客户）；工程名/配置名从客户的 `.cdkws`/`.cdkproj` 里读，不要假设叫 `DEMO_TK` 或 `BuildSet`。

## 统一编译命令：cdk-make（从 cdkproj 自动生成 mk/txt/bat 并编译）

正确用法（三个参数缺一不可）：
```
cdk-make.exe /w <工程名>.cdkws /p <工程名> /c <配置名> /d build
```

示例（仅演示格式，`DEMO_TK`/`BuildSet` 是占位名，换成客户实际的）：
```
cdk-make.exe /w DEMO_TK.cdkws /p DEMO_TK /c BuildSet /d build
```

- `/w` = workspace 文件
- `/p` = 工程名（≡ .cdkproj 中的 `<Project Name="...">`）
- `/c` = 编译配置名（≡ .cdkproj 中 `<BuildConfig Name="...">`）
- `/d` = 命令：`build` / `clean` / `rebuild`

**cdk-make 做的事**：读 `.cdkproj` → 自动生成 `.mk` / `.txt` / `.bat` → 调用 make 编译。
**从此不需要手动改 .mk**——增删文件只需改 `.cdkproj`，cdk-make 自动处理一切。

## 编译脚本(bat)
```bat
@echo off
cd /d <工程目录>
set PATH=.;<工具链>\bin;<CDK>\CSKY\MinGW\bin;<CDK>\CSKY\FlashProgrammer\Bins;%PATH%
<CDK>\cdk-make.exe /w <工程名>.cdkws /p <工程名> /c BuildSet /d rebuild
echo BUILD_EXITCODE=%errorlevel%
```
用 `rebuild` 确保 clean + 全量编译(增量缓存会骗人)。

## 路径探测(只问一次，存进度文件复用)

**先查 `PORTING_PROGRESS.md`**：如果已记录 CDK 路径，直接复用，不再问。

**没有缓存时，直接问客户一次**："CDK 装在哪个目录？"(通常是 `D:\C-Sky` 或 `C:\C-Sky`)。

得到路径后验证：确认 `<路径>\CDK\cdk-make.exe` 存在。**验证通过后写入 `PORTING_PROGRESS.md` 的关键上下文**。后续所有会话从进度文件读，永不重问。

参考值(实测，版本可能不同)：
- cdk-make：`<CDK根>\CDK\cdk-make.exe`
- 工具链(就在同一 CDK 下)：`<CDK根>\CDKRepo\Toolchain\CKV2ElfMinilib\V3.10.31\R\bin`，编译器 `csky-elfabiv2-gcc`，`-mcpu=ck801`
- make：`<CDK根>\CDK\CSKY\MinGW\bin\make.exe`
- crc32：`<CDK根>\CDK\CSKY\FlashProgrammer\Bins`
- 当前目录 `.`

## 硬规则

1. **必须整工程编译**，不能只编单文件。单文件发现不了链接层问题。
2. **每次验证前 rebuild**(≡ clean + build)，增量缓存会骗人。
3. **只改 cdkproj，不改 mk**。增删 `.c` 文件就改 `.cdkproj` 的 `<File>` 注册，然后 `cdk-make` 一键重新生成所有 mk/txt/bat。
   - `.h` 不需要注册——靠 IncludePath 的 `-I` 找到。
   - 新增子目录→改 cdkproj 的 `<IncludePath>`。
4. **判定**：`BUILD_EXITCODE=0` 且生成 `Obj/*.elf` `*.ihex` = 通过。
