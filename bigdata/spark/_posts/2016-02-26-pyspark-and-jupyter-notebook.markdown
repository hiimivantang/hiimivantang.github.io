---
layout: post
title: "How-to: Use Jupyter notebook with Apache Spark"
tags: ipython jupyter pyspark spark
---

Recently, I had a short demo to showcase the capabilities of the Hadoop / Spark mini cluster that I set up. It was a simulation of flipping 1 billion coins and finding out how many of them were heads. It won't be nice to show my colleagues a terminal running the spark job. I thought it will be nice to be running the spark job on a IPython notebook and that's how my quest of integrating PySpark and IPython started. Jupyter is a spin-off project from IPython. IPython notebooks only supported Python and now with Jupyter, you have a choice of [many different kernels][1] which can run a different programming language each.

This is a guide on setting up your Jupyter notebook, that is running a IPython kernel, to work with PySpark.


### Disclaimer: 

I have tested this only on Ubuntu precise (12.04). Please don't blame me if this guide is not working for your OS. =p 


### Pseudo-How-to:

1. create a IPython profile named "pyspark"
2. create a customized Jupyter notebook kernel spec, which specifies to use "pyspark" profile.
3. create a Python script that sets important environment variables when pyspark profile is being selected.
4. Run IPython notebook and enjoy :)



### Prerequisites:

- Jupyter notebook
- Apache Spark cluster


### How-to:

Make sure you have the above mentioned prerequisites installed before running the following in bash:

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




PS: maybe I will add more explanations to the instructions above, if there is a need to.




[1]: https://github.com/ipython/ipython/wiki/IPython-kernels-for-other-languages
