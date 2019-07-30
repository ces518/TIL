# Spring Handler Method - ResourceLoader
#### ResourceLoader
	- ResourceLoader는 리소스를 읽어오는 기능을 제공하는 Interface
	- 리소스를 읽어 Resource Type 객체를 반환한다.
	- Spring 을 사용한다면 @Autowired를 통하여 의존성 주입을 받아 바로 사용할 수 있다.
	- ApplicationContext는 ResourceLoader를 구현하고 있기때문에 ApplicationContext로도 ResourceLoader의 기능을 사용할 수 있다.

```java
/*
 * Copyright 2002-2018 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.core.io;

import org.springframework.lang.Nullable;
import org.springframework.util.ResourceUtils;

/**
 * Strategy interface for loading resources (e.. class path or file system
 * resources). An {@link org.springframework.context.ApplicationContext}
 * is required to provide this functionality, plus extended
 * {@link org.springframework.core.io.support.ResourcePatternResolver} support.
 *
 * <p>{@link DefaultResourceLoader} is a standalone implementation that is
 * usable outside an ApplicationContext, also used by {@link ResourceEditor}.
 *
 * <p>Bean properties of type Resource and Resource array can be populated
 * from Strings when running in an ApplicationContext, using the particular
 * context's resource loading strategy.
 *
 * @author Juergen Hoeller
 * @since 10.03.2004
 * @see Resource
 * @see org.springframework.core.io.support.ResourcePatternResolver
 * @see org.springframework.context.ApplicationContext
 * @see org.springframework.context.ResourceLoaderAware
 */
public interface ResourceLoader {

	/** Pseudo URL prefix for loading from the class path: "classpath:". */
	String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;


	/**
	 * Return a Resource handle for the specified resource location.
	 * <p>The handle should always be a reusable resource descriptor,
	 * allowing for multiple {@link Resource#getInputStream()} calls.
	 * <p><ul>
	 * <li>Must support fully qualified URLs, e.g. "file:C:/test.dat".
	 * <li>Must support classpath pseudo-URLs, e.g. "classpath:test.dat".
	 * <li>Should support relative file paths, e.g. "WEB-INF/test.dat".
	 * (This will be implementation-specific, typically provided by an
	 * ApplicationContext implementation.)
	 * </ul>
	 * <p>Note that a Resource handle does not imply an existing resource;
	 * you need to invoke {@link Resource#exists} to check for existence.
	 * @param location the resource location
	 * @return a corresponding Resource handle (never {@code null})
	 * @see #CLASSPATH_URL_PREFIX
	 * @see Resource#exists()
	 * @see Resource#getInputStream()
	 */
	Resource getResource(String location);

	/**
	 * Expose the ClassLoader used by this ResourceLoader.
	 * <p>Clients which need to access the ClassLoader directly can do so
	 * in a uniform manner with the ResourceLoader, rather than relying
	 * on the thread context ClassLoader.
	 * @return the ClassLoader
	 * (only {@code null} if even the system ClassLoader isn't accessible)
	 * @see org.springframework.util.ClassUtils#getDefaultClassLoader()
	 * @see org.springframework.util.ClassUtils#forName(String, ClassLoader)
	 */
	@Nullable
	ClassLoader getClassLoader();

}
```

- ResourceLoader를 우리가 Bean으로 등록하지 않았는데 대체 어떻게 사용할수 있는것일까 ?

