---
layout: post
title:  "Scheduled Tasks"
date:   2018-05-26 12:00:00 -0700
categories: java spring scheduled task
---
# Scheduled Tasks
I think we have 30+ million lines of COBOL at my shop.  Don't be shocked;
all of your money still moves around on mainframes.
Our front-ends have been browser-based java since the mid-nineties, so all that
COBOL now really provides two main functions:  1) it serves data from mainframe
databases to our web apps, and 2) it handles a huge chunk of our batch processing.
In a our big enterprise, most of the code is batch.  

At home, I don't have much need for batch.  Occasionally, though, I'll need to
write a script and will use python or bash.  Some of my toy apps lately
have required batch and/or a daemon, so I've been thinking about Spring Batch.
The concepts and components are all very similar to those used to build
mainframe batch, but it looks like an awful lot of config for
my needs, so before tackling a big framework, I thought I'd explore some simpler options.  

My first java batch stuff was just some [Spring Boot command line runner][sbclr]
apps that I put together to do ETL for that goofy little taper.io service I
built ([Extract Job][taper-io-extract] & [Trasform Job][taper-io-transform]).
OK, those are kind of janky.  Certainly not 'enterprise grade.'  

Today, I have a need to build a background app that will pull data from a
service every so often and cache it.  The @Scheduled annotation was made
available in Spring 3, I think (it's all new to me), so I am playing with that option today.  Maybe this is a perfect enterprise solution, or
maybe it's still not robust enough for something really important like moving
your money around.  In an enterprise, I'd think that you want all batch to run
under a central scheduler, but if you want to build something that runs every n
seconds all day long, and you make sure to deploy your own set of controls, then
maybe @Scheduled makes sense.  In the spirit of investigating enterprise-grade solutions, I think I'm going to use [IEX][iex] as the source of my data instead of doing my usual swapi calls.  

# My first scheduled task
I followed the [Spring Getting Started guide][spring-gs-scheduledtasks].  Easy peasy!   

**Application**
```java
package com.scotthensen.toolbox.scheduledtask;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling
public class ToolboxScheduledTaskApplication {

	public static void main(String[] args) {
		SpringApplication.run(ToolboxScheduledTaskApplication.class, args);
	}
}
```

**Task Component that logs time every 5 seconds**
```java
package com.scotthensen.toolbox.scheduledtask;

import java.text.SimpleDateFormat;
import java.util.Date;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Component
public class TaskLogTime {

	private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

	@Scheduled(fixedRate = 5000)
	public void reportCurrentTime() {
		log.info("The time is now {}", dateFormat.format(new Date()));
	}
}
```
**Results**  

![Results of my first scheduled task]({{site.baseurl}}/img/2018-05-26/time-task-console.JPG "Results of my first scheduled task")

# A task to get stock market quotes
First, I skimmed the [Spring Integration docs on Scheduled Tasks][spring-doc-scheduledtasks], so that I would have some idea of what I was doing. Then, I read through the
[IEX documentation][iex-doc] to see how to use their api.  Then, I had to refresh
my memory on [how to consume a restful service][spring-doc-getrestsvc]

**Task Component that gets a Tesla quote from IEX every 10 seconds**
```java
package com.scotthensen.toolbox.scheduledtask;

import java.text.SimpleDateFormat;
import java.util.Date;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

import com.scotthensen.toolbox.scheduledtask.domain.StockQuote;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Component
public class TaskGetQuotes {

	private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss:SSS Z");
	private static final String IEX_URL = "https://api.iextrading.com/1.0/stock/tsla/quote";

	@Scheduled(fixedDelay = 10000)
	public void getStockQuote() {
		log.info("The time is now {}", dateFormat.format(new Date()), " ... start getStockQuote");

		RestTemplate restTemplate = new RestTemplate();
		StockQuote quote = restTemplate.getForObject(IEX_URL, StockQuote.class);
		log.info(quote.toString());

		log.info("The time is now {}", dateFormat.format(new Date()), " ... stop  getStockQuote");		
	}


}
```
**Stock Quote domain model**
```java
package com.scotthensen.toolbox.scheduledtask.domain;

import java.math.BigDecimal;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

import lombok.Data;

@Data
@JsonIgnoreProperties(ignoreUnknown = true)
public class StockQuote {

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
**Results**  

![Results of my second scheduled task]({{site.baseurl}}/img/2018-05-26/time-quotes-task-console.JPG "Results:  two tasks running concurrently; the time task and the stock quote task")

# Here's a cron task
The @Scheduled annotation accepts a few optional arguments.  So far, we've used
fixedRate and fixedDelay.  There's also string versions of those: fixedRateString
and fixedDelayString.  The initialDelay and initialDelayString arguments allow us
to specify the how much lead time should be given before the scheduler starts
executing at the specified fixed delay|rate.  These are all pretty self-explanitory,
so I'm not going to build examples of each.  

There is one other attribute, though, that I want to dig into a little more; cron.
With cron, you can specify the schedule by month, day, hour, minute, second.  The
syntax is borrows from unix/linux cron expression syntax.

A cron expression is made up of the following:  
```
<sec> <min> <hour> <day> <month> <day-of-week> [<year>] <command>
```  

Notes from [Baeldung][baeldung-cron]:
- **sec**  = 0-59
- **min**  = 0-59
- **hour** = 0-23
- **day** = 1-31
- **mo**  = 1-12 or JAN-DEC
- **day-of-week** = 1-7 or SUN-SAT
- **yr**  = empty or 1970-2099 (optional)
- **"*"** = all/every
- **"?"** = any
- **"-"** = allows ranged values, eg. 18-21.
- **","** = allows multiple values, eg. FRI, SAT, SUN.
- **"/"** = allows incremental values, eg. 30/10 minutes means **:30, **:40 and **:50.
- **"L"** = last.  Can do math on this too, so L-1 days means the day before the last day of the month.
- **"W"** = weekday.  1W means the weekday nearest to the first of the month.  Warning:  this does not mean the first weekday of the month.  If the first is a Saturday, then the nearest weekday will be the day before; the last day of last month.  
- **"#"** = the nth occurrence of a given weekday in the month, eg. 6#1 is the first Friday.



Examples from [Spring's Doc][spring-doc-cron-seq-gen]:
```
- "0 0 * * * *"          = the top of every hour of every day.
- "*/10 * * * * *"       = every ten seconds.
- "0 0 8-10 * * *"       = 8, 9 and 10 o'clock of every day.
- "0 0 6,19 * * *"       = 6:00 AM and 7:00 PM every day.
- "0 0/30 8-10 * * *"    = 8:00, 8:30, 9:00, 9:30, 10:00 and 10:30 every day.
- "0 0 9-17 * * MON-FRI" = on the hour nine-to-five weekdays
- "0 0 0 25 12 ?"        = every Christmas Day at midnight
```
More examples can be found in the [Quartz][quartz-doc] docs (identical to the Oracle examples)

# A cron task to get company info from IEX :00 and :30
[Yes, I know it's a dumb example and the code does not follow DRY.]
```java
package com.scotthensen.toolbox.scheduledtask;

import java.text.SimpleDateFormat;
import java.util.Date;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

import com.scotthensen.toolbox.scheduledtask.domain.CompanyInfo;
import com.scotthensen.toolbox.scheduledtask.domain.StockQuote;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@Component
public class TaskGetFinancialUpdatesFromIEX {

	private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss:SSS Z");
	private static final String IEX_URL        = "https://api.iextrading.com/1.0";
	private static final String TESLA_QUOTE    = "/stock/tsla/quote";
	private static final String TESLA_COMPANY  = "/stock/tsla/company";


	@Scheduled(fixedDelay = 10000)
	public void getStockQuote() {
		log.info("The time is now {}", dateFormat.format(new Date()), " ... start getStockQuote");

		RestTemplate restTemplate = new RestTemplate();
		StockQuote quote = restTemplate.getForObject(IEX_URL+TESLA_QUOTE, StockQuote.class);
		log.info(quote.toString());

		log.info("The time is now {}", dateFormat.format(new Date()), " ... stop  getStockQuote");		
	}

	@Scheduled(cron = "0/30 * * * * *")
	public void getCompanyInfo() {
		log.info("The time is now {}", dateFormat.format(new Date()), " ... start getStockQuote");

		RestTemplate restTemplate = new RestTemplate();
		CompanyInfo company = restTemplate.getForObject(IEX_URL+TESLA_COMPANY, CompanyInfo.class);
		log.info(company.toString());

		log.info("The time is now {}", dateFormat.format(new Date()), " ... stop  getStockQuote");		
	}


}
```
**Results**  

![Results of the cron scheduled task]({{site.baseurl}}/img/2018-05-26/time-quotes-company-task-console.JPG "Results:  three tasks running concurrently; the time, quotes and company info task")

# Thread pools
It's pretty easy to configure thread pools too; the default is 1.  [Callicoder][callicoder]
has a nice example of this.  [After getting this far, I am realizing that the gist of the Spring docs information is covered by combining the Callicoder and Baeldung blogs.]


[baeldung-cron]:http://www.baeldung.com/cron-expressions
[callicoder]:https://www.callicoder.com/spring-boot-task-scheduling-with-scheduled-annotation/
[iex]:https://iextrading.com/
[iex-doc]:https://iextrading.com/developer/docs/
[quartz-doc]:http://www.quartz-scheduler.org/documentation/quartz-2.x/tutorials/crontrigger.html
[sbclr]:https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/CommandLineRunner.html
[spring]:https://spring.io/
[spring-gs-scheduledtasks]:https://spring.io/guides/gs/scheduling-tasks/
[spring-doc-scheduledtasks]:https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#scheduling
[spring-doc-cron-seq-gen]:https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/support/CronSequenceGenerator.html
[spring-doc-getrestsvc]:https://spring.io/guides/gs/consuming-rest/
[taper-io-extract]:https://github.com/ScottHensen/taper-job-extract-shows-for-band
[taper-io-transform]:https://github.com/ScottHensen/taper-job-transform-shows-for-band
