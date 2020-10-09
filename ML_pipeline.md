# Discovery of Machine Learning Model Pipeline

## Introduction:
Machine learning (ML) models are likely to be subject to various iterations until a satisfactory model is develop or it may as well be a project 
that has to constantly adapt to an updated dataset. In our context of students' performance on a myriad of questions from multiple products 
of SAS's Portal, data sets will be updated constantly in volume as students interact more and solve more question with out Portal. New 
students/schools join our Portal every year, so the size of our datasets are expected to keep growing. Moreover, even though students are expected
to stop using our Portal when they finish the 3rd year, these students' data are likely to be part of our ML model's training for years after they
leave.

That said, a model in constant change both in data (from which the model is trained) and the model itself is likely to benefit from large datasets'
versioning, reproducibility and experimentation. Data Version Control (DVC), a python library, will be introduced to with the purpose of sharing 
and contributing to the SAS's Data Science environment and workflow. From the DVC documentation we have:

> Data Version Control is a new type of data versioning, workflow, and experiment management software, 
> that builds upon Git (although it can work stand-alone). DVC reduces the gap between established 
> engineering tool sets and data science needs, allowing users to take advantage of new features
> while reusing existing skills and intuition.

## DVC - Overview:


### Data Versioning 

DVC is cable of handling large files and its versions effortlessly similarly to git. These files can be tracked by DVC, stored remotely in services
such as Amazon S3 and Google Cloud Storage.

```
dvc init
dvc add data/data.xml
dvc remote add -d storage s3://my-bucket/dvc-storage
dvc push
```

Other environments that may need to run the ML models can retrieve the DVC-tracker data with _dvc pull_ just like you would do on git. Similarly, 
datasets versions can be switched by checking out of a git branch or commit and the checking out of dvc.

```
git checkout <...>
dvc checkout
```

While the default is to push the data cached locally to the remote storage and to pull the opposite way, it is recommeded to avoid these kind of
operations from remote storages if the data is large. The process is similart to the previous however now DVC tracks data on external locations.

```
dvc remote add s3cache s3://mybucket/cache
dvc config cache.s3 s3cache

dvc add --external s3://mybucket/existing-data

dvc run -d data.txt \
        --external \
        -o s3://mybucket/data.txt \
        aws s3 cp data.txt s3://mybucket/data.txt
```

### Data Pipeline

Easy versioning of data and DVC pipelines improves the organization of the project, reproduction the workflow and results. **Pipeline Stages** can be
added with _dvc run_ along with its inputs (dependencies and parameters), outputs, metrics, etc., followed by _dvc push_ to save them remotely. Below
add to the pipeline the step **prepare** with parameters _-p_, dependencies _-d_ and output _-o_.

```
dvc run -n prepare \
        -p prepare.seed,prepare.split \
        -d src/prepare.py -d data/data.xml \
        -o data/prepared \
        python src/prepare.py data/data.xml
```

This and others stages added to the pipeline are added to a file _dvc.yaml_.

The command _dvc dag_ exhibits the Data Pipeline or Dependency Graph as illustrated below:

![dvc_dag](https://gitlab.sasdigital.com.br/tisas/gremlins/tars/-/raw/feature/ds-problems-list/deploy/dvc_dag.png)

The experimentation and reproducibility is possible with the command _dvc repro_. The former enables the execution of model with different values for
its parameters, for example, while avoiding unnecessary reruns of stages of the pipeline that didn't change with the new parameters, as they are
already cached by previous runs. The later allows DVC to pull the correct cached data and model version corresponding to the desired commit or branch
on Git. The figures below compare two runs with distinct parameters for training.

![dvc_differences](https://gitlab.sasdigital.com.br/tisas/gremlins/tars/-/raw/feature/ds-problems-list/deploy/dvc_differences.png)

![dvc_plot](https://gitlab.sasdigital.com.br/tisas/gremlins/tars/-/raw/feature/ds-problems-list/deploy/dvc_plot.png)
