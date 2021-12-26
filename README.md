
<!-- README.md is generated from README.Rmd. Please edit that file -->

# dyn.log

<!-- badges: start -->

[![R-CMD-check](https://github.com/bmoretz/dyn.log/workflows/R-CMD-check/badge.svg)](https://github.com/bmoretz/dyn.log/actions)
[![Codecov test
coverage](https://codecov.io/gh/bmoretz/dyn.log/branch/master/graph/badge.svg)](https://app.codecov.io/gh/bmoretz/dyn.log?branch=master)
<!-- badges: end -->

The goal of dyn.log is to be a comprehensive and dynamic configuration
driven logging package for R. While there are several excellent logging
solutions already in the R ecosystem, I always feel constrained in some
way by each of them. Every project is designed differently to achieve
its stated goal; to solve some problem, and ultimately the utility of a
logging solution is its ability to adapt to the project’s design. This
is the rai·​son d’être for dyn.log; to provide a modular design,
template mechanics and a configuration-based integration model. This
enableds the logger to integrate deeply into your design, even though it
knows nothing about it.

## Installation

#### GitHub

You can install the development version of dyn.log from
[GitHub](https://github.com/) with:

``` r
remotes::install_github("bmoretz/dyn.log")
```

#### CRAN

You can install the latest stable version of dyn.log from CRAN:

(coming soon)

### Overview

``` r
library(dyn.log)
```

<STYLE type='text/css' scoped>
PRE.fansi SPAN {padding-top: .25em; padding-bottom: .25em};
</STYLE>

#### Configuration

Everything about dyn.log is configuration driven, the package comes with
a basic configuration **default.yaml**, show below it its entirety and
broken down in the sections that follow:

``` yaml
settings:
  threshold: TRACE
  max_callstack: 5
levels:
- name: TRACE
  description: This level designates finer-grained informational events than the DEBUG.
  severity: !expr 600L
  log_style: !expr crayon::make_style("antiquewhite3")$bold
  msg_style: !expr crayon::make_style('gray60')
- name: DEBUG
  description: This level designates fine-grained informational events that are most useful to debug an application.
  severity: !expr 500L
  log_style: !expr crayon::make_style('deepskyblue2')$bold
  msg_style: !expr crayon::make_style('gray90')
- name: INFO
  description: This level designates informational messages that highlight the progress of the application at coarse-grained level.
  severity: !expr 400L
  log_style: !expr crayon::make_style('dodgerblue4')$bold
  msg_style: !expr crayon::make_style('gray100')
- name: SUCCESS
  description: This level designates that the operation was unencumbered.
  severity: !expr 300L
  log_style: !expr crayon::make_style('chartreuse')$bold
  msg_style: !expr crayon::bgGreen$bold$black
- name: WARN
  description: This level designates potentially harmful situations.
  severity: !expr 350L
  log_style: !expr crayon::make_style('darkorange')$bold
  msg_style: !expr crayon::bgYellow$bold$black
- name: ERROR
  description: This level designates error events that might still allow the application to continue running.
  severity: !expr 200L
  log_style: !expr crayon::make_style('firebrick1')$bold
  msg_style: !expr crayon::bgBlack$bold$white
- name: FATAL
  description: This level designates very severe error events that will presumably lead the application to abort.
  severity: !expr 100L
  log_style: !expr crayon::make_style('firebrick')$bold
  msg_style: !expr crayon::bgRed$bold$white
layouts:
- association: default
  seperator: !expr as.character(' ')
  new_line: !expr as.character('\n')
  formats: new_fmt_log_level(),
           new_fmt_timestamp(crayon::silver$italic),
           new_fmt_log_msg()
- association: level_msg
  seperator: !expr as.character(' ')
  new_line: !expr as.character('\n')
  formats: new_fmt_log_level(),
           new_fmt_log_msg()
```

#### Settings

The **settings** node contains the core settings of the log dispatcher,
by attribute. These are covered in detail in the *LogDispatch* section
of the manual.

> ?LogDispatch

Most of these should be fairly intuitive.

#### Levels

The **levels** node contains the log levels you want available in your
environment. When a log level is defined in the configuration, it
automatically becomes accessible via a first-class function on the
dispatcher, e.g.:

``` r
Logger$info("call info level like this")
```

<PRE class="fansi fansi-output"><CODE>#&gt; <span style='color: #0000BB; font-weight: bold;'>INFO</span> <span style='color: #555555; font-style: italic;'>[12/25/21 11:28:49 -0500]</span> <span style='color: #BBBBBB;'>call info level like this</span>
</CODE></PRE>

All levels that have been specified via the config can be found by
calling *log_levels*:

``` r
all_levels <- log_levels()

names(all_levels)
#> [1] "trace"   "debug"   "info"    "success" "warn"    "error"   "fatal"
```

And you can get detailed information about a particular log level via
*level_info()*:

``` r
level_info(log_levels("warn"))
```

<PRE class="fansi fansi-output"><CODE>#&gt; $name
#&gt; [1] &quot;WARN&quot;
#&gt; 
#&gt; $description
#&gt; [1] &quot;This level designates potentially harmful situations.&quot;
#&gt; 
#&gt; $severity
#&gt; [1] 350
#&gt; 
#&gt; $style
#&gt; $style$level
#&gt; Crayon style function, darkorange, bold: <span style='color: #BBBB00; font-weight: bold;'>example output.</span>
#&gt; 
#&gt; $style$message
#&gt; Crayon style function, bgYellow, bold, black: <span style='color: #000000; background-color: #BBBB00; font-weight: bold;'>example output.</span>
#&gt; 
#&gt; $style$example
#&gt; <span style='color: #BBBB00; font-weight: bold;'>WARN</span> - <span style='color: #000000; background-color: #BBBB00; font-weight: bold;'>This level designates potentially harmful situations.</span>
</CODE></PRE>

You’ll notice that a level has **level** and **message** style
attributes that are [crayon](https://github.com/r-lib/crayon) styles.
This is because each level has a unique render format that helps to
visually assist parsing of log information.

To see the sample format of all loaded log levels, call
*display_log_levels*:

``` r
display_log_levels()
```

<PRE class="fansi fansi-output"><CODE>#&gt; 
#&gt; <span style='color: #555555; font-weight: bold;'>TRACE</span> <span style='color: #555555;'>This level designates finer-grained informational events than the DEBUG.</span>
#&gt; 
#&gt; <span style='color: #00BBBB; font-weight: bold;'>DEBUG</span> <span style='color: #BBBBBB;'>This level designates fine-grained informational events that are most useful to debug an application.</span>
#&gt; 
#&gt; <span style='color: #0000BB; font-weight: bold;'>INFO</span> <span style='color: #BBBBBB;'>This level designates informational messages that highlight the progress of the application at coarse-grained level.</span>
#&gt; 
#&gt; <span style='color: #00BB00; font-weight: bold;'>SUCCESS</span> <span style='color: #000000; background-color: #00BB00; font-weight: bold;'>This level designates that the operation was unencumbered.</span>
#&gt; 
#&gt; <span style='color: #BBBB00; font-weight: bold;'>WARN</span> <span style='color: #000000; background-color: #BBBB00; font-weight: bold;'>This level designates potentially harmful situations.</span>
#&gt; 
#&gt; <span style='color: #BB0000; font-weight: bold;'>ERROR</span> <span style='color: #BBBBBB; background-color: #000000; font-weight: bold;'>This level designates error events that might still allow the application to continue running.</span>
#&gt; 
#&gt; <span style='color: #BB0000; font-weight: bold;'>FATAL</span> <span style='color: #BBBBBB; background-color: #BB0000; font-weight: bold;'>This level designates very severe error events that will presumably lead the application to abort.</span>
</CODE></PRE>

The default logging configuration closely resembles the fairly
ubiquitous
[log4j](https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/Level.html)
scheme.

#### Layouts

Every log message needs to have a format so the dispatcher knows what to
render on a log call. Formats are defined in the yaml config and comes
with some basic ones pre-configured.

The default log layout is a standard format: {LEVEL} - {TIMESTAMP} -
{MSG}, with space as a separator between format objects and the new line
feed *“”*.

To view the details of a log layout, you can call *log_layout_detail*:

``` r
detail <- log_layout_detail(log_layouts("default"))

names(detail)
#> [1] "formats"   "types"     "seperator" "new_line"
```

For a detailed look at these objects, and how they work please see the
“Log Layouts” *vignette*.

### Logging

The logging functionality is exposed by a R6 class, *LogDispatch*, that
is available as a package namespace variable called **Logger**. The
**Logger** will have methods that correspond to the *log levels* that
are defined in its yaml configuration, which makes logging intuitive.
When the package is loaded, the logger will appear in the top / global
environment as *Logger*.

Log messages are automatically assumed to be in standard
[glue](https://github.com/tidyverse/glue) format so local environment
variables are capturable in the log output.

#### Simple Example

The “out of the box” (OTB) configuration specifies a default vanilla log
format that displays the level that was logged, the current time-stamp
(with the default TS format), and the log message:

``` r
var1 <- "abc"; var2 <- 123; var3 <- runif(1)

Logger$debug("my log message - var1: {var1}, var2: {var2}, var3: {var3}")
```

<PRE class="fansi fansi-output"><CODE>#&gt; <span style='color: #00BBBB; font-weight: bold;'>DEBUG</span> <span style='color: #555555; font-style: italic;'>[12/25/21 11:28:49 -0500]</span> <span style='color: #BBBBBB;'>my log message - var1: abc, var2: 123, var3: 0.950028502847999</span>
</CODE></PRE>

### Customizing a Log Message

Log message layouts are exposed as an S3 type in the package called
*log_layout*. Layouts are composed from a series of objects that inherit
from *fmt_layout*.

``` r
new_log_layout(
  format = list(
    new_fmt_metric(crayon::green$bold, "sysname"),
    new_fmt_metric(crayon::red$bold, "release"),
    new_fmt_line_break(),
    new_fmt_log_level(),
    new_fmt_timestamp(crayon::silver$italic),
    new_fmt_log_msg()
  ),
  seperator = '-',
  association = "custom"
)

Logger$info("my log message - var1: {var1}, var2: {var2}, var3: {var3}", layout = "custom")
```

<PRE class="fansi fansi-output"><CODE>#&gt; <span style='color: #00BB00; font-weight: bold;'>Linux</span>-<span style='color: #BB0000; font-weight: bold;'>5.10.16.3-microsoft-standard-WSL2</span>
#&gt; <span style='color: #0000BB; font-weight: bold;'>INFO</span>-<span style='color: #555555; font-style: italic;'>[12/25/21 11:28:49 -0500]</span>-<span style='color: #BBBBBB;'>my log message - var1: abc, var2: 123, var3: 0.950028502847999</span>
</CODE></PRE>

For a detailed look at these objects, and how they work please see the
“Log Layouts” *vignette*.

> (vignette(“Log Layouts”, package = “dyn.log”)

### Logging Associations

One thing you may have noticed about the previous log layout definition
was the *association* parameter. Associations are a useful way to build
a customized log layout for your custom R6 types. This can be especially
useful in larger applications, such as
[plumber](https://github.com/rstudio/plumber/) services or
[shiny](https://github.com/rstudio/shiny) dashboards.

A TestObject is defined as below, who’s primary responsibly is to assign
a randomly generated identifier to the instance via the constructor.
There is also a method on the object that will call the logger with some
local scope variables that should be logged as well.

``` r
TestObject <- R6::R6Class(
  classname = "TestObject",

  public = list(
    id = NULL,

    initialize = function() {
      self$id <- private$generate_id()
    },

    test_method = function() {
      a <- "test"; b <- 123; c <- runif(1)

      Logger$info("these are some variables: {a} - {b} - {c}")
    }
  ),

  private = list(
    generate_id = function(n = 15) {
      paste0(sample(LETTERS, n, TRUE), collapse =  '')
    }
  )
)

obj <- TestObject$new()
obj
#> <TestObject>
#>   Public:
#>     clone: function (deep = FALSE) 
#>     id: OXDFSFRPYHNGJPX
#>     initialize: function () 
#>     test_method: function () 
#>   Private:
#>     generate_id: function (n = 15)
```

With the above class defined, we can create a custom log layout that
associated with this R6 type with a new log layout:

``` r
new_log_layout(
  format = list(
    new_fmt_literal(crayon::cyan$bold, "Object Id:"),
    new_fmt_cls_field(crayon::bgCyan$silver$bold, "id"),
    new_fmt_line_break(),
    new_fmt_log_level(),
    new_fmt_timestamp(crayon::silver$italic),
    new_fmt_log_msg(),
    new_fmt_line_break(),
    new_fmt_metric(crayon::green$bold, "sysname"),
    new_fmt_metric(crayon::red$bold, "nodename"),
    new_fmt_literal(crayon::blue$bold, "R Version:"),
    new_fmt_metric(crayon::blue$italic$bold, "r_ver"),
    new_fmt_line_break()
  ),
  association = "TestObject"
)

# notice above, "Logger$info" is called inside the context of the Test Object,
# and the variables are scoped to inside the function.
obj$test_method()
```

<PRE class="fansi fansi-output"><CODE>#&gt; <span style='color: #00BBBB; font-weight: bold;'>Object Id:</span> <span style='color: #555555; background-color: #00BBBB; font-weight: bold;'>OXDFSFRPYHNGJPX</span>
#&gt; <span style='color: #0000BB; font-weight: bold;'>INFO</span> <span style='color: #555555; font-style: italic;'>[12/25/21 11:28:49 -0500]</span> <span style='color: #BBBBBB;'>these are some variables: test - 123 - 0.943865999346599</span>
#&gt; <span style='color: #00BB00; font-weight: bold;'>Linux</span> <span style='color: #BB0000; font-weight: bold;'>WORKSTATION</span> <span style='color: #0000BB; font-weight: bold;'>R Version:</span> <span style='color: #0000BB; font-weight: bold; font-style: italic;'>4.1.2</span>
</CODE></PRE>

``` r
  
Logger$debug("this is a normal log msg")
```

<PRE class="fansi fansi-output"><CODE>#&gt; <span style='color: #00BBBB; font-weight: bold;'>DEBUG</span> <span style='color: #555555; font-style: italic;'>[12/25/21 11:28:49 -0500]</span> <span style='color: #BBBBBB;'>this is a normal log msg</span>
</CODE></PRE>

As you can see, only when the logger is invoked from inside the class
that has a custom layout associated with it does the custom layout get
used. The follow-up log call (outside the class scope) reverts back to
the standard layout settings.

For a detailed look at class associations, and how they work please see
the “Log Layouts” *vignette*.

> (vignette(“Log Layouts”, package = “dyn.log”)

## Acknowledgments

-   The pretty logger output in this document is made possible by the
    excellent [fansi](https://github.com/brodieG/fansi) package by
    [Brody Gaslam](https://twitter.com/BrodieGaslam).
