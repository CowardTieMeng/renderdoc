# 《明日方舟：终末地》GfxDebug 注入与截帧指南

本说明适用于去特征编译版 **GfxDebug**（基于 `strip-renderdoc` 分支构建，产物位于 `x64\Development\`）。

---

## 一、前置准备

1. **以管理员权限启动 GfxDebug**：
   - 右键 `x64\Development\qgfxdebug.exe` -> 选择 **“以管理员身份运行”**。
   - 确认窗口左上角标题栏末尾带有 **`(Administrator)`** 字样（游戏本体需要管理员权限，普通权限的调试器无法拉起或注入）。
2. **完全退出后台的鹰角启动器**：
   - 检查右下角系统托盘和任务管理器，若已有鹰角启动器进程在运行，请先将其**彻底退出**（否则后续拉起启动器时只会唤醒已有的后台进程，导致子进程注入链断裂）。

---

## 二、注入与启动方式（二选一）

因为《明日方舟：终末地》主程序 `Endfield.exe` 会校验是否由官方“鹰角启动器”拉起（直接启动 `Endfield.exe` 会弹窗报错 `请通过官方启动器打开游戏`），需通过以下两种方式之一注入：

### 方式一：通过启动器子进程接力注入（推荐）
利用 `Capture Child Processes` 自动沿着 `Launcher.exe -> Endfield.exe` 进程树递归注入 `gfxdebug.dll`。

1. 在 `qgfxdebug.exe` 打开 **`Launch Application`** 标签页。
2. **Executable Path（可执行文件路径）** 填写鹰角启动器路径：
   ```text
   D:\Hypergryph Launcher\Launcher.exe
   ```
3. **Working Directory（工作目录）**：留空（自动使用可执行文件所在目录）。
4. **Capture Options（截帧选项）**：
   - **[必选]** 勾选 **`Capture Child Processes`**（捕获并注入子进程）
   - **[推荐]** 勾选 **`Ref All Resources`**（保留帧内所有间接引用的贴图与 Buffer）
5. 点击右下角 **`Launch`** 拉起鹰角启动器，然后在鹰角启动器中正常点击 **“开始游戏”**。

---

### 方式二：通过全局进程钩子（Global Process Hook）注入
1. 首次使用需在顶部菜单打开 **`Tools` -> `Settings` -> `General`**，勾选 **`Enable process injection (restart required)`**，然后以管理员身份重启 `qgfxdebug.exe`。
2. 在 `Launch Application` 的 **Executable Path** 中填入游戏本体路径：
   ```text
   G:\EndField Game\Endfield.exe
   ```
3. 勾选 **`Ref All Resources`**，然后点击下方的 **`Enable Global Hook`** 按钮。
4. 正常打开鹰角启动器并点击 **“开始游戏”**。
5. 游戏启动并注入成功后，回到 `qgfxdebug.exe` 点击 **`Disable Global Hook`** 关闭全局钩子。

---

## 三、打开 `Live Capture` 窗口与游戏内截帧

1. **确认注入成功**：
   - 游戏窗口启动后，画面**左上角会出现黑底白字的 GfxDebug 状态叠层（Overlay）**，显示当前图形 API、帧率及截帧快捷键。
2. **连接并打开 `Live Capture` 窗口**：
   - 由于 `Launcher.exe` 引导进程在拉起启动器主界面后会自动退出，初始连接标签页会随之关闭；当游戏启动后，点击 `qgfxdebug.exe` 顶部菜单：
     **`File` -> `Attach to Running Instance`**
   - 在列表中展开 **`localhost`**，选中 **`Endfield [PID]`**，点击 **`Connect to App`** 即可打开实时截帧窗口。
3. **触发截帧**：
   - 在游戏内进入目标画面后，按键盘 **`F12`** 或 **`PrtScrn`** 键即可截帧；
   - 也可以切回 `qgfxdebug.exe` 的 `Live Capture` 窗口，点击 **`Capture Frame(s) Immediately`** 触发截帧。

---

## 四、截帧文件（`.rdc`）的存放与保存

1. **默认临时存放路径**：
   每按一次 `F12`，生成的 `.rdc` 文件都会立即写入系统临时目录（`%TEMP%\GfxDebug`）：
   ```text
   C:\Users\<用户名>\AppData\Local\Temp\GfxDebug
   ```
   文件名格式为：`Endfield_YYYY.MM.DD_HH.MM_frameXXXX.rdc`。
2. **打开与永久保存**：
   - **双击回放**：在 `Live Capture` 窗口右侧 `Captures collected` 列表中双击缩略图，或按 **`Ctrl + O`**（`File` -> `Open Capture`）从上述 `Temp\GfxDebug` 目录直接打开 `.rdc` 文件。
   - **永久保存**：在缩略图上**右键 -> `Save`**（或菜单 `File` -> `Save Capture As...`）将其另存到常用目录；或者直接去 `%TEMP%\GfxDebug` 目录将 `.rdc` 文件剪切/复制出来（避免关闭 `qgfxdebug.exe` 时若选择“不保存”导致临时文件被自动清理）。
3. **修改默认存放位置**：
   - 在 **`Tools` -> `Settings` -> `General`** 中可修改：
     - `Directory for temporary capture files`（按 `F12` 时的临时落地目录）
     - `Default directory for saving captures`（手动保存时的默认目录）

---

## 五、常见问题排查

| 现象 / 报错 | 原因 | 解决方法 |
| :--- | :--- | :--- |
| `GfxDebug injection failed: Failed to launch process.` | `qgfxdebug.exe` 未以管理员权限运行，无法拉起需要 UAC 提权的进程 | 关闭后右键 `qgfxdebug.exe` -> **以管理员身份运行** |
| 游戏弹窗：`请通过官方启动器打开游戏` | 在 `Launch Application` 中直接点击 `Launch` 启动了 `Endfield.exe` | 改用**方式一**（填 `D:\Hypergryph Launcher\Launcher.exe` + 勾选 `Capture Child Processes`）或**方式二**（`Enable Global Hook`） |
| 启动启动器后没有注入进游戏 | 后台已有旧的鹰角启动器进程在运行，新启动器直接退出了 | 先从系统托盘/任务管理器彻底退出鹰角启动器相关进程，再从 `qgfxdebug` 点击 `Launch` |
| 游戏内有左上角 Overlay，但 `qgfxdebug` 里看不到 `Live Capture` 标签页 | `Launcher.exe` 引导进程退出时关闭了初始标签页 | 点击菜单 **`File` -> `Attach to Running Instance`** -> 选择 `localhost` 下的 `Endfield` 重新连接 |
