

##数据绑定

    单向绑定--{{hero.name}}

    双向绑定--[(ngModel)]="hero.name"

    内建directive  *ngFor遍历元素

    Promises --异步处理模式

    如果 then() 返回另一个 promise 那种强大。这种情况下，下一个 then() 会在 promise 完结的时候被执行。这种模式可以用到把 HTTP 请求串上面，比如说(当一个请求依赖于前一个请求的结果的时候):

    directive -- 插件，指令，执行单元

##主要组件

###Module--模块

    angular 应用是模块化的，由许多模块组成

    每个模块都exports一些内容，比如class    ，函数，值 给其他模块import

    推荐app由一些列模块组成，每个模块导出一个内容


###Component--组件--

    class （model）+ telmplate（view） =》通过class的properties 和metohd与view交互

    通过组件的lifecycle控制，初始化等操作

    提供属性和方法绑定数据，业务由service实现


###Template--模板

    html代码，描述如何展示组件

    模板中组件的定义，形成父子关系


###Metadata--元数据--@Component({。。。}）

    告诉angular如何处理这个component==》

    1 selector:    'hero-list' -- 告诉angular遇到这个标签的时候创建并插入这个component处理的view

    2.templateUrl: 'app/hero-list.component.html',模板是什么，如何渲染的

    3.directives:  [HeroDetailComponent]--一组components或者directives，当前component所需的，模板中用到的组件标签只有在这里定义过才会 被处理

    4.providers:   [HeroService]---需要注入哪些类



###Data Binding

    {{value}}  ==》单向从component绑定到DOM

    [property] ='value'  ==> 单向从component绑定到DOM

    (event) ='handler'   ==> 单向从DOM绑定到component

    [(ng-model)]='propertiy' ==>双向绑定DOM到component


    <div>{{hero.name}}</div>
    <hero-detail [hero]="selectedHero"></hero-detail>
    <div (click)="selectHero(hero)"></div>

    <input [(ngModel)]="hero.name">


###Directive--指令

    angular 模板支持语法动态处理，可以根据指定的directive去动态生成DOM元素

    定义 == class + @Directive 注解

    @component是Directive的一种，继承了template-oriented功能

    有三种Directive--出现在 元素 属性

    1 components -- 在模板中最常用的directive


    2structural Directive-- 改变布局，通过增加，删除，替换DOM的元素

<div *ngFor="let hero of heroes"></div>
<hero-detail *ngIf="selectedHero"></hero-detail>

       3 Attribute directives --改变已经存在的元素的行为，在template中和普通的html属性一样

<input [(ngModel)]="hero.name">


###Service -- 工具类，通用服务，业务逻辑

    logging service

    data service

    message bus

    tax calculator

    application configuration


###Dependency Injection

    为compoent需要的service提供注入方式

    通过class的constructor方式注入

    Injector对象创建并缓存了所有依赖的Service（单例），创建完成后，调用compnent的consturctor注入

    Injector通过注册的provider知道需要哪些service注入，

    在root component定义的provider可以被所有子component使用


###Router

    <base href="/"> --在index.html的head之后，告诉router如何构建navigationURL





