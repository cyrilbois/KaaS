# R Package - callr

*Source: [r-lib/callr: Call R from R (github.com)](https://github.com/r-lib/callr)*

See Also: [Multi-Process Task Queue in R](../../../../../../../0-Slipbox/Multi-Process%20Task%20Queue%20in%20R.md)

## Contents

* [Overview](R%20Package%20-%20callr.md#overview)
* [Features](R%20Package%20-%20callr.md#features)
* [Installation](R%20Package%20-%20callr.md#installation)
* [Synchronous, one-off R processes](R%20Package%20-%20callr.md#synchronous-one-off-r-processes)
  * [Passing arguments](R%20Package%20-%20callr.md#passing-arguments)
  * [Using packages](R%20Package%20-%20callr.md#using-packages)
  * [Error handling](R%20Package%20-%20callr.md#error-handling)
  * [Standard output and error](R%20Package%20-%20callr.md#standard-output-and-error)
* [Background R processes](R%20Package%20-%20callr.md#background-r-processes)
* \[\[\#Multiple background R processes and `poll()`\|Multiple background R processes and `poll()`\]\]
* [Persistent R sessions](R%20Package%20-%20callr.md#persistent-r-sessions)
* \[\[\#Running `R CMD` commands|Running `R CMD` commands\]\]
* [Appendix: Links](R%20Package%20-%20callr.md#appendix-links)

## Overview

 > 
 > Call R from R

It is sometimes useful to perform a computation in a separate R process, without affecting the current R process at all. This packages does exactly that.

## Features

* Calls an R function, with arguments, in a subprocess.
* Copies function arguments to the subprocess and copies the return value of the function back, seamlessly.
* Copies error objects back from the subprocess, including a stack trace.
* Shows and/or collects the standard output and standard error of the subprocess.
* Supports both one-off and persistent R subprocesses.
* Calls the function synchronously or asynchronously (in the background).
* Can call `R CMD` commands, synchronously or asynchronously.
* Can call R scripts, synchronously or asynchronously.
* Provides extensible `r_process`, `rcmd_process` and `rscript_process` R6 classes, based on `processx::process`.

## Installation

Install the stable version from CRAN:

````r
install.packages("callr")
````

## Synchronous, one-off R processes

Use `r()` to run an R function in a new R process. The results are
passed back seamlessly:

````r
library(callr)
r(function() var(iris[, 1:4]))
````

````
   #>              Sepal.Length Sepal.Width Petal.Length Petal.Width
    #> Sepal.Length    0.6856935  -0.0424340    1.2743154   0.5162707
    #> Sepal.Width    -0.0424340   0.1899794   -0.3296564  -0.1216394
    #> Petal.Length    1.2743154  -0.3296564    3.1162779   1.2956094
    #> Petal.Width     0.5162707  -0.1216394    1.2956094   0.5810063
````

### Passing arguments

You can pass arguments to the function by setting `args` to the list of arguments. This is often necessary as these arguments are explicitly copied to the child process, whereas the evaluated function cannot refer to variables in the parent. 

For example, the following does not work:

````r
mycars <- cars
r(function() summary(mycars))
````

````
    #> Error: callr subprocess failed: object 'mycars' not found
````

But this does:

````r
r(function(x) summary(x), args = list(mycars))
````

````
   #>      speed           dist       
    #>  Min.   : 4.0   Min.   :  2.00  
    #>  1st Qu.:12.0   1st Qu.: 26.00  
    #>  Median :15.0   Median : 36.00  
    #>  Mean   :15.4   Mean   : 42.98  
    #>  3rd Qu.:19.0   3rd Qu.: 56.00  
    #>  Max.   :25.0   Max.   :120.00
````

Note that the arguments will be serialized and saved to a file, so if they are large R objects, it might take a long time for the child process to start up.

### Using packages

You can use any R package in the child process, just make sure to refer to it explicitly with the `::` operator.

For example, the following code creates an [igraph](https://github.com/igraph/rigraph) graph in the child, and calculates some metrics of it.

````r
r(function() { g <- igraph::sample_gnp(1000, 4/1000); igraph::diameter(g) })
````

````
    #> [1] 14
````

### Error handling

`callr` copies errors from the child process back to the main R session:

````r
r(function() 1 + "A")
````

````
    #> Error: callr subprocess failed: non-numeric argument to binary operator
````

`callr` sets the `.Last.error` variable, and after an error you can inspect this for more details about the error, including stack traces both from the main R process and the subprocess.

````r
.Last.error
````

````
    #> <callr_status_error: callr subprocess failed: non-numeric argument to binary operator>
    #> -->
    #> <callr_remote_error in 1 + "A":
    #>  non-numeric argument to binary operator>
    #>  in process 32341
````

The error objects has two parts. The first belongs to the main process, and the second belongs to the subprocess.

`.Last.error` also includes a stack trace, that includes both the main R process and the subprocess:

````r
.Last.error.trace
````

````
   #>  Stack trace:
    #> 
    #>  Process 32229:
    #>  36. callr:::r(function() 1 + "A")
    #>  37. callr:::get_result(output = out, options)
    #>  38. throw(newerr, parent = remerr[[2]])
    #> 
    #>  x callr subprocess failed: non-numeric argument to binary operator 
    #> 
    #>  Process 32341:
    #>  50. (function ()  ...
    #>  51. base:::.handleSimpleError(function (e)  ...
    #>  52. h(simpleError(msg, call))
    #> 
    #>  x non-numeric argument to binary operator
````

The top part of the trace contains the frames in the main process, and the bottom part contains the frames in the subprocess, starting with the anonymous function.

### Standard output and error

By default, the standard output and error of the child is lost, but you can request `callr` to redirect them to files, and then inspect the files in the parent:

````r
x <- r(function() { print("hello world!"); message("hello again!") },
  stdout = "/tmp/out", stderr = "/tmp/err"
)
readLines("/tmp/out")
````

````
#> [1] "[1] \"hello world!\""
````

````r
readLines("/tmp/err")
````

````
#> [1] "hello again!"
````

With the `stdout` option, the standard output is collected and can be examined once the child process finished. The `show = TRUE` options will also show the output of the child, as it is printed, on the console of the parent.

## Background R processes

`r_bg()` is similar to `r()` but it starts the R process in the background. It returns an `r_process` R6 object, that provides a rich API:

````r
rp <- r_bg(function() Sys.sleep(.2))
rp
````

````
    #> PROCESS 'R', running, pid 32379.
````

This is a list of all `r_process` methods:

````r
ls(rp)
````

````
   #>  [1] "as_ps_handle"          "clone"                 "finalize"             
    #>  [4] "format"                "get_cmdline"           "get_cpu_times"        
    #>  [7] "get_error_connection"  "get_error_file"        "get_exe"              
    #> [10] "get_exit_status"       "get_input_connection"  "get_input_file"       
    #> [13] "get_memory_info"       "get_name"              "get_output_connection"
    #> [16] "get_output_file"       "get_pid"               "get_poll_connection"  
    #> [19] "get_result"            "get_start_time"        "get_status"           
    #> [22] "get_username"          "get_wd"                "has_error_connection" 
    #> [25] "has_input_connection"  "has_output_connection" "has_poll_connection"  
    #> [28] "initialize"            "interrupt"             "is_alive"             
    #> [31] "is_incomplete_error"   "is_incomplete_output"  "is_supervised"        
    #> [34] "kill"                  "kill_tree"             "poll_io"              
    #> [37] "print"                 "read_all_error"        "read_all_error_lines" 
    #> [40] "read_all_output"       "read_all_output_lines" "read_error"           
    #> [43] "read_error_lines"      "read_output"           "read_output_lines"    
    #> [46] "resume"                "signal"                "supervise"            
    #> [49] "suspend"               "wait"                  "write_input"
````

These include all methods of the `processx::process` superclass and the new `get_result()` method, to retrieve the R object returned by the function call. Some of the handiest methods are:

* `get_exit_status()` to query the exit status of a finished process.
* `get_result()` to collect the return value of the R function call.
* `interrupt()` to send an interrupt to the process. This is equivalent to a `CTRL+C` key press, and the R process might ignore it.
* `is_alive()` to check if the process is alive.
* `kill()` to terminate the process.
* `poll_io()` to wait for any standard output, standard error, or the completion of the process, with a timeout.
* `read_*()` to read the standard output or error.
* `suspend()` and `resume()` to stop and continue a process.
* `wait()` to wait for the completion of the process, with a timeout.

## Multiple background R processes and `poll()`

Multiple background R processes are best managed with the `processx::poll()` function that waits for events (standard output/error or termination) from multiple processes. It returns as soon as one process has generated an event, or if its timeout has expired. The timeout is in milliseconds.

````r
rp1 <- r_bg(function() { Sys.sleep(1/2); "1 done" })
rp2 <- r_bg(function() { Sys.sleep(1/1000); "2 done" })
processx::poll(list(rp1, rp2), 1000)
````

````
   #> [[1]]
    #>   output    error  process 
    #> "silent" "silent" "silent" 
    #> 
    #> [[2]]
    #>   output    error  process 
    #> "silent" "silent"  "ready"
````

````r
rp2$get_result()
````

````
    #> [1] "2 done"
````

````r
processx::poll(list(rp1), 1000)
````

````
   #> [[1]]
    #>   output    error  process 
    #> "silent" "silent"  "ready"
````

````r
rp1$get_result()
````

````
   #> [1] "1 done"
````

## Persistent R sessions

`r_session` is another `processx::process` subclass that represents a persistent background R session:

````r
rs <- r_session$new()
rs
````

````
    #> R SESSION, alive, idle, pid 32412.
````

`r_session$run()` is a synchronous call, that works similarly to `r()`, but uses the persistent session. `r_session$call()` starts the function call and returns immediately. The `r_session$poll_process()` method or `processx::poll()` can then be used to wait for the completion or other events from one or more R sessions, R processes or other `processx::process` objects.

Once an R session is done with an asynchronous computation, its `poll_process()` method returns `"ready"` and the `r_session$read()` method can read out the result.

````r
rs$run(function() runif(10))
````

````
   #>  [1] 0.75342837 0.12946532 0.98800304 0.09682751 0.23944882 0.99726443
    #>  [7] 0.91098802 0.61136112 0.51781725 0.53566166
````

````r
rs$call(function() rnorm(10))
rs
````

````
   #> R SESSION, alive, busy, pid 32412.
````

````r
rs$poll_process(2000)
````

````
   #> [1] "ready"
````

````r
rs$read()
````

````
   #> $code
    #> [1] 200
    #> 
    #> $message
    #> [1] "done callr-rs-result-7de57e80bd54"
    #> 
    #> $result
    #>  [1]  0.73848421 -0.07600563 -1.18598532  0.10692265 -0.11717386 -0.24769265
    #>  [7] -0.13800969 -0.97854700 -0.30949881 -1.57689514
    #> 
    #> $stdout
    #> [1] ""
    #> 
    #> $stderr
    #> [1] ""
    #> 
    #> $error
    #> NULL
    #> 
    #> attr(,"class")
    #> [1] "callr_session_result"
````

## Running `R CMD` commands

The `rcmd()` function calls an `R CMD` command. For example, you can call `R CMD INSTALL`, `R CMD check` or `R CMD config` this way:

````r
rcmd("config", "CC")
````

````
   #> $status
    #> [1] 0
    #> 
    #> $stdout
    #> [1] "clang -mmacosx-version-min=10.13\n"
    #> 
    #> $stderr
    #> [1] ""
    #> 
    #> $timeout
    #> [1] FALSE
    #> 
    #> $command
    #> [1] "/Library/Frameworks/R.framework/Versions/4.0/Resources/bin/R"
    #> [2] "CMD"                                                         
    #> [3] "config"                                                      
    #> [4] "CC"
````

````r
#>$stdout
#>[1] "clang\n"
#>
#>$stderr
#>[1] ""
#>
#>$status
#>[1] 0
````

This returns a list with three components: the standard output, the standard error, and the exit (status) code of the `R CMD` command.

---

## Appendix: Links

* [Tools](../../../../../Tools.md)
* [Development](../../../../../../../2-Areas/MOCs/Development.md)
  \<\<\<\<\<\<\< HEAD:3-Resources/Tools/R/R Packages/General R Packages/R Package - callr.md
* [R](../../../../../../../2-Areas/Code/R/R.md)
* *3-Resources/Tools/R/R Packages/R Packages*
* [R - Run Shiny App in Background for Development](../../../../../../../2-Areas/Code/R/R%20-%20Run%20Shiny%20App%20in%20Background%20for%20Development.md)
* [R Shiny](../../../../../../../2-Areas/MOCs/R%20Shiny.md)
* *3-Resources/Tools/R/R Packages/API R Packages/R Package - plumber*
  =======
* [2-Areas/MOCs/R](../../../../../../../2-Areas/MOCs/R.md)
* *3-Resources/Tools/Developer Tools/Programming Languages/R/R Packages/R Packages*
* [R - Run Shiny App in Background for Development](../../../../../../../2-Areas/Code/R/R%20-%20Run%20Shiny%20App%20in%20Background%20for%20Development.md)
* [R Shiny](../../../../../../../2-Areas/MOCs/R%20Shiny.md)
* [R Package - plumber](../API%20R%20Packages/R%20Package%20-%20plumber.md)
  \>>>>>>> develop:3-Resources/Tools/Developer Tools/Languages/R/R Packages/General R Packages/R Package - callr.md

*Backlinks:*

````dataview
list from [[R Package - callr]] AND -"Changelog"
````