- 테스트 코드
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootMvcApplicationTests {

    @Autowired
    ResourceLoader resourceLoader;

    @Autowired
    ApplicationContext applicationContext;

    @Test
    public void contextLoads() {
        System.out.println(resourceLoader instanceof ApplicationContext);
        System.out.println(resourceLoader);
        System.out.println(applicationContext);
        System.out.println(resourceLoader == applicationContext);
    }

}
``` 

- 결과
```java
true
org.springframework.web.context.support.GenericWebApplicationContext@b9b00e0, started on Tue Jul 30 20:57:13 KST 2019
org.springframework.web.context.support.GenericWebApplicationContext@b9b00e0, started on Tue Jul 30 20:57:13 KST 2019
true
```
- ApplicationContext는 ResourceLoader를 구현하고 있기때문에 ResourceLoader Type으로 주입 받아 사용할 수 있는것이다.
- 앞서 ResourceLoader는 Resource Type 객체를 반환한다고 했다. 그렇다면 Resource 란 대체무엇일까 

#### Resource Interface
	- 스프링의 Resource 인터페이스는 저수준 리소스 접근을 추상화한 더 기능이 많은 인터페이스이다.
	- 주요 메서드
		- getInputStream(): 리소스의 위치를 찾고 오픈한뒤 리소스를 읽기위한 InputStream을 Return, 호출시마다 새로운 객체를 Return 스트림을 닫는것은 호출한 쪽에 책임이 있다.
		- exists(): 해당 리소스가 존재하는지 논리값을 Return
		- isOpen(): 해동 리소스가 오픈 스트림을 가진 하나의 핸들을 나타내는지 논리값을 Return, true일경우 'InputStream은 여러번 읽을수 없고 반드시 한번만 읽은뒤 닫아주어야 한다.' InputStreamResource 예외를 가진 일반적인 리소스 구현체에서는 false
		- getDescription(): 해당 리소스에 대한 설명을 Return, 정규화된 파일명이나, 리소스의 실제 URL이다.
```java
/*
 * Copyright 2002-2018 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.core.io;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.net.URI;
import java.net.URL;
import java.nio.channels.Channels;
import java.nio.channels.ReadableByteChannel;

import org.springframework.lang.Nullable;

/**
 * Interface for a resource descriptor that abstracts from the actual
 * type of underlying resource, such as a file or class path resource.
 *
 * <p>An InputStream can be opened for every resource if it exists in
 * physical form, but a URL or File handle can just be returned for
 * certain resources. The actual behavior is implementation-specific.
 *
 * @author Juergen Hoeller
 * @since 28.12.2003
 * @see #getInputStream()
 * @see #getURL()
 * @see #getURI()
 * @see #getFile()
 * @see WritableResource
 * @see ContextResource
 * @see UrlResource
 * @see FileUrlResource
 * @see FileSystemResource
 * @see ClassPathResource
 * @see ByteArrayResource
 * @see InputStreamResource
 */
public interface Resource extends InputStreamSource {

	/**
	 * Determine whether this resource actually exists in physical form.
	 * <p>This method performs a definitive existence check, whereas the
	 * existence of a {@code Resource} handle only guarantees a valid
	 * descriptor handle.
	 */
	boolean exists();

	/**
	 * Indicate whether non-empty contents of this resource can be read via
	 * {@link #getInputStream()}.
	 * <p>Will be {@code true} for typical resource descriptors that exist
	 * since it strictly implies {@link #exists()} semantics as of 5.1.
	 * Note that actual content reading may still fail when attempted.
	 * However, a value of {@code false} is a definitive indication
	 * that the resource content cannot be read.
	 * @see #getInputStream()
	 * @see #exists()
	 */
	default boolean isReadable() {
		return exists();
	}

	/**
	 * Indicate whether this resource represents a handle with an open stream.
	 * If {@code true}, the InputStream cannot be read multiple times,
	 * and must be read and closed to avoid resource leaks.
	 * <p>Will be {@code false} for typical resource descriptors.
	 */
	default boolean isOpen() {
		return false;
	}

	/**
	 * Determine whether this resource represents a file in a file system.
	 * A value of {@code true} strongly suggests (but does not guarantee)
	 * that a {@link #getFile()} call will succeed.
	 * <p>This is conservatively {@code false} by default.
	 * @since 5.0
	 * @see #getFile()
	 */
	default boolean isFile() {
		return false;
	}

	/**
	 * Return a URL handle for this resource.
	 * @throws IOException if the resource cannot be resolved as URL,
	 * i.e. if the resource is not available as descriptor
	 */
	URL getURL() throws IOException;

