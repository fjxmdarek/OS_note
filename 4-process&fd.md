## 进程&文件

```
task_struct
  └── files (struct files_struct *)
        └── fdtab.fd[n] (struct file *)    ← n 即文件描述符编号
              └── f_inode (struct inode *) ← vnode 等价物
```

---

## 各层结构说明

### 1. `struct task_struct` — 进程表项

文件：`include/linux/sched.h`

每个进程对应一个 `task_struct`，其中字段：

```c
struct files_struct *files;
```

指向该进程持有的所有打开文件。

---

### 2. `struct files_struct` — 进程文件描述符表

文件：`include/linux/fdtable.h`

内部包含 `struct fdtable`，其核心字段：

```c
struct file __rcu **fd;   /* 下标即 fd 编号，值为 struct file 指针 */
```

`fd[0]` 对应 stdin，`fd[1]` 对应 stdout，`fd[2]` 对应 stderr，以此类推。

---

### 3. `struct file` — 打开文件对象

文件：`include/linux/fs.h`

代表一次 `open()` 系统调用产生的实例，记录：

- `f_pos`：当前读写偏移
- `f_flags`：打开标志（O_RDONLY 等）
- `f_op`：文件操作函数表（`struct file_operations *`）
- `f_path`：路径信息（含 dentry）
- `f_inode`：指向对应 inode 的指针（缓存值）

---

### 4. `struct inode` — 文件实体（vnode 等价物）

文件：`include/linux/fs.h`

代表文件系统中唯一的文件对象，与路径无关，包含：

- `i_mode`：文件类型与权限
- `i_op`：inode 操作表（`lookup`、`create` 等）
- `i_fop`：文件 I/O 操作表（`read`、`write` 等）
- `i_sb`：所属超级块
- `i_mapping`：页缓存（地址空间）

---
| 类比 | 打开的书签 | 书本身 |

同一个文件被多次 `open()`，会产生多个 `struct file`，但它们的 `f_inode` 都指向同一个 `struct inode`。
