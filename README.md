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
我會幫您建立一個使用 odise.yml 和 requirements.txt 的自訂容器映像檔。讓我整理一個完整的步驟指南：

## 在 Taipei-1 上建立 ODISE 自訂映像檔

### 步驟 1：登入運算節點
```bash
srun -N 1 -p defq --mpi=pmix --gres=gpu:1 --ntasks-per-node 1 --pty /bin/bash
```

### 步驟 2：下載基礎映像
由於您的環境需要 Python 3.9 和 PyTorch 1.13.1，建議使用 PyTorch NGC 容器作為基礎：

```bash
# 下載 PyTorch 22.12 容器 (包含 PyTorch 1.13)
enroot import docker://nvcr.io#nvidia/pytorch:22.12-py3
```

這會生成類似 `nvidia+pytorch+22.12-py3.sqsh` 的檔案。

### 步驟 3：建立容器
```bash
enroot create --name odise_custom nvidia+pytorch+22.12-py3.sqsh
```

### 步驟 4：啟動容器並安裝套件
```bash
enroot start --root -w odise_custom /bin/bash
```

進入容器後，執行以下安裝指令：

```bash
# 更新 pip
pip install --upgrade pip

# 安裝 requirements.txt 中的套件
pip install absl-py==2.3.1
pip install aiohappyeyeballs==2.6.1
pip install aiohttp==3.12.15
pip install aiosignal==1.4.0
pip install antlr4-python3-runtime==4.9.3
pip install appdirs==1.4.4
pip install async-timeout==5.0.1
pip install attrs==25.3.0
pip install black==21.4b2
pip install boto3==1.40.25
pip install cloudpickle==3.1.1
pip install contourpy==1.3.0
pip install cycler==0.12.1
pip install einops==0.8.1
pip install filelock==3.19.1
pip install ftfy==6.3.1
pip install hydra-core==1.3.2
pip install omegaconf==2.3.0
pip install open-clip-torch==2.20.0
pip install opencv-python==4.11.0.86
pip install Pillow==9.5.0
pip install pycocotools==2.0.10
pip install pytorch-lightning==1.9.5
pip install PyYAML==6.0.2
pip install scipy==1.10.1
pip install transformers==4.36.2
pip install timm==0.6.11
pip install kornia==0.6.12

# 安裝 Detectron2
pip install 'git+https://github.com/facebookresearch/detectron2.git'

# 安裝其他重要套件
pip install scikit-image==0.24.0
pip install sentencepiece==0.2.1
pip install tqdm==4.67.1
pip install termcolor==3.1.0

# 離開容器
exit
```

### 步驟 5：匯出映像檔
```bash
enroot export --output odise_custom.sqsh odise_custom
```

### 步驟 6：清理容器
```bash
enroot remove odise_custom
exit
```

---

## 使用建立好的映像檔

### 互動式模式測試
```bash
srun \
  -N 1 \
  -p defq \
  --mpi=pmix \
  --gres=gpu:1 \
  --ntasks-per-node 1 \
  --container-image ${HOME}/odise_custom.sqsh \
  --container-writable \
  --container-remap-root \
  --pty /bin/bash
```

### 執行訓練作業（單節點 8 GPU）
```bash
srun \
  -N 1 \
  -p defq \
  --mpi=pmix \
  --gres=gpu:8 \
  --ntasks-per-node 1 \
  --container-image ${HOME}/odise_custom.sqsh \
  --container-writable \
  --container-mounts /mnt/SCRATCH/:/mnt/SCRATCH \
  python your_training_script.py
```

---

## 注意事項

1. **套件版本衝突**：您的 odise.yml 和 requirements.txt 有些套件版本不同（如 black、numpy 等），我以 requirements.txt 為主進行安裝

2. **PyTorch 版本**：基礎映像的 PyTorch 版本需與您要求的 1.13.1 匹配，NGC 的 22.12 版本包含此版本

3. **時間考量**：完整安裝所有套件可能需要 10-30 分鐘，請耐心等候

4. **儲存位置**：生成的 `odise_custom.sqsh` 會在您當前目錄，建議移動到 `${HOME}` 或專案目錄

需要我提供一個自動化安裝腳本嗎？

```bash
caijingnina-gmai-11e66a@cnode4-005:~$ ^C
caijingnina-gmai-11e66a@cnode4-005:~$ exit
exit
srun: error: cnode4-005: task 0: Exited with exit code 130
caijingnina-gmai-11e66a@Tunnel3-TRDC-02:~$ srun \
  -N 1 \
  -p NTU_Shared \
  --mpi=pmix \
  --gres=gpu:1 \
  --ntasks-per-node=1 \
  --container-image=${HOME}/odise_custom.sqsh \
  --container-writable \
  --container-remap-root \
  --pty /bin/bash
srun: job 5440 queued and waiting for resources
srun: job 5440 has been allocated resources
root@cnode4-005:/workspace#
```
