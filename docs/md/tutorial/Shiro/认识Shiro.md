
## Shiro

   记得做的第一个Web项目，系统认证授权这块就是基于Shiro实现的，当时也是第一次接触到这种类型的框架，同时是基于Spring做的集成，并且相关的配置都已经是定制好的，只需要我们根据项目情况对极少部分配置进行修改即可使用。对于其原理也只是知道个大概，具体实现细节可以说是一无所知。
      
   Shiro作为安全框架，相比其他同类型框架，在整体结构上更显得简单且更易于理解，通过对Shrio的学习，在理解里其实现原理后，对后面学习其他同类型的框架也会有很多帮助。

   > 现在各种框架盛行，为解决企业问题提供了各种优秀的解决方案，使开发者可以快速、简单、高效地搭建开发环境，用起来确实很方便，但是却产生了强烈的依赖性，现在有多少人脱离了框架还能开发出想要的功能？当然，框架我们还是要用的，但是随着学习的不断加深，在出现故障时我们及时定位问题，在当前业务需求不满足时，我们需要快速找到解决方法。

## 功能介绍

   以下是官网的一段话，直接介绍了Shiro能为我们做什么

   > Apache Shiro是一个功能强大且易于使用的Java安全框架，可执行身份验证、授权、加密和会话管理。

   ![特色](/assets/images/23_08_03/03_ShiroFeatures.png)

   主要功能：

+ Authentication：身份认证，证明用户是否具有访问系统的身份。

+ Authorization：访问控制，即确定“谁”可以访问系统的“什么”资源。
  
+ Session Management：会话管理，支持Web 或 EJB 应用程序的会话管理。
  
+ Cryptography：使用密码算法保持数据安全，主要是密码。

   其他功能：
     + Web Support:：Shiro 的 Web 支持 API 有助于轻松保护 Web 应用程序。

	 + Caching：缓存，确保能够都 Apache Shiro API 快速和高效地操作。
	
	 + Concurrency：Apache Shiro 支持多线程应用程序的并发特性。
	
	 + Testing：存在测试支持以帮助您编写单元和集成测试，并确保您的代码按预期执行。
	
	 + Run As：允许用户假设为另一个用户的身份的获取其功能权限，在管理场景中很有用。
	
	 + Remember Me：跨会话记住用户的身份，在登录时指明，省去下次登录的过程。

## 实现架构

   ![架构](/assets/images/23_08_03/03_ShiroArchitecture.png)

   + Subject( org.apache.shiro.subject.Subject) 当前与软件交互的实体（用户、第三方服务、cron任务等）的特定的安全性的“视图”。

   + SecurityManager ( org.apache.shiro.mgt.SecurityManager ) SecurityManager是Shiro架构的核心。它主要是一个综合的大对象，用于协调其托管组件以确保它们顺利协同工作。它还管理Shiro对每个应用程序用户的视图，因此它知道如何为每个用户执行安全操作。

   + Authenticator ( org.apache.shiro.authc.Authenticator ) 该Authenticator组件负责执行和响应用户的身份验证过程。当用户尝试登录时，该逻辑由Authenticator实现并执行。Authenticator通过用户配置的多个Realms，按照一定的策略，验证提供的用户是否满足用户身份。

     - AuthenticationStrategy ( org.apache.shiro.authc.pam.AuthenticationStrategy ) 当配置了多个Realm时，AuthenticationStrategy将协调调用Realm来确定用户身份验证最终成功或失败（比如有一个验证成功或每个都要验证成功）。

   + Authorizer ( org.apache.shiro.authz.Authorizer ) 是一个应用访问控制的组件。它是决定用户是否被允许执行某项操作。与Authenticator一样，Authorizer能够协调执行多个Realm来获取用户的访问角色和权限信息。

   + SessionManager ( org.apache.shiro.session.mgt.SessionManager ) 负责创建和管理用户Session的生命周期。即使没有可用的Web容器，Shiro能够在任何环境中本地管理用户会话。默认情况下是基于Web容器实现会话机制，非Web环境则可以通过内置的会话管理来提供相同的功能。可以通过SessionDAO实现会话的持久化。

     - SessionDAO ( org.apache.shiro.session.mgt.eis.SessionDAO ) 实现Session的持久化。

   + CacheManager ( org.apache.shiro.cache.CacheManager ) 因为Shiro经常访问后端数据源进行身份验证、授权和会话管理，通过缓存可以提高性能。

   + Cryptography( org.apache.shiro.crypto.\* ) 密码学是企业安全框架的扩展。Shiro的crypto包含易于使用和理解的密码、哈希（又名摘要）和不同编解码器实现的实现。Shiro的加密API简化了复杂的Java机制，使密码加密更容易。

   + Realm ( org.apache.shiro.realm.Realm ) Realm是Shiro和应用程序安全数据之间的“桥梁”。当用户需要对身份进行认证或授权时，Shiro会通过程序中配置的一个或多个Realm来查找满足的数据。Shiro将根据需要与它们协调以进行身份​​验证和授权。

