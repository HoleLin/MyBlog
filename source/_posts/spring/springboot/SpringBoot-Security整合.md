---
title: SpringBoot-SpringBoot Security整合
date: 2021-08-16 15:50:20
index_img: /img/cover/Spring.jpg
cover: /img/cover/Spring.jpg
tags:
- SpringBoot Security
categories:
- SpringBoot
updated:
type:
comments:
description:
keywords:
top_img:
mathjax:
katex:
aside:
aplayer:
highlight_shrink:
---
### 参考文献
* [Spring Boot：整合Spring Security](https://www.cnblogs.com/xifengxiaoma/p/11106220.html)
* [SpringSecurity登录原理（源码级讲解）](https://www.jianshu.com/p/a65f883de0c1)

#### 依赖

```
    // springboot_version= '2.2.2.RELEASE'
    implementation "org.springframework.boot:spring-boot-starter:$springboot_version"
    implementation "org.springframework.boot:spring-boot-starter-web:$springboot_version"
    implementation "org.springframework.boot:spring-boot-starter-security:$springboot_version"
```

#### 配置

```kotlin
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
class WebSecurityConfig : WebSecurityConfigurerAdapter() {

    @Autowired
    private val userDetailsService: UserDetailsService? = null

    @Throws(Exception::class)
    override fun configure(auth: AuthenticationManagerBuilder) {
        // 使用自定义登录身份认证组件
        auth.authenticationProvider(JwtAuthenticationProvider(userDetailsService))

    }

    override fun configure(web: WebSecurity?) {
        super.configure(web)
    }

    override fun configure(http: HttpSecurity) {
        http.cors().and().csrf().disable()
            .authorizeRequests()
            // 跨域预检请求
            .antMatchers(HttpMethod.OPTIONS, "/**").permitAll()
            .antMatchers("/login").permitAll()
            .antMatchers("/user/login").permitAll()
            .antMatchers("/user/create-user").permitAll()
            .anyRequest().authenticated();
        // 退出登录处理器
        http.logout().logoutSuccessHandler(HttpStatusReturningLogoutSuccessHandler())
        // 开启登录认证过滤器
        http.addFilterBefore(JwtLoginFilter(authenticationManager()), UsernamePasswordAuthenticationFilter::class.java)
        // 访问控制登录状态检查器
        http.addFilterBefore(
            JwtAuthenticationFilter(authenticationManager()),
            UsernamePasswordAuthenticationFilter::class.java
        )

    }

    @Bean
    override fun authenticationManager(): AuthenticationManager {
        return super.authenticationManager()
    }

    @Bean
    fun passwordEncoder(): BCryptPasswordEncoder {
        return BCryptPasswordEncoder();
    }
}
```

#### 流程

```
AbstractAuthenticationProcessingFilter#doFilter
-->子类(UsernamePasswordAuthenticationFilter)的attemptAuthentication-->AuthenticationManager
-->子类（ProviderManager）
-->AuthenticationProvider#authenticate
-->AbstractUserDetailsAuthenticationProvider#retrieveUser
-->DaoAuthenticationProvider#retrieveUser
```

* **org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter#doFilter**

```java
	// org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter#doFilter
	public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {

		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;

    // 判断当前filter是否可以处理当前请求，若不行，则交给下一个filter去处理。
		if (!requiresAuthentication(request, response)) {
			chain.doFilter(request, response);

			return;
		}

		if (logger.isDebugEnabled()) {
			logger.debug("Request is to process authentication");
		}

		Authentication authResult;

		try {
      // 最终调用子类的attemptAuthentication方法
			authResult = attemptAuthentication(request, response);
			if (authResult == null) {
				// return immediately as subclass has indicated that it hasn't completed
				// authentication
				return;
			}
      // 最终认证成功后，会处理一些与session相关的方法（比如将认证信息存到session等操作）。
			sessionStrategy.onAuthentication(authResult, request, response);
		}
		catch (InternalAuthenticationServiceException failed) {
			logger.error(
					"An internal error occurred while trying to authenticate the user.",
					failed);
			unsuccessfulAuthentication(request, response, failed);

			return;
		}
		catch (AuthenticationException failed) {
			// Authentication failed
			unsuccessfulAuthentication(request, response, failed);

			return;
		}

		// Authentication success
		if (continueChainBeforeSuccessfulAuthentication) {
			chain.doFilter(request, response);
		}
		/*
     * 最终认证成功后的相关回调方法，主要将当前的认证信息放到SecurityContextHolder中
     * 并调用成功处理器做相应的操作。
     */
		successfulAuthentication(request, response, chain, authResult);
	}
```

* **org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter#attemptAuthentication**	

```java
  // org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter#attemptAuthentication	
  public Authentication attemptAuthentication(HttpServletRequest request,
			HttpServletResponse response) throws AuthenticationException {
    // 认证请求的方式必须为POST
		if (postOnly && !request.getMethod().equals("POST")) {
			throw new AuthenticationServiceException(
					"Authentication method not supported: " + request.getMethod());
		}

		String username = obtainUsername(request);
		String password = obtainPassword(request);

		if (username == null) {
			username = "";
		}

		if (password == null) {
			password = "";
		}

		username = username.trim();

		UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(
				username, password);

		// Allow subclasses to set the "details" property
		setDetails(request, authRequest);

		return this.getAuthenticationManager().authenticate(authRequest);
	}
```

* **org.springframework.security.authentication.ProviderManager#authenticate**

```java
	// org.springframework.security.authentication.ProviderManager#authenticate
	
	/**
	 * Attempts to authenticate the passed {@link Authentication} object.
	 * <p>
	 * The list of {@link AuthenticationProvider}s will be successively tried until an
	 * <code>AuthenticationProvider</code> indicates it is capable of authenticating the
	 * type of <code>Authentication</code> object passed. Authentication will then be
	 * attempted with that <code>AuthenticationProvider</code>.
	 * <p>
	 * If more than one <code>AuthenticationProvider</code> supports the passed
	 * <code>Authentication</code> object, the first one able to successfully
	 * authenticate the <code>Authentication</code> object determines the
	 * <code>result</code>, overriding any possible <code>AuthenticationException</code>
	 * thrown by earlier supporting <code>AuthenticationProvider</code>s.
	 * On successful authentication, no subsequent <code>AuthenticationProvider</code>s
	 * will be tried.
	 * If authentication was not successful by any supporting
	 * <code>AuthenticationProvider</code> the last thrown
	 * <code>AuthenticationException</code> will be rethrown.
	 *
	 * @param authentication the authentication request object.
	 *
	 * @return a fully authenticated object including credentials.
	 *
	 * @throws AuthenticationException if authentication fails.
	 */
	public Authentication authenticate(Authentication authentication)
			throws AuthenticationException {
		Class<? extends Authentication> toTest = authentication.getClass();
		AuthenticationException lastException = null;
		AuthenticationException parentException = null;
		Authentication result = null;
		Authentication parentResult = null;
		boolean debug = logger.isDebugEnabled();

		for (AuthenticationProvider provider : getProviders()) {
			if (!provider.supports(toTest)) {
				continue;
			}

			if (debug) {
				logger.debug("Authentication attempt using "
						+ provider.getClass().getName());
			}

			try {
        // 调用Provider的方法
				result = provider.authenticate(authentication);

				if (result != null) {
					copyDetails(authentication, result);
					break;
				}
			}
			catch (AccountStatusException | InternalAuthenticationServiceException e) {
				prepareException(e, authentication);
				// SEC-546: Avoid polling additional providers if auth failure is due to
				// invalid account status
				throw e;
			} catch (AuthenticationException e) {
				lastException = e;
			}
		}

		if (result == null && parent != null) {
			// Allow the parent to try.
			try {
				result = parentResult = parent.authenticate(authentication);
			}
			catch (ProviderNotFoundException e) {
				// ignore as we will throw below if no other exception occurred prior to
				// calling parent and the parent
				// may throw ProviderNotFound even though a provider in the child already
				// handled the request
			}
			catch (AuthenticationException e) {
				lastException = parentException = e;
			}
		}

		if (result != null) {
			if (eraseCredentialsAfterAuthentication
					&& (result instanceof CredentialsContainer)) {
				// Authentication is complete. Remove credentials and other secret data
				// from authentication
				((CredentialsContainer) result).eraseCredentials();
			}

			// If the parent AuthenticationManager was attempted and successful than it will publish an AuthenticationSuccessEvent
			// This check prevents a duplicate AuthenticationSuccessEvent if the parent AuthenticationManager already published it
			if (parentResult == null) {
				eventPublisher.publishAuthenticationSuccess(result);
			}
			return result;
		}

		// Parent was null, or didn't authenticate (or throw an exception).

		if (lastException == null) {
			lastException = new ProviderNotFoundException(messages.getMessage(
					"ProviderManager.providerNotFound",
					new Object[] { toTest.getName() },
					"No AuthenticationProvider found for {0}"));
		}

		// If the parent AuthenticationManager was attempted and failed than it will publish an AbstractAuthenticationFailureEvent
		// This check prevents a duplicate AbstractAuthenticationFailureEvent if the parent AuthenticationManager already published it
		if (parentException == null) {
			prepareException(lastException, authentication);
		}

		throw lastException;
	}
```

* **org.springframework.security.authentication.dao.AbstractUserDetailsAuthenticationProvider#authenticate**

```java
  // org.springframework.security.authentication.dao.AbstractUserDetailsAuthenticationProvider#authenticate
	public Authentication authenticate(Authentication authentication)
			throws AuthenticationException {
		Assert.isInstanceOf(UsernamePasswordAuthenticationToken.class, authentication,
				() -> messages.getMessage(
						"AbstractUserDetailsAuthenticationProvider.onlySupports",
						"Only UsernamePasswordAuthenticationToken is supported"));

		// Determine username
		String username = (authentication.getPrincipal() == null) ? "NONE_PROVIDED"
				: authentication.getName();

		boolean cacheWasUsed = true;
		UserDetails user = this.userCache.getUserFromCache(username);

		if (user == null) {
			cacheWasUsed = false;

			try {
				user = retrieveUser(username,
						(UsernamePasswordAuthenticationToken) authentication);
			}
			catch (UsernameNotFoundException notFound) {
				logger.debug("User '" + username + "' not found");

				if (hideUserNotFoundExceptions) {
					throw new BadCredentialsException(messages.getMessage(
							"AbstractUserDetailsAuthenticationProvider.badCredentials",
							"Bad credentials"));
				}
				else {
					throw notFound;
				}
			}

			Assert.notNull(user,
					"retrieveUser returned null - a violation of the interface contract");
		}

		try {
      /*
       * 前检查由DefaultPreAuthenticationChecks类实现（主要判断当前用户是否锁定，过期，冻结
       * User接口）
       */
			preAuthenticationChecks.check(user);
			additionalAuthenticationChecks(user,
					(UsernamePasswordAuthenticationToken) authentication);
		}
		catch (AuthenticationException exception) {
			if (cacheWasUsed) {
				// There was a problem, so try again after checking
				// we're using latest data (i.e. not from the cache)
				cacheWasUsed = false;
				user = retrieveUser(username,
						(UsernamePasswordAuthenticationToken) authentication);
				preAuthenticationChecks.check(user);
				additionalAuthenticationChecks(user,
						(UsernamePasswordAuthenticationToken) authentication);
			}
			else {
				throw exception;
			}
		}
    // 检测用户密码是否过期
		postAuthenticationChecks.check(user);

		if (!cacheWasUsed) {
			this.userCache.putUserInCache(user);
		}

		Object principalToReturn = user;

		if (forcePrincipalAsString) {
			principalToReturn = user.getUsername();
		}

		return createSuccessAuthentication(principalToReturn, authentication, user);
	}
```

* **org.springframework.security.authentication.dao.DaoAuthenticationProvider#retrieveUser**

```java
	// org.springframework.security.authentication.dao.DaoAuthenticationProvider#retrieveUser
	protected final UserDetails retrieveUser(String username,
			UsernamePasswordAuthenticationToken authentication)
			throws AuthenticationException {
		prepareTimingAttackProtection();
		try {
			UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
			if (loadedUser == null) {
				throw new InternalAuthenticationServiceException(
						"UserDetailsService returned null, which is an interface contract violation");
			}
			return loadedUser;
		}
		catch (UsernameNotFoundException ex) {
			mitigateAgainstTimingAttack(authentication);
			throw ex;
		}
		catch (InternalAuthenticationServiceException ex) {
			throw ex;
		}
		catch (Exception ex) {
			throw new InternalAuthenticationServiceException(ex.getMessage(), ex);
		}
	}
```

```java

/**
 * Core interface which loads user-specific data.
 * <p>
 * It is used throughout the framework as a user DAO and is the strategy used by the
 * {@link org.springframework.security.authentication.dao.DaoAuthenticationProvider
 * DaoAuthenticationProvider}.
 *
 * <p>
 * The interface requires only one read-only method, which simplifies support for new
 * data-access strategies.
 *
 * @see org.springframework.security.authentication.dao.DaoAuthenticationProvider
 * @see UserDetails
 *
 * @author Ben Alex
 */
public interface UserDetailsService {
	// ~ Methods
	// ========================================================================================================

	/**
	 * Locates the user based on the username. In the actual implementation, the search
	 * may possibly be case sensitive, or case insensitive depending on how the
	 * implementation instance is configured. In this case, the <code>UserDetails</code>
	 * object that comes back may have a username that is of a different case than what
	 * was actually requested..
	 *
	 * @param username the username identifying the user whose data is required.
	 *
	 * @return a fully populated user record (never <code>null</code>)
	 *
	 * @throws UsernameNotFoundException if the user could not be found or the user has no
	 * GrantedAuthority
	 */
	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}

```

#### 示例

```kotlin
package com.holelin.security

import com.alibaba.fastjson.JSON
import com.alibaba.fastjson.JSONObject
import com.holelin.util.HttpUtils
import com.holelin.util.JwtTokenUtils
import org.springframework.security.authentication.AuthenticationManager
import org.springframework.security.authentication.event.InteractiveAuthenticationSuccessEvent
import org.springframework.security.core.Authentication
import org.springframework.security.core.AuthenticationException
import org.springframework.security.core.context.SecurityContextHolder
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter
import java.io.BufferedReader
import java.io.IOException
import java.io.InputStream
import java.io.InputStreamReader
import java.nio.charset.Charset
import javax.servlet.FilterChain
import javax.servlet.ServletException
import javax.servlet.ServletRequest
import javax.servlet.ServletResponse
import javax.servlet.http.HttpServletRequest
import javax.servlet.http.HttpServletResponse


class JwtLoginFilter(authManager: AuthenticationManager?) : UsernamePasswordAuthenticationFilter() {
    @Throws(IOException::class, ServletException::class)
    override fun doFilter(req: ServletRequest, res: ServletResponse, chain: FilterChain) {
        // POST 请求 /login 登录时拦截， 由此方法触发执行登录认证流程，可以在此覆写整个登录认证逻辑
        super.doFilter(req, res, chain)
    }

    //这个方法是用来去尝试验证用户的
    @Throws(AuthenticationException::class)
    override fun attemptAuthentication(request: HttpServletRequest, response: HttpServletResponse): Authentication {
        // 可以在此覆写尝试进行登录认证的逻辑，登录成功之后等操作不再此方法内
        // 如果使用此过滤器来触发登录认证流程，注意登录请求数据格式的问题
        // 此过滤器的用户名密码默认从request.getParameter()获取，但是这种
        // 读取方式不能读取到如 application/json 等 post 请求数据，需要把
        // 用户名密码的读取逻辑修改为到流中读取request.getInputStream()
        val body = getBody(request)
        val jsonObject: JSONObject = JSON.parseObject(body)
        var username: String = jsonObject.getString("username")
        var password: String = jsonObject.getString("password")
        username = username.trim { it <= ' ' }
        val authRequest = JwtAuthenticationToken(username, password)

        // Allow subclasses to set the "details" property
        setDetails(request, authRequest)
        return authenticationManager.authenticate(authRequest)
    }

    //成功之后执行的方法
    @Throws(IOException::class, ServletException::class)
    override fun successfulAuthentication(
        request: HttpServletRequest?, response: HttpServletResponse, chain: FilterChain?,
        authResult: Authentication?
    ) {
        // 存储登录认证信息到上下文
        SecurityContextHolder.getContext().authentication = authResult
        // 记住我服务
        rememberMeServices.loginSuccess(request, response, authResult)
        // 触发事件监听器
        if (eventPublisher != null) {
            eventPublisher.publishEvent(InteractiveAuthenticationSuccessEvent(authResult, this.javaClass))
        }
        // 生成并返回token给客户端，后续访问携带此token
        val token = JwtAuthenticationToken(null, null, authResult?.let { JwtTokenUtils.generateToken(it) })

        HttpUtils.write(response, token)
    }

    /**
     * 获取请求Body
     * @param request
     * @return
     */
    private fun getBody(request: HttpServletRequest): String {
        val sb = StringBuilder()
        var inputStream: InputStream? = null
        var reader: BufferedReader? = null
        try {
            inputStream = request.inputStream
            reader = BufferedReader(InputStreamReader(inputStream, Charset.forName("UTF-8")))
            var line: String? = ""
            while (reader.readLine().also { line = it } != null) {
                sb.append(line)
            }
        } catch (e: IOException) {
            e.printStackTrace()
        } finally {
            if (inputStream != null) {
                try {
                    inputStream.close()
                } catch (e: IOException) {
                    e.printStackTrace()
                }
            }
            if (reader != null) {
                try {
                    reader.close()
                } catch (e: IOException) {
                    e.printStackTrace()
                }
            }
        }
        return sb.toString()
    }

    init {
        authenticationManager = authManager
    }
}
```

```kotlin
package com.holelin.security

import org.springframework.security.authentication.UsernamePasswordAuthenticationToken
import org.springframework.security.authentication.dao.DaoAuthenticationProvider
import org.springframework.security.core.Authentication
import org.springframework.security.core.AuthenticationException
import org.springframework.security.core.userdetails.UserDetails
import org.springframework.security.core.userdetails.UserDetailsService
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder


class JwtAuthenticationProvider(userDetailsService: UserDetailsService?) :
    DaoAuthenticationProvider() {
    @Throws(AuthenticationException::class)
    override fun authenticate(authentication: Authentication?): Authentication {
        // 可以在此处覆写整个登录认证逻辑
        return super.authenticate(authentication)
    }

    @Throws(AuthenticationException::class)
    override fun additionalAuthenticationChecks(
        userDetails: UserDetails,
        authentication: UsernamePasswordAuthenticationToken
    ) {
        // 可以在此处覆写密码验证逻辑
        super.additionalAuthenticationChecks(userDetails, authentication)
    }

    init {
        setUserDetailsService(userDetailsService)
        passwordEncoder = BCryptPasswordEncoder()
    }
}
```

```kotlin
package com.holelin.security

import org.springframework.security.core.GrantedAuthority
import org.springframework.security.core.userdetails.User


class JwtUserDetails(
    username: String?, password: String?, enabled: Boolean, accountNonExpired: Boolean,
    credentialsNonExpired: Boolean, accountNonLocked: Boolean, authorities: Collection<GrantedAuthority?>?
) :
    User(username, password, enabled, accountNonExpired, credentialsNonExpired, accountNonLocked, authorities) {
    constructor(username: String?, password: String?, authorities: Collection<GrantedAuthority?>?) : this(
        username,
        password,
        true,
        true,
        true,
        true,
        authorities
    ) {
    }

    companion object {
        private const val serialVersionUID = 1L
    }
}
```

```kotlin
package com.holelin.security

import org.springframework.security.authentication.UsernamePasswordAuthenticationToken
import org.springframework.security.core.GrantedAuthority


/**
 * 自定义令牌对象
 * @author Louis
 * @date Jun 29, 2019
 */
class JwtAuthenticationToken : UsernamePasswordAuthenticationToken {
    var token: String? = null

    constructor(principal: Any?, credentials: Any?) : super(principal, credentials) {}
    constructor(principal: Any?, credentials: Any?, token: String?) : super(principal, credentials) {
        this.token = token
    }

    constructor(
        principal: Any?,
        credentials: Any?,
        authorities: Collection<GrantedAuthority?>?,
        token: String?
    ) : super(principal, credentials, authorities) {
        this.token = token
    }

    companion object {
        const val serialVersionUID = 1L
    }
}
```

```kotlin
package com.holelin.security

import com.holelin.util.SecurityUtils
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.security.authentication.AuthenticationManager
import org.springframework.security.web.authentication.www.BasicAuthenticationFilter
import java.io.IOException
import javax.servlet.FilterChain
import javax.servlet.ServletException
import javax.servlet.http.HttpServletRequest
import javax.servlet.http.HttpServletResponse


class JwtAuthenticationFilter @Autowired constructor(authenticationManager: AuthenticationManager?) :
    BasicAuthenticationFilter(authenticationManager) {
    @Throws(IOException::class, ServletException::class)
    override fun doFilterInternal(request: HttpServletRequest, response: HttpServletResponse, chain: FilterChain) {
        // 获取token, 并检查登录状态
        SecurityUtils.checkAuthentication(request)
        chain.doFilter(request, response)
    }
}
```

```kotlin
package com.holelin.security

import org.springframework.security.core.GrantedAuthority


class GrantedAuthorityImpl(private var authority: String) : GrantedAuthority {
    fun setAuthority(authority: String) {
        this.authority = authority
    }

    override fun getAuthority(): String {
        return authority
    }

    companion object {
        private const val serialVersionUID = 1L
    }
}
```

```kotlin
package com.holelin.security

import org.springframework.security.core.GrantedAuthority
import org.springframework.security.core.userdetails.User


class JwtUserDetails(
    username: String?, password: String?, enabled: Boolean, accountNonExpired: Boolean,
    credentialsNonExpired: Boolean, accountNonLocked: Boolean, authorities: Collection<GrantedAuthority?>?
) :
    User(username, password, enabled, accountNonExpired, credentialsNonExpired, accountNonLocked, authorities) {
    constructor(username: String?, password: String?, authorities: Collection<GrantedAuthority?>?) : this(
        username,
        password,
        true,
        true,
        true,
        true,
        authorities
    ) {
    }

    companion object {
        private const val serialVersionUID = 1L
    }
}
```

```kotlin
package com.holelin.util

import com.alibaba.fastjson.JSONObject
import com.holelin.vo.HttpResult
import org.springframework.web.context.request.RequestContextHolder
import org.springframework.web.context.request.ServletRequestAttributes
import java.io.IOException
import javax.servlet.http.HttpServletRequest
import javax.servlet.http.HttpServletResponse


object HttpUtils {
    /**
     * 获取HttpServletRequest对象
     * @return
     */
    val httpServletRequest: HttpServletRequest
        get() = (RequestContextHolder.getRequestAttributes() as ServletRequestAttributes).request

    /**
     * 输出信息到浏览器
     * @param response
     * @param message
     * @throws IOException
     */
    @Throws(IOException::class)
    fun write(response: HttpServletResponse, data: Any?) {
        response.contentType = "application/json; charset=utf-8"
        val result: HttpResult = HttpResult.ok(data)
        val json: String = JSONObject.toJSONString(result)
        response.writer.print(json)
        response.writer.flush()
        response.writer.close()
    }
}

```

```kotlin
package com.holelin.util

import com.holelin.security.GrantedAuthorityImpl
import com.holelin.security.JwtAuthenticationToken
import io.jsonwebtoken.Claims
import io.jsonwebtoken.Jwts
import io.jsonwebtoken.SignatureAlgorithm
import org.springframework.security.core.Authentication
import org.springframework.security.core.GrantedAuthority
import java.io.Serializable
import java.util.*
import javax.servlet.http.HttpServletRequest
import kotlin.collections.ArrayList
import kotlin.collections.HashMap


object JwtTokenUtils : Serializable {
	private const val serialVersionUID = 1L

	/**
	 * 用户名称
	 */
	private const val USERNAME = Claims.SUBJECT

	/**
	 * 创建时间
	 */
	private const val CREATED = "created"

	/**
	 * 权限列表
	 */
	private const val AUTHORITIES = "authorities"

	/**
	 * 密钥
	 */
	private const val SECRET = "abcdefgh"

	/**
	 * 有效期12小时
	 */
	private const val EXPIRE_TIME = (12 * 60 * 60 * 1000).toLong()

	/**
	 * 生成令牌
	 *
	 * @param userDetails 用户
	 * @return 令牌
	 */
	fun generateToken(authentication: Authentication): String {
		val claims: MutableMap<String, Any> = HashMap(3)
		claims[USERNAME] = SecurityUtils.getUsername(authentication)
		claims[CREATED] = Date()
		claims[AUTHORITIES] = authentication.authorities
		return generateToken(claims)
	}

	/**
	 * 从数据声明生成令牌
	 *
	 * @param claims 数据声明
	 * @return 令牌
	 */
	private fun generateToken(claims: Map<String, Any>): String {
		val expirationDate = Date(System.currentTimeMillis() + EXPIRE_TIME)
		return Jwts.builder().setClaims(claims).setExpiration(expirationDate).signWith(SignatureAlgorithm.HS512, SECRET)
			.compact()
	}

	/**
	 * 从令牌中获取用户名
	 *
	 * @param token 令牌
	 * @return 用户名
	 */
	private fun getUsernameFromToken(token: String): String? {
		val username: String?
		username = try {
			val claims = getClaimsFromToken(token)
			claims!!.subject
		} catch (e: Exception) {
			null
		}
		return username
	}

	/**
	 * 根据请求令牌获取登录认证信息
	 * @param token 令牌
	 * @return 用户名
	 */
	fun getAuthenticationFromToken(request: HttpServletRequest): Authentication? {
		var authentication: Authentication? = null
		// 获取请求携带的令牌
		val token = getToken(request)
		if (token != null) {
			// 请求令牌不能为空
			if (SecurityUtils.authentication == null) {
				// 上下文中Authentication为空
				val claims = getClaimsFromToken(token) ?: return null
				val username = claims.subject ?: return null
				if (isTokenExpired(token)) {
					return null
				}
				val authors = claims[AUTHORITIES]
				val authorities: MutableList<GrantedAuthority> = ArrayList()
				if (authors != null && authors is List<*>) {
					for (`object` in authors) {
						authorities.add(GrantedAuthorityImpl((`object` as Map<*, *>)["authority"] as String))
					}
				}
				authentication = JwtAuthenticationToken(username, null, authorities, token)
			} else {
				if (validateToken(token, SecurityUtils.username)) {
					// 如果上下文中Authentication非空，且请求令牌合法，直接返回当前登录认证信息
					authentication = SecurityUtils.authentication
				}
			}
		}
		return authentication
	}

	/**
	 * 从令牌中获取数据声明
	 *
	 * @param token 令牌
	 * @return 数据声明
	 */
	private fun getClaimsFromToken(token: String): Claims? {
		val claims: Claims? = try {
			Jwts.parser().setSigningKey(SECRET).parseClaimsJws(token).body
		} catch (e: Exception) {
			null
		}
		return claims
	}

	/**
	 * 验证令牌
	 * @param token
	 * @param username
	 * @return
	 */
	private fun validateToken(token: String, username: String): Boolean {
		val userName = getUsernameFromToken(token)
		return userName == username && !isTokenExpired(token)
	}

	/**
	 * 刷新令牌
	 * @param token
	 * @return
	 */
	fun refreshToken(token: String): String? {
		var refreshedToken: String?
		try {
			val claims = getClaimsFromToken(token)
			claims!![CREATED] = Date()
			refreshedToken = generateToken(claims)
		} catch (e: Exception) {
			refreshedToken = null
		}
		return refreshedToken
	}

	/**
	 * 判断令牌是否过期
	 *
	 * @param token 令牌
	 * @return 是否过期
	 */
	private fun isTokenExpired(token: String): Boolean {
		return try {
			val claims = getClaimsFromToken(token)
			val expiration: Date = claims!!.expiration
			expiration.before(Date())
		} catch (e: Exception) {
			false
		}
	}

	/**
	 * 获取请求token
	 * @param request
	 * @return
	 */
	private fun getToken(request: HttpServletRequest): String? {
		var token = request.getHeader("Authorization")
		val tokenHead = "Bearer "
		if (token == null) {
			token = request.getHeader("token")
		} else if (token.contains(tokenHead)) {
			token = token.substring(tokenHead.length)
		}
		if ("" == token) {
			token = null
		}
		return token
	}
}
```

```kotlin
package com.holelin.util

import com.holelin.security.JwtAuthenticationToken
import org.springframework.security.authentication.AuthenticationManager
import org.springframework.security.core.Authentication
import org.springframework.security.core.context.SecurityContextHolder
import org.springframework.security.core.userdetails.UserDetails
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource
import javax.servlet.http.HttpServletRequest


object SecurityUtils {
    /**
     * 系统登录认证
     * @param request
     * @param username
     * @param password
     * @param authenticationManager
     * @return
     */
    fun login(
        request: HttpServletRequest?,
        username: String?,
        password: String?,
        authenticationManager: AuthenticationManager
    ): JwtAuthenticationToken {
        val token = JwtAuthenticationToken(username, password)
        token.details = WebAuthenticationDetailsSource().buildDetails(request)
        // 执行登录认证过程
        val authentication: Authentication = authenticationManager.authenticate(token)
        // 认证成功存储认证信息到上下文
        SecurityContextHolder.getContext().authentication = authentication
        // 生成令牌并返回给客户端
        token.token = JwtTokenUtils.generateToken(authentication)
        return token
    }

    /**
     * 获取令牌进行认证
     * @param request
     */
    fun checkAuthentication(request: HttpServletRequest?) {
        // 获取令牌并根据令牌获取登录认证信息
        val authentication: Authentication? = JwtTokenUtils.getAuthenticationFromToken(request!!)
        // 设置登录认证信息到上下文
        SecurityContextHolder.getContext().authentication = authentication
    }

    /**
     * 获取当前用户名
     * @return
     */
    val username: String
        get() {
            var username = ""
            val authentication: Authentication? = authentication
            if (authentication != null) {
                val principal: Any = authentication.principal
                if (principal is UserDetails) {
                    username = principal.username
                }
            }
            return username
        }

    /**
     * 获取用户名
     * @return
     */
    fun getUsername(authentication: Authentication): String {
        var username: String = ""
        val principal: Any = authentication.principal
        if (principal is UserDetails) {
            username = principal.username
        }
        return username
    }


    /**
     * 获取当前登录信息
     * @return
     */
    val authentication: Authentication?
        get() = if (SecurityContextHolder.getContext() == null) {
            null
        } else SecurityContextHolder.getContext().authentication
}
```