	/**
	 * Return a URI handle for this resource.
	 * @throws IOException if the resource cannot be resolved as URI,
	 * i.e. if the resource is not available as descriptor
	 * @since 2.5
	 */
	URI getURI() throws IOException;

	/**
	 * Return a File handle for this resource.
	 * @throws java.io.FileNotFoundException if the resource cannot be resolved as
	 * absolute file path, i.e. if the resource is not available in a file system
	 * @throws IOException in case of general resolution/reading failures
	 * @see #getInputStream()
	 */
	File getFile() throws IOException;

	/**
	 * Return a {@link ReadableByteChannel}.
	 * <p>It is expected that each call creates a <i>fresh</i> channel.
	 * <p>The default implementation returns {@link Channels#newChannel(InputStream)}
	 * with the result of {@link #getInputStream()}.
	 * @return the byte channel for the underlying resource (must not be {@code null})
	 * @throws java.io.FileNotFoundException if the underlying resource doesn't exist
	 * @throws IOException if the content channel could not be opened
	 * @since 5.0
	 * @see #getInputStream()
	 */
	default ReadableByteChannel readableChannel() throws IOException {
		return Channels.newChannel(getInputStream());
	}

	/**
	 * Determine the content length for this resource.
	 * @throws IOException if the resource cannot be resolved
	 * (in the file system or as some other known physical resource type)
	 */
	long contentLength() throws IOException;

	/**
	 * Determine the last-modified timestamp for this resource.
	 * @throws IOException if the resource cannot be resolved
	 * (in the file system or as some other known physical resource type)
	 */
	long lastModified() throws IOException;

	/**
	 * Create a resource relative to this resource.
	 * @param relativePath the relative path (relative to this resource)
	 * @return the resource handle for the relative resource
	 * @throws IOException if the relative resource cannot be determined
	 */
	Resource createRelative(String relativePath) throws IOException;

	/**
	 * Determine a filename for this resource, i.e. typically the last
	 * part of the path: for example, "myfile.txt".
	 * <p>Returns {@code null} if this type of resource does not
	 * have a filename.
	 */
	@Nullable
	String getFilename();