### 相关术语

   如果理解了官网里介绍的这些术语，那么在进行shiro配置时，需要配置哪些信息，，分别由什么作用，怎么配置应该也会有一个较好的认识。

   **Authentication**
     是验证主体身份的过程——本质上是证明某人确实是他们所说的那个人。当身份验证尝试成功时，应用程序可以相信该主题是应用程序所期望的。

   **Authorization**
	授权，也称为访问控制，是确定是否允许用户/主体做某事的过程。它通常通过检查和解释主题的角色和权限（见下文）然后允许或拒绝访问请求的资源或功能来完成。

   **Cipher**
	密码是一种用于执行加密或解密的算法。该算法通常依赖于称为密钥的一条信息。并且加密会根据密钥而有所不同，因此如果没有它，解密将非常困难。

	密码有不同的变体。块密码适用于通常具有固定大小的符号块，而流密码适用于连续的符号流。对称密码使用相同的密钥进行加密和解密，而非对称密码使用不同的密钥。如果非对称密码中的密钥不能从另一个密钥导出，则可以公开共享一个密钥，创建公钥/私钥对。

   **Credential**
	凭据是验证用户/主题身份的一条信息
	。在验证尝试验证提交它们的用户/主体实际上是关联用户期间，一个（或多个）凭证与主体一起提交。凭证通常是只有特定用户/主题才能知道的非常机密的东西，例如密码或 PGP 密钥或生物特征或类似机制。

	这个想法是，对于委托人，只有一个人会知道与该委托人“配对”的正确凭证。如果当前用户/主题提供了与存储在系统中的证书匹配的正确证书，那么系统可以假定并相信当前用户/主题确实是他们所说的那个人。信任程度随着更安全的凭证类型（例如生物特征签名 > 密码）而增加。

   **Cryptography**
	密码学是一种通过隐藏信息或将其转换为无意义的信息来保护信息免遭不受欢迎的访问的做法，这样其他人就无法阅读它。Shiro 专注于密码学的两个核心元素：使用公钥或私钥对电子邮件等数据进行加密的密码，以及对密码等数据进行不可逆加密的哈希（又名消息摘要）。

   **Hash**
	散列函数是将输入源（有时称为消息）单向、不可逆地转换为编码散列值（有时称为消息摘要）。它通常用于密码、数字指纹或具有底层字节数组的数据。

   **Permission**
	至少在 Shiro 的解释中，权限是描述应用程序中原始功能的语句，仅此而已。权限是安全策略中最低级别的结构。他们只定义应用程序可以做什么。他们没有描述“谁”能够执行这些操作。许可只是行为声明，仅此而已。

   **Principal**
	主体是应用程序用户（主题）的任何标识属性
	“识别属性”可以是任何对您的应用程序有意义的东西——用户名、姓氏、名字、社会安全号码、用户 ID 等等。就是这样——没什么疯狂的。

   **Realm**
	领域是可以访问特定于应用程序的安全数据（例如用户、角色和权限）的组件。它可以被认为是一个安全特定的 DAO（数据访问对象）。Realm 将这种特定于应用程序的数据转换为 Shiro 可以理解的格式，因此无论存在多少数据源或您的数据可能是多么特定于应用程序，Shiro 都可以反过来提供一个易于理解的主题编程 API。

	领域通常与数据源（例如关系数据库、LDAP 目录、文件系统或其他类似资源）具有一对一的关联。因此，Realm 接口的实现使用特定于数据源的 API 来发现授权数据（角色、权限等），例如 JDBC、文件 IO、Hibernate 或 JPA，或任何其他数据访问 API。

   **Role**
   角色的定义与交互的系统关联。在许多应用程序中，人们使用它来隐式定义安全策略充其量只是一个模糊的概念。Shiro 更喜欢将角色简单地解释为权限的命名集合。就是这样 - 聚合一个或多个权限声明的应用程序唯一名称。

	这是一个比许多应用程序使用的隐含定义更具体的定义。如果您选择让您的数据模型反映 Shiro 的假设，您会发现您将拥有更多控制安全策略的权力。

   **Session**
	会话是与在一段时间内与软件系统交互的单个用户/主题相关联的有状态数据上下文。当主体使用应用程序时，可以从会话中添加/读取/删除数据，并且应用程序可以稍后在必要时使用这些数据。当用户/主题退出应用程序或由于不活动而超时时，会话将终止。

	对于熟悉 HttpSession 的人来说，ShiroSession具有相同的用途，除了 Shiro 会话可以在任何环境中使用，即使没有可用的 Servlet 容器或 EJB 容器。

   **Subject**
	主题只是花哨的安全术语，基本上意味着应用程序用户的特定于安全的“视图”
	。一个主题并不总是需要反映一个人 - 它可以代表一个调用您的应用程序的外部进程，或者可能是一个在一段时间内间歇性执行某些事情的守护程序系统帐户（例如 cron 作业）。它基本上是对应用程序做某事的任何实体的表示。

