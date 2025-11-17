---
title: "Installing and running Phastest with Apptainer on HPC"
published: true
categories:
  - Blog
  - Tutorial
tags:
  - singularity
  - apptainer
  - hpc
  - containers
---

[Apptainer](https://apptainer.org/) (formerly Singularity) is a container platform that allows you to create and run containers that package up software in a way that is portable and reproducible. Apptainer is the preferred container platform for HPC clusters as each container is only a single file and users don't need root access to run the containers.

In this post, I'll detail how to install [Phastest](https://phastest.ca/), a tool for rapid identificationrapid identification, annotation and visualization of prophage sequences within bacterial genomes and plasmids.

## Setup (once)
SSH into the login node (hint: use the VS Code Remote extension). Create a directory to hold the container and its data:

```bash
CONTAINER_DIR=$HOME/containers/
mkdir -p $CONTAINER_DIR/bin/
```

Download and extract the Phastest Apptainer container and database:

```bash
wget -O $CONTAINER_DIR/phastest-docker.zip https://phastest.ca/download_file/phastest-docker
unzip $CONTAINER_DIR/phastest-docker.zip "phastest/*" -d $CONTAINER_DIR
rm $CONTAINER_DIR/phastest-docker.zip
```

Download and extract the Phastest database (3GB):
```bash
wget -O $CONTAINER_DIR/docker-database.zip https://phastest.ca/download_file/docker-database
unzip $CONTAINER_DIR/docker-database.zip "DB/*" -d $CONTAINER_DIR/phastest/phastest-app-docker
rm $CONTAINER_DIR/docker-database.zip
```

Create wrapper script to run Phastest using Apptainer and copy results to current working directory:
```bash
cat > $CONTAINER_DIR/bin/phastest << 'EOF'
#! /bin/sh
set -e
set -o pipefail

output_dir="phastest-results"

# Parse command line arguments.
while getopts ":i:m:a:s:o:-:" opt; do
    case $opt in
        i)
            input_type=$OPTARG
            ;;
        m)
            anno_mode=$OPTARG
            ;;
        a)
            accession=$OPTARG
            ;;
        s)
            sequence=$OPTARG
            ;;
        o)
            output_dir=$OPTARG
            ;;
        -)
            case $OPTARG in
                yes)
                    skip_confirmation=1
                    ;;
                silent)
                    silent=1
                    ;;
                phage-only)
                    complete_annotation=0
                    phage_only=1
                    ;;
                *)
                    echo "Invalid option: --$OPTARG" >&2
                    exit 1
                    ;;
            esac
            ;;
        \?)
            echo "Invalid option: -$OPTARG" >&2
            exit 1
            ;;
        :)
            echo "Option -$OPTARG requires an argument." >&2
            exit 1
            ;;
    esac
done

PHASTEST=$HOME/containers/phastest/

apptainer run \
  --hostname slurmctld \
  --bind $PHASTEST/phastest-app-docker/sub_programs/ncbi-blast-2.3.0+:/BLAST+ \
  --bind $PHASTEST/phastest-app-docker/sub_programs/ncbi-blast-2.3.0+:/root/BLAST+ \
  --bind $PHASTEST/phastest-app-docker:/phastest-app \
  --bind $PHASTEST/phastest-app-docker:/root/phastest-app \
  --bind $PHASTEST/phastest_inputs:/phastest_inputs \
  --writable-tmpfs \
  docker://wishartlab/phastest-docker-single \
  phastest \
    -i "$input_type" \
    $( [ -n "$anno_mode" ] && echo "-m $anno_mode" ) \
    $( [ -n "$accession" ] && echo "-a $accession" ) \
    $( [ -n "$sequence" ] && echo "-s $sequence" ) \
    $( [ -n "$skip_confirmation" ] && echo "--yes" ) \
    $( [ -n "$silent" ] && echo "--silent" ) \
    $( [ -n "$phage_only" ] && echo "--phage-only" )

# move the output file to the current working directory
if [[ $input_type != "genbank" ]]; then
    filename=$(basename $sequence)
    job_id="${filename%.*}"
else 
    job_id="$accession"
fi

mkdir -p $PWD/$output_dir
rm -rf $PWD/$output_dir/$job_id
mv $PHASTEST/phastest-app-docker/JOBS/$job_id $PWD/$output_dir/$job_id
echo "Results moved to $PWD/$output_dir/$job_id"
EOF
```

Make the wrapper script executable (ensure .local/bin is in your PATH):
```bash
install -m 755 $CONTAINER_DIR/bin/phastest "$HOME/.local/bin/phastest"
```

## Usage
You can now run Phastest using the `phastest` command. For example, to analyse a GenBank file with accession `NC_000907.1`, run:

```bash
phastest -i genbank -a NC_000907.1 --yes --phage-only
```
```
Running PHASTEST
Job ID: NC_000907.1
Available space of /phastest-app/JOBS is 20G
Handle gbk file...
Generating fna file from gbk ...
NC_000907.1.fna created!
Generating ptt file from gbk ...
Generating faa file from gbk ...
Running phage search ...
Progress: [==================== ] 100%
Fork is done ...
Scanning for phage regions ...
Annotating proteins in regions ...
Get true regions ...
true_defective_prophage.txt generated!
Cleaning up ...
Program exit!
```






