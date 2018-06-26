---
layout: post
title:  "Stock Portfolio Web App"
date:   2018-06-20 12:00:00 -0700
categories: java spring boot tdd clean architecture spock
---
# Beating a Dead Horse
My last three posts have used the same theme -- getting some quotes from a third-party service, caching them, serving them from the cache and displaying them.  Instead of just leaving these pieces as little more than a proof-of-concept, I thought I'd add another couple posts on this theme to build out a functioning web app.  This will let me continue to work on my TDD and add BDD tests with Spock.  If I don't get too bored or distracted, I would also like to go through the exercises of testing the front end with Selenium (maybe with GEB on top).

*[I spent some time thinking about building this app in a real-world adaptation of Uncle Bob's Clean Architecture, but I got bogged down with the optimal project structure and with identifying which ideas of Bob's would be confusing/ignored by support team programmers.  I'm going to let it roll around in my head a little longer and tackle that in a future post, so this project will adhere to more conventional patterns, but it will make use of some of Bob's ideas.]*  


### Mock of the web page
Here's a rough idea of what I am planning to build.  

![Quotes Front-End Mock]({{site.baseurl}}/img/2018-06-14/mock.JPG "Quotes Front-End Mock")

It will handle three actions:
1. Add a security (by entering a symbol in the symbol column)
2. Remove a security (by clicking the [-] button)
3. Calculate value per shares (by entering the number of shares)


### First, the Service:  
**Use Cases**
* Get Quote for a given Symbol
* Get All Securities we're following
* Get Quotes for Each Security in our List
* Add Security to our List
* Delete a Security from our List
* Calculate Value of given Share Quantity

**Spring Starter Project... portfolio-svc**  
Start a new spring boot project with the following:
* web
* jpa
* redis
* h2
* actuator
* lombok

