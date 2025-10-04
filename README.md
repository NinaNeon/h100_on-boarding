# h100_on-boarding


https://hackmd.io/@MasonWu/r1p9Hb88gg

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
這一段輸出顯示：你目前登入的節點 **（Tunnel3-TRDC-02）並不是計算節點**，而是登入用的「入口節點（login node）」或「tunnel node」── 它沒有安裝 Slurm 執行工具（`srun`, `sbatch`），所以你不能在這裡直接跑 GPU 任務。

---

✅ **解釋一下：**

* `Tunnel3-TRDC-02` 是「入口機」（login node / tunnel node）
* `cnode4-009` 才是你剛剛分配到的「運算節點（compute node）」
* Slurm 命令 (`srun`, `sbatch`, `sinfo`, `squeue`) 只能在有安裝 Slurm 的環境使用。通常會在主集群節點（例如 `jb1.frontier.nvidia.com` 之後連進的主控節點）執行。

---

### ✅ 正確的做法：

請你重新連線到 **主控節點** 或 **具備 Slurm 的節點**，然後再使用 `srun`。

你可以用下面這個命令進入正確環境（你之前成功過）：

```bash
ssh -i ~/.ssh/id_ed25519 \
  -o "ProxyCommand=ssh -i ~/.ssh/id_ed25519 -W %h:%p caijingnina-gmai-11e66a@jb1.frontier.nvidia.com -p 2222" \
  caijingnina-gmai-11e66a@4.tun.tp1-cluster.nvidia.com
```

進去後你會看到提示字樣像是：

```
Welcome to Base Command Manager 10.0
```

並且可以用：

```bash
module load slurm
sinfo
```

這樣才可以跑：

```bash
srun -N 1 -p NTU_Shared --gres=gpu:1 --pty /bin/bash
```

---

🔹 **重點：**

* Tunnel node (`Tunnel3-TRDC-02`) ➜ ❌ 沒有 Slurm，不能跑 GPU 任務
* Cluster node (`tp1-cluster.nvidia.com` 或 `cnode4-*`) ➜ ✅ 可以跑 Slurm 指令

---

要不要我幫你整理一份 **從登入 → 載入模組 → 啟動 GPU → 執行 PyTorch 測試** 的完整「一鍵流程範例」？
貼上就能直接使用（從登入節點開始一路執行）。

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

