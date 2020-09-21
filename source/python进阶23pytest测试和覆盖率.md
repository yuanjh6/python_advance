# python进阶23pytest测试和覆盖率
插件安装：pip install pytest-cov  
命令：pytest --cov=src --cov-report=html  
src：python源代码路径（文件夹形式，不支持模块or模块.py等形式）  
注意：文件夹下所有**符合文件名:test_*.*_test.py都必须能跑通，否则html报表中只有函数定义**，没有函数内的代码执行情况。  

其他插件:  
1.多重校验 pytest-assume   
2.设定执行顺序 pytest-ordering  
3.失败重跑 pytest-rerunfailures  
4.显示进度条 pytest-sugar  
5.pytest-pep8，就是在做pytest测试时，自动检测代码是否符合PEP 8规范的插件。  
6.pytest-mock是一个pytest的插件，安装即可使用。 它提供了一个名为mocker的fixture，仅在当前测试function或method生效，而不用自行包装。  

## 参考
[转]Pytest 基础教程：https://blog.csdn.net/u011331731/article/details/108189950  
pytest的一些实用插件实践：https://www.jianshu.com/p/7df6d781f100  
【pytest】pytest-cov ：统计代码测试覆盖率:https://blog.csdn.net/waitan2018/article/details/104400749  
pytest-cov插件计算单元测试代码覆盖率:https://blog.csdn.net/u011519550/article/details/86367137  