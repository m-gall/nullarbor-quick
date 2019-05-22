# Fast nullarbor

### 

This is a modified version of nullarbor. In the current version of nullarbor, each isolate is analysed sequentially, which can lead to extremely long run-times for large batches of samples. The 'hacks' implemented here should lead to improved run time. This has been achieved by splitting the analyses into two scripts - one dealing with isolate level analyses, the other population. The isolate perl script can be kicked off for each sample separately; once these analyses are complete, the population perl script can be executed over the directory containing the isolate outputs.

The major changes can be summarised as follows:

- The 'isolate' vs 'population' analyses have been split into two perl scripts. This allows the user to run single isolates in parallel, and then gather for the population level analyses. This has been implemented to improve run-times for large batches.
- Roary has been switched off as this feature is not required by our users.
- Plasmid annotation has been added as a feature using the abricate plamid database. The output is captured in the report in the table 'plasmidome'. 


### Usage

#### With a docker image, with all the tools installed

Requirements:
- Docker installed

##### Pull the docker image from docker hub

```console
$ mkdir nullarbor-quick

$ cd nullarbor-quick

$ git pull m-gall/nullarbor-quick

$ docker pull pansapiens/nullarbor:latest ## nb this image is around ~18 GBs

$ cd nullarbor-quick; mkdir fastq; mv $dir_with_fastq fastq

$ pwd=$path/nullarbor-quick

```

Start up an interactive session in the docker container:

`$ sudo docker run -it --entrypoint=/bin/bash -v $pwd:/tmp pansapiens/nullarbor:latest`

The files on your computer have been mounted in the tmp folder of the docker container:

`$ cd /tmp/.../nullarbor-quick/v2.0/bin`

The same inputfile described for nullarbor can be provided, but each isolate must be written to a single. Something like the following might be used to quickly generate these from the batch inputfile:

```console

$ for row in {1..$NO_SAMPLES};
  do
  echo $row;
  sed -n ${row}p $input_batch_file > ${outputdir}/input.${row}.tsv;
done

```
Once these files have been generated, the nullarbor-fast-isolate perl script can be kicked off for each sample separately:

`$ perl nullarbor-fast-isolate.pl --name '${SAMPLE-NAME}-isolate' --ref ${PATH}/reference/${reference.fa} --input ${PATH}/fastq/input.${row}.tsv --outdir ${PATH}/output/ --assembler skesa --force --run --cpus ${CPUS}`

NB: The same outdir must be specified for the isolate analyses

Once each of the isolate analyses have been completed, the population-level analyses can be run

`$ perl nullarbor-fast-population.pl --name '${RUN-NAME}-batch' --ref ${PATH}/reference/${reference.fa} --input ${PATH}/fastq/input.batch.tsv --outdir ${PATH}/output/ --assembler skesa --force --run --cpus ${CPUS}`

The final report should be contained in the 'report' directory, whereas the isolate folder contains a shorter isolate-only report.

#### On a system where the relevant tools have already been installed (e.g. HPC)

```console

$ mkdir nullarbor-quick

$ cd nullarbor-quick

$ git pull m-gall/nullarbor-quick

```

Load the relevant modules. On a system where nullarbor has been installed, make sure you provide the path to the nullarbor-quick scripts, else risk running the version of the nullarbor perl script already installed on the system.

`$ perl nullarbor-quick/bin/nullarbor-fast-isolate.pl --name '${SAMPLE-NAME}-isolate' --ref ${PATH}/reference/${reference.fa} --input ${PATH}/fastq/input.${row}.tsv --outdir ${PATH}/output/ --assembler skesa --force --run --cpus ${CPUS}`

