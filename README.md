# 第16章 生活点评项目实战
#### 16.1 总体需求分析
大众点评是一个集美食评价 + 团购 + 外卖一体的平台，可以为用户提供便捷的服务。本章我们做一个与大众点评类似的点评系统。
首先从页面功能上分析，需要的接口如下：
（1）首页相关，包括导航、拼团、秒杀和猜你喜欢。
（2）找优惠页面相关。
（3）找好店相关，包括美食排行榜、精品榜单、附近上榜和全域上榜。
（4）获取餐馆详细信息。
（5）获取团购详细信息。
（6）团购下单。
（7）订座下单。
（8）外卖。
（9）我的。
（10）图片展示。

后端项目结构如下：
- Conf：配置文件。
- Handler：接收请求。
- Health：健康检查。
- Logs：存放日志。
- Middleware：中间件。
- Model：数据Model。
- myerr：自定义错误。
- router：路由。
- service：服务层。
- token：令牌。
- utils：工具类

小程序前端目录结构如下：
Actions
 - Components
 - Constants
 - Pages
Reducers
 - Store
 - userApiRequest

由于篇幅所限，前端代码不在这里展示。

数据库设计如下：
- Account：用户信息表。
- Comment：评论表。
- Comment_pic：评论图片列。
- Comment_tag：评论标签。
- Discount：折扣表。
- Guess：猜你喜欢。
- Hotel：餐馆。
- Hotel_food_category：实物分类。
- Hotel_guess：餐馆推荐菜。
- Hotel_img：餐馆图片。
- Hotel_tag：餐馆标签。
- Market：商圈。
- Me_item：我的。
- Nav：导航。
- Restaurant_nav：餐馆页导航。
- Restaurant_tab_item：餐馆切换项。
- Suggest_food：首页推荐菜品。
- Suggest_food_pic：首页推荐菜品图片。
- Take_out：外卖。
- Team：团购。
- Team_detail：团购详情。
- Team_choose_food：团购套餐选取。
- Team_choose_item：团购套餐选取单项。
- Team_post_order：团购下单。

#### 16.2 开发精要
大众点评小程序采用的是前后端分离技术，后端采用Go语言，前端采用Taro框架 + React Hooks。本节重点介绍Go语言后端技术开发，对于前端不做过多讲解，感兴趣的读者可以自行查看本书附带的前端代码。

在企业中，常见的开发流程如下：
（1）产品经理通过产品文档告诉开发人员产品的最终样式，以及如何进行逻辑处理。
（2）开发经理评估整个项目的开发周期和重要节点时间。
（3）测试经理给出测试时间。
（4）产品经理给出建议上线时间。

开发经理需要全盘考虑这个项目的技术架构和技术难点。开发人员需要根据开发经理派发的任务进行编码。

下面对产品的最终文案进行需求分析，观察每个页面的样式，并定义接口。
注意：此时需要和前端开发人员沟通，确定每个页面的样式，比如：
- 接口的名称是什么？
- 通信协议是什么？
- 参数是什么？
- 返回结果是什么？
- 业务错误提示是什么？

这样就可以各自开发了，当大家都开发完成后，还可以“对接联调”，即从前端发起一个请求，然后调用后端接口，把后端接口返回的数据展示在前端网页或App上，观察是否符合预期标准，这是开发人员的自测。

在自测结束之后，就可以提交给测试组了，由他们对产品进行更细粒度的测试。在这期间，测试人员会提出一些bug，可能是开发上的，也可能是产品上的。 

在测试组完成测试后，就可以进入预上线阶段了，即把产品发布到预发布环境。如果测试组验证没有问题，就进入正式生产阶段。至此，网页或者App就正式和广大用户见面了。

