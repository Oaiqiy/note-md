---
title: "Springboot"
date: 2021-07-08T13:26:50Z
draft: false
tags: ["spring","java"]
categories: ["note"]
---

## Unit1

1. Application.java

    * @SpringBootApplication  includes

       1. @SpringBootConfiguration
           >special @Configuration
       2. @EnableAutoConfiguration
           >enable spring boot auto config
       3. @ComponentScan
           >enable component scan  
           >scan something such as @Component @Controller @Service
    * main() Application.run(Application.class, args);

2. ApplicationTest.java
    * test context
    * @RunWith(SpringRunner,.class)
    * @SpringBootTest
    * regard as Application.java

3. Spring Application
    * Spring MVC process web request with controller (MVC -> Model View Controlller)
        > @Controller  
        > @GetMappeing processes HTTP GET request  
        > use Thymeleaf templates path: /src/main/resoureces/templates
    * define view
        > static resources path: /src/main/resources/static/  
    * test controller
        > MockMvc  

## Unit2

1. *lombok* dynamically generate getter, setter and so on  
   @Data
   @RequiredArgsConstructor  
   @Slf4j  automatic generate SLF4J logger  
2. @RequestMapping general processing request uesd for class *@RequestMapping(method=RequestMethod.GET)*
   @GetMapping @PostMapping @PutMapping @DeleteMapping @PatchMapping used for method
3. design view include JavaServer Pages\Thymeleaf\FreeMarker\Mustache\template based on Groovy

    * Thymeleaf  
    @{} is relative path /src/main/resource/static  
    image `<img th:src="@{}"/>`  
    `<p th:text="${name}"></p>` name is a attribute of Model  
    `<div th:each="element : ${test}><div>` repeat redering  
    `<input name="output" type="test" th:value"${}>` input data
    `<form method="POST" th:object="${design"}>` form data

4. process form submit
    * when submit form, data will be binded with object.
    * `redirect:` redirect prefix
    * `<form>` action attribute used to specify URL  
      e.g. `<form th:action="@{test}">`
5. check form input
    * use Bean Validation API\Hibernate Validattor
    * @NOTNull not null
    * @NOTNull not null and not 0
    * @Size(min=`int`, message=`String`) control size
    * @CreditCardNumber check credit card number from *hibernate validattor*
    * @Pattern regular expression
    * @Valid the object need check after blind before use the method  
    e.g.

    ```java
    @PostMapping
    public String processOrder(@Valid Order order, Errors errors) {
    if (errors.hasErrors()) {
      return "orderForm";
    }
    
    log.info("Order submitted: " + order);
    return "redirect:/";
    }
    ```

6. show check error

    ```html
            <label for="name">Name: </label>
        <input type="text" th:field="*{name}"/>
    <!-- end::allButValidation[] -->
        <span class="validationError"
                th:if="${#fields.hasErrors('name')}"
                th:errors="*{name}">Name Error</span>
    ```

7. view controller

    ```java
    package tacos.web;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
    import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

    @Configuration
    public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }
    }
    ```

8. cache templates
    * application.properties `spring.thymeleaf.cache=false` disable cache

## Unit3

