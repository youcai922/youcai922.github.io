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
'@startuml 声明一个图形的起始
participant Tars
actor Copper
database Murphy
'participant、actor、database代表不同的角色类型（还有control和entity）
Tars -> Copper
Copper --> Murphy: morse code
'两个角色之间的消息 ， ->为实现，-->为虚线
'点状箭头:-[#black,dotted]    条状箭头:-[#green,dashed]   加粗箭头:-[#gray,bold]
'消息后加：可以对消息添加说明
@enduml
'@enduml声明一个对图形的结束
```

![diagram-2524278456546615833](https://youcai922.github.io/99.src/img/diagram-2524278456546615833.png)

角色种类展示

![diagram-6776278217798399884](https://youcai922.github.io/99.src/img/diagram-6776278217798399884.png)

另外plantUML也可以结合html提高图形的美化

```
@startuml
actor "公司\n<b>老板</b>" as copper #0091ff
actor "公司\n员工" as murphy #red

copper -[#orange]> murphy: 管理和<font color=red>关爱</font>
'[#blue][#orange][#black][#green][#gray]
@enduml
```

效果展示：

![diagram-11971723348357067519](https://youcai922.github.io/99.src/img/diagram-11971723348357067519.png)



```
@startuml
scale 1024*768
'scale代表图形大小
[--> Tars: "They" provides data inside singularity
'[-->表示传入到当前序列图的消息，-->]表示传出当前序列图的消息

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



--------------

#### Activity diagram(活动图)

```
@startuml
start
'strat表示开始图标，stop表示结束
if (exec Lazarus?) then (yes)
'if到endif代表条件判断
    :find a livable planet;
    '注意分号
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

```
@startuml

|Romilly|
start
repeat
    :record the data from black hole;
    :keep waiting;
repeat while (Copper & Brand are not back?)
'repeat while (Copper & Brand are not back?)和while (has more video tapes?)是两种不同的循环方式

|#AntiqueWhite|Copper|
'||声明泳道，|#AntiqueWhite|是泳道的颜色

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

```
@startuml
|main|
start
if(Graphviz installed?) then (yes)
:process all \ndiagrams;
'泳道
|#green|Activie|
-> this is yes;
'注释
floating note left: this is a note
else(no)
:process only
__sequence__ and __activity__ diagrams]
'组合
partition Runing{
:暂存款打包;
:暂存款冲正;
'移除箭头 分离
detach
}
'箭头
-[#blue,dashed]->
note right
this note is on several
end note
endif
'颜色
'特殊领域语言（SDL）
#HotPink:ending of the process<
end
@enduml
```

![diagram-13752496942680108645](https://youcai922.github.io/99.src/img/diagram-13752496942680108645.png)



-------------------

### 类图：

```
@startuml
namespace controller #DDDDDD {
	class TestController {
		- QuestionServicen questionService;
		+ Question getQuestion(@DBExchange,int questionId);
	}
	TestController <--> spring.aop.DBExchangeHandler : Spring AOP
}

namespace service #DDDDDD {
	interface QuestionService {
		+ {abstract} Question getMessage(int questionId);
	}
	class DbImpl {
		+ Question getMessage(int questionId);
	}
	class RedisImpl {
		+ Question getMessage(int questionId);
	}
	DbImpl .up.|> QuestionService
	RedisImpl .up.|> QuestionService
}


namespace spring.bean #DDDDDD {
	class StateControl {
		- QuestionService questionService;
		- RedisImpl redisImpl;
		- DbImpl dbImpl;

		+ StateControl(QuestionService questionService);
		+ Question getMessage(int questionId);
	}
	StateControl --> service.QuestionService
	StateControl --> service.RedisImpl
	StateControl --> service.DbImpl
}

namespace spring.aop #DDDDDD {
	class DBExchangeHandler {
		- StateControl stateControl;
		- void before(JoinePoint joinePoint,@DBExchange dBExchange);
	}
	DBExchangeHandler -down-> spring.bean.StateControl : get really status for QuestionService
}

annotation DBExchange

DBExchange ... controller.TestController : label annotation
controller.TestController *--> service.QuestionService
@enduml
```

效果展示：

![img](https://img-blog.csdnimg.cn/img_convert/afc1cf496ba772091e03ca34a5d74895.png)