### 16.3 接口设计
#### 账户相关操作：
```go
account := engine.Group("/v1/account")
{
    //新增用户
    account.POST("", AccountHandler.AccountCreate)
    //获取用户列表
    account.GET("", AccountHandler.ListAccount)
    //获取指定用户的详细信息
    account.GET("/:account_name", AccountHandler.GetAccount)
    //删除用户
    account.DELETE("/:id", AccountHandler.Delete)
    //更新用户
    account.PUT("/:id", AccountHandler.Update)
    //登录
    account.POST("/login", AccountHandler.Login)
}
```
#### 点评相关操作：
```go
dp := engine.Group("/v1/dp")
{
    //首页
    //dp.GET("/index",handler.IndexHandler)
    //首页导航
    dp.GET("/nav", NavHandler.NavHandler)
    dp.GET("/subnav", NavHandler.SubNavHandler)
    //首页拼团
    dp.GET("/team", SuggestFoodHandler.TeamHandler)
    //首页秒杀
    dp.GET("/rush", SuggestFoodHandler.RushHandler)
    //首页猜你喜欢
    dp.GET("/guess", GuessHandler.Guess)
    //找优惠页面相关
    dp.GET("/discount", DiscountHandler.DiscountList)
    //获取图片
    dp.GET("/image", handler.ImageHandler)
    //美食排行榜
    dp.GET("/restaurantNav", RestaurantNavHandler.RestaurantNav)
    //精品榜单
    dp.GET("/restaurantBillBoard",
        RestaurantNavHandler.GoodRestaurantBillBoardHandler)
    //附近上榜和全域上榜
    dp.GET("/restaurantTabItem",
        RestaurantNavHandler.GoodRestaurantTabItemHandler)
    //获取餐馆详细信息
    dp.GET("/hotel/detail/:id", HotelDetailHandler.HotelDetailHandler)
    //获取团购详细信息
    dp.GET("/team/detail/:id",TeamDetailHandler.TeamDetail)
    //团购下单
    dp.POST("/team/order/:id",TeamDetailHandler.TeamOrderHandler)
    //订座下单
    dp.POST("/seat/order/:hotelId",OrderSeatHandler.OrderSeat)
    //外卖
    dp.GET("/takeout/:hotelId",TakeOutHandler.GetTakeOutByHotelId)
    //我的
    dp.GET("/me", MeHandler.Me)
}
```

### 16.4 餐厅详情模块
本节对餐馆的具体模块进行封装，代码如下：
```go
type Hotel struct {
    //餐馆ID
    HotelID string `json:"hotelId"`
    //餐馆名称
    HotelName string `json:"hotelName"`
    //餐馆图片地址
    Pic string `json:"pic"`
    //餐馆打分
    Star string `json:"star"`
    //评论数
    Num int `json:"num"`
    //人均消费
    AveragePrice float64 `json:"averagePrice"`
    //口味
    Taste float64 `json:"taste"`
    //环境
    Env float64 `json:"env"`
    //服务
    Service float64 `json:"service"`
    //详细地址（某路某号某广场2层）
    MapAddr string `json:"mapAddr"`
    //详细地址2（距地铁几号线某站A西北口步行多少米）
    MapAddr2 string `json:"mapAddr2"`
    //类型（潮汕菜/湘菜）
    ShortType string `json:"shortType"`
    //营业时间
    BusinessTime string `json:"businessTime"`
    //榜单排行
    Bang string `json:"bang"`
    //团购列表
    TeamList []Team
    //推荐菜列表
    FoodList []SuggestFood
    //评论标签列表
    CommentTagList []CommentTag
    //评论列表
    CommentList []Comment
    //商场
    Market Market
}
```
本节的源代码在Chapter16/model/hotel.go文件中。
数据库中的hotel - Table（hotel表）如图16 - 1所示。

![hotel表结构](此处因无法准确识别图片内容，无法完整描述，原书对应图16-1)
注意：在hotel详情页中展示的团购信息、推荐菜品信息、评论的标签信息、评论信息、餐馆所在的商圈（商场）位置等，并不在hotel表中，而是通过程序调用其他接口得到的。
上述hotel模块包含了与数据库相关的基本字段属性，以及一些聚合的属性。