### 关键模块

  ***EnvironmentLoaderListener***
	
  该类实现了ServletContextListener，因此我们在web.xml中配置\<listeners\>后即可启用，主要目的是为了构建Enviroment对象并保存到ServleteContext中。
```java
public class EnvironmentLoaderListener extends EnvironmentLoader implements ServletContextListener {

    public void contextInitialized(ServletContextEvent sce) {
        initEnvironment(sce.getServletContext());
    }

    public void contextDestroyed(ServletContextEvent sce) {
        destroyEnvironment(sce.getServletContext());
    }
}

```

  ***Environment***

  Environment是一个Shiro容器，主要目的是提供SecurityManager对象。
```java
public interface Environment {

    SecurityManager getSecurityManager();
}
```

  ***SecurityManager***

  SecurityManager是Shiro框架中最核心的类，用户的认证、鉴权都是基于它来完成的，包含认证、会话管理、RememberMe等功能，这也是开发者需要理解并配置较多的类。
```java
public interface SecurityManager extends Authenticator, Authorizer, SessionManager {

    Subject login(Subject subject, AuthenticationToken authenticationToken) throws AuthenticationException;

    void logout(Subject subject);

    Subject createSubject(SubjectContext context);

}
```

  ***Authenticator***

  实现了用户的身份认证逻辑，也就是密码校验，注意我们是通过对AuthenticationToken进行认证，在改Token中封装了用户提交的信息，开发者只需要根据这些信息构建AuthenticationInfo对象，密码的校验过程是有Shiro代理完成的。
```java
public interface Authenticator {

    public AuthenticationInfo authenticate(AuthenticationToken authenticationToken)
            throws AuthenticationException;
}
```

  ***Authorizer***

  实现了用户权限的鉴权，包括权限和角色，鉴权过程在整个系统的生命周期中是比较频繁的，我们应该配置CacheManager来提高系统效率，避免频繁的进行DAO操作。
