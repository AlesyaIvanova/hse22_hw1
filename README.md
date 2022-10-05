# hse22_hw1

## Обязательная часть
### Подготовка файлов 

- Создала символические ссылки для доступа к файлам.

```
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```

- Установила seed 716 и выбрала случайно 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs.

```
seqtk sample -s716 oil_R1.fastq 5000000 > sub1.fastq
seqtk sample -s716 oil_R2.fastq 5000000 > sub2.fastq
seqtk sample -s716 oilMP_S4_L001_R1_001.fastq 1500000 > matepairs.fastq
seqtk sample -s716 oilMP_S4_L001_R2_001.fastq 1500000 > matepairs2.fastq
```

### Оценка качества 

- С помощью программы fastQC и multiQC оценила качество исходных чтений и получила по ним общую статистику.
```
mkdir fastqc
ls sub* matepairs* | xargs -tI{} fastqc -o fastqc {}
mkdir multiqc
multiqc -o multiqc fastqc
```

![Скрин1](https://github.com/AlesyaIvanova/hse22_hw1/blob/main/picture1.png)
![Скрин2](https://github.com/AlesyaIvanova/hse22_hw1/blob/main/picture2.png)

- С помощью программ platanus_trim и platanus_internal_trim подрезала чтение по качеству и удалила адаптеры.
```
platanus_trim sub*
platanus_internal_trim matepair*
```

- Удалила первоначальные файлы.
```
rm sub1.fastq
rm sub2.fastq
rm matepairs.fastq 
rm matepairs2.fastq
```

- С помощью программ fastQC и multiQC оценила качество подрезанных чтений и получила по ним общую статистику.
```
mkdir fastqc_trim
ls sub* matepairs*| xargs -tI{} fastqc -o fastqc_trim {}
mkdir multqctrim
multiqc -o multqctrim fastqc_trim
```

![Скрин3](https://github.com/AlesyaIvanova/hse22_hw1/blob/main/picture3.png)
![Скрин4](https://github.com/AlesyaIvanova/hse22_hw1/blob/main/picture4.png)

### Контиги, скаффолды
- С помощью программы “platanus assemble” собрала контиги из подрезанных чтений.
```
time platanus assemble -o Poil -f sub1.fastq.trimmed sub2.fastq.trimmed 2> assemble.log
```

- Проанализировала полученные контиги 
    + Ссылка на Google Colab: 
    + Блокнот:

- С помощью программы “platanus scaffold” собрала скаффолды из контигов, а также из подрезанных чтений.
```
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matepairs.fastq.int_trimmed matepairs2.fastq.int_trimmed 2> scaffold.log
```

- Уменьшила промежутки.
```
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matepairs.fastq.int_trimmed  matepairs2.fastq.int_trimmed 2> gapclose.log
```

Ссылка на ноутбук та же.

- Удалила подрезанные чтения.
```
rm matepairs.fastq.int_trimmed
rm matepairs2.fastq.int_trimmed
rm sub1.fastq.trimmed
rm sub2.fastq.trimmed
```

- Перенесла все в одну папку.

```
mkdir MainTask
mv Poil* MainTask/
mv oil* MainTask/
mv *.log MainTask/
mv fast* MainTask/
mv mult* MainTask/
```

- В папке MainTask создала папку platanus и перенесла туда файлы, созданные программой platanus.

```
cd MainTask
mkdir platanus
mv Poil* platanus/
mv *.log platanus/
```

## Бонусная часть
### Дополнительная сборка 1: чтений в 2 раза меньше

- Создала папку ExtraTask, в ней Seq1. В папке Seq1 выполнила те же действия, но с изменением кол-ва чтений (для paired-end 2_500_000 чтений, для mate-pairs 750_000 чтений.

```
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq

seqtk sample -s716 oil_R1.fastq 2500000 > sub1.fastq
seqtk sample -s716 oil_R2.fastq 2500000 > sub2.fastq
seqtk sample -s716 oilMP_S4_L001_R1_001.fastq 750000 > matepairs.fastq
seqtk sample -s716 oilMP_S4_L001_R2_001.fastq 750000 > matepairs2.fastq

mkdir fastqc
ls sub* matepairs* | xargs -tI{} fastqc -o fastqc {}
mkdir multiqc
multiqc -o multiqc fastqc

platanus_trim sub*
platanus_internal_trim matepair*

rm sub1.fastq
rm sub2.fastq
rm matepairs.fastq 
rm matepairs2.fastq

mkdir fastqc_trim
ls sub* matepairs*| xargs -tI{} fastqc -o fastqc_trim {}
mkdir multqctrim
multiqc -o multqctrim fastqc_trim

time platanus assemble -o Poil -f sub1.fastq.trimmed sub2.fastq.trimmed 2> assemble.log

time platanus scaffold -o Poil -c Poil_contig.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matepairs.fastq.int_trimmed matepairs2.fastq.int_trimmed 2> scaffold.log

time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matepairs.fastq.int_trimmed  matepairs2.fastq.int_trimmed 2> gapclose.log

rm matepairs.fastq.int_trimmed
rm matepairs2.fastq.int_trimmed
rm sub1.fastq.trimmed
rm sub2.fastq.trimmed
```

- Ссылка на Google Colab та же.

- Статистики:
 ![Скрин5](https://github.com/AlesyaIvanova/hse22_hw1/blob/main/picture5.png)
 ![Скрин6](https://github.com/AlesyaIvanova/hse22_hw1/blob/main/picture6.png)

### Дополнительная сборка 2: чтений в 4 раза меньше

- В папке ExtraTask создала папку Seq2. В папке Seq2 выполнила те же действия, но с изменением кол-ва чтений (для paired-end 1_250_000 чтений, для mate-pairs 375_000 чтений.

```
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq

seqtk sample -s716 oil_R1.fastq 1250000 > sub1.fastq
seqtk sample -s716 oil_R2.fastq 1250000 > sub2.fastq
seqtk sample -s716 oilMP_S4_L001_R1_001.fastq 375000 > matepairs.fastq
seqtk sample -s716 oilMP_S4_L001_R2_001.fastq 375000 > matepairs2.fastq

mkdir fastqc
ls sub* matepairs* | xargs -tI{} fastqc -o fastqc {}
mkdir multiqc
multiqc -o multiqc fastqc

platanus_trim sub*
platanus_internal_trim matepair*

rm sub1.fastq
rm sub2.fastq
rm matepairs.fastq 
rm matepairs2.fastq

mkdir fastqc_trim
ls sub* matepairs*| xargs -tI{} fastqc -o fastqc_trim {}
mkdir multqctrim
multiqc -o multqctrim fastqc_trim

time platanus assemble -o Poil -f sub1.fastq.trimmed sub2.fastq.trimmed 2> assemble.log

time platanus scaffold -o Poil -c Poil_contig.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matepairs.fastq.int_trimmed matepairs2.fastq.int_trimmed 2> scaffold.log

time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matepairs.fastq.int_trimmed  matepairs2.fastq.int_trimmed 2> gapclose.log

rm matepairs.fastq.int_trimmed
rm matepairs2.fastq.int_trimmed
rm sub1.fastq.trimmed
rm sub2.fastq.trimmed
```

- Ссылка на Google Colab та же.

 - Статистики:
    
    
 ### Выводы
 - См. блокнот.
