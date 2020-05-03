Feign 的使用与踩过的坑
---

#### 一、Feign 的使用

​        Feign 是一个声明式的伪 RPC 的 REST 客户端，它用了基于接口的注解方式，很方便的客户端配置，刚开始使用时还不习惯，感觉是在客户端写服务端的代码，Spring Cloud 给 Feign 添加了支持 Spring MVC 注解，并整合 Ribbon 及 Eureka 进行支持负载均衡、Hystrix 进行熔断回调，本质上还是通过**注册中心**和**网关**找到其他服务的公开接口并通过 Http 请求调用，获取并解析返回报文。

Feign 的使用很简单，有以下几步：

1、添加依赖。

2、启动类添加 @EnableFeignClients 注解支持。

3、建立 FeignClient 接口，并在接口中定义需调用的服务方法。

4、完善 Fallback 对象，获取 FeignClient 返回的报错信息。

5、使用 FeignClient 接口，Bean 注入 FeignClient 接口即可。

```xml
<!-- 1.添加依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
//================================================================
//  2.打开注解
//================================================================
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class FeignClientApplication {
 
    public static void main(String[] args) {
        SpringApplication.run(FeignClientApplication.class, args);
    }
}
```

```java
//================================================================
//  3.建立 FeignClient 接口，并在接口中定义需调用的服务方法
//================================================================
import org.hzero.luwx.api.controller.v1.dto.FeignCommonResponseDTO;
import org.hzero.luwx.api.controller.v1.dto.OrderDetailDTO;
import org.hzero.luwx.domain.vo.OrderVO;
import org.hzero.luwx.infra.feign.fallback.DemoFeignFallBack;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestParam;

@FeignClient(value = "demo-service", path = "/v1/demos", fallbackFactory = DemoFeignFallBack.class)
public interface DemoFeign {

    @GetMapping("/{organizationId}/list")
    FeignCommonResponseDTO<OrderVO> list(@RequestBody final OrderDetailDTO detailDTO, @RequestParam String demo);
}

@Data
public class FeignCommonResponseDTO<T> {
    /**
     * 失败
     */
    private Boolean failed;

    /**
     * 错误消息
     */
    private String message;

    /**
     * ResponseEntity 中的 body 属性，获取返回的真正数据
     */
    private T body;

    public FeignCommonResponseDTO() {
        this.failed = false;
    }
    
    public FeignCommonResponseDTO(T body) {
        this.body = body;
    }
}
```

```java
//================================================================
//  4.完善 Fallback 对象，获取 FeignClient 返回的报错信息
//================================================================
import feign.hystrix.FallbackFactory;
import org.hzero.luwx.api.controller.v1.dto.FeignCommonResponseDTO;
import org.hzero.luwx.api.controller.v1.dto.OrderDetailDTO;
import org.hzero.luwx.domain.vo.OrderVO;
import org.hzero.luwx.infra.feign.DemoFeign;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

@Component
public class DemoFeignFallBack implements FallbackFactory<DemoFeign> {

    private static final Logger LOG = LoggerFactory.getLogger(DemoFeignFallBack.class);

    @Override
    public DemoFeign create(Throwable throwable) {
        return new DemoFeign() {
            @Override
            public FeignCommonResponseDTO<OrderVO> list(OrderDetailDTO detailDTO, String demo) {
                LOG.error("Feign error fallback [method: list()], cause by ", throwable);
                final FeignCommonResponseDTO<OrderVO> response = new FeignCommonResponseDTO<>();
                response.setFailed(true);
                response.setMessage(throwable.getMessage());
                return response;
            }
        };
    }
}
```

#### 二、定义与规范

##### 1、Feign

- 方法名可以与被调接口不一致，但 **url 必须完全一致且是全路径**（一般是 “/v1” 起头）。
- 单参数没有限制，但是**多参数**必须加上 **@RequestParam("xxx")**，参数个数可以与被调接口给的不同，但必传字段必须相同。Spring 会自动组装匹配被调接口的参数。
- 对象必须加上 **@RequestBody** 而且**只能存在一个**，**复杂对象参数会强行将 GET 转换成 POST 请求**，有可能导致接收方出现请求不匹配的错误。对象名可以不同但是**属性字段**必须相同，Spring 会自动组装匹配被调接口的对象。
- 返回值按照接口给出的返回值来确定，一般的被调接口返回的都是 ResponseEntity，使用示例中通用的返回接收 DTO 既可以获取到需要的返回数据对象，也可以获取到报错信息。被调方法的返回值类型可以与主调方法不一致，只需要属性字段相同就会做适当匹配转换。甚至可以用 String 接收一切返回值，再做解析。

##### 2、Fallback

- 必须加上 **@Component** 注解。
- 规范的写法是示例中的 return new DemoFeign()，实现每个方法的回调方法，打出对应的日志。