**Add Spock and Jacoco**  
I want to run my unit tests with jUnit, but I plan to test my use cases with Spock, so we need to add its dependencies to our gradle.build.  *[I am borrowing the dependencies from Geoffroy Warin's [spring-spock-mvc github repo][github-geowarin].]*  

Spock requires Groovy, so I also added the Eclipse Groovy Development Tools Plug-in via the Eclipse MarketPlace [From Eclipse IDE: Help -> Eclipse MarketPlace -> Find: groovy.  Install Groovy Development Tools.].  

So that we can measure our code coverage, I'm also adding jacoco and a gradle task called jacocoTestReport.

**gradle.build**  
```groovy
buildscript {
	ext {
		springBootVersion = '2.0.2.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'groovy'
apply plugin: 'jacoco'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.scotthensen.portfolio'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
	// Groovy
	compile 'org.codehaus.groovy:groovy-all:2.4.7'

	// Spring
	compile 'org.springframework.boot:spring-boot-starter-actuator'
	compile 'org.springframework.boot:spring-boot-starter-data-jpa'
	//compile 'org.springframework.boot:spring-boot-starter-data-redis'
	compile 'org.springframework.boot:spring-boot-starter-web'

	// Other
	compileOnly 'org.projectlombok:lombok'

	// Test
	testCompile 'org.springframework.boot:spring-boot-starter-test'
	testCompile 'org.spockframework:spock-core:1.1-groovy-2.4'
	testCompile 'org.spockframework:spock-spring:1.1-groovy-2.4'
	testCompile 'com.h2database:h2'
	testRuntime 'cglib:cglib-nodep:3.2.4'
}

jacoco {
    toolVersion = "0.7.9"
    reportsDir = file("$buildDir/reports/tests/jacoco")
}
jacocoTestReport {
    reports {
        xml.enabled false
        csv.enabled false
        html.destination file("${buildDir}/reports/tests/jacoco")
    }
}
```
**Build a test for Quote Entity**  
```java
package com.scotthensen.portfolio.svc.tests.unit.quote;

import java.math.BigDecimal;

import org.assertj.core.api.Assertions;
import org.junit.Test;

import com.scotthensen.portfolio.svc.quote.Quote;

public class QuoteTest {

	@Test
	public void givenValidArgs_whenAllArgConstructorCalled_thenQuoteIsConstructed() {

		// Given valid arguments
		String     teslaSym = "TSLA";
		BigDecimal teslaBid = new BigDecimal("100.50");
		BigDecimal teslaAsk = new BigDecimal("101.50");

		// When AllArgsConstructor executes
		Quote p = new Quote(null, teslaSym, teslaBid, teslaAsk);

		// Then a quote with the given data is constructed
		Assertions.assertThat(p.getSymbol()).isEqualTo(teslaSym);
		Assertions.assertThat(p.getBid()).isEqualByComparingTo(teslaBid);
		Assertions.assertThat(p.getAsk()).isEqualTo(teslaAsk);
	}
}

```

**Make it pass**  
```java
package com.scotthensen.portfolio.svc.quote;

import java.math.BigDecimal;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Entity
@AllArgsConstructor
@NoArgsConstructor(access=AccessLevel.PRIVATE, force=true)
public class Quote
{
	@Id
	@GeneratedValue(strategy=GenerationType.AUTO)
	private Long id;

	private String symbol;
	private BigDecimal bid;
	private BigDecimal ask;
}
```

**Build a helper class to generate test quotes**  
I don't want to rebuild my test quotes, so I'm going to use a little helper class to return two standard quotes.
```java
package com.scotthensen.portfolio.svc.tests.helpers;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;

import com.scotthensen.portfolio.svc.quote.Quote;


public class QuoteHelper {

	private String teslaSym     = "TSLA";
	private BigDecimal teslaBid = new BigDecimal("100.50");
	private BigDecimal teslaAsk = new BigDecimal("101.50");

	private String     samSym   = "SAM";
    private	BigDecimal samBid   = new BigDecimal("200.50");
	private BigDecimal samAsk   = new BigDecimal("201.50");

	public Quote getQuote1()
	{
		return new Quote(null, teslaSym, teslaBid, teslaAsk);
	}

	public Quote getQuote2()
	{
		return new Quote(null, samSym, samBid, samAsk);
	}
	public List<Quote> getQuotes()
	{
		List<Quote> quotes = new ArrayList<>();
		quotes.add(new Quote(null, teslaSym, teslaBid, teslaAsk));
		quotes.add(new Quote(null, samSym, samBid, samAsk));
		return quotes;
	}
}
```

**Build a test for Portfolio Entity**  
```java
package com.scotthensen.portfolio.svc.tests.unit.portfolio;


import java.util.List;

import org.assertj.core.api.Assertions;
import org.junit.Test;

import com.scotthensen.portfolio.svc.portfolio.Portfolio;
import com.scotthensen.portfolio.svc.quote.Quote;
import com.scotthensen.portfolio.svc.tests.helpers.QuoteHelper;

public class PortfolioTest {

	@Test
	public void givenValidArgs_whenAllArgConstructorCalled_thenPortfolioIsConstructed() {

		// Given valid arguments
		Long   portfolioId   = 1L;
		String portfolioName = "test portfolio";
		QuoteHelper helper   = new QuoteHelper();
		List<Quote> quotes   = helper.getQuotes();

		// When AllArgsConstructor executes
		Portfolio p = new Portfolio(portfolioId, portfolioName, quotes);

		// Then a portfolio with Tesla and Sam Adams is constructed
		Assertions.assertThat(p.getId()).isEqualTo(portfolioId);
		Assertions.assertThat(p.getPortfolioName()).isEqualTo(portfolioName);
		Assertions.assertThat(p.getQuotes()).containsAll(quotes);
	}

}
```

**Make it pass**  
```java
package com.scotthensen.portfolio.svc.portfolio;

import java.util.List;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.ManyToMany;
import javax.validation.constraints.NotNull;

import com.scotthensen.portfolio.svc.quote.Quote;

import lombok.AccessLevel;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Entity
@NoArgsConstructor(access=AccessLevel.PRIVATE, force=true)
@AllArgsConstructor
public class Portfolio
{
	@Id
	@GeneratedValue(strategy=GenerationType.AUTO)
	private Long id;

	@NotNull
	private String portfolioName;

	@ManyToMany(targetEntity=Quote.class)
	private List<Quote> quotes;
}
```

**Write a JPA Test for Portfolio**  
```java
package com.scotthensen.portfolio.svc.tests.integ.portfolio;

import java.util.ArrayList;
import java.util.List;

import org.assertj.core.api.Assertions;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;
import org.springframework.test.context.junit4.SpringRunner;

import com.scotthensen.portfolio.svc.portfolio.Portfolio;
import com.scotthensen.portfolio.svc.quote.Quote;
import com.scotthensen.portfolio.svc.tests.helpers.QuoteHelper;

@DataJpaTest
@RunWith(SpringRunner.class)
public class PortfolioJpaTest {

	@Autowired
	private TestEntityManager testEntityManager;

	@Test
	public void givenValidPortfolio_whenJpaPersist_thenPortfolioPersists()
	{
		String portfolioName = "Test Porfolio";
		QuoteHelper helper   = new QuoteHelper();
		Quote  q1 = this.testEntityManager.persist(helper.getQuote1());
		Quote  q2 = this.testEntityManager.persist(helper.getQuote2());
		List<Quote> quotes   = new ArrayList<Quote>();
		quotes.add(q1);
		quotes.add(q2);

		Portfolio p  = this.testEntityManager.persistAndFlush(new Portfolio(null,portfolioName,quotes));

		Assertions.assertThat(p.getId()).isNotNull();
		Assertions.assertThat(p.getPortfolioName()).isEqualTo(portfolioName);
		Assertions.assertThat(p.getQuotes()).containsAll(quotes);
	}
}
```

**Write a Repository Test for Portfolio**  
```java
package com.scotthensen.portfolio.svc.tests.integ.portfolio;

import java.util.ArrayList;
import java.util.List;

import org.assertj.core.api.Assertions;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.scotthensen.portfolio.svc.portfolio.PorfolioRepository;
import com.scotthensen.portfolio.svc.portfolio.Portfolio;
import com.scotthensen.portfolio.svc.quote.Quote;
import com.scotthensen.portfolio.svc.tests.helpers.QuoteHelper;

@DataJpaTest
@RunWith(SpringRunner.class)
public class PortfolioRepositoryTest
{
	@Autowired
	private PorfolioRepository repository;

	@Test
	public void givenValidPortfolio_whenSave_thenPortfolioPersists()
	{
		String portfolioName = "test porfolio";
		QuoteHelper helper = new QuoteHelper();
		List<Quote> quotes = new ArrayList<Quote>();
		quotes.add(helper.getQuote1());
		quotes.add(helper.getQuote2());

		Portfolio p = repository.save(new Portfolio(null, portfolioName, quotes));

		Assertions.assertThat(p.getId()).isNotNull();
		Assertions.assertThat(p.getPortfolioName()).isEqualTo(portfolioName);
		Assertions.assertThat(p.getQuotes().containsAll(quotes));
	}
}
```

**Make the Portfolio JPA/Repo tests pass**  
```java
package com.scotthensen.portfolio.svc.portfolio;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface PorfolioRepository extends JpaRepository<Portfolio, Long>{

}
```  

**Write a JPA Test for Quote**  
```java
package com.scotthensen.portfolio.svc.tests.integ.quote;

import org.assertj.core.api.Assertions;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;
import org.springframework.test.context.junit4.SpringRunner;

import com.scotthensen.portfolio.svc.quote.Quote;
import com.scotthensen.portfolio.svc.tests.helpers.QuoteHelper;

@DataJpaTest
@RunWith(SpringRunner.class)
public class QuoteJpaTest {

	@Autowired
	private TestEntityManager testEntityManager;

	@Test
	public void givenValidQuote_whenJpaPersist_thenPersistsQuote()
	{
		QuoteHelper helper = new QuoteHelper();
		Quote q1 = helper.getQuote1();

		Quote q = this.testEntityManager.persistAndFlush(q1);

		Assertions.assertThat(q.getSymbol()).isEqualTo(q1.getSymbol());
		Assertions.assertThat(q.getBid()).isEqualTo(q1.getBid());
		Assertions.assertThat(q.getAsk()).isEqualTo(q1.getAsk());
	}
}
```

**Write a Repository Test for Quote**  
```java
package com.scotthensen.portfolio.svc.tests.integ.quote;

import org.assertj.core.api.Assertions;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.junit4.SpringRunner;

import com.scotthensen.portfolio.svc.quote.Quote;
import com.scotthensen.portfolio.svc.quote.QuoteRepository;
import com.scotthensen.portfolio.svc.tests.helpers.QuoteHelper;

@DataJpaTest
@RunWith(SpringRunner.class)
public class QuoteRepositoryTest
{
	@Autowired
	private QuoteRepository repository;

	@Test
	public void givenValidQuote_whenSave_thenQuotePersists()
	{
		QuoteHelper helper = new QuoteHelper();
		Quote q1 = helper.getQuote1();

		Quote q = repository.save(q1);

		Assertions.assertThat(q.getSymbol()).isEqualTo(q1.getSymbol());
		Assertions.assertThat(q.getAsk()).isEqualTo(q1.getAsk());
		Assertions.assertThat(q.getBid()).isEqualTo(q1.getBid());
	}
}
```

**Make the Quote JPA/Repo tests pass**  
```java

package com.scotthensen.portfolio.svc.quote;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface QuoteRepository extends JpaRepository<Quote, Long>{

}
```

### Build the Service's Use Cases  
Up to this point, I've really just slapped together classes based on my last post.  Now, I will add the first use case:  get some quotes.  

Here, I will depart from the standard Spring project structure, and group all of the classes used for this use case into one package.  I imagine that, if the use case involved more classes, or multiple implementations, we could justify breaking the package into smaller pieces, but I am just going to throw them all into one bucket.  This is the *'screaming architecture'* that Uncle Bob refers to in his vision for clean architecture.  *[Note:  I am just borrowing some of his ideas, I am not using all of the injections and interfaces that he recommends.]*  

To give you an idea of how I am organizing the components, here is a view of the project structure after writing one use case (code below).

**Project Structure with one use case**  

![Project Structure]({{site.baseurl}}/img/2018-06-14/ProjectStructure-1.JPG "Project Structure with one use case")


Here is the spec for that use case, written in BDD statements:
* *Given* a valid symbol (TSLA),  
*When* a request for a quote is made,  
*Then* the quote is returned.  

* *Given* an invalid symbol (xxx),  
*When* a request for a quote is made,  
*Then* an Http 404 (not found) is returned in the response.  

* *Given* a mix of valid and invalid symbols (TSLA, xxx, SAM),  
*When* a request for a quote is made,  
*Then* the quotes are returned for the valid symbols (TSLA and SAM) are returned in alphabetical order.  

**Write a test (a 'spec') for the get-quotes use case**  
```groovy
package com.scotthensen.portfolio.svc.tests.integ

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.http.HttpStatus
import org.springframework.web.client.HttpClientErrorException

import com.scotthensen.portfolio.svc.usecases.getquotes.GetQuotes
import com.scotthensen.portfolio.svc.usecases.getquotes.IexQuoteSvc
import com.scotthensen.portfolio.svc.usecases.getquotes.GetQuotesRequest
import com.scotthensen.portfolio.svc.usecases.getquotes.GetQuotesResponse

import spock.lang.Narrative
import spock.lang.Shared
import spock.lang.Specification
import spock.lang.Stepwise
import spock.lang.Title

@Narrative(""" The IEX Quote service is called to get a list quotes.""")
@Title('Integration test with IEX Quote service')

@Stepwise
class GetQuotesSpec extends Specification {

	@Shared
	def GetQuotes quotes = new IexQuoteSvc();
	@Shared
	def GetQuotesResponse quotesResponse = new GetQuotesResponse();

	def List<String> symbols = new ArrayList<>();

	def "Get a quote for a valid symbol"() {

		given: "a valid symbol (TSLA)"
		symbols.add("TSLA")
		GetQuotesRequest quotesRequest = new GetQuotesRequest(symbols);

		when: "a request for a quote is made"
		quotesResponse = quotes.getQuotes(quotesRequest);

		then: "the quote is returned"
		quotesResponse.getQuotes().get(0).getSymbol() == "TSLA"
	}
	def "Get a quote for an invalid symbol"() {

		given: "an invalid symbol (xxx)"
		symbols.add("xxx")
		GetQuotesRequest quotesRequest = new GetQuotesRequest(symbols);

		when: "a request for a quote is made"
		quotesResponse = quotes.getQuotes(quotesRequest);

		then: "an Http 404 (not found) is returned in the response"
		quotesResponse.getHttpException().getStatusCode() == HttpStatus.NOT_FOUND
		quotesResponse.getQuotes() == null
	}
	def "Get a quote for a missing symbol"() {

		given: "a missing symbol"
		GetQuotesRequest quotesRequest = new GetQuotesRequest();

		when: "a request for a quote is made"
		quotesResponse = quotes.getQuotes(quotesRequest);

		then: "an HttpClientErrorException is returned in the response"
		quotesResponse.getHttpException().getStatusCode() == HttpStatus.BAD_REQUEST
		quotesResponse.getQuotes() == null
	}
	def "Get multiple quotes; some are valid symbols and some aren't"() {

		given: "a mix of valid and invalid symbols (TSLA, xxx, SAM)"
		symbols.add("TSLA")
		symbols.add("xxx")
		symbols.add("SAM")
		GetQuotesRequest quotesRequest = new GetQuotesRequest(symbols);

		when: "a request for a quote is made"
		quotesResponse = quotes.getQuotes(quotesRequest);

		then: "the quotes are returned for the valid symols (in alphabetical order)"
		quotesResponse.getQuotes().get(0).getSymbol() == "SAM"
		quotesResponse.getQuotes().get(1).getSymbol() == "TSLA"

	}
}
```  

All sorts of classes were identified as missing after writing that first given-then-when test.  Here are the GetQuotes interface, the request and response, and the implementation and entities that I used to pull the quotes from IEX.  

**GetQuotes interface**  
```java
package com.scotthensen.portfolio.svc.usecases.getquotes;

public interface GetQuotes
{
	public GetQuotesResponse getQuotes(GetQuotesRequest request);
}
```  

**GetQuotesRequest**  
```java
package com.scotthensen.portfolio.svc.usecases.getquotes;

import java.util.List;

import org.springframework.stereotype.Component;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Component
@Data
@NoArgsConstructor //(access = AccessLevel.NONE)
@AllArgsConstructor
public class GetQuotesRequest
{
	private List<String> symbols;
}
```  

**GetQuotesResponse**  
```java
package com.scotthensen.portfolio.svc.usecases.getquotes;

import java.util.List;

import org.springframework.web.client.HttpClientErrorException;

import com.scotthensen.portfolio.svc.quote.Quote;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class GetQuotesResponse
{
	private List<Quote> quotes;
	private HttpClientErrorException httpException;
}
```  

**IexQuoteSvc implementation**  
Warning:  This is still pretty crumby looking.    

*[This is a good example of how estimates for open-source-based development can go off the rails.  I threw all that code above together in an hour or two and tested it with a single quote, which IEX returns as a single JSON object.  It was easy-peasy.  If I had to give an estimate based on that, I'd say give me a couple hours to build the basic get-quotes use case.  But, when I replaced that quick test with a request to pull multiple quotes at once (something much more reasonable for tracking an entire portfolio), I ran into all sorts of problems.  It took me three nights to figure out how that one line - Spring's RestTemplate - maps nested LinkedHashMaps into an object.  I'm still not happy with the solution, and of course I want to remove the if/else and try/catch blocks, but I'm kind of sick of messing with it right now.  :)  Hopefully, in a few days, my repo will have a better version of this class.  If not... this is a good example of how tech debt is created, I guess. ]*  

