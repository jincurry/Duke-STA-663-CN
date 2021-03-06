
.. code:: python

    import numpy as np

Spark on a local mahcine using 4 nodes
======================================

Started with

.. code:: bash

    SPARK_WORKER_MEMORY=512m MASTER=local[4] IPYTHON_OPTS="notebook" pyspark

If you have a Spark cluster, just set

.. code:: bash

    MASTER=spark://IP:PORT

Everything else works the same way.

Using Spark in standalone prograsm
----------------------------------

.. code:: python

    from pyspark import SparkConf, SparkContext
    conf = (SparkConf()
             .setMaster("local[4]")
             .setAppName("STA663")
             .set("spark.executor.memory", "4g"))
    sc = SparkContext(conf = conf)

Check that the SparkContext object is available.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: python

    sc




.. parsed-literal::

    <pyspark.context.SparkContext at 0x10fecee10>



Introduction to Spark concepts with a data manipulation example
---------------------------------------------------------------

Adapted from scala version in Chapter 2: Introduction to Data Analysis
with Scala and Spark of Advanced Analytics with Spark (O'Reilly 2015)

.. code:: python

    import os
    
    if not os.path.exists('documentation'):
        ! curl -o documentation https://archive.ics.uci.edu/ml/machine-learning-databases/00210/documentation
    if not os.path.exists('donation.zip'):
        ! curl -o donation.zip https://archive.ics.uci.edu/ml/machine-learning-databases/00210/donation.zip
    ! unzip -n -q donation.zip
    ! unzip -n -q 'block_*.zip'
    if not os.path.exists('linkage'):
        ! mkdir linkage
    ! mv block_*.csv linkage
    ! rm block_*.zip


.. parsed-literal::

    
    10 archives were successfully processed.


Info about the data set
^^^^^^^^^^^^^^^^^^^^^^^

Please see the ``documentation`` file.

If we are running Spark on Hadoop, we need to transfer files to HDFS
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

    ! hadoop fs -mkdir linkage
    ! hadoop fs -put block_*.csv linkage

.. code:: python

    rdd = sc.textFile('linkage')

Actions trigger execution and return a non-RDD result
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: python

    rdd.first()




.. parsed-literal::

    u'"id_1","id_2","cmp_fname_c1","cmp_fname_c2","cmp_lname_c1","cmp_lname_c2","cmp_sex","cmp_bd","cmp_bm","cmp_by","cmp_plz","is_match"'



.. code:: python

    rdd.take(10)




.. parsed-literal::

    [u'"id_1","id_2","cmp_fname_c1","cmp_fname_c2","cmp_lname_c1","cmp_lname_c2","cmp_sex","cmp_bd","cmp_bm","cmp_by","cmp_plz","is_match"',
     u'37291,53113,0.833333333333333,?,1,?,1,1,1,1,0,TRUE',
     u'39086,47614,1,?,1,?,1,1,1,1,1,TRUE',
     u'70031,70237,1,?,1,?,1,1,1,1,1,TRUE',
     u'84795,97439,1,?,1,?,1,1,1,1,1,TRUE',
     u'36950,42116,1,?,1,1,1,1,1,1,1,TRUE',
     u'42413,48491,1,?,1,?,1,1,1,1,1,TRUE',
     u'25965,64753,1,?,1,?,1,1,1,1,1,TRUE',
     u'49451,90407,1,?,1,?,1,1,1,1,0,TRUE',
     u'39932,40902,1,?,1,?,1,1,1,1,1,TRUE']



.. code:: python

    def is_header(line):
        return "id_1" in line

Transforms return an RDD and are lazy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: python

    vals = rdd.filter(lambda x: not is_header(x))
    vals




.. parsed-literal::

    PythonRDD[4] at RDD at PythonRDD.scala:42



.. code:: python

    vals.count()




.. parsed-literal::

    5749132



Now it is evaluated
^^^^^^^^^^^^^^^^^^^

.. code:: python

    vals.take(10)




.. parsed-literal::

    [u'37291,53113,0.833333333333333,?,1,?,1,1,1,1,0,TRUE',
     u'39086,47614,1,?,1,?,1,1,1,1,1,TRUE',
     u'70031,70237,1,?,1,?,1,1,1,1,1,TRUE',
     u'84795,97439,1,?,1,?,1,1,1,1,1,TRUE',
     u'36950,42116,1,?,1,1,1,1,1,1,1,TRUE',
     u'42413,48491,1,?,1,?,1,1,1,1,1,TRUE',
     u'25965,64753,1,?,1,?,1,1,1,1,1,TRUE',
     u'49451,90407,1,?,1,?,1,1,1,1,0,TRUE',
     u'39932,40902,1,?,1,?,1,1,1,1,1,TRUE',
     u'46626,47940,1,?,1,?,1,1,1,1,1,TRUE']



Each time we access vals, it is *reconstructed* from the original sources
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spark maintains a DAG of how each RDD was constructed so that data sets
can be reconstructed - hence *resilient distributed datasets*. However,
this is inefficient.

.. code:: python

    # vals is reconstructed again
    vals.first()




.. parsed-literal::

    u'37291,53113,0.833333333333333,?,1,?,1,1,1,1,0,TRUE'



Spark allows us to persist RDDs that we will be re-using
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: python

    vals.cache()




.. parsed-literal::

    PythonRDD[4] at RDD at PythonRDD.scala:42



.. code:: python

    # now vals is no longer reconstructed but retrieved from memory
    vals.take(10)




.. parsed-literal::

    [u'37291,53113,0.833333333333333,?,1,?,1,1,1,1,0,TRUE',
     u'39086,47614,1,?,1,?,1,1,1,1,1,TRUE',
     u'70031,70237,1,?,1,?,1,1,1,1,1,TRUE',
     u'84795,97439,1,?,1,?,1,1,1,1,1,TRUE',
     u'36950,42116,1,?,1,1,1,1,1,1,1,TRUE',
     u'42413,48491,1,?,1,?,1,1,1,1,1,TRUE',
     u'25965,64753,1,?,1,?,1,1,1,1,1,TRUE',
     u'49451,90407,1,?,1,?,1,1,1,1,0,TRUE',
     u'39932,40902,1,?,1,?,1,1,1,1,1,TRUE',
     u'46626,47940,1,?,1,?,1,1,1,1,1,TRUE']



.. code:: python

    vals.take(10)




.. parsed-literal::

    [u'37291,53113,0.833333333333333,?,1,?,1,1,1,1,0,TRUE',
     u'39086,47614,1,?,1,?,1,1,1,1,1,TRUE',
     u'70031,70237,1,?,1,?,1,1,1,1,1,TRUE',
     u'84795,97439,1,?,1,?,1,1,1,1,1,TRUE',
     u'36950,42116,1,?,1,1,1,1,1,1,1,TRUE',
     u'42413,48491,1,?,1,?,1,1,1,1,1,TRUE',
     u'25965,64753,1,?,1,?,1,1,1,1,1,TRUE',
     u'49451,90407,1,?,1,?,1,1,1,1,0,TRUE',
     u'39932,40902,1,?,1,?,1,1,1,1,1,TRUE',
     u'46626,47940,1,?,1,?,1,1,1,1,1,TRUE']



Parse lines and work on them
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: python

    def parse(line):
        pieces = line.strip().split(',')
        id1, id2 = map(int, pieces[:2])
        scores = [np.nan if p=='?' else float(p) for p in pieces[2:11]]
        matched = True if pieces[11] == 'TRUE' else False
        return [id1, id2, scores, matched]

.. code:: python

    mds = vals.map(lambda x: parse(x))

.. code:: python

    mds.cache()




.. parsed-literal::

    PythonRDD[10] at RDD at PythonRDD.scala:42



.. code:: python

    match_counts = mds.map(lambda x: x[-1]).countByValue()

.. code:: python

    for cls in match_counts:
        print cls, match_counts[cls]


.. parsed-literal::

    False 5728201
    True 20931


Summary statistics
^^^^^^^^^^^^^^^^^^

.. code:: python

    mds.map(lambda x: x[2][0]).stats()




.. parsed-literal::

    (count: 5749132, mean: nan, stdev: nan, max: nan, min: nan)



.. code:: python

    mds.filter(lambda x: np.isfinite(x[2][0])).map(lambda x: x[2][0]).stats()




.. parsed-literal::

    (count: 5748125, mean: 0.712902470443, stdev: 0.3887583258, max: 1.0, min: 0.0)



Takes too long on laptop - skip
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

stats = [mds.filter(lambda x: np.isfinite(x[2][i])).map(lambda x:
x[2][i]).stats() for i in range(3)]

for stat in stats: print stat

Using the MLlib for Regression
------------------------------

Adapted from `example <https://spark.apache.org/examples.html>`__ in
Spark doucmentation.

.. code:: python

    from pyspark.mllib.classification import LogisticRegressionWithSGD
    from pyspark.mllib.regression import LabeledPoint
    
    def parsePoint(md):
        return LabeledPoint(md[-1], md[2])
    
    full_count = mds.count()
    
    # Only use columns with less than 20% missing as features
    idxs = [i for i in range(9) if 
            mds.filter(lambda p: np.isfinite(p[2][i])).count() > 0.8*full_count]
    
    data = mds.filter(lambda p: np.all(np.isfinite(np.array(p[2])[idxs]))).map(lambda p: parsePoint(p))
    train_data, predict_data = data.randomSplit([0.9, 0.1])
    
    model = LogisticRegressionWithSGD.train(train_data)
    
    labelsAndPreds = predict_data.map(lambda p: (p.label, model.predict(p.features)))
    trainErr = labelsAndPreds.filter(lambda (v, p): v != p).count() / float(predict_data.count())
    
    print "Training Error = " + str(trainErr)


.. parsed-literal::

    [0, 2, 4, 5, 6, 7, 8]
    5160175 574313
    5734488 5160175 574313
    Training Error = 0.00356774093569


References
----------

-  `Spark documetnation <https://spark.apache.org/documentation.html>`__
-  `Spark examples <https://spark.apache.org/examples.html>`__
-  `Learning
   Spark <http://www.amazon.com/Learning-Spark-Lightning-Fast-Data-Analysis/dp/1449358624/ref=sr_1_3?s=books&ie=UTF8&qid=1428533070&sr=1-3&keywords=spark>`__
-  `Advanced Analytics with
   Spark <http://www.amazon.com/Advanced-Analytics-Spark-Patterns-Learning/dp/1491912766/ref=sr_1_4?s=books&ie=UTF8&qid=1428533070&sr=1-4&keywords=spark>`__
-  `Data
   Algorithms <http://www.amazon.com/Data-Algorithms-Recipes-Scaling-Hadoop/dp/1491906189/ref=pd_sim_b_6?ie=UTF8&refRID=04ADYDTN1VCVADVB7HY6>`__

