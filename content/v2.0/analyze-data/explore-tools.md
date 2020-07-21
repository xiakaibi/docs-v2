---
title: Explore data science tools
description: >
  Use ... to learn about and explore data science tools.
menu:
  v2_0:
    parent: Analyze data
weight: 102
---


InfluxDB is built to integrate any(is this true?) data science tool, language, or library. Start by exploring integrations with our communities' most popular data science tools:

- Explore Notebooks

  - Zeppelin Notebooks:

    - Spark

  - Jupyter Notebooks. 
    1. First, set up a pre-populated Jupyter Notebook Docker container.
    2. Next, select one of our Jupyter Notebooks to explore your data.
  
  (basic: this notebook includes a Python script to stream data from InfluxDB 2.0 (Stream-influx2.ipynb), a bash script to run InfluxDB 2.0 and Telegraf (run.sh), and a Telegraf configuration file (telegraf.conf).

  (anomaly detection: this notebook includes two Python scripts.  to stream data from InfluxDB 2.0 using the Python client, and return it as a Pandas DataFrame.

    - [Query and stream data from InfluxDB 2.0 using Flux](https://github.com/influxdata/Io/blob/master/notebooks/Basic/Stream-Influx2.ipynb)
    
    - Pandas

    - For anomaly detection
      
      - [BIRCH from scikit-earn with  ADTK library](https://github.com/influxdata/Io/blob/master/notebooks/Anomaly-Detection/BIRCH%20and%20InfluxDB.ipynb)
      - [Level Shift anomaly detection](https://github.com/influxdata/Io/blob/master/notebooks/Anomaly-Detection/Level-Shift%20Anomaly%20Detection.ipynb)

    - Forecasting
