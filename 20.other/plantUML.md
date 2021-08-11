参考：https://zhuanlan.zhihu.com/p/19901245

### plantUML安装和新建

在idea上安装PlantUML插件

![plantUML1](https://youcai922.github.io/99.src/img/plantUML2.png)

创建一个PlantUML文件

![plantUML1](https://youcai922.github.io/99.src/img/plantUML1.png)





### plantUML支持如下UML图：

- Sequence diagram（序列图）
- Usecase diagram
- Class diagram
- Activity diagram（活动图）
- Component diagram
- State diagram
- Object diagram
- GUI Wireframe

#### Sequence diagram

```
@startuml
participant Tars
actor Copper
database Murphy

Tars -> Copper
Copper --> Murphy: morse code
@enduml
```

![diagram-2524278456546615833](https://youcai922.github.io/99.src/img/diagram-2524278456546615833.png)

内容解释：

- @startuml 声明一个图形的起始

- participant、actor、database代表不同的角色类型（还有control和entity）
- ![diagram-6776278217798399884](https://youcai922.github.io/99.src/img/diagram-6776278217798399884.png)
- 两个角色之间的消息 ， ->为实现，-->为虚线
- 消息后加：可以对消息添加说明
- @enduml声明一个对图形的结束

另外plantUML也可以结合html提高图形的美化

```
@startuml
actor "公司\n<b>老板</b>" as copper #0091ff
actor "公司\n员工" as murphy #red

copper -[#orange]> murphy: 管理和<font color=red>关爱</font>
@enduml
```

效果展示：

![diagram-11971723348357067519](https://youcai922.github.io/99.src/img/diagram-11971723348357067519.png)



```
@startuml
scale 1024*768
[--> Tars: "They" provides data inside singularity

activate Tars
Tars -> Copper: sending data
activate Copper

Copper -> Copper: translate it to morse code
activate Murphy

Copper -> Murphy: send morse code through watch

Copper -> Tars: ask for next batch
deactivate Copper

Murphy -> Murphy: record and parse morse code

Murphy -->]: figured out the formula

deactivate Murphy
deactivate Tars
@enduml
```

效果展示：

![img](https://pic2.zhimg.com/80/ba12461d4887c272ac783475d3edd465_720w.jpg)

内容解释：

- scale代表图形大小
- [-->表示传入到当前序列图的消息，-->]表示传出当前序列图的消息

--------------



#### Activity diagram

```
@startuml
start

if (exec Lazarus?) then (yes)
    :find a livable planet;
    :save **human beings**;
else (no)
    :keep adapting,
    __keep farming__ and <font color=red>keep dying</font>;
endif

stop
@enduml
```

效果展示：

![diagram-3512339533571661316](https://youcai922.github.io/99.src/img/diagram-3512339533571661316.png)

内容解释：

- strat表示开始图标，stop表示结束
- if到endif代表条件判断
- 注意分号



```
@startuml

|Romilly|
start
repeat
    :record the data from black hole;
    :keep waiting;
repeat while (Copper & Brand are not back?)

|#AntiqueWhite|Copper|

:enter the Endurance;

while (has more video tapes?)
    :watch it;
    :cry;
endwhile

end
@enduml
```

效果展示：

![diagram-13852912287513279042](https://youcai922.github.io/99.src/img/diagram-13852912287513279042.png)

内容解释：

- repeat while (Copper & Brand are not back?)和while (has more video tapes?)是两种不同的循环方式
- ||声明泳道，|#AntiqueWhite|是泳道的颜色



```
@startuml
start
:first planet: Miller;

fork
    :Romilly: stay in the Endurance;
fork again
    :Copper et al: go to planet Miller;
    :giant wave comes;
    fork
        :Copper found wave, but helpless;
    fork again
        :Brand is racing against the wave;
    fork again
        :Doyle wait for Brand;
        :Doyle died;
        kill
    endfork
    :they finally left the planet;
endfork
@enduml
```

内容解释：

- fork，fork again，endfork 用来描述并发线程

- kill 终结一个线程，plantuml的例子中使用 detach，经测试，detach 不可用

![img](https://pic2.zhimg.com/80/52b2e16ea81348e32157f0556c59e30d_720w.jpg)

```
@startuml
scale 2
:Ready;
:next(o)|
:Receiving;
split
 :nak(i)<
 :ack(o)>
split again
 :ack(i)<
 :next(o)
 on several line|
 :i := i + 1]
 :ack(o)>
split again
 :err(i)<
 :nak(o)>
split again
 :foo/
split again
 :i > 5}
stop
end split
:finish;
@enduml
```

生成效果：

![img](https://pic4.zhimg.com/80/fa3200e6b6ed142fd77e2b924a2f1dab_720w.jpg)