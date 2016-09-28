#2016年自动化系 香菇滑稽大赛

- 编译条件：
  - Windows下
    - Visual Studio 2012(VC11)（如果不想这么干，请ignore掉项目文件）
    - Opencv 2.4.10（必须）
    - 请自行编辑属性表文件并且放置到项目目录下（与.vcproj同目录），两个配置文件的名称分别为OpenCV_release_property.props和OpenCV_debug_property.props，这是写死在.vcproj里的（如果不想这么干，请ignore掉项目文件）
  - Ubuntu下
    - jsoncpp
    - opencv 
    - cmake


- **各项目的简介请参见各项目的readme.md文件**


- 以下是规则书




#香菇滑稽 Version3 Update2

> 修订：Du\wxk14<br>
> 规则：Du <br>
> 鸣谢：Pierre ;Starlight 

## 剧本简介

参见https://github.com/hzsydy/XiangGuHuaJi/blob/dev-3.0/Scenarios/

## 游戏规则

###概述

领土扩张~~核弹战争~~游戏

- 通过合纵连横为自己争取优势

- 玩家消耗资源放置一次性兵力~~炸逼~~以获得领土主权
- 国家领土为之提供资源

### 回合时序

第0回合

- 选择出生点（调用玩家AI）

从第1回合开始

- （调用玩家AI）


- 外交阶段

  - 结算外交指令，更新国家的外交状态

- 军事阶段

  - 结算军事指令，在自己或同盟国家上，部署兵力


- 所有人部署兵力之后，统一计算全图的影响力，更新领土归属
- 设置新的首都，然后
    - 改变包围状态

- 经济阶段

    - 收取当前本国领土上的资源
    - 随着时间的推移，每个玩家获得线性于回合数的资源 ~~也就是低保~~ 

- 检查胜负

    - 一个失去首都和所有领土的玩家将被迫终止游戏。
      - 没有领土但是有首都的玩家，作为流亡政府还是有收入的，不会退出游戏，参见 **资源**。
    - （乱斗规则）任何一个玩家，存活到回合结束**并**在回合内完成了剧本要求，游戏立即终止
    - 最终游戏的排名顺序（从最重要到次重要）
      - 完成剧本要求
        - 仅限乱斗
      - 坚持的回合数目
        - 坚持得越久，排名越高 ~~不论你曾如何卧薪尝胆/苟且偷生~~
      - 全场游戏最后一回合后的领土面积
        - 若在全场游戏最后一回合因为失去所有城市而退出游戏：因为你的领土在最后一回合结束的时候已经全部变成无主领土了，你的最终领土面积为0。
      - 离开游戏时的资源数量

### 出生点的选择

按次序选择出生点，次序如下：

- 竞价（每个人出一定的初始资源）
- 出价相同则优先上一轮得分低的
- 再相同则优先代码字节数低的

###地图

- 地图尺寸是给定的，预计最大50*50

- 土地

  - 四边形格
  - 主权
    - 有两种状态：中立，某国领土
    - 游戏开始时，除了出生点，所有土地都是中立的


- 地形
    - 地形包括攻击系数和防御系数两部分，这将影响军事阶段

- 资源
    - 资源的富裕程度一般和地形是一一对应的（ ~~因为我们没空给每个点都自设资源~~），剧本指明的特殊点除外。

- 土地的特殊效果
    - 乱斗规则特有，由剧本指明

###外交阶段

外交阶段是改变外交状态的阶段。

- 视野
  - 外交关系决定了你的视野（地图视野 + 国家内政视野）。
  - 地图视野
    - 你的视野为：对你联盟的国家领土以及附近的1格范围 + 对你中立/停战的国家领土。你能观察到这些土地的归属。
    - 对于你没有视野的土地，你只能获得土地的攻防参数和资源数。
      - 事实上，土地的攻防参数和资源数是你所早就知道的：因为我们的地图是恒定不变且在赛前公开的。~~你可以先验的用if-else来规定你AI的策略，比如我们都知道六教比较肥（误）~~
  - 国家内政视野
    - 战争/未发现：你无法获得此国家的任何内政信息
    - 中立/停战：你可以获得此国家的
      - 领土总面积；首都位置；与该国家处于战争和停战状态的国家列表。
    - 同盟：你可以获得此国家的
      - 领土总面积；首都位置，当前库存总量；该国家与所有国家的外交关系列表。