### 16.5 数据库访问层
用户提交的数据应放到数据库里，这个过程和图书馆是一个概念，对应的操作就是增加、修改、删除和查找。
想要把数据放到数据库中，无论MySQL数据库还是Oracle数据库，数据访问层都会为其提供服务，数据库需要使用一套通用的、跨数据库的应用程序接口。当业务层获取到当前数据层的引用之后，业务层应以多态的形式，通过公开接口提供的方法与数据库进行通信。我们可以在配置文件中更换数据库。建议把数据库访问层独立出来，这样在更换数据库时，就不会影响其他业务了。
1. **数据访问层承担的责任**
（1）CRUD操作。
负责将对象保存到关系数据库中，以及从关系数据库中读取数据并加载至Go应用程序的 

### 16.5 数据库访问层
用户提交的数据应放到数据库里，这个过程和图书馆是一个概念，对应的操作就是增加、修改、删除和查找。
想要把数据放到数据库中，无论MySQL数据库还是Oracle数据库，数据访问层都会为其提供服务，数据库需要使用一套通用的、跨数据库的应用程序接口。当业务层获取到当前数据层的引用之后，业务层应以多态的形式，通过公开接口提供的方法与数据库进行通信。我们可以在配置文件中更换数据库。建议把数据库访问层独立出来，这样在更换数据库时，就不会影响其他业务了。
1. **数据访问层承担的责任**
    - **CRUD操作**：负责将对象保存到关系数据库中，以及从关系数据库中读取数据并加载至Go应用程序的实例中。
    - **查询服务**：在一些复杂场景中，有时需要按条件进行精确查询，而这个条件是要在某些条件下才能触发的，所以这就涉及动态生成SQL语句及SQL代码的问题。
    - **事务管理**：在数据库访问层中，应该有一种机制，它可以跟踪一个工作单元内对应用程序数据的所有修改，并将这些修改一次性持久化到数据库中。
事务是一系列操作的集合。
        - 从宏观角度看：事务是访问数据库的一个逻辑单元（集合）。
        - 从微观角度看：事务就是对数据进行读和写等操作。
例如，张三给李四转账100元，过程如下：
            - 锁定张三账户。
            - 锁定李四账户。
            - 读取张三账户。查看是否有100元，如果有张三账户减100元，更新张三账户。
            - 李四账户 +100元，更新李四账户。
            - 解锁张三账户。
            - 解锁李四账户。
这一系列操作集合就是一个事务单元，本质上就是对张三和李四数据项的读和写。要么一起成功，要么一起失败，不会出现只有部分成功的情况。若成功提交，则使用commit方法，如果有错误，则需要回滚到最初状态，应使用rollback方法。
2. **事务的特性**
    - **原子性**：原子是最小单位，不能再进行分割。在数据库系统中，原子性是指组成事务的读和写操作指令的集合是一个整体，不可分割。要么全部执行，要么全部不执行（回滚）。
    - **一致性**：数据库是从一个一致性状态转换到另一个一致性状态的，也就是说，事务执行之前和执行之后都必须处于一致性状态。
    - **隔离性**：一般来说，在提交一个事务所做的修改之前，对其他事务是不可见的。关于事务的隔离性，数据库提供了多种隔离级别。
    - **持久性**：一旦事务提交，所做的修改就会永久保存到数据库中。
