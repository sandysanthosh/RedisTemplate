To use Redis cache in your Spring Boot project with Spring Data Redis, follow these steps:

### 1. Add Dependencies

First, you need to add the required dependencies to your `pom.xml` (for Maven) or `build.gradle` (for Gradle).

**Maven:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

**Gradle:**
```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
implementation 'org.springframework.boot:spring-boot-starter-cache'
```

### 2. Configure Redis

Configure Redis in your `application.properties` or `application.yml` file.

**application.properties:**
```properties
spring.redis.host=localhost
spring.redis.port=6379
```

**application.yml:**
```yaml
spring:
  redis:
    host: localhost
    port: 6379
```

### 3. Enable Caching in Spring Boot

Enable caching by adding the `@EnableCaching` annotation to one of your configuration classes.

```java
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableCaching
public class CacheConfig {
}
```

### 4. Define a Redis Cache Manager

You can customize the Redis cache manager as needed. Here is an example:

```java
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.redis.RedisCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10))  // Set cache expiration
                .disableCachingNullValues();
        
        return RedisCacheManager.builder(connectionFactory)
                .cacheDefaults(config)
                .build();
    }
}
```

### 5. Use Caching Annotations

Use caching annotations like `@Cacheable`, `@CachePut`, and `@CacheEvict` in your service methods.

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @Cacheable(value = "users", key = "#userId")
    public User getUserById(Long userId) {
        // Method implementation to fetch user from database
        return userRepository.findById(userId).orElse(null);
    }
}
```

- `@Cacheable`: Caches the result of the method.
- `@CachePut`: Updates the cache with the method result.
- `@CacheEvict`: Removes the entry from the cache.

### 6. Test Your Configuration

Run your Spring Boot application and test the caching behavior. Verify that the method is not being called repeatedly and that the data is being served from the cache.

### Example Project Structure

```
src/
├── main/
│   ├── java/
│   │   ├── com/
│   │   │   ├── example/
│   │   │   │   ├── config/
│   │   │   │   │   └── RedisConfig.java
│   │   │   │   ├── service/
│   │   │   │   │   └── UserService.java
│   │   │   │   └── Application.java
│   ├── resources/
│   │   ├── application.properties
```

By following these steps, you can successfully integrate Redis caching into your Spring Boot project.