- 侵犯  
  - 只有处于 **战争** 状态的国家之间才会发生领土变更。
  - 中立领土不属于任何一个国家（或，属于一个虚构的，永远和所有人处于战争的不存在的中立国家）

- 联防
  - 同盟国家可以在对方的领土内放置兵力，视作有 **军通**。
  - 如果两个AI处于同盟，并且对同一个AI宣战，那么这两个AI被视作“战争中的盟友”。在战争中，防御时影响力叠加计算，称为 **联防** 。
  - 在退出同盟（任何一方的态度变成KeepNeutral/DeclareWar）的回合
    - 联防立即失效（这是由于游戏回合时序的问题）。战争期间可能导致你的领土被瞬间击破。
    - 双方被迫回收放置在对方领土上的兵力，参见 **投放兵力** 。注意，由于时序的问题，这个过程发生在军事阶段之前。

- 外交态度

  - 包括三种：保持中立KeepNeutral、默许同盟FormAlliance、战争DeclareWar

- 国际关系和外交态度的关系总结表

  | 代码中名称 | Undiscovered |              Neutral               |                AtTruce                |       Allied       | AtWar |
  | :---: | :----------: | :--------------------------------: | :-----------------------------------: | :----------------: | :---: |
  |       |     未发现      |                 中立                 |                  停战                   |         同盟         |  战争   |
  |  注释   |   还没有发现的国家   |                默认状态                |         等于中立，但x回合内不能宣战；不能签同盟          | 自己对自己的国际关系始终标记为同盟。 |       |
  | 可能来自  |      -       |           停战（自动）；未发现（自动）           | 同盟（当至少一方态度不为“默许同盟”）；战争（当双方态度均不为为“战争”） |         中立         |  中立   |
  | 可能变为  |      中立      | 同盟（当双方态度均为“默许同盟”）；战争（当至少一方态度为“战争”） |                  中立                   |         停战         |  停战   |



### 军事阶段

军事阶段是改变地图领土归属的阶段。这一阶段通过消耗资源投放兵力~~炸逼~~来实现。

兵力~~炸逼~~是一个一次性的操作，并不会留存在地图上，只有领土的变更会继承到下一回合。

- 兵力~~炸逼~~的投放
  - 兵力~~炸逼~~可以投放在自己的领土上（也即，自己的领土永远是 **军通** 的一部分），或者有 **军通** 的同盟领土上。
  - 兵力~~炸逼~~的价格是**线性（如果滚雪球太严重可能搞成非线性）**而**离散** 的。
- 领土变更的计算
  - 每单位 **有效** 的兵力对周围 **可以达到的点** 产生【被攻击格】攻击力\*一个随距离衰减的系数的攻击。相当于卷积一个高斯核然后再乘一个系数。
    - 你的进攻路线必须连续。也即，进攻不能导致飞地。实现方法为计算你可以击破的格子位置，并且舍去所有和你当前领土不联通的部分。
      - 计算方法：对于每个你布置炸逼的地方，在此处乘以衰减系数的矩阵。最后将所有的矩阵相加。
      - 例如，隔河击穿了沿线的所有地方，然而却没有击穿桥头堡，那么隔河的部分由于全部相当于飞地所以无法被攻下。
      - 因此坚守桥头堡是非常重要的。例如小桥烧烤店。
  - 每单位 **有效** 的兵力（包括同盟的 **联防**）对周围产生【防御格】防御力\*同一个系数的防御。
  - 如果存在进攻方的攻击高出防御方的防御，并且达到了阈值X，该格领土被**击破** ，从下回合开始改变归属而属于此时攻击最高的进攻方。
- 首都与 **包围**
  - 首都
    - 玩家指定的一个地点。如果玩家不指定，默认为上一回合的值。初始值被置为出生点。
    - 首都可以不设在自己领地而是设在有**军通**的同盟领地上。
    - 如果首都被设置在了不合法的地点（一般是由于时序问题），你将暂时（本回合）失去首都，你的所有领地在本回合结束时视为被**包围** 。你可以指定一个新首都，但是得等到下回合结束的时候才能设立新首都。
      - 设在同盟军通领土上，但本回合同盟被撤除会导致这一现象。
      - 如果你指定的首都位置在本回合被攻陷，也会导致这一现象。
    - 除了作为你的政治中心之外，首都也有一定的攻击力 ~~参考明代和清代将首都设在北京~~ 。只要你的首都本回合存在，就相当于在首都位置丢下等效于每回合x的兵力~~炮兵阵地（误）~~，x为你每回合收入的30%等效的炸弹规模，向上取整。
      - tips:在初期，利用频繁改变首都快速扩张领地是非常重要的~~红警里面拖着基地车到处跑~~
  - 包围
    - 一种状态。在一回合结束时，所有没有 **军通** 到首都的领地视为被**包围** 。
    - 在被**包围** 的地区中，由**你的有效兵力** 产生的防御力视为0。
      - 但是你依然可以投放兵力以进一步进攻。（敌后开花）
      - 但是盟友的有效兵力依然可以产生有效防御。
        - ~~是啊我们就是鼓励支援盟友~~ 
        - ~~如果你不救盟友 下回合你盟友就全被敌人吞掉了哟 你因为停战原因来不及背刺~~

