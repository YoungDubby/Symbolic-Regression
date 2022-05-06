# gplearn的使用
作为一种一种监督学习方法，符号回归（symbolic regression）试图发现某种隐藏的数学公式，以此利用特征变量预测目标变量。  
全自动地发现知识、模式和规律。  
符号回归的具体实现方式是遗传算法（genetic algorithm）。一开始，一群天真而未经历选择的公式会被随机生成。此后的每一代中，最「合适」的公式们将被选中。随着迭代次数的增长，它们不断繁殖、变异、进化，从而不断逼近数据分布的真相。  

# gplearn：
主要组成成分：SymbolicRegressor和SymbolicTransformer  
• SymbolicRegressor是回归器。它利用遗传算法得到的公式，直接预测目标变量的值。  
• SymbolicTransformer是转换器。它并不直接预测目标变量，而是转化原有的特征、输出新的特征，这在特征工程的阶段尤为有效  
## 用户可以自定义函数，即树节点
## 公式的适应度fitness （类似于score、loss）
1. SymbolicRegressor的适应度有三种，都是机器学习里常见的error function：  
	• mae: mean absolute error  
	• mse: mean squared error  
	• rmse: root mean squared error  
2. SymbolicTransformer会最大化输出的新特征与目标变量之间的相关系数的绝对值：  
	• pearson：皮尔逊积矩相关系数（Pearson product-moment correlation coefficient）  
	默认设置。因为它衡量线性相关度，所以适用于下一个预测器（分类或回归）是线性模型的情况。  
	• spearman：斯皮尔曼等级相关系数（Spearman's rank correlation coefficient）。因为它衡量单调相关度，所以适用于下一个预测器（分类或回归）是决策树类模型的情况。  
## 公式的进化
初始化阶段，有population_size课公式树被随机生成，每棵树深度init_depth（min,max），每棵树的初始方式由init_method决定（grow从下往上、full最后一层变量常数、其他全是函数、half and half 一样一半）。在模拟自然选择中，适应度低的公式淘汰，每一代里有tournament_size个公式被随机选中，其中适应度最高的是胜利者进入下一代，size越小选择压力就越大。n_jobs使用并行操作，一代划分为几个线程，全部完毕后进入下一代  
## 进入下一代的方式
1. 繁衍（完全不改变优胜者），直接进入  
2. 交叉：优胜者中随机选子树替换另一个公式树的子树（一般是次高者））  
3. 子树变异：由p_subtree_mutation，子树被随机子树完全代替  
4. hoist变异：p_hoist_mutation，子树中选子树提升到原来位置  
5. 点变异：p_point_replace随机改变一个节点  
## 公式树的膨胀和解决办法
1. 膨胀：当公式更加复杂，计算速度越来越慢，适应度却毫无提升
2. 对抗膨胀：
  - 适应度函数中加节俭系数，由parsimony_coeffcient控制，惩罚太复杂公式，auto。
  - 使用hoist变异，削减过大公式树。 
  - max_samples每一代选取部分样本衡量适应度，提升速度保证公式多样性。
