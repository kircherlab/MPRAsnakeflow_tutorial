## MPRA workshop directory 
- In the following directory the MPRAsnakeflow run is shown on example data from the proximal regulator project (TODO: reference to project/ another repo?)
- TODO: Adding background information of the data here

## Processing:
1. The corresponding data was downloaded and stored in the `example_data/` directory
2. Next the `experiment.csv` was generated and added to the same folder. 
    - For this run there is one condition (HepG2) with three technical replicates. Each replicate contains a forward (barcode-forward `DNA/RNA_BC_F`), reverse (barcode-reverse `DNA/RNA_BC_R`), and index (unique molecular identifier `DNA/RNA_UMI`) read for DNA and RNA.
    - In general: This file has the following header (`Condition,Replicate,DNA_BC_F,DNA_UMI,DNA_BC_R,RNA_BC_F,RNA_UMI,RNA_BC_R`). For each replicate to be analyzed the table is filled with condition name (`Condition`), replicate number (`Replicate`) and names of the following three files for DNA and RNA, respectifly. First the barcode forward reads (`DNA_BC_F`, `RNA_BC_F`), DNA / RNA UMI file (`DNA_UMI`, `RNA_UMI`) and the barcode reverse reads (`DNA_BC_R`, `RNA_BC_R`).
The workflow expects the file names to be in the format <condition>_<type(dna/rna)>-<replicate>_<Fwd/Rev/BC>.fastq.gz
3. The final `experiment.csv` file is added to the configuration file here called `workshop_config.yaml`. This file is specified as config file later in the snakemake call.

## Explaining the config
- MPRAsnakeflow can be divided into two steps: 
    1. Assignment workflow: Assign the barcodes used to the corresponding sequences
    2. Experiment workflow: Outputs count tables for the sequences used (requires the assignment workflow to be finished)
- `global` Global section of the config
  - There are global settings (e.g. number of splits) which are set for both workflows. 
  - Number of splits (`split_number`) the files are split into to make jobs of the jobs computable in parallel.
- `assignments` Assignment section of the config
  - Give your assignment a name (here `assignMPRAworkshop`)
  - Check the length of your barcodes (`bc_length`) (here `15`) 
    - check the length of the sequences in the barcode files of your samples
  - Check the length of the designed sequences (`sequence_length`) (you can set a range to catch sequencing or mapping errors) (here `min: 195`, `max: 205`)
  - Set the alignment start (you can set a range to catch mapping errors) (here `min: 14`, `max: 16`)
  - Set the minimal allowed mapping quality (`min_mapping_quality`) (here: `1`)
  - Add the path to the read data
    - `FW`: path to forward reads
    - `BC`: path to barcode reads
    - `REV`: path to reverse reads
  - Add the path to the reference design file
  - `configs` section
    - You can name an assignment configuration in camel-case (here `standardConfig`)
        - `min_support`: minimal number a barcode needs to be observed (here `3`)
        - `fraction`: A barcode is assigned to a sequence if it is assigned at least x% of the barcode is assigned to this sequence (here `0.7` so at least 70% of one barcode need to be assigned to the sequence to be assigned to it)

- `experiments` Experiment section
    - You can name your experiment (here `workshop_data`)
    - `bc_length`: length of the barcode (here `15`)
    - `umi_length`: length of the umi (here `16`)
    - `data_folder`: path to the read data (here `./example_data/counts`)
    - `experiment_file`: path to the file of the experiment table (i.e. condition, replicate and name of read files given) (here `./example_data/experiment.csv`)
    - `demultiplex`: If demultiplexing is required it can be set to `true` but if you have umi, forward and reverse reads you don't need this to happen (here `false`)
    - `assignments`: assign the experiment to an assignment of choise (you can configure multiple assignments and experiments in one config file)
      - name of the assignment itself `MPRAworkshop`:
        - `type` is set to `config`
        - `assignment_name`: name of the assignment you want this experiment to depend on (here `assignMPRAworkshop`)
        - `assignment_config`: name of the assignment config you want this experiment to depend on (here `standardConfig`)
    - `design_file`: your design file (probably same as the reference file of the assignment section if you used the assignment workflow) (here `./example_data/design/workshop_design.fa`)
    - `label_file`: file to the labels of your sequences (here `./example_data/design/workshop_labels.tsv`)


## Running the workflow (local)
- Install snakemake and conda (if not already installed) as shown [here](https://snakemake.readthedocs.io/en/v7.32.3/getting_started/installation.html)
- If you are running the workflow locally, you can use the following command to check if you set everything up correctly (remove the -n to run the workflow):
    - Required modifications:
        - Add number of threads (`n_threads`)
        - Add the path to the MPRAsnakeflow Snakefile (`--snakefile`)
        - Add the path to the conda directory (`--conda-prefix`)
```bash
snakemake -c < n_threads > --use-conda  --configfile workshop_config.yaml --snakefile < path to your MPRAsnakeflow directory + workflow/Snakefile > --conda-prefix < conda installation path > --keep-going 
```
- If assignment and experimental workflow are used the number of jobs shown by snakemake is 262

## Running the workflow (Slurm)
- If you are using a cluster with the job management system SLURM, you can use the following command to check if you set everything up correctly (remove the -n to run the workflow):
- Required modifications:
    - Add the path to the MPRAsnakeflow directory (for Snakefile (`--snakefile`) and sbatch config (`--cluster-config`))
    - Add the path to the conda directory (`--conda-prefix`)
    - cluster status script
    - Maximal number of allowed jobs in parallel (`--jobs`)
```bash
snakemake --use-conda  --configfile workshop_config.yaml --snakefile < path to your MPRAsnakeflow directory + workflow/Snakefile > --conda-prefix < conda installation path > --keep-going --cluster-config < path to the MPRAsnakeflow directory + config/sbatch.yml > < if you have a cluster status script: --cluster-status status.py > --cluster "sbatch --parsable --nodes=1 --ntasks-per-node={cluster.threads} --mem {cluster.mem} -t {cluster.time} -p {cluster.queue} -o {cluster.output} -e {cluster.error}"  --jobs < maximal number of allowed jobs in parallel > --cluster-cancel scancel -n
```
- If assignment and experimental workflow are used the number of jobs shown by snakemake is 262