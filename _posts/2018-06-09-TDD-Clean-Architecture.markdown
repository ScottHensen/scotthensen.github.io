---
layout: post
title:  "TDD with Spring"
date:   2018-06-11 12:00:00 -0700
categories: java spring boot rest tdd
---
# An Old Man Tries Test-Driven Development
For a couple of years now, TDD just made sense to me, but it is just not a habit.  I have a ton of green field coding coming up at work, so I wanted to try and get in the habit of working with a TDD mindset.  

For my first attempt, I am coding a little service and front-end as I follow along with Josh and Mario's [Bootiful Testing presentation][bootiful-testing].  I struggled to get the service contract bit working, so for now, I've commented that out.  I'll come back to it after doing some reading.

### Spring Boot Starter  
Start a new spring boot project with the following:
* actuator
* jpa
* web
* h2
* lombok
* cloud contract verifier

### Entity
* Write **PortfolioTest**  

```java  
package com.hensen.toolbox.portfolio.service;

import org.assertj.core.api.Assertions;
import org.junit.Test;

public class PortfolioTest {

	@Test
	public void shouldConstructPortfolio() {

		Portfolio p = new Portfolio(1L, "TSLA", 100);
		Assertions.assertThat(p.getSymbol()).isEqualTo("TSLA");
	}
}
```
* Write **Portfolio**  

```java
package com.hensen.toolbox.portfolio.service;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Entity
@AllArgsConstructor
@NoArgsConstructor
public class Portfolio {

	@Id
	@GeneratedValue
	private Long id;
	private String symbol;
	private int shares;
}
```
### Persisting Entity

* Write **Portfolio JPA Test**

```java
package com.hensen.toolbox.portfolio.service;

import org.assertj.core.api.Assertions;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;
import org.springframework.test.context.junit4.SpringRunner;

@DataJpaTest
@RunWith(SpringRunner.class)
public class PortfolioJpaTest {

	@Autowired
	private TestEntityManager testEntityManager;

	@Test
	public void shouldPersistViaJpa() {

		Portfolio p = this.testEntityManager.persistAndFlush(new Portfolio(null, "TSLA", 100));
		Assertions.assertThat(p.getId()).isNotNull();
		Assertions.assertThat(p.getSymbol()).isNotEmpty();
		Assertions.assertThat(p.getSymbol()).isEqualToIgnoringCase("TSLA");
		Assertions.assertThat(p.getShares()).isEqualTo(100);
	}

}
```

* Write **Portfolio Repository Test**

```java
package com.hensen.toolbox.portfolio.service;

import org.assertj.core.api.Assertions;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.junit4.SpringRunner;

@DataJpaTest
@RunWith(SpringRunner.class)
public class PortfolioRepositoryTest {

	@Autowired
	private PortfolioRepository repository;

	@Test
	public void shouldPersist() {

		Portfolio p = repository.save(new Portfolio(1L, "TSLA", 100));
		Assertions.assertThat(p.getId()).isNotNull();
		Assertions.assertThat(p.getSymbol()).isEqualToIgnoringCase("TSLA");
		Assertions.assertThat(p.getShares()).isEqualTo(100);
	}
}
```

* Write **Portfolio Repository**

```java
package com.hensen.toolbox.portfolio.service;

import org.springframework.data.jpa.repository.JpaRepository;

public interface PortfolioRepository extends JpaRepository<Portfolio, Long> {

}
```

### Controller  
* Write **Portfolio Rest Controller Test**  

```java
package com.hensen.toolbox.portfolio.service;

import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

import java.util.Collections;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mockito;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.context.junit4.SpringRunner;

@WebMvcTest
@RunWith(SpringRunner.class)
public class PortfolioRestControllerTest {

	@MockBean
	private PortfolioRepository portfolioRepository;

	@Autowired
	private MockMvc mvc;

	@Test
	public void givenHttpRequest_whenGetPortfolio_thenAllPortfoliosReturned() throws Exception {

		Mockito.when(this.portfolioRepository.findAll())
				.thenReturn(Collections.singletonList(new Portfolio(null, "TSLA", 100)));

		this.mvc.perform(MockMvcRequestBuilders.get("/portfolios"))
				.andExpect(status().isOk())
				.andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8_VALUE))
				.andExpect(jsonPath("@.[0].symbol").value("TSLA"));
	}
}
```

* Write **Portfolio Rest Controller**

```java
package com.hensen.toolbox.portfolio.service;

import java.util.Collection;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class PortfolioRestController {

	private final PortfolioRepository portfolioRepository;

	public PortfolioRestController(PortfolioRepository portfolioRepository) {
		this.portfolioRepository = portfolioRepository;
	}

	@GetMapping("/portfolios")
	Collection<Portfolio> portfolios() {
		return portfolioRepository.findAll();
	}
}
```

### My source code
[Github: portfolio-service][github-service]  
[Github: portfolio][github-client] (just a stub to verify an http get request)





[bootiful-testing]:https://www.youtube.com/watch?v=k3M_yifZvYg
[github-service]:https://github.com/ScottHensen/portfolio-service
[github-client]:https://github.com/ScottHensen/portfolio