```java
public interface Authorizer {

    boolean isPermitted(PrincipalCollection principals, String permission);

    boolean isPermittedAll(PrincipalCollection subjectPrincipal, String... permissions);

    void checkPermission(PrincipalCollection subjectPrincipal, String permission) throws AuthorizationException;

    boolean hasRole(PrincipalCollection subjectPrincipal, String roleIdentifier);

    boolean hasAllRoles(PrincipalCollection subjectPrincipal, Collection<String> roleIdentifiers);

    void checkRole(PrincipalCollection subjectPrincipal, String roleIdentifier) throws AuthorizationException;
	
	// ...
    
}
```

  ***SessionManager***

  会话管理主要是为了实现认证保持，即一次认证成功后基于Cookie/Session机制，使下次会话不需要重复认证。在集群环境中，如果通过容器来管理会话那样就没办法实现会话共享，我们可以通过一些分布式会话管理框架来实现。
```java
public interface SessionManager {

    Session start(SessionContext context);

    Session getSession(SessionKey key) throws SessionException;
}
```

  ***SubjectDAO***
  实现Subject对象的保存，比如将Subject保存的Session或者到数据库。一般情况下，我们通过 SecurityUtils.getSubject()获取Subject对象，这里没有提供get方法。

```java
public interface SubjectDAO {

    Subject save(Subject subject);

    void delete(Subject subject);
}
```

  ***RememberMeManager***

  记住我，实现用户在会话超时后，不需要重新登录即可进入系统，默认是基于Cookie实现，主要将用户信息写入到Cookie中，并设置一个有效期，比如14天。
```java
public interface RememberMeManager {

    PrincipalCollection getRememberedPrincipals(SubjectContext subjectContext);

    void forgetIdentity(SubjectContext subjectContext);

    void onSuccessfulLogin(Subject subject, AuthenticationToken token, AuthenticationInfo info);

    void onFailedLogin(Subject subject, AuthenticationToken token, AuthenticationException ae);

    void onLogout(Subject subject);
}
```

  ***CacheManager***

  缓存，上面说到在鉴权时通过缓存提高API效率，还可以缓存Session实现Session快速存取。
```java
public interface CacheManager {
    <K, V> Cache<K, V> getCache(String var1) throws CacheException;
}
```

  ***Realm***

  Realm是开发者经常需要重写的，也是Shiro的扩展点，我们可以根据需要配置多个Realm，这样可以实现多因子认证，比如用户名、邮箱、指纹等。一般可以通过继承AuthenticatingRealm重写抽象方法实现。
```java
public interface Realm {

    String getName();

    boolean supports(AuthenticationToken token);

    AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException;

}
```

  ***Subject***

  Subject也是Shiro中非常重要的对象，在系统中，我们的操作基本都是调用其内部方法，最终由SecurityManager代理完成。比如常见的登录、登出、是否有角色、是否有权限等。
```java
public interface Subject {

	Object getPrincipal();
	
	PrincipalCollection getPrincipals();
	
	void login(AuthenticationToken token) throws AuthenticationException;

   // ...
}
```

  ***AuthenticationToken***

  用户提交的请求数据我们一般会进行封装，因为除了用户名密码，可能还会有一些其他信息，比如动态码、RememberMe标识，或者基于令牌的认证。
```java
public interface AuthenticationToken extends Serializable {

    Object getPrincipal();

    Object getCredentials();

}
```

  ***SubjectContext***

  Subject上下文，保存了Subject相关的数据，比如Subject、SecurityManager、Session等，在Subject间相互隔离。
```java
public interface SubjectContext extends Map<String, Object> {

    Subject getSubject();

    PrincipalCollection getPrincipals();

    PrincipalCollection resolvePrincipals();

    Session getSession();

    Session resolveSession();

    boolean isAuthenticated();

    boolean isSessionCreationEnabled();

    boolean resolveAuthenticated();

    AuthenticationInfo getAuthenticationInfo();

    AuthenticationToken getAuthenticationToken();

}
```

  ***ShiroFilter***

  Shiro过滤器链，前面一直提到基于Web的认证基本都是通过Filter实现，在一个Web项目中会有很多个Filter，而且会按照配置顺序执行，隐藏Shiro的过滤器会定义成链的模式，同时也是为了方便配置。开发者会根据业务需求配置过滤规则，最终都会由FilterChainResolver解析成过滤器链。