```java
package com.scotthensen.portfolio.svc.usecases.getquotes;

import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

import org.springframework.core.ParameterizedTypeReference;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.client.HttpClientErrorException;
import org.springframework.web.client.RestTemplate;

import com.scotthensen.portfolio.svc.quote.Quote;


class IexQuoteSvc implements GetQuotes {

	private static final String IEX_BASE_URL      = "https://api.iextrading.com/1.0/stock/market/batch?symbols=";
	private static final String IEX_REQUEST_TYPES = "&types=quote";

	public GetQuotesResponse getQuotes(GetQuotesRequest request)
	{
		RestTemplate             restTemplate = new RestTemplate();
		GetQuotesResponse        response     = new GetQuotesResponse();
		HttpClientErrorException exception    = null;

		if (request == null || request.getSymbols() == null) {
			exception =
				new HttpClientErrorException(
						HttpStatus.BAD_REQUEST,
						"GetQuotesRequest must contain a symbol");
		}
		else
		{
			try
			{
				ParameterizedTypeReference<HashMap<String,IexQuote>> quotesMap =
						new ParameterizedTypeReference<HashMap<String,IexQuote>>() {};

				ResponseEntity<HashMap<String,IexQuote>> iexResponse =
					restTemplate.exchange(
						buildStringUrl(request),
						HttpMethod.GET,
						null,
						quotesMap);

				if ((iexResponse.getStatusCode() != HttpStatus.OK) || (iexResponse.getBody().isEmpty()))
				{
					exception =
						new HttpClientErrorException(
							HttpStatus.NOT_FOUND,
							"Symbol(s) provided in GetQuotesRequest were not found at IEX");

				}
				else
				{
					List<Quote> quotes =
						iexResponse.getBody().entrySet()
							.stream()
							.sorted(Map.Entry.<String, IexQuote>comparingByKey())
							.map(q -> new Quote(null,
											    q.getValue().getQuote().getSymbol(),
											    q.getValue().getQuote().getIexBidPrice(),
											    q.getValue().getQuote().getIexAskPrice()  )	)
							.collect(Collectors.toList());

					response.setQuotes(quotes);
				}
			}
			catch (HttpClientErrorException e)
			{
				exception =
					new HttpClientErrorException(
						HttpStatus.NOT_FOUND,
						"Symbol(s) provided in GetQuotesRequest were not found at IEX");
			}
		}
		response.setHttpException(exception);
		return response;
	}

	private String buildStringUrl(GetQuotesRequest request)
	{
		String urlSymbols = String.join(",", request.getSymbols());
		String url        = IEX_BASE_URL + urlSymbols + IEX_REQUEST_TYPES;

		return url;
	}

}
```  

