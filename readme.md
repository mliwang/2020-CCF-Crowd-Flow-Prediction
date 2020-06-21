- #### 赛题简介
    * **背景**：2019新型冠状病毒（COVID-19）感染的肺炎疫情发生对人们生活生产的方方面面产生了重要影响。人口的流动聚集，客观上加大了疫情传播的风险和防控的难度。出于对公共卫生、重大公共利益相关影响的为研究目的，为进一步掌握人员流动聚集动向，针对疫情相关的做重点区域人群聚集密度预测。
    * **任务**：选取北京市若干不同类别的重点区域，提供各区域历史多天分小时人群密度数据。同时，提供北京六环内历史多天分小时的网格（200*200）人群密度、北京市迁入和迁出指数、网格间联系强度指数，预测接下来每天分小时北京市重点区域的人群密度。评分指标基于RMSE，具体公式如下：</br>
    ![eq1.svg](https://github.com/agave233/233/raw/master/equation-1.svg?raw=true)
    * **成绩**：2/1009（A榜得分0.15482，B榜得分0.1397）
    * **比赛地址**：https://www.datafountain.cn/competitions/428

  

- #### 难点与挑战   
    1. **时间序列长期预测的困难性**
    题目给出的数据是1.17～2.15共计30天的历史人群密度数据，要求预测接下来9天每个区域的分小时人群密度，所以要预测的序列长度为216，属于长期序列预测问题。此外，由于30天历史数据大部分是在春节前后，而这期间的人群密度分布和疫情期间的分布是不同的，如何从少于30天的历史数据能准确预测未来9天的人群密度是困难的。
    2. **复杂的两级时间周期性**
    和一般的序列预测问题不同，本题目需要考虑天级别的周期性(daily periodicity)的周级别的周期性(weekly periodicity)。
    3. **区域之间人员流动的空间依赖性**
    不同区域的人群密度并不是独立的，区域的人流量变化是由于人群流动性的存在，各个区域在空间上存在关联性，在预测人群密度时建模空间特征也是一个很有挑战的问题。

- #### 数据分析
    赛题给出的数据包括区域历史人群密度和网格联系等数据，对数据进行探索分析：
    ###### 1. 人群密度总体分布
    首先对所有区域的平均人群密度分布进行统计，如下图所示。可以发现在1.24之前由于疫情才刚开始发展并且还未到春节，北京市的人群密度很大，在春节后受疫情影响人群密度明显变小，并且在原本假期结束的时间点之后人群密度一直很小，在2.2之后的两周才呈现出一定的规律性，所以春节左右的时间段对应的历史数据应该清除，否则会引入过多的噪声，最后训练模型时选择的数据是2.2到2.15期间的历史数据。
     ![北京人流量统计](https://github.com/agave233/233/raw/master/bj_flow_heatmap1.png?raw=true)
     ###### 2. 周期性分析
    * 2.1  Weekly Periodicity
    对每个区域的周级别周期性进行分析，统计区域每天的平均人群密度进行可视化，如下图是北京大学第三医院和华联商厦每天的平均人群密度曲线图。从图中可以发现：（1）区域满足每周一定的周期性，例如图中两个区域的人群密度在一周内工作日较大，而在周末会迅速减小，下一周的人群密度又会按照同样的周期性；（2）不同区域的周级别周期性是不同的；（3）周期性是从历史数据最近的两周才呈现出来。
    ![周级别周期](https://github.com/agave233/233/raw/master/period-1.png?raw=true)
    * 2.2  Daily Periodicity
    对每个区域的天级别周期性进行分析，对区域一周内的人群密度分小时进行可视化，如下图是医院和购物中心对应的曲线图。人群密度每天呈现出明显的周期性规律，每天的人群密度在不同小时分布是类似的。此外，通过曲线图还可以发现工作日的天级别周期性相似，而周末的天级别周期性却相差较大，这是因为工作日的人群出行规律和周末存在差异。夜间和凌晨的人群密度在不同天波动不会很大，但是中午的人群密度会有很大波动，可以认为夜晚的人群密度相对稳定，而不同天的人群密度变化主要是因为白天的差异所致。
    ![天级别周期](https://github.com/agave233/233/raw/master/period-2.png?raw=true)
     ###### 3. 趋势性分析
    * 3.1  区域间趋势的关联性（粗粒度分析）
    首先从天的粒度进行趋势的分析，每个区域的人群密度在一段时间内满足一定的趋势性，由于疫情的原因人们出行的可能性减小，所以大部分区域的人群密度呈现出减小的趋势，例如火车站类型的区域人群密度持续减小；而有些类型的区域，例如购物中心在疫情前期人群密度减小但是随着疫情逐渐得到控制，人们也会选择到商场进行一些必要的购物，所以这些区域的人群密度又会呈现逐渐增加的趋势。
    ![类别趋势](https://github.com/agave233/233/raw/master/trend-1.png?raw=true)
    * 3.2  不同时间段的趋势分析（细粒度分析）
    本节从小时的粒度对区域的人群密度趋势进行分析，如下图是最近三周内对应天的人群密度变化曲线，可以发现从第二周到第四周在工作日每周对应天的人群密度有明显增长，但是夜间（23点到6点）的人群密度却基本没有变化，所以在每天不同时间段内的增长趋势差异大，白天尤其是中午时间段增长趋势很大，但在凌晨增常趋势却很小，在建模时需要考虑这些因素。
    ![分级趋势](https://github.com/agave233/233/raw/master/trend-2_.png?raw=true)

- #### 模型框架
    ![模型框架](https://github.com/agave233/233/raw/master/model-2.png?raw=true)
    算法的整个框架如下图所示，分为数据层，模型层和融合层，最终的输出结果由树模型预测和规则模型预测两部分组成。数据层我们选择了网格联系强度、天气数据、区域历史人群密度和区域属性数据作为模型的输入，模型具体说明如下。

     ###### 1. 建模思路
     由上一节的分析可知，人群密度的时序数据具有两级周期性，如果直接按照时间的顺序直接预测9天的人群密度，序列长度为216，这样连续预测长序列的误差很大，为了解决这一问题，我们是使用了两个维度的预测方法进行结合。如下图所示，由于30天历史数据前期的人群密度与要预测时间段差别大，所以我们仅使用2.2之后的数据作为训练数据。
     * 1.1 水平方向的预测
     基于历史数据每天的相同小时进行预测，这样训练24个回归树模型进行预测，将预测的序列长度缩短到了9；
     * 1.2 垂直方向的预测
     对历史训练数据进行统计建模，得到周级别的周期系数和天级别的周期系数然后进行预测。
     ![建模思路](https://github.com/agave233/233/raw/master/overall.png?raw=true)
     ###### 2. 树回归模型
     树回归模型部分的框架如下图所示，首先进行特征提取得到小时级别的时序特征和区域属性特征，并且基于网格联系强度构建区域的关联流动图，通过graph embedding算法提取空间特征，然后将时序特征、区域属性特征和空间特征输入回归树模型对未来人群密度进行连续预测。
    <div align="center">  
     <img src="https://github.com/agave233/233/raw/master/model-tree.png?raw=true" width="400" height="320"/>
    </div>

    **2.1 特征提取**

    我们首先从以下几个方面构建特征：时序特征、属性编码特征和天气节假日等特征。

    * 时序特征：前一天同一时刻的人群密度，前三天同一时刻的平均人群密度；
    * 天气节假日特征：当前天是第几天，当前天是否为周末，前一天是否为周末，当前天的最低气温和是否下雨，前一天的最低气温和是否下雨；
    * 属性编码特征：每个区域的历史平均人群密度，区域面积，区域对应类型的平均人群密度。

    </br>

    **2.2 构建区域关联图**

    题目给出的数据是200m*200m的网格联系强度，但是预测目标并不是网格而是区域的人群密度，网格和区域之间没有严格的对应关系（区域可能包含多个网格，网格内也可能有多个区域），所以如何基于网格构建区域之间的关联图是一个问题。
    因为区域中心所在的网格往往代表来这个区域最核心的人群密度信息，所以在这里我们直接按照数据给定的区域中心所在网格这一关系来构建区域关联图。需要注意的是，有些区域所在的中心网格并没有在网格联系强度数据中出现，等价于网格缺失，所以对这些区域需要重新寻找距离区域中心最近的网格来代表这个区域。最终可以构建24个加权有向图，分别对应24个小时下区域之间的关系网络，边上的权重表示区域间的联系强度。

    <div align="center">  
     <img src="https://github.com/agave233/233/raw/master/graph.png?raw=true" width="480" height="250"/>
    </div>

    **2.3 空间特征学习**
    构建区域关联图之后对区域的特征空间进行提取，时刻t的有向图中存在区域A指向区域B的连边表示t时刻A到B有一定的人群流动性，所以我们选择了基于随机游走的图嵌入算法来学习24个小时对应的空间特征。在实验中我们选择了node2vec和deepwalk两种算法，最终使用的是实际效果更好的node2vec。

    **2.4 小时级别的人群密度预测**
    使用LightGBM[1]和XGBoost[2]两个回归树模型分别对结果进行预测，对结果进行融合。分别用p和q表示h小时对应的两个模型，则第i个区域在第d天h小时的人群密度预测值为：

    </br>
    ![eq1.svg](https://github.com/agave233/233/raw/master/equation-2.svg?raw=true)

    其中g的增长趋势因子，在下一节的规则模型会介绍到。因为由于训练数据的时间范围很短，难以将趋势信息提取并使用回归树模型训练，所以预测的结果不完全满足区域人群密度的增长趋势，为了克服这一问题我们对回归树预测的结果乘上趋势因子以获得更准确的预测结果。
     ###### 3. 规则模型
     如下图所示，在进行规则建模[3]时，我们考虑了四个因素增长趋势因子(growth)、基础人群密度(base)、周级别的周期因子(alpha)和天级别的周期因子(beta)。
     <div align="center">  
     <img src="https://github.com/agave233/233/raw/master/model-rule.png?raw=true" width="400" height="271"/>
    </div>

    3.1 增长趋势因子
    由之前的数据分析可知，一个区域在不同时间段的增长趋势不同（比如白天增长趋势明显，但夜间相对稳定），所以在计算增长趋势因子时我们将一天24小时划分为三个集合，分别为S_1=\{23,0,1,2,3,4,5,6\},S_2=\{7,8,9,19,20,21,22\},S_3=\{10,11,12,13,14,15,16,17,18\}。S1集合包含23点到早上6点，在这个时间段活动人群最少，其人群密度相对稳定；S2集合包含早上6点到8点和晚上19点到22点，这个时间段人们开始出行活动和回家，其人群密度有一定的趋势性；S3集合包含早上10点到下午18点，这个时间段是人们活动的主要时间，其人群密度呈现出明显的趋势性。计算公式如下：

    </br>
    ![eq1.svg](https://github.com/agave233/233/raw/master/equation-3.svg?raw=true)

    growth表示的是区域i最近一周呈现出的增长趋势，开根号进行趋势平滑，其中Sh表示时间h所在的集合，E1表示预测天的前一周的天数集合，E2表示E1的前一周的天数集合。为了增强趋势因子的鲁棒性，我们还考虑了区域所属类型的增长趋势因子，加权得到最终区域i的增长趋势因子为：
    </br>
    ![eq1.svg](https://github.com/agave233/233/raw/master/equation-4.svg?raw=true)

    w0,w1,w2是三个加权参数，满足w0+w1+w2=1，第一项表示单个区域计算得到的趋势因子，第二项表示区域所属类型1的所有区域平均趋势因子，第三项表示区域所属类型2的所有区域平均趋势因子（如北京南站的类型是“交通设施；火车站”，其类型1是“交通设施”，类型2是“火车站”）。

    </br>
    3.2 基础人群密度

    我们选择最近三天的平均人群密度base和回归树模型预测的第一天人群密度base进行加权作为基础人群密度，计算公式如下：
    </br>
    ![eq1.svg](https://github.com/agave233/233/raw/master/equation-5.svg?raw=true)

    3.3 周和天级别的周期因子_
    周级别的周期因子是一周七天为周期的分布系数，我们选择的是最近一周的周期分布系数作为预测的周期因子（也尝试过最近两周并且进行加权融合，但效果不如最近一周的效果好），计算公式如下：
    </br>
    ![eq1.svg](https://github.com/agave233/233/raw/master/equation-6.svg?raw=true)

    天级别的周期因子是一天24小时为周期的分布系数，考虑到周末和工作日的出行活动差别较大，分别对工作日和周末的24小时统计的分布系数求平均得到对应天级别的周期因子：

    </br>
    ![eq1.svg](https://github.com/agave233/233/raw/master/equation-7.svg?raw=true)

     ###### 4. 模型融合
    在实验中我们发现回归树模型预测周末人群密度的效果很好，但工作日预测效果相对较差，所以我们去掉训练数据的周末后又按照同样的训练回归树模型预测工作日的人群密度。最后对回归树模型的预测结果和规则模型的预测结果进行加权融合。规则模型和回归树模型两种建模方式侧重点不同，从两个维度方向去预测，融合后准确率有明显的提升。此外，由于预测的是未来9天的人群密度，而周期性为7天，所以我们用前两天的预测结果乘上趋势因子作为最后两天的预测结果，从而避免连续预测的误差传递问题。

- #### 深度学习模型尝试
    上面的建模思路虽然考虑到了空间关联特征，但是题目给出的网格人群密度数据并没有使用，而这部分数据显然对区域的预测也有意义，为了能利用这部分数据，我们尝试使用深度时空残差网络DeepST模型[4]预测网格的人群密度。
    由于题目给出的网格数据是分散的，首先要把网格在地理空间上进行恢复，得到L*L的人群密度图像，如下图是早上7点以天安门为中心，大小为131\*131的网格人群密度分布图，然后通过这些网格图像分别构建相邻时间、周期性、趋势性的三维张量，并且结合天气、区域属性等特征作为DeepST的输入，预测所有网格在下一个时间段的人群密度。由于时间原因这部分并没有继续往下做，接下来的思路是通过DeepST预测出网格的人群密度后，通过算法建立网格和区域的映射关系，从而将网格的预测结果可以对应到区域的人群密度，从而提高预测的准确性。

    <div align="center">  
     <img src="https://github.com/agave233/233/raw/master/bj_grid_heatmap.jpg?raw=true" width="500" height="414"/>
    </div>

- #### 参考
    [1] Tianqi Chen and Carlos Guestrin. XGBoost: A Scalable Tree Boosting System. In 22nd SIGKDD Conference on Knowledge Discovery and Data Mining, 2016
    [2] Guolin Ke, Qi Meng, Thomas Finley, Taifeng Wang, Wei Chen, Weidong Ma, Qiwei Ye, Tie-Yan Liu. "LightGBM: A Highly Efficient Gradient Boosting Decision Tree". Advances in Neural Information Processing Systems 30 (NIPS 2017), pp. 3149-3157.
    [3] dropout, 时间序列规则法快速入门, https://www.jianshu.com/p/31e20f00c26f
    [4] Zhang J , Zheng Y , Qi D . Deep Spatio-Temporal Residual Networks for Citywide Crowd Flows Prediction[J]. 2016.