	/**
	 * Return a description for this resource,
	 * to be used for error output when working with the resource.
	 * <p>Implementations are also encouraged to return this value
	 * from their {@code toString} method.
	 * @see Object#toString()
	 */
	String getDescription();

}
```

- Resource 구현체
	- 1. UrlResource
		- java.net.URL을 Wrapping 하고 일반적으로 URL로 접근할수 있는 파일, HTTP, FTP등 과 같은 객체에 접근시 사용한다.
	- 2. ClassPathResource
		- 클래스패스에서 얻어와야하는 리소스를 나타낸다.
		- Thread Context Loader, ClassLoader , Class Load시 주어진 클래스를 모두 사용한다.
		- 파일시스템에 존재하면 java.io.File과 같은 해결책을 지원하지만 jar에 존재하는 클래스패스 리소스는 지원하지않으며 파일시스템으로 확장하지도 않는다.
	- 3. FileSystemResource
		- java.io.File 핸들에 대한 구현체이다.
	- 4. ServletContextResource
		- 웹 어플리케이션 루트 경로내에서 상대경로를 인터프리팅하는 ServletContext 리소스에대한 구현체이다.
	- 5. InputStreamResource
		- InputStream에 대한 구현체이다. 적용가능한 특정 Resource 구현체가 없을때만 사용 가능하다.
		- ByteArrayResource 나 파일 기반 Resource구현체가 가능한 곳에서 선호한다.
		- 다른 구현체와 달리 이미 오픈된 Resource에 대한 디스크립터이다.
		- 즉 isOpen() 은 true를 리턴한다.
	- 6. ByteArrayResource
		- Byte 배열에 대한 구현체이다.
		- 주어진 바이트배열에서 컨텐츠 로드시 유용하다.

#### ResourceLoaderAware Interface
- ResourceLoaderAware 인터페이스는 ResourceLoader 참조와 함께 제공되기를 기대하는 객체를 식별하는 특별한 마커(marker) 인터페이스이다.
- 이를 구현하는 Class가 ApplicationContext에 Bean으로 등록 되었을때 ApplicationContext는 해당 클래스를 ResourceLoaderAware로 인식한다.
- 그런 다음 자신을 Argument로 setResourceLoader 메서드를 호출해 의존성을 주입한다.
```java
/*
 * Copyright 2002-2019 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.context;

import org.springframework.beans.factory.Aware;
import org.springframework.core.io.ResourceLoader;

/**
 * Interface to be implemented by any object that wishes to be notified of the
 * {@link ResourceLoader} (typically the ApplicationContext) that it runs in.
 * This is an alternative to a full {@link ApplicationContext} dependency via
 * the {@link org.springframework.context.ApplicationContextAware} interface.
 *
 * <p>Note that {@link org.springframework.core.io.Resource} dependencies can also
 * be exposed as bean properties of type {@code Resource}, populated via Strings
 * with automatic type conversion by the bean factory. This removes the need for
 * implementing any callback interface just for the purpose of accessing a
 * specific file resource.
 *
 * <p>You typically need a {@link ResourceLoader} when your application object has to
 * access a variety of file resources whose names are calculated. A good strategy is
 * to make the object use a {@link org.springframework.core.io.DefaultResourceLoader}
 * but still implement {@code ResourceLoaderAware} to allow for overriding when
 * running in an {@code ApplicationContext}. See
 * {@link org.springframework.context.support.ReloadableResourceBundleMessageSource}
 * for an example.
 *
 * <p>A passed-in {@code ResourceLoader} can also be checked for the
 * {@link org.springframework.core.io.support.ResourcePatternResolver} interface
 * and cast accordingly, in order to resolve resource patterns into arrays of
 * {@code Resource} objects. This will always work when running in an ApplicationContext
 * (since the context interface extends the ResourcePatternResolver interface). Use a
 * {@link org.springframework.core.io.support.PathMatchingResourcePatternResolver} as
 * default; see also the {@code ResourcePatternUtils.getResourcePatternResolver} method.
 *
 * <p>As an alternative to a {@code ResourcePatternResolver} dependency, consider
 * exposing bean properties of type {@code Resource} array, populated via pattern
 * Strings with automatic type conversion by the bean factory at binding time.
 *
 * @author Juergen Hoeller
 * @author Chris Beams
 * @since 10.03.2004
 * @see ApplicationContextAware
 * @see org.springframework.core.io.Resource
 * @see org.springframework.core.io.ResourceLoader
 * @see org.springframework.core.io.support.ResourcePatternResolver
 */
public interface ResourceLoaderAware extends Aware {

	/**
	 * Set the ResourceLoader that this object runs in.
	 * <p>This might be a ResourcePatternResolver, which can be checked
	 * through {@code instanceof ResourcePatternResolver}. See also the
	 * {@code ResourcePatternUtils.getResourcePatternResolver} method.
	 * <p>Invoked after population of normal bean properties but before an init callback
	 * like InitializingBean's {@code afterPropertiesSet} or a custom init-method.
	 * Invoked before ApplicationContextAware's {@code setApplicationContext}.
	 * @param resourceLoader the ResourceLoader object to be used by this object
	 * @see org.springframework.core.io.support.ResourcePatternResolver
	 * @see org.springframework.core.io.support.ResourcePatternUtils#getResourcePatternResolver
	 */
	void setResourceLoader(ResourceLoader resourceLoader);

}
```

- ResourceLoaderAware를 사용하는 이유 ?
	- ApplicationContext가 ResourceLoader를 구현하고있기때문에 ApplicationContext를 직접 사용해도 된다.
	- 하지만 리소스를 로드하는 목적이라면 리소스 로드용 ResourceLoader를 사용하는 편이 낫다.