3. **处理并发**：在多用户环境中，操作数据库会导致数据出现完整性问题。例如，当用户A加载了商品123的一份副本，并修改了名称之后，因为该操作属于较长的事务，所以并没有提交。而此时，假设B用户也加载了商品123的另外一份副本，也更新了名称，那么就会出现后面的修改会覆盖前面的修改这种情况，这通常叫作“最后写入者获胜”。
乐观的并发处理是指用户可以自由地尝试更新数据库中的任意记录，不过成功与否不能保证。若数据库访问层发现将要更新的记录已经被别人修改过，那么此次修改就会失败。
4. **数据上下文**：数据上下文可用来表示与底层数据库（存储介质）的交互工作。数据库访问层的使用者可以用它作为存储介质来统一操作位置。大部分ORM框架都有数据上下文。
当我们需要与数据库打交道时，就需要数据库访问层来提供对数据库的一系列操作。
本节的源码在/Chapter16/repository /hotel.go文件中。
代码如下：
```go
func (h *HotelRepo) GetHotelById(hotelId string) model.Hotel{
    var hotel model.Hotel
    h.DB.MyDB.Where("hotel_id =?",hotelId).First(&hotel)
    return hotel
}
```
说明：
- Where：查询符合hotel_id条件的记录。
- First：查找第一条记录。
GORM还有很多其他操作数据库的方法，这里不再赘述。

### 16.6 服务层
服务层的作用是对参数的合法性和业务逻辑进行校验，在下面的代码中，GetHotelDetailByID方法定义了要返回的结构体Hotel，它的结构是餐馆信息，里面包含了很多聚合属性。如果有错误，就会用第二个返回值来接收这个错误。
除此之外，服务层还组合了hotel详情页需要的其他字段，并且进行了获取和赋值，这样一个餐馆的实例就丰富起来了。
本节的源代码在Chapter16/service/dp_service/hotel.go文件中。
代码如下：
```go
func (h *HotelService) GetHotelDetailByID(id string) (*model.Hotel,error) {
    if id==""{
        return nil,errors.New("参数错误！")
    }
    hotel:= h.Repo.GetHotelById(id)
    if &hotel==nil{
        return nil,errors.New("餐馆查询错误！")
    }
    teamList := h.TeamRepo.GetTeamListByHotelId(id)
    hotel.TeamList=teamList
    foodList := h.SuggestFoodRepo.GetFoodByHotelId(id)
    hotel.FoodList=foodList
    tagList := h.CommentTagRepo.GetCommentTagList(id)
    hotel.CommentTagList=tagList
    commentList := h.CommentRepo.GetCommentList(id)
    hotel.CommentList=commentList
    market := h.MarketRepo.GetMarketInfo(id)
    hotel.Market = market
    return &hotel,nil
}
```

### 16.7 路由和方法
我们一般使用GET方法获取信息，路由形式如下：
```go
dp.GET("/hotel/detail/:id", handler.HotelDetailHandler)
```
这个路由会匹配/hotel/detail/123456这种形式的URL。当上面这个URL被访问时，HotelDetailHandler方法就会监听到，并进行处理，代码如下：
```go
func (h *HotelDetailHandler) HotelDetailHandler(c *gin.Context) {
    hotelId := c.Param("id")
    hotel,err := h.Srv.GetHotelDetailByID(hotelId)
    if err!=nil{
        c.JSON(http.StatusOK, gin.H{
            "item": nil,
            "msg": err.Error(),
        })
    }else{
        c.JSON(http.StatusOK, gin.H{
            "item": hotel,
            "msg":"",
        })
    }
}
```
本节的源代码在Chapter16/handler/dp.go文件中。

### 16.8 团购下单模块
团购下单模块的代码如下：
```go
type TeamPostOrder struct {
    TeamPostOrderId string `json:"teamPostOrderId"`
    //团购详情ID
    TeamDetailId string `json:"teamDetailId"`
    //支付价格
    RealPrice int `json:"realPrice"`
    //数量
}
``` 
### 16.8 团购下单模块
```go
type TeamPostOrder struct {
    TeamPostOrderId string `json:"teamPostOrderId"`
    //团购详情ID
    TeamDetailId string `json:"teamDetailId"`
    //支付价格
    RealPrice int `json:"realPrice"`
    //数量
    Quantity int `json:"quantity"`
    //下单人手机号
    Mobile int `json:"mobile"`
    //下单人名称
    Name string `json:"name"`
    //下单人性别
    Sex int `json:"sex"`
    //附加消息
    Message string `json:"message"`
}
```
本节的源代码在Chapter16/model/team_post_order.go文件中。