#### 三、HZero 与 Feign

问题：Job 中包含 Feign 调用，在该项目所依赖的包的环境下，被调服务无法接收到主调服务拓传的 token 等信息，获取 tenantId 为空。 

思路：首先，Job 由 Scheduler 服务统一调用，需要在 Scheduler 的 bootstrap.yml 配置中**打开分享上下文权限** hystrix.shareSecurityContext:  true，Job 服务中应该就能获取到 Scheduler 传过去的 token。其次，因为 Feign 调用会获取线程池中的另一个线程进行调用，为了获得拓传的 token，需要在**主调服务**中依赖相应的包，主调服务也需要打开 hystrix 的分享上下文权限，保险起见还需要添加 Feign 通用拦截器（附-1）。最后逐步调式分析空指针产生原因。

原理：主调服务打开 hystrix.shareSecurityContext 并添加拦截器后，一切 Feign 调用方法都会在发送的 Http 请求头部拼接 token 信息，由被调服务自动解析头部 token（前提是主调服务能获取到 token）。这就和中台用户登录以后请求后端接口传 token 是一个道理。

解决方法：      

1. 主调服务依赖 org.hzero.starter:hzero-starter-feign-replay:1.0.0.RELEASE 包，bootstrap.yml 中打开hystrix.shareSecurityContext:  true，Scheduler 服务也开启该权限。     
2. 在主调服务、拦截器、被调服务中打断点逐步排查问题，主调服务中能获取到 customerDetails 和 token，但是 hzero-starter-feign-replay 包的 JwtRequestInterceptor 拦截器中获取不到 customerDetails，只能获取token，这样被调服务当然获取不到。
3. Job 不行，则改用 postman 直接封装 token 来访问 controller，调用 Feign，拦截器、被调服务中都可以获取到 customerDetails，说明拓传没有问题，则确定是 Job 匿名用户 userId = -1 导致 customerDetails 为空。



#### 附-1

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import feign.RequestInterceptor;
import feign.RequestTemplate;
import io.choerodon.core.oauth.CustomUserDetails;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.jwt.JwtHelper;
import org.springframework.security.jwt.crypto.sign.MacSigner;
import org.springframework.security.jwt.crypto.sign.Signer;
import org.springframework.security.oauth2.provider.authentication.OAuth2AuthenticationDetails;

import javax.annotation.PostConstruct;
import java.util.Collections;

/**
 * <p>
 * fegin + oauth2 调用，实现token传递
 * </p>
 *
 * @author 黄晟 2019/05/08 12:42 PM
 */
@Configuration
public class FeignRequestInterceptor implements RequestInterceptor {
    private static final Logger LOGGER = LoggerFactory.getLogger(FeignRequestInterceptor.class);
    private static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();
    private static final String OAUTH_TOKEN_PREFIX = "Bearer ";
    private Signer signer;
    private CustomUserDetails defaultUserDetails;

    @PostConstruct
    private void init() {
        this.signer = new MacSigner("choerodon");
        this.defaultUserDetails = new CustomUserDetails("default", "unknown", Collections.emptyList());
        this.defaultUserDetails.setUserId(0L);
        this.defaultUserDetails.setOrganizationId(0L);
        this.defaultUserDetails.setLanguage("zh_CN");
        this.defaultUserDetails.setTimeZone("CCT");
    }

    @Override
    public void apply(RequestTemplate template) {
        try {
            String token = null;
            if (SecurityContextHolder.getContext() != null && SecurityContextHolder.getContext().getAuthentication() != null && SecurityContextHolder.getContext().getAuthentication().getDetails() instanceof OAuth2AuthenticationDetails) {
                OAuth2AuthenticationDetails details = (OAuth2AuthenticationDetails) SecurityContextHolder.getContext().getAuthentication().getDetails();
                if (details.getTokenType() != null && details.getTokenValue() != null) {
                    token = details.getTokenType() + " " + details.getTokenValue();
                } else if (details.getDecodedDetails() instanceof CustomUserDetails) {
                    token = "Bearer " + JwtHelper.encode(OBJECT_MAPPER.writeValueAsString(details.getDecodedDetails()), this.signer).getEncoded();
                }
            }

            if (token == null) {
                token = "Bearer " + JwtHelper.encode(OBJECT_MAPPER.writeValueAsString(this.defaultUserDetails), this.signer).getEncoded();
            }

            template.header("Jwt_Token", new String[]{token});
//            this.setLabel(template);
        } catch (Exception var4) {
            LOGGER.error("generate jwt token failed {}", var4);
        }

    }

    private void setLabel(RequestTemplate template) {
//        if (HystrixRequestContext.isCurrentThreadInitialized()) {
//            String label = (String) RequestVariableHolder.LABEL.get();
//            if (label != null) {
//                template.header("X-Eureka-Label", new String[]{label});
//            }
//        }

    }
}
```



