##   微服务架构与实战

###  OAuth2
*	产生背景
	*	开放系统间授权
		*	联合登陆
		*	开放api平台
	*	现代微服务安全
*	定义与原理
	*	最简向导 -- https://medium.com/@darutk/the-simplest-guide-to-oauth-2-0-8c71bd9a15bb
	*	什么是oauth2.0
		*	用户rest/apis的代理授权框架
		*	基于令牌token的授权，在无需暴露用户密码的情况下，使应用可以访问用户数据
			*	第三方登录
		*	解耦认证与授权
	*	主要角色
		*	资源拥有者--用户，微信账号主人
		*	应用程序 -- 第三方程序,需要访问用户资源的应用
		*	授权服务器 -- 认证后，颁发令牌
		*	资源服务器 -- web站点，维护用户的数据
	

### OAuth 2.0的运行流程  --http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html
*	应用场景
	*	用户我 【资源拥有者】  ===使用===》 云打印网站【第三方应用程序，客户端】 ===打印==》 我的google照片【资源服务器】
	*	主要问题：
		*	告诉账号密码给第三方程序不安全，万一程序被破解，密码遭泄露
		*	第三方直接通过账号密码登录可以获取用户的所有资料的权限，但是实际上只需要访问照片的权限
	*	需求：不告诉第三方程序密码，限定访问权限的资源访问授权，让"客户端"安全可控地获取"用户"的授权，与"服务商提供商"进行互动。