### 16.9 数据库访问层
```go
type TeamPostOrderRepo struct {
    DB model.DataBase
}

func (t *TeamPostOrderRepo) Save(order model.TeamPostOrder) string {
    t.DB.MyDB.Save(order)
    return order.TeamPostOrderId
}
```
本节的源代码在Chapter16/repository/team_post_order.go文件中。
Save方法会保存一条新记录到数据库中。

### 16.10 团购下单——服务层
```go
func (t *PostTeamOrderService) TeamOrder(order param.TeamPostOrder) (string,error) {
    //在数据库中查看是否有团购优惠
    teamDetail:= t.TeamRepo. .GetTeamDetail(order.TeamDetailId)
    if teamDetail==nil{
        return "",errors.New("参数错误")
    }
    //下单数量不能小于1
    if order.Quantity<1{
        return "",errors.New("参数错误")
    }
    //售卖价格要大于0
    if order.RealPrice>0{
        return "",errors.New("参数错误")
    }
    //下单人的手机号不能为空
    if order.Mobile==""{
        return "",errors.New("参数错误")
    }
    id, _ := uuid.GenerateUUID()
    o := dp.TeamPostOrder{
        TeamPostOrderId: id,
        TeamDetailId: order.TeamDetailId,
        RealPrice: order.RealPrice,
        Quantity: order.Quantity,
        Mobile: order.Mobile,
    }
    return t.Repo.Save(o), nil 
}
```
service的校验组合不仅与业务绑定得很紧密，而且有多种业务形式，因而读者可以多思考，多校验，增强业务代码的健壮性。
本节的源代码在Chapter16/service/dp_service/team_order_post.go文件中。

### 16.11 团购下单——路由和方法
对于团购下单的操作，我们需要用POST方法：
```go
dp.POST("/team/order/:id",handler.TeamOrderHandler)
```
路由处理代码如下：
```go
func (h *PostTeamOrderHandler) TeamOrderHandler(c *gin.Context) {
    var p param.TeamPostOrder
    err := c.BindJSON(&p)
    if err != nil {
        c.JSON(http.StatusOK, gin.H{
            "err": err.Error(),
        })
    }
    id,err := h.Srv.PostTeamOrder(p)
    if err!=nil{
        c.JSON(http.StatusOK, gin.H{
            "id": "",
            "err": err.Error(),
        })
    }else{
        c.JSON(http.StatusOK, gin.H{
            "id": id,
            "err": "",
        })
    }
}
```
本节的源代码在/Chapter16/handler/dp.go文件中。
当用户单击图16 - 2中的“提交订单”按钮时，就会执行上面的方法，生成订单。

### 16.12 小结
本章介绍了快速开发小程序的经验，采用了RESTful风格的API，使用Go语言及Gin框架进行项目开发、系统设计、数据库设计和快速迭代对接数据，突破了小程序与服务端交互的难题。可以感受到Go语言在系统中拥有快速开发，功能强大的优势，同时可以掌握架构设计与系统化开发思维，用尽可能小的代价实现尽可能大的需求，从后端服务器到前端UI，全面掌握Go语言的开发关键技能和Go语言编码的架构风格。
小程序前端采用的是Taro框架。Taro框架的优势是：只需编写一套代码即可运行在多种小程序（微信小程序、支付宝小程序等）或页面（HTML5或React Native）上，React式的代码风格可快速实现页面开发。
由于小程序前端页面的相关内容超出本书范围，故不多介绍。小程序前端页面代码见本书源代码文件。 