### 经济阶段

经济阶段是收取资源的阶段。

- 计算公式：下一回合的资源 = 上一回合的资源 * （1 - 腐败系数*土地面积） + （土地收入 + 城市收入）

  - 土地收入：你领土上所有资源之和
  - 城市收入：来自你首都的工资~~低保~~。
    - 这意味着流亡政府（没有自己领土，只是靠同盟领地中的首都苟活）也能领到来自城市收入的工资~~低保~~。
  - 腐败系数：每单位土地让你造成的资源收获损耗。 ~~领土越大，中央集权越弱，腐败越严重~~ 


- tips
    - ~~计算你每回合产生的资源是一件吃时间的事情，如果没有时间计算的话，不妨试试直接存储上回合你的资源量，相减就知道你上回合产了多少，这回合也差不多~~

### TIPS

- 利用回合的时序的战术动作
  - 攻击一个中立国家：主动宣战，并在宣战的回合部署重兵，几乎可以保证打出先手速攻。
    - ~~友情提示：如果不能一回合达到战术目的，请考虑准备预备队。~~
    - 如果一回合打崩对面首都当然更好 背刺效果拔群 俗称闪电战
  - 利用同盟军通瞬移：主动请求同盟，同时尝试在对方领土的边缘部署兵力。如果对方不同意同盟，你在对方边境的部署命令将被忽略；而如果对方同意了同盟，你将能够立即在很远的地方扩展出自己的疆土。
    - ~~友情提示：下回合被卖了就傻逼了~~
  - 流亡政府：在本国领土即将被攻占，感觉无力回天时，立即将首都部署到自己最强的一个同盟国领土中。原则上说，你的命运从此就掌握在大哥手中了（大哥一旦取消同盟，回合结束的时候你可能就没有首都了）；不过，因为你对大哥只有益处（联防）没有害处，大哥可能不会劝退你。
    - ~~友情提示：这样虽然可以苟活很久，但是如果你到最后一回合还没有获得自己的领土的话，大哥是很有动机在最后一回合劝退你的~~



## 选手接口

TODO

## 比赛形式

【赛程慢慢来嘛】

- 积分赛~~天梯模式~~

  根据标准规则和标准地图进行 *一局比赛* 。标准地图是公开的。

  关于 *一局比赛* 的定义，参见本段最后。

- 初赛

  初赛的时候将根据标准规则和初赛特制的地图进行 *一轮比赛* ，前7名进入决赛（刨掉乱斗模式的胜利者之外的）

  关于 *一轮比赛* 的定义，参见本段最后。

- 乱斗模式

  每两周一次，采用特别制定的地图和特别制定的胜利条件和特殊规则，决出胜负。每个乱斗周期结束的时候，进行 *一轮比赛* ，前三名作为本届乱斗的优胜者保送进入决赛。【并且应该还有其他优惠】

  特制地图将在该轮次乱斗的前一周公布。

- 决赛

  决赛将采用标准规则和决赛特制的地图进行 *一轮比赛*。

- 一局比赛的定义

  一局比赛中，将由赛程规定数目的玩家参与比赛，获得初始资源，并且决出优胜者（或相对优胜者）。关于如何决出优胜者，参见 **游戏规则-胜利条件** 。

- 一轮比赛的定义

  一轮比赛将由若干局比赛组成。在第一局比赛中，所有玩家的出生点将获得等量的资源。从第二局比赛开始，上一局中的优胜者（或相对优胜者）获得上一局初始资源的1.5倍。持续此过程，直到有玩家的初始资源为原来资源的1.5^5倍，则此玩家获得优胜并从下一局游戏开始被剔除。持续此过程直到足够多的玩家（赛程规定数目的玩家）获得优胜。