**IexQuote**  
```java
package com.scotthensen.portfolio.svc.usecases.getquotes;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
@JsonIgnoreProperties(ignoreUnknown = true)
public class IexQuote
{
	private IexQuoteDetails quote;

}
```  

**IexQuoteDetail**  
```java

package com.scotthensen.portfolio.svc.usecases.getquotes;

import java.math.BigDecimal;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
@JsonIgnoreProperties(ignoreUnknown = true)
public class IexQuoteDetails
{
	private String     symbol;
	private String     companyName;
	private String     primaryExchange;
	private String     sector;
	private String     calculationPrice;
	private BigDecimal open;
	private BigDecimal openTime;
	private BigDecimal close;
	private BigDecimal closeTime;
	private BigDecimal high;
	private BigDecimal low;
	private BigDecimal latestPrice;
	private String     latestSource;
	private String     latestTime;
	private BigDecimal latestUpdate;
	private BigDecimal latestVolume;
	private BigDecimal iexRealtimePrice;
	private BigDecimal iexRealtimeSize;
	private BigDecimal iexLastUpdated;
	private BigDecimal delayedPrice;
	private BigDecimal delayedPriceTime;
	private BigDecimal extendedPrice;
	private BigDecimal extendedChange;
	private BigDecimal extendedChangePercent;
	private BigDecimal extendedPriceTime;
	private BigDecimal change;
	private BigDecimal changePercent;
	private BigDecimal iexMarketPercent;
	private BigDecimal iexVolume;
	private BigDecimal avgTotalVolume;
	private BigDecimal iexBidPrice;
	private BigDecimal iexBidSize;
	private BigDecimal iexAskPrice;
	private BigDecimal iexAskSize;
	private BigDecimal marketCap;
	private BigDecimal peRatio;
	private BigDecimal week52High;
	private BigDecimal week52Low;
	private BigDecimal ytdChange;
}
```  

I think I am going to stop here.  There are still additional use cases to develop, but I am going to switch to the front-end now.  My plan is to build the basic web page to display a list of quotes.  Once I get the front-end and back-end talking, I'll give Selenium a try to test the web page, then I'll circle back and finish the use cases.  

### My source code
[Github: portfolio-service][github-portfolio-svc]  



[github-geowarin]:https://github.com/geowarin/spring-spock-mvc
[github-portfolio-svc]:https://github.com/ScottHensen/portfolio-svc
