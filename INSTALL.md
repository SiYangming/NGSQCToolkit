# NGSQCToolkit 安装与配置

NGSQCToolkit（NGS QC Toolkit）是 NIPGR 用 Perl 编写的 NGS 数据质控工具包，面向 Illumina 与 Roche 454 平台，提供质量统计与过滤、3' 端修剪、去 N、同聚物修剪、格式转换等功能。

- 引用：Patel RK, Jain M (2012). *NGS QC Toolkit: a toolkit for quality control of next generation sequencing data.* PLoS ONE 7(2): e30619.
- 本仓库归档版本：**2.3.3**

> 仓库根目录自带的 `README.md` 与 `NGSQCToolkitv2.3.3_manual.pdf` 为软件随附说明，未做改动。

## 一、目录结构

```
NGSQCToolkit_v2.3.3/
├── QC/                  IlluQC.pl（单线程）、IlluQC_PRLL.pl（并行，推荐）
│                       454QC.pl、454QC_PE.pl、454QC_PRLL.pl
│                       lib/（自带 Parallel::ForkManager 等 pure-perl 模块）
├── Trimming/            TrimmingReads.pl、AmbiguityFiltering.pl、HomopolymerTrimming.pl
├── Statistics/          AvgQuality.pl、N50Stat.pl
├── Format-converter/    FastqToFasta.pl、FastqTo454.pl、
│                       SangerFastqToIlluFastq.pl、SolexaFastqToIlluFastq.pl
└── NGSQCToolkitv2.3.3_manual.pdf
```

## 二、依赖

脚本为 Perl 编写，需要以下 CPAN 模块：

| 模块 | 用途 |
|---|---|
| `String::Approx` | 模糊串匹配（接头/引物检测核心依赖） |
| `YAML` | 统计/配置读取 |
| `GD` | QC 图形输出 |
| `GD::Text`（GDTextUtil 发行版） | 图中文本 |
| `GD::Graph`（GDGraph 发行版） | 图形绘制 |
| `Parallel::ForkManager` | 并行版 `IlluQC_PRLL.pl` 使用，已自带于 `QC/lib/`，无需单独安装 |

其中 `GD`、`GD::Text`、`GD::Graph` 为 XS 模块，需要先提供 libgd 开发库。

## 三、获取软件

本仓库即为软件本体：

```bash
# 方式一：克隆仓库
git clone https://github.com/SiYangming/NGSQCToolkit.git

# 方式二：从 Release 下载
# https://github.com/SiYangming/NGSQCToolkit/releases/tag/v2.3.3
```

放置到目标目录：

```bash
mkdir -p /path/to/install/
cp -r NGSQCToolkit /path/to/install/NGSQCToolkit_v2.3.3
chmod +x /path/to/install/NGSQCToolkit_v2.3.3/QC/*.pl
chmod +x /path/to/install/NGSQCToolkit_v2.3.3/Trimming/*.pl
chmod +x /path/to/install/NGSQCToolkit_v2.3.3/Statistics/*.pl
chmod +x /path/to/install/NGSQCToolkit_v2.3.3/Format-converter/*.pl
```

## 四、安装依赖

任选一种方式。

### 方式一：系统包管理器 + cpanm

```bash
# Debian / Ubuntu
sudo apt-get install -y libgd-dev
cpanm -n String::Approx YAML GD GDTextUtil GDGraph

# RHEL / CentOS
sudo yum install -y gd-devel
cpanm -n String::Approx YAML GD GDTextUtil GDGraph

# macOS（Homebrew）
brew install gd
cpanm -n String::Approx YAML GD GDTextUtil GDGraph
```

### 方式二：conda

```bash
conda create -n ngsqctoolkit -c conda-forge -c bioconda \
  perl perl-app-cpanminus \
  perl-string-approx perl-yaml perl-gd perl-gdgraph perl-gdtextutil
conda activate ngsqctoolkit
```

只需文本统计、不需要 QC 图形时，可暂不安装 GD 系模块。

## 五、配置

```bash
# 按需把各功能目录加入 PATH
echo 'export PATH=$PATH:/path/to/install/NGSQCToolkit_v2.3.3/QC/' >> ~/.bashrc
echo 'export PATH=$PATH:/path/to/install/NGSQCToolkit_v2.3.3/Trimming/' >> ~/.bashrc
echo 'export PATH=$PATH:/path/to/install/NGSQCToolkit_v2.3.3/Statistics/' >> ~/.bashrc
source ~/.bashrc
```

`QC/` 下脚本会 require 同目录的 `QC/lib/`，**不要把脚本单独拷贝出去运行**。

## 六、验证安装

```bash
# 语法检查
perl -c /path/to/install/NGSQCToolkit_v2.3.3/QC/IlluQC_PRLL.pl
perl -c /path/to/install/NGSQCToolkit_v2.3.3/Trimming/TrimmingReads.pl

# 依赖加载检查
perl -e 'require String::Approx; require YAML; require GD; print "deps OK\n"'
```

## 七、版本与许可

- 版本：**2.3.3**
- License：官方未声明 SPDX 许可，主页与论文中标注为 free / open source、学术免费使用
