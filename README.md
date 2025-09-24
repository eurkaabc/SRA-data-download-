# SRA Data Download and Fastq Conversion

## 介绍
此脚本用于从 NCBI 的 SRA 数据库下载 `.sra` 文件，并将其转换为 `.fastq.gz` 格式。它使用了 **SRAToolkit** 工具包中的 `fastq-dump` 或 `pfastq-dump` 命令，将每个 `.sra` 文件转换成对应的 `.fastq.gz` 文件，支持双端（paired-end）数据。

## 步骤

### 1. 下载并安装 **SRAToolkit**
首先，您需要安装 **SRAToolkit**。在您的机器上创建一个目录并下载工具包：

```bash
# 创建工具存放目录
mkdir ~/SRAToolkit

# 进入该目录
cd ~/SRAToolkit

# 下载 SRAToolkit 2.10.9 版本的工具包
wget https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/2.10.9/sratoolkit.2.10.9-ubuntu64.tar.gz

# 解压下载的工具包
tar xzvf sratoolkit.2.10.9-ubuntu64.tar.gz

```

### 2. 使用 prefetch 下载 .sra 文件####
下载完 SRAToolkit 后，使用 prefetch 命令来获取 SRA 数据文件。例如：
```bash
# 下载指定的 SRA 数据文件
prefetch SRR1553610  # 用您实际的 SRA 访问号替换 SRR1553610
```

### 3. 批量转换 .sra 文件为 .fastq.gz 格式###

假设您已经下载了多个 .sra 文件，现在我们可以将它们批量转换为 .fastq.gz 格式。以下是转换文件的脚本：

```bash
# Define the source directory containing the .sra files 
rawdata_dir="/home/Weixin.Zhang/20250918_pediatirc_tumor/rhabdoid_tumor/GSE71506/rawdata"

# Define the output directory for the fastq.gz files
output_dir="/home/Weixin.Zhang/20250918_pediatirc_tumor/rhabdoid_tumor/GSE71506/1.data"
mkdir -p "$output_dir"  # Create the output directory if it doesn't exist

# Run the process in the background with nohup
nohup bash -c "
  for sra_file in $rawdata_dir/*.sra; do
    # Get the base name of the file (without extension)
    base_name=\$(basename \"\$sra_file\" .sra)

    # Run fastq-dump (or pfastq-dump if you prefer)
    fastq-dump --split-files --gzip -O \"$output_dir\" \"\$sra_file\"

    # Optionally, check if output files were created and moved correctly
    echo \"Processed \$base_name.sra -> \$output_dir/\$base_name_1.fastq.gz and \$output_dir/\$base_name_2.fastq.gz\"
  done
" > "$output_dir/process_log.txt" 2>&1 &

echo "Process started in the background. Check the log file at $output_dir/process_log.txt"
```

### 4. 脚本详细解释 ###
目录定义

- `rawdata_dir`：存放 .sra 文件的源目录。

- `output_dir`：保存转换后 .fastq.gz 文件的目标目录。

#### 批量转换 ####

该脚本将遍历 `rawdata_dir` 中的所有 `.sra` 文件，使用 `fastq-dump` 进行转换。每个 `.sra` 文件将被转换为两个 `.fastq.gz` 文件（假设数据是双端的）。所有转换过程将在后台运行，以避免因会话断开而中断。

#### 日志输出 ####

转换过程的详细日志将被记录到 process_log.txt 文件中，您可以随时检查该文件以查看转换进度。

### 5. 使用 `nohup` 进行后台运行 ###

使用 nohup 确保即使在会话断开后，转换过程依然会在后台继续执行。输出会重定向到 process_log.txt 文件，您可以通过以下命令查看进度：
```
# 查看处理日志
`tail -f /home/Weixin.Zhang/20250918_pediatirc_tumor/rhabdoid_tumor/GSE71506/1.data/process_log.txt`
```

### 6. 注意事项 ###

- 确保目标目录有足够的存储空间来保存 .fastq.gz 文件，尤其是如果您有大量数据的话。

- `fastq-dump` 默认会将 `.sra` 文件分成两部分，分别对应 `_1.fastq.gz` 和 `_2.fastq.gz`。如果是单端数据，则只会生成 `_1.fastq.gz` 文件。

- 运行时，如果遇到任何错误，可以通过查看日志文件 process_log.txt 获取详细的错误信息。

### 总结 ###

该脚本自动化了从 NCBI 下载 `.sra` 数据并将其转换为 `.fastq.gz` 格式的过程，适用于大规模数据的批量处理。您只需指定源数据目录和输出目录，其他过程会自动执行。
