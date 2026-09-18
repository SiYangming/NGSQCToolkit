# NGSQCToolkit 使用示例

下文中的 `path/to/`、`/path/to/install/` 为占位路径，请按实际环境替换。示例输入为双端 FASTQ。

```bash
NGSQC=/path/to/install/NGSQCToolkit_v2.3.3
```

## 示例 1：质量过滤（IlluQC_PRLL.pl）

`IlluQC_PRLL.pl` 是并行版 QC 脚本，按质量与长度阈值过滤 reads，输出高质量数据与统计图表。

```bash
NGSQC=/path/to/install/NGSQCToolkit_v2.3.3
mkdir -p path/to/NGSQCToolkit
cd path/to/NGSQCToolkit

# -pe 后依次为 R1、R2；2 选择文件类型；1 选择内置接头/引物库编号
# -l 高质量碱基长度百分比阈值；-s 高质量 PHRED 阈值；-c CPU 数；-o 输出目录
$NGSQC/QC/IlluQC_PRLL.pl \
  -pe path/to/data/fragment.1.fastq path/to/data/fragment.2.fastq \
  2 1 -l 60 -s 13 -c 4 -o fragment

$NGSQC/QC/IlluQC_PRLL.pl \
  -pe path/to/data/jumping.1.fastq path/to/data/jumping.2.fastq \
  2 1 -l 60 -s 13 -c 4 -o jumping
```

输出为过滤后的高质量 reads（按脚本约定，文件名在原名后追加 `_filtered`）。

## 示例 2：3' 端质量截短（TrimmingReads.pl）

```bash
$NGSQC/Trimming/TrimmingReads.pl \
  -i path/to/data/jumping.1.fastq \
  -irev path/to/data/jumping.2.fastq \
  -r 30
```

输出文件名在输入文件名后追加 `_trimmed`。

## 示例 3：去除接头序列（自定义接头/引物文件）

```bash
cat > primer_adaptor.txt <<'EOF'
AATGATACGGCGACCACCGAGATCTACACTCTTTCCCTACACGACGCTCTTCCGATCT
CAAGCAGAAGACGGCATACGAGATCGGTCTCGGCATTCCTGCTGAACCGCTCTTCCGATCT
AATGATACGGCGACCACCGAGATCTACACTCTTTCCCTACACGACGCTCTTCCGATCT
AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGTAGATCTCGGTGGTCGCCGTATCATT
CAAGCAGAAGACGGCATACGAGATCGGTCTCGGCATTCCTGCTGAACCGCTCTTCCGATCT
AGATCGGAAGAGCGGTTCAGCAGGAATGCCGAGACCGATCTCGTATGCCGTCTTCTGCTTG
TTTTTTTTTTAATGATACGGCGACCACCGAGATCTACAC
TTTTTTTTTTCAAGCAGAAGACGGCATACGA
TACACTCTTTCCCTACACGACGCTCTTCCGATCT
GTGACTGGAGTTCAGACGTGTGCTCTTCCGATCT
TACACTCTTTCCCTACACGACGCTCTTCCGATCT
AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGTA
GTGACTGGAGTTCAGACGTGTGCTCTTCCGATCT
AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC
EOF

# 第 2 个位置参数指定自定义接头/引物文件，随后 1 表示按文件过滤
$NGSQC/QC/IlluQC_PRLL.pl \
  -pe jumping.1.fastq_trimmed jumping.2.fastq_trimmed \
  primer_adaptor.txt 1 -l 60 -s 13 -c 4 -o jumping

cp jumping/jumping.1.fastq_trimmed_filtered jumping.1.fastq
cp jumping/jumping.2.fastq_trimmed_filtered jumping.2.fastq
```

## 示例 4：去除含 N 的 reads（AmbiguityFiltering.pl）

```bash
$NGSQC/Trimming/AmbiguityFiltering.pl \
  -i fragment/fragment.1.fastq_filtered \
  -irev fragment/fragment.2.fastq_filtered \
  -p 5

cp fragment/fragment.1.fastq_filtered_ambFiltered fragment.1.fastq
cp fragment/fragment.2.fastq_filtered_ambFiltered fragment.2.fastq
```

## 示例 5：其它可用脚本

`Statistics/`、`Format-converter/`、`Trimming/` 下另有若干独立脚本，可直接调用：

- `Statistics/AvgQuality.pl`：统计平均质量
- `Statistics/N50Stat.pl`：N50 统计
- `Trimming/HomopolymerTrimming.pl`：同聚物修剪
- `Format-converter/FastqToFasta.pl`、`FastqTo454.pl`：FASTQ 转 FASTA / 454 格式
- `Format-converter/SangerFastqToIlluFastq.pl`、`SolexaFastqToIlluFastq.pl`：FASTQ 变体间转换
- `QC/IlluQC.pl`（单线程）、`QC/454QC.pl`、`QC/454QC_PE.pl`、`QC/454QC_PRLL.pl`：其它平台/模式 QC

各脚本参数以仓库内 `NGSQCToolkitv2.3.3_manual.pdf` 为准。

## 参数说明

- `-pe`：双端输入，后依次为 R1、R2 文件。
- QC 脚本的位置参数：文件类型/平台选择，以及接头/引物库（内置库编号，或自定义序列文件）。
- `-l`：高质量碱基长度百分比阈值。
- `-s`：高质量 PHRED 阈值。
- `-c`：CPU 数（并行度）。
- `-o`：输出目录（QC）或输出文件名（Trimming、Statistics）。
- `-i` / `-irev`：输入单端 / 双端文件。
- `-r`：3' 端固定截短长度。
- `-p`：`AmbiguityFiltering.pl` 的最大 N 百分比阈值。

约束：

- `TrimmingReads.pl` 的 `-q`（质量截短）与 `-l/-r`（固定截短）互斥。
- `AmbiguityFiltering.pl` 的 `-c/-p/-t5/-t3` 四者任选其一。
- 以上互斥关系由脚本自身校验。

## 注意事项

- 各脚本依赖同目录 `QC/lib/`（`Parallel::ForkManager` 等），不要脱离该目录单独运行脚本。
- QC 图形输出依赖 GD / GD::Text / GD::Graph；只需文本统计时可不安装 GD 系模块。
