#!/bin/bash

# Step 1: Set the hostname for the compute node
sudo hostnamectl set-hostname compute2

# Step 2: Update and install required dependencies
sudo apt update
sudo apt install -y build-essential munge libmunge-dev libmunge2 \
  libmysqlclient-dev libssl-dev libpam-dev libnuma-dev perl

# Step 3: Download and extract the Slurm tarball
wget https://download.schedmd.com/slurm/slurm-21.08.8.tar.bz2
tar -xvjf slurm-21.08.8.tar.bz2 slurm-21.08.8

# Step 4: Compile and install Slurm
cd slurm-21.08.8/
./configure --prefix=/home/ubuntu/slurm-21.08.8/
make
sudo make install

# Step 5: Configure Munge
sudo cp -r /tmp/munge.key /etc/munge/
sudo chown -R munge: /etc/munge /var/log/munge/
sudo chmod 0700 /etc/munge /var/log/munge/
sudo systemctl enable munge
sudo systemctl start munge

# Step 6: Configure Slurm
# Copy Slurm configuration from the controller
sudo mkdir -p /etc/slurm /etc/slurm-llnl/
sudo cp -r /tmp/slurm.conf /etc/slurm/
sudo cp -r /tmp/slurm.conf /etc/slurm-llnl/

# Step 7: Configure Slurm services
sudo cp slurmd.service /etc/systemd/system/
sudo systemctl enable slurmd
sudo systemctl start slurmd

# Step 8: Set up directories for Slurm daemon
sudo mkdir -p /var/spool/slurmd
sudo chown -R ubuntu:ubuntu /var/spool/slurmd/
sudo chmod 0755 /var/spool/slurmd/

# Step 9: Verify Munge and Slurm daemon statuses
sudo systemctl status munge
sudo systemctl status slurmd

# Step 10: Environment variables for Slurm
export LD_LIBRARY_PATH="/home/ubuntu/slurm-21.08.8/lib:$LD_LIBRARY_PATH"
export PATH="/home/ubuntu/slurm-21.08.8/sbin/:$PATH"
export PATH="/home/ubuntu/slurm-21.08.8/bin/:$PATH"

echo 'export LD_LIBRARY_PATH="/home/ubuntu/slurm-21.08.8/lib:$LD_LIBRARY_PATH"' >> ~/.bashrc
echo 'export PATH="/home/ubuntu/slurm-21.08.8/sbin/:$PATH"' >> ~/.bashrc
echo 'export PATH="/home/ubuntu/slurm-21.08.8/bin/:$PATH"' >> ~/.bashrc

# Source the updated bashrc
source ~/.bashrc

# Step 11: Verify compute node communication with the controller
sinfo