*	RFC 6749流程图
	*	![](http://www.ruanyifeng.com/blogimg/asset/2014/bg2014051203.png)
	*	step 1 :  用户打开第三方应用程序云打印，云打印要求用户给与授权，访问用户的google照片
	*	step 2 :  用户同意给与客户端授权
	*	step 3 :  客户端使用step2的授权，向google的认证服务器申请令牌
	*	stpe 4 :  认证服务器对客户端进行认证，验证完成后，发放令牌（可指定权限）
	*	step 5 :  客户端使用令牌，向google照片服务器申请获取照片请求
	*	step 6 :  google资源服务器确认令牌后，返回请求的资源信息
*	客户端的授权模式 -- http://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html
	*	授权码模式（authorization code）
		*	是功能最完整、流程最严密的授权模式。
		*	指的是第三方应用先申请一个授权码，然后再用该码获取令牌
		*	步骤：
			*	第一步，云打印提供一个google授权服务器的地址
				*	云打印 嵌入 的google示例： `https://google.com/oauth/authorize?response_type=code&client_id=CLIENT_ID&redirect_uri=https://yundayin.com/callback&scope=read`	
  					*	response_type参数表示要求返回授权码（code）	
  					*	client_id参数让 google 知道是谁在请求 (通常google认证服务对接时，google分配)
  					*	redirect_uri参数是 google 接受或拒绝请求后的跳转网址
  					*	scope参数表示要求的授权范围
  			*	第二步，用户跳转后，google 网站会要求用户登录，然后询问是否同意给予 云打印 网站授权。用户表示同意，这时 google 网站就会跳回redirect_uri参数指定的网址。跳转时，会传回一个授权码
  				*	https://yundayin.com/callback?code=AUTHORIZATION_CODE
  					*	code为授权码
  			*	第三步，云打印 网站拿到授权码以后，就可以在后端，向 google认证服务器 网站请求令牌
  				*	https://b.com/oauth/token?client_id=CLIENT_ID&client_secret=CLIENT_SECRET&grant_type=authorization_code&code=AUTHORIZATION_CODE&redirect_uri=CALLBACK_URL
  					*	client_id参数和client_secret参数用来让 google 确认 云打印 的身份（client_secret参数是保密的，因此只能在后端发请求）
  						*	client_id , secret 是 云打印 接入 google 授权服务时，google分配给云打印的 账号，密码
  			*	第四步，google 认证服务器网站收到请求以后，就会颁发令牌。具体做法是向redirect_uri指定的网址，发送一段 JSON 数据。
  				*	JSON 数据中，access_token字段就是令牌，云打印 网站在后端拿到了
  		*	说明：
  			*	第三方程序需要和 认证服务器接入时，需要确定第三方程序需要访问哪些api资源，什么权限，并且分配id和密码给第三方接入
	*	简化模式（implicit）
		*	不通过第三方应用程序的服务器，直接在浏览器中向认证服务器申请令牌，跳过了"授权码"这个步骤，因此得名。所有步骤在浏览器中完成，令牌对访问者是可见的，且客户端不需要认证。
		*	步骤：
			*	第一步，A 网站提供一个链接，要求用户跳转到 B 网站，授权用户数据给 A 网站使用。
				*		https://b.com/oauth/authorize?response_type=token&client_id=CLIENT_ID&redirect_uri=CALLBACK_URL&scope=read
					*	response_type参数为token，表示要求直接返回令牌。
			*	第二步，用户跳转到 B 网站，登录后同意给予 A 网站授权。这时，B 网站就会跳回redirect_uri参数指定的跳转网址，并且把令牌作为 URL 参数，传给 A 网站。
				*	https://a.com/callback#token=ACCESS_TOKEN
					*	令牌的位置是 URL 锚点（fragment），而不是查询字符串（querystring），这是因为 OAuth 2.0 允许跳转网址是 HTTP 协议，因此存在"中间人攻击"的风险，而浏览器跳转时，锚点不会发到服务器，就减少了泄漏令牌的风险。	
		*	这种方式把令牌直接传给前端，是很不安全的。
	*	密码模式（resource owner password credentials）
		*	如果你高度信任某个应用，RFC 6749 也允许用户把用户名和密码，直接告诉该应用。
		*	步骤：
			*	第一步，A 网站要求用户提供 B 网站的用户名和密码。拿到以后，A 就直接向 B 请求令牌。
			*	第二步，B 网站验证身份通过后，直接给出令牌。注意，这时不需要跳转，而是把令牌放在 JSON 数据里面，作为 HTTP 回应，A 因此拿到令牌。
	*	客户端模式（client credentials）
		*	适用于没有前端的命令行应用，即在命令行下请求令牌。
		*	步骤：
			*	第一步，A 应用在命令行向 B 发出请求。
				*	https://oauth.b.com/token?grant_type=client_credentials&client_id=CLIENT_ID&client_secret=CLIENT_SECRET
			*	第二步，B 网站验证通过以后，直接返回令牌。
		*	这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。

#### 更新令牌
*	B 网站颁发令牌的时候，一次性颁发两个令牌，一个用于获取数据，另一个用于获取新的令牌（refresh token 字段）。令牌到期前，用户使用 refresh token 发一个请求，去更新令牌。
	*	https://b.com/oauth/token?grant_type=refresh_token&client_id=CLIENT_ID&client_secret=CLIENT_SECRET&refresh_token=REFRESH_TOKEN

#### 案例   http://www.ruanyifeng.com/blog/2019/04/github-oauth.html

#### 代码
##### spring 实现： spring security + oauth2  -- https://projects.spring.io/spring-security-oauth/docs/oauth2.html
*	spring oauth 2 两种主要角色：
	*	Authorization Service 授权服务 ： 通过spring mvc 的 controller 来处理 token 申请请求
		*	AuthorizationEndpoint ： 授权服务controller , 请求路径/oauth/authorize
		*	TokenEndpoint : token 申请cotroller , 请求路径： /oauth/token
	*	Resource Service 资源服务
		*	OAuth2AuthenticationProcessingFilter ： 
*	token store
	*	JwtTokenStore
		*	JwtAccessTokenConverter：TokenEnhancer的子类,帮助程序在JWT编码的令牌值和OAuth身份验证信息之间进行转换（在两个方向上），同时充当TokenEnhancer授予令牌的时间。
		*	TokenEnhancer：在AuthorizationServerTokenServices 实现存储访问令牌之前增强访问令牌的策略。