# h100_on-boarding


ssh -J caijingnina-gmai-11e66a@jb1.frontier.nvidia.com:2222 caijingnina-gmai-11e66a@4.tun.tp1-cluster.nvidia.com




ssh -i ~/.ssh/id_ed25519 \
  -o "ProxyCommand=ssh -i ~/.ssh/id_ed25519 -W %h:%p caijingnina-gmai-11e66a@jb1.frontier.nvidia.com -p 2222" \
  caijingnina-gmai-11e66a@4.tun.tp1-cluster.nvidia.com



caijingnina-gmai-11e66a@Tunnel3-TRDC-02:~$ srun -N 1 -p NTU_Shared --gres=gpu:1 --pty /bin/bash
srun: job 5409 queued and waiting for resources
srun: job 5409 has been allocated resources
groups: cannot find name for group ID 1003
caijingnina-gmai-11e66a@cnode4-009:~$


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
