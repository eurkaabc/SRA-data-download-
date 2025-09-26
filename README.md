# SRA Data Download and Fastq Conversion

## Introduction
This script is designed to download `.sra` files from the NCBI SRA database and convert them into `.fastq.gz` format. It utilizes the **SRAToolkit** package's `fastq-dump` or `pfastq-dump` commands to convert each `.sra` file into the corresponding `.fastq.gz` files, supporting paired-end data.


### 1. Download and Install **SRAToolkit**

First, you need to install the **SRAToolkit**. Create a directory on your machine and download the toolkit:


```bash
# Create a directory for the tools
mkdir ~/SRAToolkit

# Navigate into the directory
cd ~/SRAToolkit

# Download SRAToolkit version 2.10.9
wget https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/2.10.9/sratoolkit.2.10.9-ubuntu64.tar.gz

# Extract the downloaded toolkit
tar xzvf sratoolkit.2.10.9-ubuntu64.tar.gz

```

### 2. Use `prefetch`to Download `.sra` Files
Once SRAToolkit is installed, you can use the prefetch command to download SRA data files. For example:
```bash
# Download a specific SRA data file
prefetch SRR1553610  # Replace SRR1553610 with your actual SRA accession number
```

### 3. Batch Convert `.sra` Files to `.fastq.gz` Format

After downloading your `.sra` files, you can batch convert them into `.fastq.gz` format. Here's the script to do that:

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

### 4. Script Breakdown
#### Directory Definitions:

- `rawdata_dir`：存放 .sra 文件的源目录。

- `output_dir`：保存转换后 .fastq.gz 文件的目标目录。

#### Batch Conversion ####

The script will loop through all `.sra` files in the `rawdata_dir` and use `fastq-dump` to convert them. Each `.sra` file will be converted into two `.fastq.gz` files (for paired-end data). The entire conversion process runs in the background to avoid interruption due to session disconnection.

#### Logging: ####

Detailed logs of the conversion process are saved to `process_log.txt`. You can monitor this file at any time to check the conversion progress.

### 5. Running the Script in the Background with `nohup`  ###

To ensure the conversion continues in the background even if your session disconnects, we use `nohup`. The output will be redirected to `process_log.txt`, which you can monitor as follows:
```
# View the processing log
`tail -f /home/Weixin.Zhang/20250918_pediatirc_tumor/rhabdoid_tumor/GSE71506/1.data/process_log.txt`
```

### 6. Considerations ###

- Ensure that your target directory has sufficient storage space to hold the `.fastq.gz` files, especially if you're working with large datasets.

- By default, `fastq-dump` splits the `.sra` files into two parts, resulting in `_1.fastq.gz` and `_2.fastq.gz` files for paired-end data. For single-end data, only `_1.fastq.gz` files will be generated.

- If any errors occur during the process, the details will be logged in `process_log.txt`, where you can find more information about the issues.

### Conclusion ###

This script automates the process of downloading `.sra` data from NCBI and converting it to .fastq.gz format, making it ideal for large-scale batch processing. Simply specify the source directory for your data and the output directory, and the rest of the process will run automatically.
