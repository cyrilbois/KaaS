# R Package - DBI

*Source: [R Database Interface • DBI (r-dbi.org)](https://dbi.r-dbi.org/) | [r-dbi/DBI: A database interface (DBI) definition for communication between R and RDBMSs (github.com)](https://github.com/r-dbi/DBI)*

## Contents

* [Overview](R%20Package%20-%20DBI.md#overview)
* [Installation](R%20Package%20-%20DBI.md#installation)
* [Example](R%20Package%20-%20DBI.md#example)
* [Class structure](R%20Package%20-%20DBI.md#class-structure)
* [Further Reading](R%20Package%20-%20DBI.md#further-reading)
* [Appendix: Links](R%20Package%20-%20DBI.md#appendix-links)

## Overview

The DBI package helps connecting R to database management systems
(DBMS). DBI separates the connectivity to the DBMS into a “front-end”
and a “back-end”. The package defines an interface that is implemented
by *DBI backends* such as:

* [RPostgres](https://rpostgres.r-dbi.org),
* [RMariaDB](https://rmariadb.r-dbi.org),
* [RSQLite](https://rsqlite.r-dbi.org),
* [odbc](https://github.com/r-dbi/odbc),
* [bigrquery](https://github.com/r-dbi/bigrquery),

and many more, see the [list of backends](https://github.com/r-dbi/backends#readme). R scripts and packages use DBI to access various databases through their DBI backends.

The interface defines a small set of classes and methods similar in spirit to Perl’s [DBI](https://dbi.perl.org/), Java's [JDBC](https://www.oracle.com/java/technologies/javase/javase-tech-database.html), Python’s [DB-API](https://www.python.org/dev/peps/pep-0249/), and Microsoft’s [ODBC](https://en.wikipedia.org/wiki/ODBC). It supports the following operations:

* connect/disconnect to the DBMS
* create and execute statements in the DBMS
* extract results/output from statements
* error/exception handling
* information (meta-data) from database objects
* transaction management (optional)

## Installation

Most users who want to access a database do not need to install DBI directly. It will be installed automatically when you install one of the database backends:

* [RPostgres](https://rpostgres.r-dbi.org) for PostgreSQL,
* [RMariaDB](https://rmariadb.r-dbi.org) for MariaDB or MySQL,
* [RSQLite](https://rsqlite.r-dbi.org) for SQLite,
* [odbc](https://github.com/r-dbi/odbc) for databases that you can access via [ODBC](https://en.wikipedia.org/wiki/Open_Database_Connectivity),
* [bigrquery](https://github.com/r-dbi/bigrquery)

You can install the released version of DBI from [CRAN](https://CRAN.R-project.org) with:

````R
install.packages("DBI")
````

And the development version from [GitHub](https://github.com/) with:

````R
# install.packages("devtools")
devtools::install_github("r-dbi/DBI")
````

## Example

The following example illustrates some of the DBI capabilities:

````r
library(DBI)
# Create an ephemeral in-memory RSQLite database
con <- dbConnect(RSQLite::SQLite(), dbname = ":memory:")

dbListTables(con)
#> character(0)
dbWriteTable(con, "mtcars", mtcars)
dbListTables(con)
#> [1] "mtcars"

dbListFields(con, "mtcars")
#>  [1] "mpg"  "cyl"  "disp" "hp"   "drat" "wt"   "qsec" "vs"   "am"   "gear"
#> [11] "carb"
dbReadTable(con, "mtcars")
#>    mpg cyl  disp  hp drat    wt  qsec vs am gear carb
#> 1 21.0   6 160.0 110 3.90 2.620 16.46  0  1    4    4
#> 2 21.0   6 160.0 110 3.90 2.875 17.02  0  1    4    4
#> 3 22.8   4 108.0  93 3.85 2.320 18.61  1  1    4    1
#> 4 21.4   6 258.0 110 3.08 3.215 19.44  1  0    3    1
#> 5 18.7   8 360.0 175 3.15 3.440 17.02  0  0    3    2
#> 6 18.1   6 225.0 105 2.76 3.460 20.22  1  0    3    1
#> 7 14.3   8 360.0 245 3.21 3.570 15.84  0  0    3    4
#> 8 24.4   4 146.7  62 3.69 3.190 20.00  1  0    4    2
#> 9 22.8   4 140.8  95 3.92 3.150 22.90  1  0    4    2
#>  [ reached 'max' / getOption("max.print") -- omitted 23 rows ]

# You can fetch all results:
res <- dbSendQuery(con, "SELECT * FROM mtcars WHERE cyl = 4")
dbFetch(res)
#>    mpg cyl  disp hp drat    wt  qsec vs am gear carb
#> 1 22.8   4 108.0 93 3.85 2.320 18.61  1  1    4    1
#> 2 24.4   4 146.7 62 3.69 3.190 20.00  1  0    4    2
#> 3 22.8   4 140.8 95 3.92 3.150 22.90  1  0    4    2
#> 4 32.4   4  78.7 66 4.08 2.200 19.47  1  1    4    1
#> 5 30.4   4  75.7 52 4.93 1.615 18.52  1  1    4    2
#> 6 33.9   4  71.1 65 4.22 1.835 19.90  1  1    4    1
#> 7 21.5   4 120.1 97 3.70 2.465 20.01  1  0    3    1
#> 8 27.3   4  79.0 66 4.08 1.935 18.90  1  1    4    1
#> 9 26.0   4 120.3 91 4.43 2.140 16.70  0  1    5    2
#>  [ reached 'max' / getOption("max.print") -- omitted 2 rows ]
dbClearResult(res)

# Or a chunk at a time
res <- dbSendQuery(con, "SELECT * FROM mtcars WHERE cyl = 4")
while(!dbHasCompleted(res)){
  chunk <- dbFetch(res, n = 5)
  print(nrow(chunk))
}
#> [1] 5
#> [1] 5
#> [1] 1
dbClearResult(res)

dbDisconnect(con)
````

## Class structure

There are four main DBI classes. Three which are each extended by
individual database backends:

* `DBIObject`: a common base class for all DBI.
* `DBIDriver`: a base class representing overall DBMS properties. Typically generator functions instantiate the driver objects like `RSQLite()`, `RPostgreSQL()`, `RMySQL()` etc.
* `DBIConnection`: represents a connection to a specific database
* `DBIResult`: the result of a DBMS query or statement.

All classes are *virtual*: they cannot be instantiated directly and instead must be sub-classed.

## Further Reading

* [Databases using R](https://db.rstudio.com/) describes the tools and best practices in this ecosystem.
* The [DBI project site](https://www.r-dbi.org/) hosts a blog where recent developments are presented.
* [A history of DBI](https://r-dbi.github.io/DBI/articles/DBI-history.html) by David James, the driving force behind the development of DBI, and many of the packages that implement it.

---

## Appendix: Links

* [Tools](../../../../../Tools.md)
* [Development](../../../../../../../2-Areas/MOCs/Development.md)
  \<\<\<\<\<\<\< HEAD:3-Resources/Tools/R/R Packages/Database R Packages/R Package - DBI.md
* [R](../../../../../../../2-Areas/Code/R/R.md)
  =======
* [2-Areas/MOCs/R](../../../../../../../2-Areas/MOCs/R.md)
  \>>>>>>> develop:3-Resources/Tools/Developer Tools/Languages/R/R Packages/Database R Packages/R Package - DBI.md
* [Databases](../../../../../../../2-Areas/MOCs/Databases.md)

*Backlinks:*

````dataview
list from [[R Package - DBI]] AND -"Changelog"
````