```java
public class ShiroFilter extends AbstractShiroFilter {

    @Override
    public void init() throws Exception {
        WebEnvironment env = WebUtils.getRequiredWebEnvironment(getServletContext());

        setSecurityManager(env.getWebSecurityManager());

        FilterChainResolver resolver = env.getFilterChainResolver();
        if (resolver != null) {
            setFilterChainResolver(resolver);
        }
    }
}
```

  ***Filter***

  在DefaultFilter中，我们可以看到这样的定义，主要是为过滤器提供了别名方便开发者进行配置。
```java
    anon(AnonymousFilter.class),
    authc(FormAuthenticationFilter.class),
    authcBasic(BasicHttpAuthenticationFilter.class),
    authcBearer(BearerHttpAuthenticationFilter.class),
    logout(LogoutFilter.class),
    noSessionCreation(NoSessionCreationFilter.class),
    perms(PermissionsAuthorizationFilter.class),
    port(PortFilter.class),
    rest(HttpMethodPermissionFilter.class),
    roles(RolesAuthorizationFilter.class),
    ssl(SslFilter.class),
    user(UserFilter.class),
    invalidRequest(InvalidRequestFilter.class);
```

  > 一般配置如下：
  /login.html = anon;        //表示不需认证也可访问
  /login.html = authc;       //表示表单POST提交到/login.html请求时会根据提交的参数进行身份认证
  /system     = authcBasic;  //根据请求头Authorization:BASIC Base64(用户名:密码)的形式完成认证
  /system     = authcBearer; //根据请求头Authorization:Bearer Token的形式完成认证
  /logout     = logout;      //退出登录
  /authz      = perms[update,*:add,*:delete:*]; //表示具有操作权限可访问，支持'*'通配符
  /index.html = port[8080];  //表示8080请求端口才能访问
  //权限字符串由 type:action:instance 组成，在rest服务中，通过权限与请求方法构建权限字符串[perm:mapping]，请求方法与action对应关系如下：
  // GET|HEAD|OPTIONS|TRACE  -> read
  // PUT                     -> update
  // POST|MKCOL              -> create
  /           = rest

## 认证流程

   ![认证流程](/assets/images/23_08_03/03_ShiroAuthenticationSequence.png)

   第1步：应用程序代码调用该Subject.login方法，传入代表最终用户的主体和凭据的构造AuthenticationToken实例。

   第2步：Subject实例，通常是一个类通过调用DelegatingSubject委托给应用程序，实际的身份验证工作从这里开始。SecurityManagersecurityManager.login(token)

   第3步：SecurityManager作为基本的综合组件，接收令牌并通过调用简单地委托给其内部Authenticator实例authenticator.authenticate(token)。一般是ModularRealmAuthenticator实例，它支持在身份验证期间协调一个或多个Realm实例。ModularRealmAuthenticator本质上为 Apache Shiro提供了PAM样式的范例（其中每个Realm 都是 PAM 术语中的“模块”）。

   第4步：如果Realm为应用程序配置了多个，ModularRealmAuthenticator实例将Realm使用其配置的 启动多重身份验证尝试策略AuthenticationStrategy。Realms在调用 进行身份验证之前、期间和之后，AuthenticationStrategy将根据每个Realm的执行结果做出最终认证的决定。

   第5步：根据每一个配置Realm，判断supports是否提交AuthenticationToken。如果匹配则以token为参数调用Realm的getAuthenticationInfo方法。该方法代表了Realm的一次身份验证尝试。

## 结束语

  关于shiro，很多都是介绍框架关键组件，比如SecurityManager，大部分功能都是围绕该类展开，后面会通过实战，一步一步了解以上提到的各个模块在项目运行期间如何实现最终的用户身份认证以及对应的访问权限控制，以及细化到其具体的实现流程与模块间的时序调用关系。

## 参考

 [https://shiro.apache.org/introduction.html](https://shiro.apache.org/introduction.html)

 [https://www.infoq.com/articles/apache-shiro/](https://www.infoq.com/articles/apache-shiro/)