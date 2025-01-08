# SeqArc
SeqArc compresses next-generation sequencing data in [FASTQ](http://en.wikipedia.org/wiki/Fastq) format. 

As one of few tools capable of handling data of various omics, species, read length, sequence generation and other features, SeqArc strikes the best balance between compression ratio and performance. 

It is achieved by block-united processing. A fixed volume of reads will be considered as a block, and reads within a block are encoded altogether. Each read contains ID, base sequence and quality scores. The three parts are compressed independently, with the most suitable algorithms respectively.

ID is compressed with adaptive binning and range coding. 
Base sequence will be mapped to reference if index built, replaced with alignment information and encoded with range coder. The base sequence unable to map will be estimated with 16-order model and encoded with range coder. 
Quality scores are estimated with some pre-set pattern and encoded with range coder. For users interested in higher compression ratio and slight modification of Q-scores, the R-Block algorithms are equipped within SeqArc. 

Besides, the compression output of SeqArc is packed with encapsulation format, so different parts can be independently accessed. It also conveniently supports following iteration and replacement of algorithms.

## Usage

### Params

    To build Taxonomic Classification index:
        SeqArc index -K </path/to/the/directory/where/multiple/ref.fa/within>
    To build Alignment index (default type is Minimizer):
        SeqArc index -r </path/to/ref.fa>

    To compress with Taxonomic classification (software auto decide which fa to use, whose alignment index would be used after):
        SeqArc compress [options] -K </path/to/ref.fa/directory> -i <input_file> -o <output_file>.arc -t 8
    To compress with a specific alignment index:
        SeqArc compress [options] -r </path/to/ref.fa> -i <input_file>  -o <output_file>.arc -t 8

    To decompress a file which was compressed with Taxonomic classification:
        SeqArc decompress [options] -K </path/to/ref.fa/directory> -i <result_prefix.arc> -o <output_file> -t 8
    To decompress a file which was compressed with alignment index specified:
        SeqArc decompress [options] -r </path/to/ref.fa> -i <result_prefix.arc> -o <output_file> -t 8

    Options:
            --aligntype         INT       Choose the index type to build FASTA index or compress FASTQ
                    1                     HASH(not recommend)
                    2                     Minimizer(HASH-based), recommended and default option
                    3                     LongSeq

            -r,--ref            STR       Path of reference FASTA
            -K,--clsdb          STR       Path of directory with multiple reference FASTA in
            -J,--clsmin         FLOAT     Minimum ratio of reads to hit for determination, default as [0.6]
            -L,--clsnum         INT       Read number for classification, default as [10000]

            -i,--input          STR       Input file path
            --pipein            BOOL      Input is pipe
            -o,--output         STR       Output file path
            -f,--force          BOOL      Force overwrite target file
            --rawtype           BOOL      Decompression to mgzip format, a parallel gzip format designed by BGI
            -t,--thread         INT       Thread num for multi-threading, default as [8]
            --blocksz           INT       Set block size, default is [50M]
            --calcmd5           BOOL      Calculation of md5sum for whole FASTQ file(s) during compression
            --ccheck            BOOL      Check compression file is 100%% complete and lossless during compression.
                                          Considerably more time-consuming for highest data safety.
            --dcheck            BOOL      Check decompression file is 100%% complete and lossless during writing to disk.
                                          Slightly more time-consuming for higher data safety
            -h,--help                     Get instructions of this software


### Practice
&nbsp;

### I. Index Building

Taxonomic Classification is a brand new function in Version 2.0.0. Now we can build a taxonomic classification index of multiple FASTA files, and it will analyze reads and allocate appropriate FASTA during compression. You need to download all the FASTA files corresponded to your FASTQ files (you can download high-frequency and model organism from NCBI if you have no idea what species you need) before you download and gunzip the **names.dmp.gz** and **nodes.dmp.gz** to the directory you put those FASTA files. Then, you can build the index with the following command:

    SeqArc index -K /path/to/the/directory/where/multiple/ref.fa/within

If you don't want the Taxonomic Classification function, you can always choose the traditional method, where alignment index of a specific FASTA file being built. The following command helps you to build a Minimizer index of fasta for alignment afterwards:

    SeqArc index -r /path/to/ref.fa

You can also build a HASH index with additional option **--aligntype 1**:

    SeqArc index -r /path/to/ref.fa --aligntype 1

Index file will be built within the same directory as ref.fa. And you can also compression without index, and it will still work out with lower compression ratio.

Note: the BWA algorithm has been deprecated in v2.0.4 or later version

### II. Compression

As taxonomic classification index being built, the easiest way to compress single-end file is:

    SeqArc compress -K /path/to/ref.fa/directory -i /path/to/test1.fq -o /path/to/test.arc

If you don't want to use the taxonomic classification function (which only costs **10s** for compression), you can also specify the alignment index you want. Here is the command to compress single-end file:

    SeqArc compress -r /path/to/ref.fa -i /path/to/test1.fq -o /path/to/test.arc

And for pipein:

    cat test.fq | SeqArc compress -r /path/to/ref.fa --pipein -o /path/to/test.arc

Note: If the index was built with **--aligntype**, the compression parameters **must** be given the same type of index as well if you want to use the index.

Note2: If **no index included**, just delete the **-r /path/to/ref.fa** in commands and compression will still work without index.

Note3: For a super strict usage scenario, the parameter **--ccheck** should be added, where data will be decompressed at once to ensure 100% data safety with the cost of halving compression speed.

Note4: You can directly compress **xxx.fq.gz** and everything will still work out. This **gz** include **normal gzip and mgzip**, which is a parallel gzip format designed by BGI and here is the link: https://github.com/BGIResearch/mgzip

Note5: Based on users' feedback, we have added a function to calculate MD5 of whole raw FASTQ file(s), so you can check the MD5 conveniently. However, it is a function badly decreasing compression speed, so if you want a satisfactory performance, please do not add **--calcmd5** in your parameters.

Note6: PE compression don't support.
&nbsp;

### III. Regular Decompression
For single-end compression result compressed with taxonomic classification, the command to decompress test.arc is:

    SeqArc decompress -r /path/to/ref.fa/directory -i test.arc -o restore

For SE, a file named **restore** will be created to current directory.

For compression result compressed with specific alignment index, the command is:

    SeqArc decompress -r /path/to/ref.fa -i test.arc -o back

Note: Again, if **no index** involved in compression, just **delete** the **-r /path/to/ref.fa**.

&nbsp;

### IV. Decompression to standard-output
If you want to directly pass the decompressed reads to downstream analysis tools, do not use the parameter **-o** and they will be delivered to stdout.

For single-end data:

    SeqArc decompress -r ref.fa -i test.arc

&nbsp;

### End
