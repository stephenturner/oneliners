<meta http-equiv="Content-Type" content="text/html; charset=utf-8">

# 生信单行脚本

生信息有用的单行脚本 (and [some, more generally useful](#etc)).  


## contents

- [来源](#sources)
- [awk、sed基础](#basic-awk--sed)
- [awk、sed生信单行程序](#awk--sed-for-bioinformatics)
- [sort,uniq和cut等等](#sort-uniq-cut-etc)
- [find,xargs,和GNU parallel](#find-xargs-and-gnu-parallel)
- [seqtk](#seqtk)
- [GFF3 Annotations](#gff3-annotations)
- [其他有用的简称 .bashrc](#other-generally-useful-aliases-for-your-bashrc)
- [更多...](#etc)



## Sources

* <http://gettinggeneticsdone.blogspot.com/2013/10/useful-linux-oneliners-for-bioinformatics.html#comments>
* <http://sed.sourceforge.net/sed1line.txt>
* <https://github.com/lh3/seqtk>
* <http://lh3lh3.users.sourceforge.net/biounix.shtml>
* <http://genomespot.blogspot.com/2013/08/a-selection-of-useful-bash-one-liners.html>
* <http://biowize.wordpress.com/2012/06/15/command-line-magic-for-your-gene-annotations/>
* <http://genomics-array.blogspot.com/2010/11/some-unixperl-oneliners-for.html>
* <http://bioexpressblog.wordpress.com/2013/04/05/split-multi-fasta-sequence-file/>
* <http://www.commandlinefu.com/>



## Basic awk & sed

[[返回顶部](#contents)]


提取文件中的2, 4, and 5 列:

    awk '{print $2,$4,$5}' file.txt


输出第五列等于abc123的行:

    awk '$5 == "abc123"' file.txt


输出第五列不是abc123的行:

    awk '$5 != "abc123"' file.txt


输出第七列以字母a-f开头的行:

    awk '$7  ~ /^[a-f]/' file.txt


输出第七列不是以字母a-f开头的行:

    awk '$7 !~ /^[a-f]/' file.txt


计算第二列不重复的值保存在哈希arr中 (一个值只保存一次):

    awk '!arr[$2]++' file.txt


输出第三列的值比第五列大的行:

    awk '$3>$5' file.txt


计算文件中第一列的累加值，输出最后的结果:

    awk '{sum+=$1} END {print sum}' file.txt


计算第二列的平均值:

    awk '{x+=$2}END{print x/NR}' file.txt


用bar替换文件中所有的foo:

    sed 's/foo/bar/g' file.txt


消除行开头空和格制表符:

    sed 's/^[ \t]*//' file.txt


消除行结尾的空格和制表符:

    sed 's/[ \t]*$//' file.txt


消除行中开头和结尾的空格和制表符:

    sed 's/^[ \t]*//;s/[ \t]*$//' file.txt


删除空行:

    sed '/^$/d' file.txt


删除包含‘EndOfUsefulData’的行及其后所有的行:

    sed -n '/EndOfUsefulData/,$!p' file.txt




## awk & sed for bioinformatics

## 生信单行sed,awk

[[返回](#contents)]


Returns all lines on Chr 1 between 1MB and 2MB in file.txt. (assumes) chromosome in column 1 and position in column 3 (this same concept can be used to return only variants that above specific allele frequencies):

输出Chr为1在1M和2M之间的所有行。（假设）染色体在第一列，位点在第三列（基于同样的假设可以用来返回类似特定等位基因频率的变异）

    cat file.txt | awk '$1=="1"' | awk '$3>=1000000' | awk '$3<=2000000'


Basic sequence statistics. Print total number of reads, total number unique reads, percentage of unique reads, most abundant sequence, its frequency, and percentage of total in file.fq:
基本序列统计。输出总的reads数，不重复的reads总数，不重复reads百分比，最大冗余的序列及其频度以及总占比百分数。

    cat myfile.fq | awk '((NR-2)%4==0){read=$1;total++;count[read]++}END{for(read in count){if(!max||count[read]>max) {max=count[read];maxRead=read};if(count[read]==1){unique++}};print total,unique,unique*100/total,maxRead,count[maxRead],count[maxRead]*100/total}'


转换.bam为.fastq:

    samtools view file.bam | awk 'BEGIN {FS="\t"} {print "@" $1 "\n" $10 "\n+\n" $11}' > file.fq


Keep only top bit scores in blast hits (best bit score only):
只取blast采样中的顶级位点的分数（最高的位点分）

    awk '{ if(!x[$1]++) {print $0; bitscore=($14-1)} else { if($14>bitscore) print $0} }' blastout.txt


Keep only top bit scores in blast hits (5 less than the top):
只取blast采样中的顶级位点的分数（比顶级少于5的）

    awk '{ if(!x[$1]++) {print $0; bitscore=($14-6)} else { if($14>bitscore) print $0} }' blastout.txt


分割多序列FASTA文件为单序列FASTA文件

    awk '/^>/{s=++d".fa"} {print > s}' multi.fa

输出fasta文件中的每条序列的序列名称和长度

    cat file.fa | awk '$0 ~ ">" {print c; c=0;printf substr($0,2,100) "\t"; } $0 !~ ">" {c+=length($0);} END { print c; }'

转化FASTQ文件为FASTA:

    sed -n '1~4s/^@/>/p;2~4p' file.fq > file.fa

从第二行开始每四行取值（从FASTQ文件提取序列）。

    sed -n '2~4p' file.fq

输出中剔除第一行：

    awk 'NR>1' input.txt

输出20-80行:

    awk 'NR>=20&&NR<=80' input.txt

计算二，三行列的和并追加到每行后输出

    awk '{print $0,$2+$3}' input.txt

计算fastq文件平均reads的长度

    awk 'NR%4==2{sum+=length($0)}END{print sum/(NR/4)}' input.fastq

转化VSF文件为BED文件

sed -e 's/chr//' file.vcf | awk '{OFS="\t"; if (!/^#/){print $1,$2-1,$2,$4"/"$5,"+"}}'


## sort, uniq, cut, etc.

[[返回开头](#contents)]

输出带行号的内容:

    cat -n file.txt

去重复行计数

    cat file.txt | sort -u | wc -l


找到两文件都有的行（假设两个文件都是无重复行，重定向执行‘wd -l’计算同样行的行数）

    sort file1 file2 | uniq -d

    # 安全的方法
    sort -u file1 > a
    sort -u file2 > b
    sort a b | uniq -d

    # 用comm的方法
    comm -12 file1 file2

对文件按照第九列数字顺序排序（g按照常规数值，k列）

    sort -gk9 file.txt

找到第二列出现最多的字符串

    cut -f2 file.txt | sort | uniq -c | sort -k1nr | head


从文件中随机取10行

    shuf file.txt | head -n 10

输出所有三个所可能的DNA序列

    echo {A,C,T,G}{A,C,T,G}{A,C,T,G}


Untangle an interleaved paired-end FASTQ file. If a FASTQ file has paired-end reads intermingled, and you want to separate them into separate /1 and /2 files, and assuming the /1 reads precede the /2 reads:

解开一列交错paired-end fastq文件。如果fastq文件有乱序paired-end reads，你想将其分离成单独的/1，/2的文件保存，这里假设/1 reads 在/2 前面：

    cat interleaved.fq |paste - - - - - - - - | tee >(cut -f 1-4 | tr "\t" "\n" > deinterleaved_1.fq) | cut -f 5-8 | tr "\t" "\n" > deinterleaved_2.fq

Take a fasta file with a bunch of short scaffolds, e.g., labeled `>Scaffold12345`, remove them, and write a new fasta without them:

将一个fasta文件转成一系列短的scaffolds。比如，标签 ">Scaffold12345"，然后移出他们，保存一个去掉他们的新文件：

    samtools faidx genome.fa && grep -v Scaffold genome.fa.fai | cut -f1 | xargs -n1 samtools faidx genome.fa > genome.noscaffolds.fa

Display hidden control characters:

显示一个隐藏的控制字符：

    python -c "f = open('file.txt', 'r'); f.seek(0); file = f.readlines(); print file" 


## find, xargs, and GNU parallel

[[返回](#contents)]


*通过  <https://www.gnu.org/software/parallel/>.* 载 GNU parallel


搜索文件夹及其子目录中名称为 .bam 文件（目录也算）:

    find . -name "*.bam"


删除上面搜到的文件列表(不可逆的危险操作，谨慎使用！删除之前请自习确认)

    find . -name "*.bam" | xargs rm


将所有.txt 文件修改为.bak(例如在对*.txt做操作之前用于文件备份)

    find . -name "*.txt" | sed "s/\.txt$//" | xargs -i echo mv {}.txt {}.bak | sh


Chastity filter raw Illumina data (grep reads containing `:N:`, append (-A) the three lines after the match containing the sequence and quality info, and write a new filtered fastq file):

对Illumina数据做Chastity过滤（grep 查询 包含`:N:`，用（-A）选项第三列信息附加在匹配的包含一个序列质量信息后，并保存为一个新的fasta文件）

    find *fq | parallel "cat {} | grep -A 3 '^@.*[^:]*:N:[^:]*:' | grep -v '^\-\-$' > {}.filt.fq"


通过parallel并行运行12个FASTQC任务

    find *.fq | parallel -j 12 "fastqc {} --outdir ."

通过parallel给bam做索引，通过`--dry-run`打印测试这些命令，实际上并未做执行。
    find *.bam | parallel --dry-run 'samtools index {}'

## seqtk

[[back to top](#contents)]

* Seqtk项目托管地址<https://github.com/lh3/seqtk>。Seqtk是一个快捷轻量的处理FASTA和FASTQ格式基因序列的工具。他可以是先FASTA和FASTQ无缝处理和转化，同时支持gzip格式的压缩文件。

把FASTQ转化为FASTA:

    seqtk seq -a in.fq.gz > out.fa

转化ILLUMINA 1.3+ 格式FASTQ为FASTA，并且以小于20的mask bases获得小写字母(第一命令行)或者到N（第二）。
    seqtk seq -aQ64 -q20 in.fq > out.fa
    seqtk seq -aQ64 -q20 -n N in.fq > out.fa

折叠长FASTA/Q行，并且去除其注释：

    seqtk seq -Cl60 in.fa > out.fa

转化多行FASTQ到四行FASTQ:

    seqtk seq -l0 in.fq > out.fq

反转FASTA/Q序列:

    seqtk seq -r in.fq > out.fq

用序列文件中的名称（比如name.1st）提取序列,一个虚列名一行:

    seqtk subseq in.fq name.lst > out.fq


利用序列文件中的”reg.bed“r信息提取地理信息的序列:

    seqtk subseq in.fa reg.bed > out.fa

编码‘reg.bed’地理信息到小写

    seqtk seq -M reg.bed in.fa > out.fa


从两个大的paired FASTQ文件提取10000个read pairs（记得用同样的随机种子保持 paire）

    seqtk sample -s100 read1.fq 10000 > sub1.fq
    seqtk sample -s100 read2.fq 10000 > sub2.fq


利用Phred公式从两头修剪低质量bases:

    seqtk trimfq in.fq > out.fq


从左端修剪5bp，从右端修剪10bp的。

    seqtk trimfq -b 5 -e 10 in.fa > out.fa


    seqtk seq -l0 -1 interleaved.fq > deinterleaved_1.fq
    seqtk seq -l0 -2 interleaved.fq > deinterleaved_2.fq




## GFF3 Annotations

[[back to top](#contents)]


输出GFF3文件中标注的所有的序列

    cut -s -f 1,9 yourannots.gff3 | grep $'\t' | cut -f 1 | sort | uniq


检测GFF3文件中标注的所有性状类型。

    grep -v '^#' yourannots.gff3 | cut -s -f 3 | sort | uniq


检测GFF3文件中标注的基因数量。

    grep -c $'\tgene\t' yourannots.gff3


从GFF3文件中提取所有的基因ID

    grep $'\tgene\t' yourannots.gff3 | perl -ne '/ID=([^;]+)/ and printf("%s\n", $1)'


输出GFF3文件每个基因的长度

    grep $'\tgene\t' yourannots.gff3 | cut -s -f 4,5 | perl -ne '@v = split(/\t/); printf("%d\n", $v[1] - $v[0] + 1)'

FASTA头列转化为GFF格式（假设头的长度，附加在”\_length“ ,和[Velvet](http://www.ebi.ac.uk/~zerbino/velvet/) assembled transcripts)）

    grep '>' file.fasta | awk -F "_" 'BEGIN{i=1; print "##gff-version 3"}{ print $0"\t BLAT\tEXON\t1\t"$10"\t95\t+\t.\tgene_id="$0";transcript_id=Transcript_"i;i++ }' > file.gff




## Other generally useful aliases for your .bashrc

## 有用的别名(.bashrc)

[[back to top](#contents)]


提示符修改为`user@hostname:/full/path/cwd/:$ `形式

    export PS1="\u@\h:\w\\$ "

避免反复敲诸如`cd ../../..`的命令（也可以用[autojump](https://github.com/joelthelion/autojump），让你在飞速的转换目录 

    alias ..='cd ..'
    alias ...='cd ../../'
    alias ....='cd ../../../'
    alias .....='cd ../../../../'
    alias ......='cd ../../../../../'

向前和向后浏览

    alias u='clear; cd ../; pwd; ls -lhGgo'
    alias d='clear; cd -; ls -lhGgo'


覆盖文件时候，先确认

    alias mv="mv -i"
    alias cp="cp -i"  
    alias rm="rm -i"


我最喜欢的”ls“别名

    alias ls="ls -1p --color=auto"
    alias l="ls -lhGgo"
    alias ll="ls -lh"
    alias la="ls -lhGgoA"
    alias lt="ls -lhGgotr"
    alias lS="ls -lhGgoSr"
    alias l.="ls -lhGgod .*"
    alias lhead="ls -lhGgo | head"
    alias ltail="ls -lhGgo | tail"
    alias lmore='ls -lhGgo | more'


对cut空格和逗号，分割文件

    alias cuts="cut -d \" \""
    alias cutc="cut -d \",\""


解压缩tar包

    alias tarup="tar -zcf"
    alias tardown="tar -zxf"


或者可以用更普遍的‘extract’函数

    # 源于ABSG(Advanced Bash Scripting Guide)中 Mendel Cooper的建议
  
    extract () {
       if [ -f $1 ] ; then
           case $1 in
            *.tar.bz2)      tar xvjf $1 ;;
            *.tar.gz)       tar xvzf $1 ;;
            *.tar.xz)       tar Jxvf $1 ;;
            *.bz2)          bunzip2 $1 ;;
            *.rar)          unrar x $1 ;;
            *.gz)           gunzip $1 ;;
            *.tar)          tar xvf $1 ;;
            *.tbz2)         tar xvjf $1 ;;
            *.tgz)          tar xvzf $1 ;;
            *.zip)          unzip $1 ;;
            *.Z)            uncompress $1 ;;
            *.7z)           7z x $1 ;;
            *)              echo "don't know how to extract '$1'..." ;;
           esac
       else
           echo "'$1' is not a valid file!"
       fi
    }



使用别名"mcd"创建一个目录，并且cd到该目录

    function mcd { mkdir -p "$1" && cd "$1";}


跳转到上级目录，并且列出其内容

    alias u="cd ..;ls"

一个好看的grep

    alias grep="grep --color=auto"


刷新你的`.bashrc`

    alias refresh="source ~/.bashrc"

编辑你的`.bashrc`

    alias eb="vi ~/.bashrc"

常用错误别称

    alias mf="mv -i"
    alias mroe="more"
    alias c='clear'


使用 [pandoc](http://johnmacfarlane.net/pandoc/)转化markdown文档为PDF格式:

    # 用法: mdpdf document.md document.md.pdf
    alias mdpdf="pandoc -s -V geometry:margin=1in -V documentclass:article -V fontsize=12pt"


对当前目录搜索关键词(`ft "mytext" *.txt`):

    function ft { find . -name "$2" -exec grep -il "$1" {} \;; }

## Etc

[[返回](#contents)]

重复运行上一条命令:

    sudo !!

列出最近最常用的命令行参数(通常是文件)

    'ALT+.' or '<ESC> .'

敲出了部分命令，删除这些输入，查你忘记的明亮，拉回命令，继续输入(<CTRL+u>删除光标之前的输入，<CTRL+y>恢复上个C-U删除字符)

    <CTRL+u> [...] <CTRL+y>

跳到一个目录，执行命令，然后返回当前目录(()的用法)
 
    (cd /tmp && ls)

记时秒表 (输入`Enter` or `ctrl-d` 停止):

    time read

把上次执行的命令生成一个脚本

    echo "!!" > foo.sh

重用上次命令的所有参数

    !*

列出或者删除一个目录中所有不匹配的特定后缀的文件（例如，列出所有不是压缩的文件，删除所有不以.foo和.bar后缀的文件）

    ls !(*.gz)
    rm !(*.foo|*.bar)

利用上次的命令，但是不需要他的的参数（重新输入参数）:

    !:- <new_last_argument>

激活一个快捷的编辑器，输入，编辑长的，复杂，巧妙的命令:

    fc

输出一个特定的行（比如 42行）

    sed -n 42p <file>

终结一个冻结的ssh session(会车换行，敲~键，在敲下.键）

    [ENTER]~.

利用grep去除文件的空行，结果保存到新文件

    grep . filename > newfilename

查找大文件（例如，大于500M的）

    find . -type f -size +500M


利用截取列（例如，一个tab分割文件的第五个域）

    cut -f5 --complement

查找包含特定字符的文件（`-l` 只输出文件名, `-i` 忽略大小写 `-r` 遍历子目录）

    grep -lir "some text" *

