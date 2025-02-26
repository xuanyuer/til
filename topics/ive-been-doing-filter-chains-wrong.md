# I've been doing Filter Chains wrong

For anyone who's ever worked on a Spring Boot project, especially one that involves Spring Security, it's likely that you'd encounter a filter chain bean getting created. Let's say, for an application, we want to keep some endpoints unsecured and everything else secured:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity httpSecurity) throws Exception {
    return httpSecurity
            .authorizeHttpRequests(registry ->
                    registry
                        .requestMatchers("/monitoring", "/swagger-ui").permitAll()
                        .requestMatchers(EndpointRequest.toAnyEndpoint()).permitAll()
                        .anyRequest.authenticated())
            .build();
}
```

In the above example, we can see that the intention is for any request to the `/monitoring`, `/swagger-ui` or Spring's actuator endpoints should be permitted without any sort of authentication, and everything else has to be authenticated. Things get a little more complicated when I introduce, say, a custom filter to the mix. Let's say I add a Filter that validates some custom headers:

```java
public class MyFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws Exception {
        String foo = request.getHeader("foo");

        if (StringUtils.isBlank(foo) || !foo.equals("Foo")) {
            throw IllegalArgumentException("You need to Foo!");
        }

        filterChain.doFilter(request, response);
    }
}
```

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity httpSecurity, MyFilter myFilter) throws Exception {
    return httpSecurity
            .authorizeHttpRequests(registry ->
                    registry
                        .requestMatchers("/monitoring", "/swagger-ui").permitAll()
                        .requestMatchers(EndpointRequest.toAnyEndpoint()).permitAll()
                        .anyRequest.authenticated())
            .addFilterAfter(myFilter, AuthorizationFilter.class)
            .build();
}
```

When we do the above, what happens is that even when we call `/monitoring` or `/swagger-ui`, we end up triggering `MyFilter`. This happens because `authorizeHttpRequests` only customizes the behaviour of `AuthorizationFilter` in Spring's filter chain, and NOT the filter chain itself. If we don't want `MyFilter` to execute, we'll need to alter the behaviour of `MyFilter`. In my experience, we usually have people doing this:

```java
public class MyFilter extends OncePerRequestFilter {

    @Override
    protected boolean shouldNotFilter(HttpServletRequest request) throws Exception {
        return request.getServletPath().equals("/monitoring")
            || request.getServletPath().equals("/swagger-ui")
            || request.getServletPath().startsWith("/actuator");
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws Exception {
        String foo = request.getHeader("foo");

        if (StringUtils.isBlank(foo) || !foo.equals("Foo")) {
            throw IllegalArgumentException("You need to Foo!");
        }

        filterChain.doFilter(request, response);
    }
}
```

You can see that for the above, we definite a set of rules to tell Spring not to filter when the request mathces some URL patterns. The above example is simplistic, but things get complicated when the request has to match various patterns of the defined paths. You'd have to maintain a list of these, and apply them to BOTH your filter chain AND the custom filter (or even multiple filters!). This is not fun, and not very maintable in my opinion.

Thankfully, I learnt something today. I learnt that you can have multiple filter chains, and each chain executes based on YOUR rules.

```java
@Bean
@Order(1)
public SecurityFilterChain unsecuredFilterChain(HttpSecurity httpSecurity) throws Exception {
    return httpSecurity
            .securityMatchers(configurer ->
                    configurer.requestMatchers(EndpointRequest.toAnyEndpoint())
                              .requestMatchers("/monitoring", "/swagger-ui"))
            .authorizeHttpRequests(registry -> registry.anyRequest.permitAll())
            .build();
}

@Bean
@Order(2)
public SecurityFilterChain securedFilterChain(HttpSecurity httpSecurity, MyFilter myFilter) throws Exception {
    return httpSecurity
            .securityMatchers(AbstractREquestMatcherRegistry::anyRequest)
            .authorizeHttpRequests(registry -> registry.anyRequest.authenticated())
            .addFilterAfter(myFilter, AuthorizationFilter.class)
            .build();
}
```

You can even give chains a priority, as shown above using `@Order`, so Spring would evaluate one chain over another. This means that if the preceeding chain already matches the request, then subsequent chains will be skipped. In the above example, if my request path is `/monitoring`, the `unsecuredFilterChain` will be evaluated first and it would match the request. It then only uses the unsecured filter chain to authenticate, without `MyFilter` every coming into the picture. However, if I had another request with a path of `/foo`, then `unsecuredFilterChain` would not match, but `securedFilterChain` would match and would also bring in `MyFilter`.

Now we no longer have to worry about filters executing when they shouldn't. We also don't have to maintain a set of endpoints for different scenerios and have them evaludated in each individual filter. Life is better now.

Happy chaining everyone!
