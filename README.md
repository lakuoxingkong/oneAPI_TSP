
# 面向TSP的改良GA算法

使用了Intel oneAPI对遗传算法的问题初始化、种群初始化、评估、选择、编译方面进行了并行优化，并且还运用了FPGA进行加速处理，提升了遗传算法解决旅行商问题的效率。



## 编码策略和种群初始化
直接用城市的序号进行编码，对每一个个体的基因随机地赋予城市数量范围之内的城市编号，
然后如果出现了城市编号重复的情况就尝试继续生成随机数去重。
## 初始化城市坐标
使用pair来表示每一个城市的坐标。将城市坐标的随机生成运用oneAPI进行并行化处理，
把存储城市坐标的容器传入到buffer里面，再提交到队列，进行并行处理。其中使用oneapi::dpl::uniform_int_distribution来生成0,
100范围之内的随机数，用以作为城市的坐标。

## 适应函数
由于基因所对应的是城市的编号，记此编号为i，记城市数组为c,所对应的城市坐标就是
c(i)。要计算一条路径中的总花费，就是要计算个体DNA中的每一对相邻的基因所对应的城市的距离
计算出来，并将他们相加（最后一个基因所对应的城市要计算和第一个基因所对应城市的距离）。
计算评估值的公式如下，i表示第j个个体中的第i个基因。</br>
![alt text](https://github.com/QuietHuihui/oneAPI_TSP/blob/main/formula_img/dANDeval.gif?raw=true)
</br>
在计算的过程中，为了便于将过程并行化，把种群展开成一个一维的向量，然后再利用oneAPI进行并行计算。


## 选择算子
采用轮盘赌的选择方式。把每一个个体的适应值eval(j)加起来成为总的适应值。然后将每一个个体的适应值去除以总的适应值，得到被选择的概率p(j)。
然后再对第j个个体，将每一个p(j)前面的所有概率累加起来得到P(j)。接下来生成随机数rand,如果rand大于P(j-1)而小于P(j)，
那么就选择第j个个体。最终选择完成之后，将选择出来的个体替换掉初始的种群。此处对p(j)的计算使用了oneAPI进行并行改良。
## 交叉算子
交叉策略是单点交叉，生成随机数，若其小于交叉概率，则可以进行交叉。交叉时生成随机点位，
将相邻两个个体进行交叉，第一个个体的右部与第二个个体的左部交换，得到两个新的个体。然后分别对这两个个体进行基因去重。
## 变异算子
生成随机数，若小于变异概率则个体可进行变异。变异时生成两个随机数，对应其DNA的两个标，将对应的两个基因进行交换完成一个个体的变异。

## 运行结果
时间单位均为毫秒ms。
参数：运行100代，种群数量为145000，城市数量为700，交叉概率为0.9，变异概率为0.1</br>
![alt text](https://github.com/QuietHuihui/oneAPI_TSP/blob/main/data_img/SEQ.png?raw=true)
![alt text](https://github.com/QuietHuihui/oneAPI_TSP/blob/main/data_img/PAR.png?raw=true)
![alt text](https://github.com/QuietHuihui/oneAPI_TSP/blob/main/data_img/Par&Seq.png?raw=true)
![alt text](https://github.com/QuietHuihui/oneAPI_TSP/blob/main/data_img/SUB.png?raw=true)

