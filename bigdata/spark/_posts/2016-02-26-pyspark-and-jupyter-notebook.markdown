---
layout: post
title: "How-to: Use Jupyter notebook with Apache Spark"
---


This blog post is a guide in setting up your Jupyter notebook environment to work with PySpark. 

Pre-requisite:
  - Jupyter notebook
  - Apache Spark cluster


```bash
ipython profile create pyspark

jupyter notebook --generate-config
sudo pip install py4j
mkdir -p ~/.ipython/kernels/pyspark
touch ~/.ipython/kernels/pyspark/kernel.json
```

insert the following into ~/.jupyter/jupyter_notebook_config.py

```python
c = get_config()

c.NotebookApp.ip = '*'
c.NotebookApp.open_browser = False
c.NotebookApp.port = 1337 # or whatever you want; be aware of conflicts
```


insert the following into ~/.ipython/kernels/pyspark/kernel.json

```json
{
 "display_name": "pySpark (Spark 1.5.0)",
 "language": "python",
 "argv": [
  "/home/vagrant/anaconda2/bin/python",
  "-m",
  "IPython.kernel",
  "--profile=pyspark",
  "-f",
  "{connection_file}"
 ]
}
```


insert the following into ~/.ipython/profile_pyspark/startup/00-pyspark-setup.py

```python
import os
import sys

os.environ['SPARK_HOME']='/opt/cloudera/parcels/CDH/lib/spark'   #replace with your spark home directory
spark_home = os.environ.get('SPARK_HOME', None)
if not spark_home:
        raise ValueError('SPARK_HOME environment variable is not set')

sys.path.insert(0, os.path.join(spark_home, 'python'))
sys.path.insert(0, os.path.join(spark_home, 'python/lib/py4j-0.8.1-src.zip'))

#java_home = os.environ.get('JAVA_HOME', None)
#if not java_home:
#       raise ValueError('JAVA_HOME environment variable is not set')
#sys.path.insert(0, os.path.join(java_home, '/usr/lib/jvm/java-7-oracle-cloudera'))

os.environ['JAVA_HOME']='/usr/lib/jvm/java-7-oracle-cloudera'
#os.environ['PYSPARK_SUBMIT_ARGS']='--executor-memory 500M pyspark-shell'
#os.environ['PYSPARK_SUBMIT_ARGS']='--executor-memory 500M --executor-cores 4  --num-executors 20 pyspark-shell'
execfile(os.path.join(spark_home, 'python/pyspark/shell.py'))
```




Enjoy using PySpark with Jupyter notebook:

```bash
ipython notebook
```
