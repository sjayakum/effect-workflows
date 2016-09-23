# effect-workflows
DIG workflow processing for the EFFECT project.

## Installation

1. Download and install conda - https://www.continuum.io/downloads
2. Install conda env - `conda install -c conda conda-env`
3. Create the environment `conda env create`. This will create a virtual environment named effect-env (The name is defined in environment.yml)
4. Switch to the environment using `source activate effect-env`

<B>NOTE: You should build the environment on the same hardware/os you're going to run the job</B>


## Running script to convert PostgreSQL to CDR
1. Follow above instructions to create conda environment - Steps 1-3
2. Switch to the effect-env: `source activate effect-env`
3. Execute: 

  ```
  python postgresToCDR.py --host <postgreSQL hostname> --user <db username> --password <db password> \
                          --database <databasename> --table <tablename> --output <output filename>`
  ```


## Running script to convert CSV,JSON,XML,CDR data into a format that should be used for Karma Modeling
1. Follow above instrucrions to create conda environment - Steps 1-3
2. Switch to the effect-env: `source activate effect-env`
3. Execute:

  ```
  python generateDataForKarmaModeling.py --input <input filename> --output <output filename> \
        --format <input format-csv/json/xml/cdr> --source <a name for the source> \
        --separator <column separator for CSV files>
  ```

  Example Invocations:
  ```
  python generateDataForKarmaModeling.py --input ~/github/effect/effect-data/nvd/sample/nvdcve-2.0-2003.xml \
            --output nvd.jl --format xml --source nvd


  python generateDataForKarmaModeling.py --input ~/github/effect/effect-data/hackmageddon/sample/hackmageddon_20160730.csv \
            --output hackmageddon.jl --format csv --source hackmageddon


  python generateDataForKarmaModeling.py --input ~/github/effect/effect-data/hackmageddon/sample/hackmageddon_20160730.jl \
            --output hackmageddon.jl --format json --source hackmageddon
  ```

## Loading data in HIVE

1. Login to AWS and create a tunnel - `ssh -L 8888:localhost:8888 hadoop@ec2-52-42-169-124.us-west-2.compute.amazonaws.com`
2. Access Hue on http://localhost:8888
3. See hiveQueries.sql for examples

## Running the workflow

To build the python libraries required by the workflows,

1. Edit make.sh and update the path to `dig-workflows`
2. Run `./make.sh`. This will create `lib\python-lib.zip` that can be attached with the `--py-files` option to the spark workflow
3. Copy the `python-lib.zip` file to AWS - `scp lib/python-lib.zip hadoop@ec2-52-42-169-124.us-west-2.compute.amazonaws.com:/home/hadoop/effect-workflows/`
4. Login to AWS and run the workflow

```
ssh -L 8888:localhost:8888 hadoop@ec2-52-42-169-124.us-west-2.compute.amazonaws.com
spark-submit --deploy-mode client  \
    --py-files /home/hadoop/effect-workflows/python-lib.zip \
    /home/hadoop/effect-workflows/effectWorkflow.py \
    cdr hdfs://ip-172-31-19-102/user/effect/data/cdr-out text
```

## Extras

* To remove the environment run `conda env remove -n effect-env`
* To see all environments run `conda env list`

