## 스프링부트 설정.
### 프로퍼티를 스캔하는 설정
```
//application.yml
server:
  port: 20000
  servlet:
    contex-path: /app
```
```
//database.yml
spring:
  datasource:
    jdbcUrl: jdbc:postgresql://localhost:5432/mydb
    username: postgres
    password: 1234
    driver-class-name: org.postgresql.Driver
```

```
package com.hive.ansj

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.builder.SpringApplicationBuilder
import org.springframework.boot.context.properties.ConfigurationPropertiesScan
import org.springframework.boot.runApplication

@SpringBootApplication
@ConfigurationPropertiesScan
class AnsjApplication

fun main(args: Array<String>) {
    val appHome = System.getenv("APP_HOME")
    SpringApplicationBuilder(AnsjApplication::class.java)
        .profiles("development")
        .headless(false)
        .properties(
            "spring.config.location="+
                    "file:$appHome/properties/database.yml" +
                    ", file:$appHome/properties/application.yml"
        ).application().run(*args)
}
```
#### ConfigurationPropertiesScan
- 어노테이션명 그대로 ConfigurationProperties를 스캔한다.
#### SpringApplicationBuilder
- 이것을 이용하면 배너와 실행설정 등을 커스터마이징할 수 있다.

```
package com.hive.ansj.config

import org.springframework.boot.context.properties.ConfigurationProperties
import org.springframework.boot.jdbc.DataSourceBuilder
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.data.jpa.repository.config.EnableJpaRepositories
import org.springframework.orm.jpa.JpaTransactionManager
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter
import javax.sql.DataSource

@Configuration
@EnableJpaRepositories(
    basePackages = ["com.hive.ansj.repository"],
    entityManagerFactoryRef = "an",
    transactionManagerRef = "imsi"
)
class DataContextConfig {
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    fun dataSource(): DataSource = DataSourceBuilder.create().build()

    @Bean
    fun entityManager(): LocalContainerEntityManagerFactoryBean =
        (LocalContainerEntityManagerFactoryBean()).apply{
            dataSource = dataSource()
            setPackagesToScan("com.hive.ansj.entity")
            jpaVendorAdapter = HibernateJpaVendorAdapter()
        }

    @Bean
    fun transactionManager() = JpaTransactionManager(entityManager().`object`!!)
}
```

#### Hibernate
hibernate를 사용하기 위해서는 configuration에 2가지가 등록되어야 함

LocalSessionFactoryBean: SessionFactory에 대한 Factory Bean.. SessionFactory를 생성하는 객체를 등록시켜 줌 Spring에서 사용할 DataSource와 Entity가 위치한 Package들에 대한 검색을 모두 포함하게  됨

HibernateTransactionManager: PlatformTransactionManager를 구현한 Hibernate용 TransactionManager를 등록해야지 됨, Spring에서 @EnableTransactionManager와 같이 사용되어 @Transactional annotation을 사용할 수 있게 됨.

#### JPA
Spring에서 Hibernate기반의 JPA를 사용하기 위해서는 다음 Bean이 필요함

LocalContainerEntityManagerFactoryBean: SessionFactoryBean과 동일한 역할을 함. DataSource와 Hibernate Property, Entity가 위치한 Package를 지정한다. Hibernate 기반으로 동작하는 것을 지정해 JpaVendor를 설정해야 함

HibernateJpaVendorAdaptor: Hibernate Vendor 와 JPA간의 Adaptor를 설정함.

JpaTransactionManager: DataSource와 EntityManagerFactoryBean에서 생성되는 EntityManagerFactory를 지정하는 Bean이다.

