```java
package com.springsource.html5expense.config;

import java.util.HashMap;
import java.util.Map;

import javax.sql.DataSource;

import org.apache.commons.collections.map.HashedMap;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ImportResource;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;
import org.springframework.jdbc.datasource.SimpleDriverDataSource;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.JpaVendorAdapter;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.Database;
import org.springframework.orm.jpa.vendor.HibernateJpaDialect;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import org.springframework.web.multipart.MultipartResolver;
import org.springframework.web.multipart.commons.CommonsMultipartResolver;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

import com.springsource.html5expense.controller.ExpenseController;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.serviceImpl.JpaExpenseServiceImpl;

/**
 * Configuration for application @Components such as @Services, @Repositories, and @Controllers.
 * Loads externalized property values required to configure the various application properties.
 * Not much else here, as we rely on @Component scanning in conjunction with @Inject by-type autowiring.
 *
 */
@Configuration
@EnableTransactionManagement
@PropertySource("/config.properties")
@ImportResource({ "classpath:spring-security.xml"})
@ComponentScan(basePackageClasses = {JpaExpenseServiceImpl.class,ExpenseController.class,Expense.class})

public class ComponentConfig {
	
	@Autowired
	Environment env;
	@Bean
    public DataSource dataSource()  {
        SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
        dataSource.setUrl(String.format("jdbc:postgresql://%s:%s/%s", env.getProperty("database.host"), env.getProperty("database.port"),
        		    env.getProperty("database.name")));
        dataSource.setDriverClass(org.postgresql.Driver.class);
        dataSource.setUsername(env.getProperty("database.username"));
        dataSource.setPassword(env.getProperty("database.password"));
        return dataSource;
    }
	
	@Bean
	public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
	    LocalContainerEntityManagerFactoryBean emfb = new LocalContainerEntityManagerFactoryBean();
	    emfb.setJpaVendorAdapter( jpaAdapter());
	    emfb.setDataSource(dataSource());
	    emfb.setJpaPropertyMap(createPropertyMap());
	    emfb.setJpaDialect(new HibernateJpaDialect());
	    emfb.setPersistenceUnitName("sample");
	    emfb.setPackagesToScan(new String[]{Expense.class.getPackage().getName()});
	    return emfb;
	}
	
	public Map<String,String> createPropertyMap()
	{
		Map<String,String> map= new HashedMap();
		map.put(org.hibernate.cfg.Environment.HBM2DDL_AUTO, "create");
		map.put(org.hibernate.cfg.Environment.HBM2DDL_IMPORT_FILES, "import.sql");
		map.put("hibernate.c3p0.min_size", "5");
		map.put("hibernate.c3p0.max_size", "20");
		map.put("hibernate.c3p0.timeout", "360000");
		map.put("hibernate.dialect", "org.hibernate.dialect.PostgreSQLDialect");
		return map;
		
	}
	
	@Bean
	public JpaVendorAdapter jpaAdapter() {
	    HibernateJpaVendorAdapter hibernateJpaVendorAdapter = new HibernateJpaVendorAdapter();
	    hibernateJpaVendorAdapter.setShowSql(true);
	    hibernateJpaVendorAdapter.setDatabase(Database.POSTGRESQL);
	    hibernateJpaVendorAdapter.setShowSql(true);
	    hibernateJpaVendorAdapter.setGenerateDdl(true);
	    return hibernateJpaVendorAdapter;
	}


	@Bean
	public PlatformTransactionManager transactionManager() {
	    final JpaTransactionManager transactionManager =
	        new JpaTransactionManager();

	    transactionManager.setEntityManagerFactory(entityManagerFactory().getObject());
	    Map<String,String> jpaProperties = new HashMap<String,String>();
	    jpaProperties.put("transactionTimeout","43200");
	    transactionManager.setJpaPropertyMap(jpaProperties);

	    return transactionManager;
	}
	
	@Bean
	public MultipartResolver multipartResolver(){
		CommonsMultipartResolver multipartResolver = new org.springframework.web.multipart.commons.CommonsMultipartResolver();
		multipartResolver.setMaxUploadSize(10000000);
		return multipartResolver;
	}
	
	@Bean
    public InternalResourceViewResolver internalResourceViewResolver() {
        InternalResourceViewResolver internalResourceViewResolver = new InternalResourceViewResolver();
        internalResourceViewResolver.setPrefix("/WEB-INF/views/");
        internalResourceViewResolver.setSuffix(".jsp");
        return internalResourceViewResolver;
    }
	
}
```

```java
package com.springsource.html5expense.config;

import java.util.List;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.MessageSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.context.annotation.PropertySource;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.context.support.ResourceBundleMessageSource;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.HandlerMethodReturnValueHandler;
import org.springframework.web.multipart.commons.CommonsMultipartResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

import com.springsource.html5expense.controller.ExpenseController;
import com.springsource.html5expense.model.Expense;
import com.springsource.html5expense.serviceImpl.JpaExpenseServiceImpl;


@Configuration
@Import(ComponentConfig.class)
@PropertySource("/config.properties")
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter
 {
     @Override
     public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {

     }

     @Value("${debug}")
    private boolean debug;

    private int maxUploadSizeInMb = 20 * 1024 * 1024;

    @Bean(name = "multipartResolver")
    public CommonsMultipartResolver commonsMultipartResolver() {
        CommonsMultipartResolver commonsMultipartResolver = new CommonsMultipartResolver();
        commonsMultipartResolver.setMaxUploadSize( maxUploadSizeInMb  );
        return commonsMultipartResolver;
    }

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    @Bean
    public MessageSource messageSource() {
        String[] baseNames = "messages,errors".split(",");
        ResourceBundleMessageSource resourceBundleMessageSource = new ResourceBundleMessageSource();
        resourceBundleMessageSource.setBasenames(baseNames);
        return resourceBundleMessageSource;
    }

    @Bean
    public static PropertySourcesPlaceholderConfigurer placeholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }

    @Override
    public void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> returnValueHandlers) {

    }

}
```