1. JDBC
   1. JdbcTemplate
       >@Repository  
       >@Autowired

       ```java
       @Override
       public Ingredient findOne(String id) {
           return jdbc.queryForObject(
               "select id, name, type from Ingredient where id=?",
               new RowMapper<Ingredient>() {
               public Ingredient mapRow(ResultSet rs, int rowNum) 
                   throws SQLException {
                   return new Ingredient(
                       rs.getString("id"), 
                       rs.getString("name"),
                       Ingredient.Type.valueOf(rs.getString("type")));
               };
               }, id);
       }
       ```

   2. insert
       1. insert data use Jdbc.update()

           ```java
           @Override
           public Ingredient save(Ingredient ingredient) {
               jdbc.update(
                   "insert into Ingredient (id, name, type) values (?, ?, ?)",
                   ingredient.getId(), 
                   ingredient.getName(),
                   ingredient.getType().toString());
                   return ingredient;
           }
           ```

       2. complex insert
       update() recevie a PrepareStatementCreator and a KeyHolder. The key Holder will provide the key content.

           ```java
           @Override
           public Taco save(Taco taco) {
               long tacoId = saveTacoInfo(taco);
               taco.setId(tacoId);
               for (Ingredient ingredient : taco.getIngredients()) {
               saveIngredientToTaco(ingredient, tacoId);
               }

               return taco;
           }

           private long saveTacoInfo(Taco taco) {
               taco.setCreatedAt(new Date());
               PreparedStatementCreator psc =
                   new PreparedStatementCreatorFactory(
                       "insert into Taco (name, createdAt) values (?, ?)",
                       Types.VARCHAR, Types.TIMESTAMP
                   ).newPreparedStatementCreator(
                       Arrays.asList(
                           taco.getName(),
                           new Timestamp(taco.getCreatedAt().getTime())));

               KeyHolder keyHolder = new GeneratedKeyHolder();
               jdbc.update(psc, keyHolder);

               return keyHolder.getKey().longValue();
           }

           private void saveIngredientToTaco(
                   Ingredient ingredient, long tacoId) {
               jdbc.update(
                   "insert into Taco_Ingredients (taco, ingredient) " +
                   "values (?, ?)",
                   tacoId, ingredient.getId());
           }
           ```

       3. @SessionAttributes("`parameter`") add session attribute
       @SeesionAttribute take out
       SessionStatus.setComplete() clear
       @ModelAttribute add and take out

       4. SimpleJdbcInsert  

           ```java
               @Override
               public Order save(Order order) {
                   order.setPlacedAt(new Date());
                   long orderId = saveOrderDetails(order);
                   order.setId(orderId);
                   List<Taco> tacos = order.getTacos();
                   for (Taco taco : tacos) {
                   saveTacoToOrder(taco, orderId);
                   }

                   return order;
               }

               private long saveOrderDetails(Order order) {
                   @SuppressWarnings("unchecked")
                   Map<String, Object> values =
                       objectMapper.convertValue(order, Map.class);

                   values.put("placedAt", order.getPlacedAt());
                   //Not compatible

                   long orderId =
                       orderInserter
                           .executeAndReturnKey(values)
                           .longValue();
                   return orderId;
               }

               private void saveTacoToOrder(Taco taco, long orderId) {
                   Map<String, Object> values = new HashMap<>();
                   values.put("tacoOrder", orderId);
                   values.put("taco", taco.getId());
                   orderTacoInserter.execute(values);
               }
           ```

2. JPA *Java Persistence API*
    1. @Entity identify entity  
    @Id primary key  
    @GeneratedValue  
    @ManyToMany  declare a new table
    @Table(name=`String`)  
    @PrePersist before persistence execute the method  
    2. repository  
    extends CrudREpository<`ObjectType`,`IdType`>  
    3. customize JPA repository  
    through method name define a operation  

    ```java
    List<Order> readOrderByDeliveryZipAndPlacedAtBetween(String deliveryZip, Date startDate, date endDate);
    ```

    `read``Order``By``DeliveryZip``And``PlacedAt``Between`

    @Query("`sql string`")

## Unit4

