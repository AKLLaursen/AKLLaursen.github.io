---
layout: post
author: andreas
---
For various purposes, I have been wanting to set up a database with the aim of hosting securities data on my [server](https://www.digitalocean.com/) for some time now. I thought that I would document the process as I went along, such that it would be easy to do again, should I ever need to. Further, it might be both interesting and useful for other people.

An easy accessible database is very useful if you want to check the odd model, theory or just want to do a quick visualisation. Accessing services such as [Quandl](https://www.quandl.com/) can quickly get very time consuming, once you need a large number of time series.

As for the specific database, a number of options is available, but really the choice boils down to either a [relational database](https://en.wikipedia.org/wiki/Relational_database) ([Microsoft SQL Server](https://www.microsoft.com/en-us/server-cloud/), [Oracle](https://www.oracle.com/database/index.html), [MySQL](https://www.mysql.com/), [Postgresql](http://www.postgresql.org/)) or a [NoSQL database](https://en.wikipedia.org/wiki/NoSQL) ([MongoDB](https://www.mongodb.com/), [CouchDB](http://couchdb.apache.org/), [Redis](http://redis.io/)). Being very familiar with relational databases in general and [SQL](https://en.wikipedia.org/wiki/SQL) in particular, this is a natural choice for me. I go with Postgresql as 1) it is free and open source and 2) I am already familiar with it as this site, [akll.io](www.akll.io), is built using Postgresql through [Django](https://www.djangoproject.com/).

## Setting Up the Postgresql Database

The database will be hosted on a [Ubuntu Server 14.04](http://www.ubuntu.com/server) platform. I have still to migrate the server to 16.04, even though Digital Ocean has made it available. Postgresql is ridiculously easy to set up. To install SSH into the server and run.

```
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
```

The [contrib package](http://www.postgresql.org/docs/9.1/static/contrib.html) provides porting tools, analysis utilities, and plug-in features that are not part of the core PostgreSQL system. As part of the installation, a user account called `postgres` is created. This user can be used to check that Postgres is installed and working as intended.

```
sudo -i -u postgres
psql

psql (9.4.5)
Type "help" for help.

\q
```

Next up is creating a new role. One might want to associate an existing account with the given role, or one might want to create a new account. At any rate the role is assigned by entering the following using the `postgres` account.

```
createuser --interactive
Enter name of role to add: andreas   
Shall the new role be a superuser? (y/n) y
```

Here I choose to be a superuser in order to have all privileges. Each user must also be associated with a database of the same name. It can be created by entering.

```
createdb andreas
```

Thus the `psql` command can be invoked as the given user (in my case andreas). That's it for setting up the server. Of course there's a lot more to relational database administration, but as this is a small server only accessed by one person, there is no reason to get fancy with user rights and restrictions.

We can then create and access the actual securities database by running the following commands in the terminal. The standard in postgreql is to use all lower-case names, however due to problems with RODBC, I had to have the database name be capitalized. I have no idea as to why.

```
createdb Securities
psql Securities
```

Next, it is time to set up the database. In this post I will cover how to set up equities in the [style](https://www.quantstart.com/articles/Securities-Master-Database-with-MySQL-and-Python) of Michael Halls-Moore of [QuantStart](https://www.quantstart.com/). I will however append a bit as I go along. Thus, we will be creating 4 simple tables:

* exchange: The exchanges to obtain prices from - Dimensional table.
* data_provider: The providers of the data - Dimensional table.
* symbol: The ticker symbols - Dimensional table.
* daily_price: The actual prices - Fact table.

First I create an equity schema.

```SQL
create schema equities;
```

And then I build the tables, starting with the exchange table and then building the data_provider, symbol and daily_price tables.

```SQL
create table equities.exchange (
  id_exchange serial not null primary key,
  abbreviation varchar(32) not null,
  name varchar(255) not null,
  city varchar(255) null,
  country varchar(255) null,
  currency varchar(64) null,
  timezone_offset varchar(6) null,
  created_datetime timestamp not null,
  last_updated_datetime timestamp not null
);

create table equities.data_vendor (
  id_data_vendor serial not null primary key,
  name varchar(64) not null,
  website_url varchar(255) null,
  support_email varchar(255) null,
  created_datetime timestamp not null,
  last_updated_datetime timestamp not null
);

create table equities.symbol (
  id_symbol serial not null primary key,
  id_exchange serial not null references equities.exchange(id_exchange),
  ticker varchar(32) not null,
  instrument varchar(64) not null,
  name varchar(255) null,
  sector varchar(255) null,
  currency varchar(32) null,
  created_datetime timestamp not null,
  last_updated_datetime timestamp not null
);

create table equities.daily_price (
  id_daily_price serial not null primary key,
  id_data_vendor serial not null references equities.data_vendor(id_data_vendor),
  id_symbol serial not null references equities.symbol(id_symbol),
  date timestamp not null,
  created_datetime timestamp not null,
  last_updated_datetime timestamp not null,
  open decimal(19, 4) null,
  high decimal(19, 4) null,
  low decimal(19, 4) null,
  close decimal(19, 4) null,
  adj_close decimal(19, 4) null,
  volume bigint null
);

```

Note the use of foreign keys employed to connect dimensions to the fact table through their respective primary keys.

## Connect to the Postgresql Database Using R

In R it is easy to connect to a variety of relational databases using the [ODBC](https://en.wikipedia.org/wiki/Open_Database_Connectivity) database interface [RODBC](https://cran.r-project.org/web/packages/RODBC/index.html). However, setting up ODBC on a Ubuntu server is unfortunately quite a hassle. First we need to install the unixodbc and odbc-postgresql packages, which installs an open source implementation of the ODBC API and the official PostgreSQL ODBC driver. As the versions of these packages hosted in Ubuntu's package manager were either outdated or bugged, I found that I had to build both packages from source in order to make it work. A quick overview of how to do that is given below, starting with unixodbc.

```
wget ftp://ftp.unixodbc.org/pub/unixODBC/unixODBC-2.3.4.tar.gz
tar xf unixODBC-2.3.4.tar.gz
cd unixODBC-2.3.4
./configure --disable-gui --disable-drivers --enable-stats=no --enable-iconv --with-iconv-char-enc=UTF8 --with-iconv-ucode-enc=UTF16LE
make
sudo make install
```
Now the files have been installed in a local library to we need to add `/usr/local/lib` to the end of the `/etc/ld.so.conf` file and run:

```
sudo ldconfig
```

Next we build the odbc-postgresql package in the same way.

```
wget https://ftp.postgresql.org/pub/source/v9.6.1/postgresql-9.6.1.tar.gz
tar xf postgresql-9.6.1.tar.gz
cd postgresql-9.6.1
./configure --with-unixodbc=/usr/local/bin/odbc_config 
make
sudo make install
```

We can then setup the connection by editing a few files. Set up the driver in `/usr/local/etc/odbcinst.ini`.

```
[PostgreSQL]
Description     = PostgreSQL ODBC driver (Unicode 9.5)
Driver          = /usr/local/lib/psqlodbcw.so
Debug           = 0
CommLog         = 1
UsageCount      = 1
```

Next set up the database connections. In our case, we add the following to the `/usr/local/etc/odbc.ini` file.

```
[ODBC Data Sources]
securities = The securities database

[securities]
Driver      = PostgreSQL
ServerName  = localhost
Port        = 5432
Database    = Securities
Username    = user
Password    = password
Protocol    = 9.6.1
Debug       = 1
```

Here the username and password should of course reflect the database user and password. We can then test the connection by running.

```
isql -v securities
```

Which if everything is set up correct returns the following output.

```
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
SQL> 
```

## Interfacing With R

We are now ready to use the ODBC driver to interface between R and PostgreSQL. We do this by utilizing the [RODBC](https://cran.r-project.org/web/packages/RODBC/index.html) package. This is unfortunately also a bit complicated as RODBC off the box from CRAN comes with iODBC libraries that does not work. So we will compile it from source as we did with the unix packages in the former section.

First we download the source package and set a path such that the compiler uses the unixODBC library instead of the iODBC library. Next we install the package.

```
wget cran.r-project.org/src/contrib/RODBC_1.3-14.tar.gz
DYLD_LIBRARY_PATH=/usr/local/lib
sudo R CMD INSTALL RODBC_1.3-13.tar.gz
```

We can now test the connection in R.

```R
library("RODBC")
odbcDataSources()
```

Which returns the output:

```
  securities
"PostgreSQL"
```

So we have a working connection.

## Obtaining Stock Symbols and Exchanges

Next we turn towards populating the actual tables. Using the [S&P500 Index](http://us.spindices.com/indices/equity/sp-500) as an example, we can, as QuantStart, use [Wikipedia](https://en.wikipedia.org/wiki/List_of_S%26P_500_companies) to get a complete list of the symbols of the S&P500 Index. Scraping html tables is incredible easy in R using [Hadley Wickham's](http://hadley.nz/) [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/) inspired package [rvest](https://github.com/hadley/rvest). At the same time we populate the exchange table manually, (S&P 500 stocks are only traded on [NYSE](https://www.nyse.com/index) and [Nasdaq](http://www.nasdaq.com/)).

The script for obtaining the symbols and the exchanges and populating the tables is given below. To run all the following scripts, you should import these functions.

```R
import::from(magrittr, `%>%`, `%<>%`, `%$%`)
import::from(xml2, read_html)
import::from(rvest, html_node, html_nodes, html_table, html_text, html_attr)
import::from(dplyr, transmute, bind_rows, left_join, select, inner_join, mutate)
import::from(stringr, str_extract, str_replace_all)
import::from(RODBC, odbcConnect, sqlQuery, odbcClose)
import::from(httr, GET, content)
import::from(jsonlite, fromJSON)
```

Two functions get the symbols and the exchanges and two functions inserts these into the postgresql database

```R
# Function obtaining symbols
get_symbols <- function() {
  
  # Set current time
  current_time <- Sys.time()
  
  # Parse entire wiki url
  wiki_symbols <- read_html("https://en.wikipedia.org/wiki/List_of_S%26P_500_companies")
  
  # Obtain first table, cast as data frame and extract relevant information
  wiki_symbols %>%
    html_node("h2 + table") %>%
    html_table() %>%
    transmute(ticker = `Ticker symbol`,
              instrument = "Stock",
              name = Security,
              sector = `GICS Sector`,
              currency = "USD",
              created_datetime = current_time,
              last_updated_datetime = current_time)
  
}

# Function obtaining exchanges
get_exchanges <- function() {
  
  # Set current time
  current_time <- Sys.time()
  
  # Relevant exchange information
  exchanges <- list(
    list(
      exchange = "NYSE",
      name = "New York Stock Exchange",
      city = "New York",
      country = "United States of America",
      currency = "USD",
      timezone_offset = "-05:00"
    ),
    list(
      exchange = "NASDAQ",
      name = "National Association of Securities Dealers Automated Quotations",
      city = "New York",
      country = "United States of America",
      currency = "USD",
      timezone_offset = "-05:00"
    )
  ) %>%
    bind_rows()
  
  # Parse entire wiki url
  wiki_symbols <- read_html("https://en.wikipedia.org/wiki/List_of_S%26P_500_companies")
  
  # Obtain first row of table, cast as data frame and extract names of symbols 
  # and links identifying their respective exchanges.
  wiki_symbols %>%
    html_nodes("td:nth-child(1) .text") %>%
    {
      data.frame(
        symbol = (.) %>%
          html_text(),
        exchange = (.) %>%
          html_attr("href") %>%
          str_extract("[^https://www.].?[a-z-_]*") %>%
          toupper(),
        stringsAsFactors = FALSE
      )
    } %>% 
    left_join(exchanges) %>%
    transmute(ticker = symbol,
              abbreviation = exchange,
              name,
              city,
              country,
              currency,
              timezone_offset,
              created_datetime = current_time,
              last_updated_datetime = current_time)
  
}

# Function inserting exchanges into postgresql database
insert_exchanges <- function(exchanges, user, password) {
  
  # Get unique exchange names
  exchanges %<>%
    select(-ticker) %>%
    unique()
  
  # Paste exchange names and values to single strings
  ex_names <- names(exchanges) %>%
    paste(collapse = ", ")
  ex_values <- apply(exchanges, 1, paste, collapse = "', '") %>%
    paste(collapse = "'), ('") %>%
    paste0("('", ., "')")
  
  # Create sql query
  query <- paste("insert into equities.exchange",
                 "(",
                 ex_names,
                 ")",
                 "values",
                 ex_values,
                 ";")
  
  # Set connection, sedn query and close connection
  ch <- odbcConnect("securities", uid = user, pwd = password)
  sqlQuery(ch, query)
  odbcClose(ch)
  
}

# Function inserting symbols into postgresql securities database
insert_symbols <- function(symbols, exchanges, user, password) {
  
  # Get exchange names
  ex_names <- exchanges %>%
    select(-ticker) %>% 
    names() %>%
    paste(collapse = ", ")
  
  # query to obtain exchange ID's in order to populate foreign key table
  ex_query <- paste("select id_exchange,",
                    ex_names,
                    "from equities.exchange;")
  
  # Connect to database and send query
  ch <- odbcConnect("securities", uid = user, pwd = password)
  ex <- sqlQuery(ch, ex_query, stringsAsFactors = FALSE)
  
  # Join exchange id's on symbols and obtain dataframe to populate database table
  sym <- symbols %>%
    inner_join(exchanges %>% select(ticker, abbreviation),
               by = c("ticker" = "ticker")) %>% 
    inner_join(ex %>% select(abbreviation, id_exchange),
               by = c("abbreviation" = "abbreviation")) %>%
    transmute(id_exchange,
              ticker,
              instrument,
              name,
              sector,
              currency,
              created_datetime,
              last_updated_datetime)
  
  # Paste symbol column names and values to single strings
  sym_names <- sym %>%
    names() %>%
    paste(collapse = ", ")
  sym_data <- sym %>%
    mutate(name = str_replace_all(name, "'", " ")) %>% 
    apply(1, paste, collapse = "', '") %>%
    paste(collapse = "'), ('") %>%
    paste0("('", ., "')")
  
  # Create sql query
  sym_query <- paste("insert into equities.symbol",
                     "(",
                     sym_names,
                     ")",
                     "values",
                     sym_data,
                     ";")
  
  # Send query and close connection
  sqlQuery(ch, sym_query)
  odbcClose(ch)
  
}

# Run functions
get_exchanges() %>%
  insert_exchanges(user = "Insert user",
                   password = "Insert password")

get_symbols() %>%
  insert_symbols(get_exchanges(),
                 user = "Insert user",
                 password = "Insert password")

```

Of course the user and password information should be replaced..

## Obtaining Stock Qoutes

Finally, we turn towards obtaining the actual stock quotes. I do this using the [Yahoo Finance](http://finance.yahoo.com/) API. It is by far the easiest way to obtain the quotes, as the individual series are easy to identify using the ticker symbols and the data returned in a nice [JSON](http://www.json.org/) format. First however, I populate the data vendor table using the function given below.

```R
# Insert a data vendor to the database
insert_data_vendor <- function(data_vendor, user, password) {
  
  # Get current time
  current_time <- Sys.time()
  
  # Create query to populate the postgresql database
  query <- paste0("insert into equities.data_vendor ",
                  "(name, website_url, support_email, created_datetime, last_updated_datetime)",
                  "values ",
                  "('",
                  paste(data_vendor$name, data_vendor$website_url,
                        data_vendor$support_email, current_time, current_time,
                        sep = "', '"),
                  "');")
  
  # Connect to database, send query and close connection.
  ch <- odbcConnect("securities", uid = user, pwd = password)
  sqlQuery(ch, query)
  odbcClose(ch)
  
}

data_vendor <- data.frame(name = "Yahoo Finance",
                          website_url = "https://finance.yahoo.com",
                          support_email = "None")

insert_data_vendor(data_vendor, "User Name", "Password")

```

Next we obtain the prices. The function and function call is given below. Note that the function checks against the database in order to remove doublets. This functionality could also be implemented in the other functions, should the need arise, (it probably will).

```R
# Insert stock quotes function
insert_stock_quotes <- function(date_from = "2016-08-01", date_to = Sys.Date(), user, password) {
  
  # Set current time
  current_time <- Sys.time()
  
  # Symbols query
  sym_query <- "select id_symbol, ticker, instrument, name, sector, currency from equities.symbol;"
  
  # Vendor query
  ven_query <- "select id_data_vendor from equities.data_vendor where name = 'Yahoo Finance';"
  
  # Connect to database and send queries
  ch <- odbcConnect("securities", uid = user, pwd = password)
  sym <- sqlQuery(ch, sym_query, stringsAsFactors = FALSE)
  ven <- sqlQuery(ch, ven_query, stringsAsFactors = FALSE)
  
  # Extract ticker symbols
  ticker <- sym %$%
    ticker
  
  # Iterate over each symbol.
  json_res <- lapply(ticker, function(x) {
    
    # Paste ticker symbols to construct call to Yahoo API
    url <- paste0('https://query.yahooapis.com/v1/public/yql?q=select+*+from+yahoo.finance.historicaldata+where+symbol+in+("',
                  x,
                  '")+and+startDate="',
                  date_from,
                  '"+and+endDate="',
                  date_to,
                  '"&format=json&diagnostics=true&env=store://datatables.org/alltableswithkeys&callback=')
    
    # Get prices and convert result from json to R list.
    json <- GET(url) %>%
      content(as = "text", encoding = "UTF-8") %>%
      fromJSON()
    
    json %$%
      query %$%
      results %$%
      quote
    
  }) %>% 
    bind_rows()
  
  
  # Extract prices from list object and join to get id_symbol foreign key
  res <- json_res %>%
    transmute(id_data_vendor = ven$id_data_vendor,
              ticker = Symbol,
              date = Date,
              created_datetime = current_time,
              last_updated_datetime = current_time,
              open = Open,
              high = High,
              low = Low,
              close = Close,
              adj_close = Adj_Close,
              volume = Volume) %>%
    inner_join(sym %>%
                 select(id_symbol, ticker)) %>% 
    select(-ticker)
  
  # Concatenate to string
  res_string <- res %>% 
    apply(1, paste, collapse = "', '") %>%
    paste(collapse = "'), ('") %>%
    paste0("('", ., "')")
  
  # Delete existing data
  del <- paste0("delete from equities.daily_price where date between '",
                date_from,
                "' and '",
                date_to,
                "' and id_symbol in ('",
                paste(res$id_symbol %>% unique, sep = ', '),
                "')")
  
  sqlQuery(ch, del)
  
  # Insert new data
  query <- paste0("insert into equities.daily_price ",
                  "(id_data_vendor, date, created_datetime, last_updated_datetime, open, high, low, close, adj_close, volume, id_symbol)",
                  "values ",
                  res_string)
    
  sqlQuery(ch, query)
  odbcClose(ch)
  
}

# Insert stock quotes function
insert_stock_quotes(date_from = "2016-08-01",
                    date_to = Sys.Date(),
                    user = "User Name",
                    password = "Password")
```

And that's all. Of course, if one wanted to keep the data updated, the insert_stock_quotes function could be set to run each day via a [Cron](https://en.wikipedia.org/wiki/Cron) job.