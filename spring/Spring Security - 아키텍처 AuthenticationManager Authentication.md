# Spring Security - ArcheTecher_AuthenticationManager Authentication
- Spring Security에서 인증 (Authenticaiton) 은 AuthenticationManager가 담당한다.

#### AuthenticationManager
```java
public interface AuthenticationManager {
	// ~ Methods
	// ========================================================================================================

	/**
	 * Attempts to authenticate the passed {@link Authentication} object, returning a
	 * fully populated <code>Authentication</code> object (including granted authorities)
	 * if successful.
	 * <p>
	 * An <code>AuthenticationManager</code> must honour the following contract concerning
	 * exceptions:
	 * <ul>
	 * <li>A {@link DisabledException} must be thrown if an account is disabled and the
	 * <code>AuthenticationManager</code> can test for this state.</li>
	 * <li>A {@link LockedException} must be thrown if an account is locked and the
	 * <code>AuthenticationManager</code> can test for account locking.</li>
	 * <li>A {@link BadCredentialsException} must be thrown if incorrect credentials are
	 * presented. Whilst the above exceptions are optional, an
	 * <code>AuthenticationManager</code> must <B>always</B> test credentials.</li>
	 * </ul>
	 * Exceptions should be tested for and if applicable thrown in the order expressed
	 * above (i.e. if an account is disabled or locked, the authentication request is
	 * immediately rejected and the credentials testing process is not performed). This
	 * prevents credentials being tested against disabled or locked accounts.
	 *
	 * @param authentication the authentication request object
	 *
	 * @return a fully authenticated object including credentials
	 *
	 * @throws AuthenticationException if authentication fails
	 */
	Authentication authenticate(Authentication authentication)
			throws AuthenticationException;
}
```

인자로 받은 Authentication 객체가 유저에 대한 인증 정보를 담고있다.
해당 인증 정보가 유효하다면 UserDetailsService에서 Return한 객체 [Principal] 을 담고있는 Authentication 객체를 리턴한다.

#### 인증 과정에서 발생하는 예외들
- DisabledException
    - 계정이 비활성화 되어있는 경우
- LockedException
    - 계정이 잠겨 있는 경우
- BadCredentialsException
    - 비밀번호가 일치하지 않는 경우


기본 구현체는 ProviderManager 를 사용한다.
직접 AuthenticationManager를 구현해도 되지만 대부분의 Application에서 기본 구현체 정도에서 해결이 된다.
- 직접 구현하는 경우는 매우 드물다.

#### 인증 프로세스
- 유저를 생성하고, 로그인 폼에서 정상적인 인증 요청을 했을경우
- ProviderManager의 authentcation 메서드로 진입을 한다.
    - 이때 Authentication 객체를 파라메터로 받는다.
    - Authentcation 객체에는 principal에 해당하는 username 정보와 credentials에 해당하는 password가 들어오게 된다.
    - authorities 권한에 해당하는 정보는 아직 알 수 없기때문에 빈값이 들어온다.
- 해당 요청에 대한 인증을 처리할수 있는 Provider를 찾아 이를 처리한 뒤 인증정보가 담긴 Authentication 객체를 반환한다.

```java
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
            result = provider.authenticate(authentication);

            if (result != null) {
                copyDetails(authentication, result);
                break;
            }
        }
        catch (AccountStatusException e) {
            prepareException(e, authentication);
            // SEC-546: Avoid polling additional providers if auth failure is due to
            // invalid account status
            throw e;
        }
        catch (InternalAuthenticationServiceException e) {
            prepareException(e, authentication);
            throw e;
        }
        catch (AuthenticationException e) {
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

ProviderManager는 인증을 직접 처리하는것이 아닌 다른 여러 Provider를 사용해서 진행한다.
- 다른 여러 Provider들에게 위임하는 구조

- 현재 파라메터로 들어온 Authentication 객체는 UsernameAuthenticationToken 타입
- Authentication 객체의 타입마다 처리할 수 있는 Provider가 다르다.

UsernameAuthenticationToken.java
```java
public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {

	private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

	// ~ Instance fields
	// ================================================================================================

	private final Object principal;
	private Object credentials;

	// ~ Constructors
	// ===================================================================================================

	/**
	 * This constructor can be safely used by any code that wishes to create a
	 * <code>UsernamePasswordAuthenticationToken</code>, as the {@link #isAuthenticated()}
	 * will return <code>false</code>.
	 *
	 */
	public UsernamePasswordAuthenticationToken(Object principal, Object credentials) {
		super(null);
		this.principal = principal;
		this.credentials = credentials;
		setAuthenticated(false);
	}

	/**
	 * This constructor should only be used by <code>AuthenticationManager</code> or
	 * <code>AuthenticationProvider</code> implementations that are satisfied with
	 * producing a trusted (i.e. {@link #isAuthenticated()} = <code>true</code>)
	 * authentication token.
	 *
	 * @param principal
	 * @param credentials
	 * @param authorities
	 */
	public UsernamePasswordAuthenticationToken(Object principal, Object credentials,
			Collection<? extends GrantedAuthority> authorities) {
		super(authorities);
		this.principal = principal;
		this.credentials = credentials;
		super.setAuthenticated(true); // must use super, as we override
	}

	// ~ Methods
	// ========================================================================================================

	public Object getCredentials() {
		return this.credentials;
	}

	public Object getPrincipal() {
		return this.principal;
	}

	public void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException {
		if (isAuthenticated) {
			throw new IllegalArgumentException(
					"Cannot set this token to trusted - use constructor which takes a GrantedAuthority list instead");
		}

		super.setAuthenticated(false);
	}

	@Override
	public void eraseCredentials() {
		super.eraseCredentials();
		credentials = null;
	}
}
```

현재 ProviderManager는 AnnoymousAuthenticationProvider만을 가지고 있기때문에 UsernamePasswordAuthenticationToken에 대한 인증을 처리할 수 없다.

현재 ProviderManager에게 처리 할수 있는 Provider가 존재하지 않을경우 ProviderManager의 Parent에게 위임한다.
- ProviderManager의 Parent가 인증 요청을 위임받아, 처리할수 있는 Provider를 찾은뒤 인증을 처리한다.
- 이때 ProviderManager와 ProviderManager의 Parent는 다른 ProviderManager이며 처리 가능한 Provider도 다르다.

> DaoAuthenticationProvider를 사용하며, DaoAuthenticationProvider가 우리가 직접 구현한 UserDetailsService를 사용한다.