1. configure Spring Security

    ```java
    @Configuration
    @EnableWebSecurity
    public class SecurityConfig extends WebSecurityConfigurerAdapter {

    }
    ```

    user information  storage  

    1. based on memory  

        ```java
        @Override
        protected void configure(AuthenticationManagerBuilder auth)
            throws Exception {
            auth
            .inMemoryAuthentication()
                .withUser("buzz")
                .password("infinity")
                .authorities("ROLE_USER")
                .and()
                .withUser("woody")
                .password("bullseye")
                .authorities("ROLE_USER");
            
        }  
        ```

    2. based on JDBC

        spring security provide database define  
        table users: username password enabled  
        table authority: username authority  
        table groups: id name  
        table group_members: group_id name  
        table group_authorities: group_id authority  

        customize user information query

        ```java
            @Autowired
            dataSource dataSource;
            protected void configure(AuthenticationManagerBuilder auth)
            throws Exception {

            auth
            .jdbcAuthentication()
                .dataSource(dataSource)
                .usersByUsernameQuery(
                    "select username, password, enabled from Users " +
                    "where username=?")
                .authoritiesByUsernameQuery(
                    "select username, authority from UserAuthorities " +
                    "where username=?");

            }
            ```

        Query basic agreement: the only parameter is **username**

        transcode: .passwordEncoder(`Stirng`)

    3. LDAP as back end *Lightweight Directory Access Protocol*

         ```java
            @Override
            protected void configure(AuthenticationManagerBuilder auth)
                throws Exception {
                auth
                .ldapAuthentication()
                    .userSearchFilter("(uid={0})")
                    .groupSearchFilter("member={0}");
            }
        ```

    4. customize user authenticate  *Spring Data repository*
        1. define a entity *User* implements UserDetails

            ```java
            @Override
            public Collection<? extends GrantedAuthority> getAuthorities() {
                return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
            }
            ```

            every user has the authority *ROLE_USER*
        2. define a repository interface extends CrudRepository<User, Long>
            define a method *findByUsername*(`String`)
        3. create user details service

            ```java
            @Service
            public class UserRepositoryUserDetailsService 
                    implements UserDetailsService {

            private UserRepository userRepo;

            @Autowired
            public UserRepositoryUserDetailsService(UserRepository userRepo) {
                this.userRepo = userRepo;
            }
            
            @Override
            public UserDetails loadUserByUsername(String username)
                throws UsernameNotFoundException {
                User user = userRepo.findByUsername(username);
                if (user != null) {
                return user;
                }
                throw new UsernameNotFoundException(
                                "User '" + username + "' not found");
            }

            }
            ````

            add config

            ```java
            @Autowired
            private UserDetailsService userDetailsService;

            @Bean
            public PasswordEncoder encoder() {
                return new StandardPasswordEncoder("53cr3t");
            }

            @Override
            protected void configure(AuthenticationManagerBuilder auth)
                throws Exception {

                auth
                .userDetailsService(userDetailsService)
                .passwordEncoder(encoder());

            }
            ```

        4. register

2. protect Web request

    ```java
     @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
        .authorizeRequests()
            .antMatchers("/design", "/orders")
            .access("hasRole('ROLE_USER')")
            .antMatchers("/", "/**").access("permitAll")
            //access() + SpEL

        .and()
            .formLogin()
            .loginPage("/login")

        .and()
            .logout()
            .logoutSuccessUrl("/")

        .and()
            .csrf()
            .ignoringAntMatchers("/h2-console/**")

        .and()
            .headers()
            .frameOptions()
                .sameOrigin()

        ;
    }
    ```

    know who the user is

    1. add parameter Principal  
    use principal.getName() to get userName
    2. add parameter Authentication  
    use authentication.getPrincipal() to get a Object and then use coercion to get User
    3. add parameter @AuthenticationPrincipal User user
    4. can be used in anywhere

        ```java
        Authentication authentcation = SecuurityContextHolder.getContext().getAuthentication();
        User user = (User) authentication.getPrincipal();
        ```

## Unit5

1. configure
    > application.properties / application.yml

    1. Servlet port

        ```yaml
        server:
            port:9090
        ```

    2. datasource

        ```yaml
        spring:
            datasource:
                url: jdbc:mysql://localhost/table
                username: root
                password: root
                schema:
                    - name-schema.sql
                data:
                    - name.sql
        ```

    3. log

        ```yml
        logging:
            file: path
            level:
                root: WARN
                org:
                    springframework:
                        scurity: DEBUG
        ```

    4. create specific attribute

        ```yml
        test:
            example: ${spring.application.name}
        ```

    5. customize configuration attribute  
        @ConfiguraionProperties(prefix=`String`)

        configure meta data `src/main/resources/META-INF`

        ```json
            {
                "properties": [
                    {
                        "name":"taco.orders.page-size",
                        "type":"java.lang.String",
                        "descrption":"lalala"                    
                    }
                ]
            }
        ```

    6. profile  

    application-{profile name}.yml

    --spring.profiles.active={profile name}

    @Profile({profile name}) before method of class definition


