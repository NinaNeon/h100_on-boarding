# h100_on-boarding

```bash
ssh -J caijingnina-gmai-11e66a@jb1.frontier.nvidia.com:2222 caijingnina-gmai-11e66a@4.tun.tp1-cluster.nvidia.com
```


```bash
ssh -i ~/.ssh/id_ed25519 \
  -o "ProxyCommand=ssh -i ~/.ssh/id_ed25519 -W %h:%p caijingnina-gmai-11e66a@jb1.frontier.nvidia.com -p 2222" \
  caijingnina-gmai-11e66a@4.tun.tp1-cluster.nvidia.com
```

```bash
caijingnina-gmai-11e66a@Tunnel3-TRDC-02:~$ srun -N 1 -p NTU_Shared --gres=gpu:1 --pty /bin/bash
srun: job 5409 queued and waiting for resources
srun: job 5409 has been allocated resources
groups: cannot find name for group ID 1003
caijingnina-gmai-11e66a@cnode4-009:~$
```
```bash
caijingnina-gmai-11e66a@cnode4-009:~$ sbatch gpu.sh
sbatch: error: Unable to open file gpu.sh
caijingnina-gmai-11e66a@cnode4-009:~$ nano gpu.sh
caijingnina-gmai-11e66a@cnode4-009:~$ sbatch gpu.sh
Submitted batch job 5410
caijingnina-gmai-11e66a@cnode4-009:~$ squeue -u $USER
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
              5409 NTU_Share     bash caijingn  R       3:02      1 cnode4-009
caijingnina-gmai-11e66a@cnode4-009:~$ cat slurm-<jobID>.out
bash: jobID: No such file or directory
caijingnina-gmai-11e66a@cnode4-009:~$
```

✅ 完美！你已经确认拿到一张 **H100 80GB HBM3**，现在在 **计算节点 cnode4-009** 上，GPU 空闲可以使用。

你跑 `sbatch gpu.sh` 报错是因为当前目录下没有 `gpu.sh` 文件。
要用 `sbatch` 提交任务，你需要先准备一个 **Slurm Job Script**。

---

### 最小化范例：gpu.sh

在 `cnode4-009` 上运行：

```bash
nano gpu.sh
```

粘贴以下内容（按需修改）：

```bash
#!/bin/bash
#SBATCH -J test_gpu          # 任务名称
#SBATCH -p NTU_Shared        # 分区
#SBATCH -N 1                 # 节点数
#SBATCH --gres=gpu:1         # 请求 1 张 GPU
#SBATCH -t 00:30:00          # 最长运行时间 (30 分钟)
#SBATCH -o slurm-%j.out      # 标准输出 (%j = job ID)

# 打印节点信息和 GPU 状态
hostname
nvidia-smi

# 如果要跑 Python 程序，记得加载模块/conda
# module load anaconda3
# conda activate myenv
# python train.py
```

保存后（`Ctrl+O` → `Enter` → `Ctrl+X`），再提交：

```bash
sbatch gpu.sh
```

查看队列状态：

```bash
squeue -u $USER
```

查看输出：

```bash
cat slurm-<jobID>.out
```

---

👌 你现在已经成功提交了 **Job 5410**：

```bash
Submitted batch job 5410
```

这时候输出文件的名字会是

```
slurm-5410.out
```

所以你需要这样查看：

```bash
cat slurm-5410.out
```

---

### 小技巧

1. 如果你以后不想手动改 `<jobID>`，可以直接：

   ```bash
   ls slurm-*.out
   ```

   就能看到所有输出文件。

2. 如果 job 还没跑完，文件可能还没生成或内容不全。
   可以一边等一边看：

   ```bash
   tail -f slurm-5410.out
   ```

---

要不要我帮你改 `gpu.sh`，加上一段 **Python (PyTorch CUDA 测试代码)**？这样跑完后输出文件里会直接显示 GPU 能不能被 PyTorch 用。

