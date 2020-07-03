####   springboot + shrio + jwt

####  shrio

* shiro中用户认证和用户授权是分开的。用户认证（可以理解为登陆）叫Authentication，用户授权叫Authorization，
* Realm 是存储客户端安全数据如用户、角色、权限等的一个组件，可以理解为DAO。
* shiro先用filter拦截请求，然后调用realm获取用户的认证、授权信息。













####  参考

* https://blog.csdn.net/u010606397/article/details/104110093

  