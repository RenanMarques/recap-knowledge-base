# JPA

## Auditing

Spring Data JPA adds auditing features to your application.

Annotate a `@Configuration` class with the `@EnableJpaAuditing` annotation in order to enable that functionality.

```java
@Configuration
@EnableTransactionManagement
@EnableJpaAuditing
public class DomainConfig() {
  // omitted
}
```

### Created and Last Modified Dates

The `@CreatedDate` and `@LastModifiedDate` annotations keep track of the creation and updating dates on fields annotated with them.

You can specify the `dateTimeProviderRef` to indicate the bean instance that calculates the time when auditing. This is useful for defining the correct format of input in the database.

The following example sets the dateTimeProviderRef with a bean witch generates data for fields with the type `OffsetDateTime`.

```java
@Configuration
@EnableTransactionManagement
@EnableJpaAuditing(dateTimeProviderRef = "auditingDateTimeProvider")
public DomainConfig() {
  @Bean
  public DateTimeProvider auditingDateTimeProvider() {
      return () -> Optional.of(OffsetDateTime.now());
  }
}

@Entity
@Table(name = "foo")
public class Foo {
  @Column
  @CreatedDate
  private OffsetDateTime created;

  @Column
  @LastModifiedDate
  private OffsetDateTime updated;

  // omitted
}
```

