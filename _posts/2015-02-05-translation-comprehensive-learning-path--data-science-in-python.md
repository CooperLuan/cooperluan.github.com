---
layout: post
title: "[译] Comprehensive learning path – Data Science in Python & Python 科学计算的学习路径"
description: ""
category: [翻译, Python, Data, 科学计算]
tags: [翻译, Python, 科学计算, Data]
---
{% include JB/setup %}

原文 [Comprehensive learning path – Data Science in Python](http://www.analyticsvidhya.com/learning-paths-data-science-business-analytics-business-intelligence-big-data/learning-path-data-science-python/)

## 从 Python 小白到 Kaggler(一个机器学习竞赛网站)

- 运行环境
- 学习 Python 基础
- Python 正则
- [注] 前 3 个步骤都可以在网上找到系统的资料 在这里一笔带过
- 学习 Python 的科学计算库 NumPy, SciPy, Matplotlib and Pandas
- 数据可视化
- 学习 Scikit-learn 和机器学习
- 练习
- 深度学习

## 学习 Python 的科学计算库 NumPy, SciPy, Matplotlib and Pandas

学习从这里开始变得有趣了, 下面是库的基本介绍和练习方法

- 按照 [NumPy tutorial](http://wiki.scipy.org/Tentative_NumPy_Tutorial) 练习一遍, 着重 Numpy arrays 为接下来要做的事情打好基础
- 翻开 [Scipy tutorial](http://docs.scipy.org/doc/scipy/reference/tutorial/) 看基础部分, 有选择的看接下来的部分
- 看这个 [ipython notebook](http://nbviewer.ipython.org/github/jrjohansson/scientific-python-lectures/blob/master/Lecture-4-Matplotlib.ipynb) 到 animations 部分
- 看 Pandas, pandas 为 python 提供了 DataFrame 功能(类似 R), 这也是应该好好话时间练习的地方.
  pandas 将会是所有中型规模数据分析最有效的工具, 可以从简短介绍开始看, [10 minutes to pandas](http://pandas.pydata.org/pandas-docs/stable/10min.html)
  再看更详细的 [tutorial on pandas](http://www.gregreda.com/2013/10/26/intro-to-pandas-data-structures/)

也可以看看 [Exploratory Data Analysis with Pandas](http://www.analyticsvidhya.com/blog/2014/08/baby-steps-python-performing-exploratory-analysis-python/)
和 [Data munging with Pandas](http://www.analyticsvidhya.com/blog/2014/09/data-munging-python-using-pandas-baby-steps-python/)
这两篇文章

### 额外资源

- 如果你需要一本 Pandas & Numpy 的书 建议 [Python for Data Analysis](http://www.amazon.com/Python-Data-Analysis-Wrangling-IPython/dp/1449319793)
- 有很多 Pandas 的教程 [在这里](http://pandas.pydata.org/pandas-docs/stable/tutorials.html)

### 任务

解决 [assignment from CS109 course](http://nbviewer.ipython.org/github/cs109/2014/blob/master/homework/HW1.ipynb)

## 数据可视化

现在到了整个流程中的享受阶段了, Scikit-learn 是 Python 机器学习最有用的库, [这里有简介](http://www.analyticsvidhya.com/blog/2015/01/scikit-learn-python-machine-learning-tool/) 
练习 [CS109 course from Harvard](http://cs109.github.io/2014/pages/schedule.html) 的 10/18 课  
这里你大概会了解到机器学习概论, 监督学习算法如 回归/决策树/组合建模 和非监督学习算法如聚类. 
跟着这些课程做完 [作业](http://cs109.github.io/2014/pages/homework.html) 

### 资源

- 如果有一本书必须读的话 就是 [Programming Collective Intelligence](http://www.amazon.com/Programming-Collective-Intelligence-Building-Applications/dp/0596529325) 一部经典好书
- 课程 [Machine Learning course from Yaser Abu-Mostafa](https://www.edx.org/course/learning-data-caltechx-cs1156x)
- 如果你需要详细的技术细节的话 看 [Machine learning course from Andrew Ng](https://www.coursera.org/course/ml) 并做 Python 练习
- Scikit-learn 上的教程

### 作业

在 Kaggle 上挑战 [这个](http://www.kaggle.com/c/data-science-london-scikit-learn)

## 练习

你现在学会所有需要用的技能了, 接下来关键是要练习, 没有比在 Kaggle 上和那里的人竞赛更好的了.  
在 [Kaggle](http://www.kaggle.com/) 上找一个竞赛, 把你所学都用到上面

## 深度学习

现在你已经学到了机器学习的大部分技术, 到了可以深度学习的时候了, [这里](http://www.analyticsvidhya.com/blog/2014/06/deep-learning-attention/) 有一个很好的机会可以了解深度学习  
我自己是一个深度学习新手, 所以接受这些建议时多个心眼.  
最全面的深度学习资源在 [deeplearning.net](http://deeplearning.net/) 这里有课程/数据集/题目/教程  
也可以在 [course from Geoff Hinton](https://www.coursera.org/course/neuralnets) 学习神经网络基础
