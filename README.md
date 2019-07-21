# GretlGraphiteConnector

This package sends metrics to the Graphite monitoring tool which collects, stores, and displays time-series data in real time. The package is written for the open-source statistics and econometrics software ([gretl](http://gretl.sourceforge.net/)).

Gretl calls through shell commands the unix tool ([netcat](http://netcat.sourceforge.net/)) -- hence, this package is only working under linux or some other unix system **but not Windows**.

## Main function
The package's main function is ```GretlGraphiteConnector()```. The function takes the following arguments:
```
GretlGraphiteConnector (const string use_case_name "Name of use case to build metric prefixes",
                        const string identifier[null] "Identifier, e.g. team name",
                        const string GRAPHITE_HOST_DEV[null] "Server URL for dev-runmode",
                        const string GRAPHITE_HOST_PRD[null] "Server URL for prd-runmode",
                        const string INSTANCE_NAME_DEV[null] "Instance name of the dev-runmode",
                        const string INSTANCE_NAME_PRD[null] "Instance name of the prd-runmode",
                        const bool dev[0] "Current run mode: 0=prd (default), 1=dev",
      			const bool netcat[1] "Use netcat: 0=use socket (unsupported), 1=use netcat (default)")
```
## The *defaultGC()* function
The user can also specify default values for *GRAPHITE_HOST_DEV*, *GRAPHITE_HOST_PRD*, *INSTANCE_NAME_DEV* and *INSTANCE_NAME_PRD* which are internally set by the ```defaultGC()``` function:
```
function bundle defaultGC (void)
/* Set default property and connection value */
    bundle self = null
    string self.identifier = "MyTeam"
    string self.use_case_name = ""
    string self.GRAPHITE_HOST_DEV = "some_server-dev.openshift.foo.com"
    string self.GRAPHITE_HOST_PRD = "some_server-prd.openshift.foo.com"
    string self.INSTANCE_NAME_DEV = "test"
    string self.INSTANCE_NAME_PRD = "prod"

    return self
end function
```

## The *sendTimestampNow*
This function simply sends the current Unix time-stamp to Graphite for a given metric name:
```
function void sendTimestampNow (bundle *self, const string metric_name "name of the metric")
```

## The *sendWithDate*
This function allows you to set a metric name, a metric value and a date string. Optionally, the metric can be appended for bulk load:
```
function void sendWithDate (bundle *self, const string metric_name,
                            const string metric_value "Value of the metric",
                            const string send_date "Date string: either Unix timestamp or use the format %Y-%m-%d",
                            const bool buffer[0] "Append metric to bulk load (default: No)")
```

## Sample script (see also "./src/graphite_sample.inp")
```
clear
set verbose off

# Set working dir
string wd = "/home/at/git/graphite_gretl_connector/src/

# Load the gretl script file
include "@wd/graphite.inp" --force

# Set up the GC struct (in gretl terms aka bundle) with default
# connection settings
string use_case_name = "SomeUseCase"
bundle GC = GraphiteConnector(use_case_name)

# Example 1: Send the current timesampt as the metric
string metric_name = "current_ts_new"
sendTimestampNow(&GC, metric_name)

# Example 2: Set some metric value with some date string
string metric_name = "SomeImportantMetric"
string some_value = "22"
string date_string = sprintf("%d", $now[1])
sendWithDate(&GC, metric_name, some_value, date_string)


# Bulk load data
string metric_name = "bulk"
loop i=1..3 -q
	# This is actually just the filling of the buffer     
    sendWithDate(&GC, metric_name, sprintf("%d", $i^2), sprintf("%d", $now[1]), 1)		# use the current timestamp
    if i<3
        sleep(2)		# artifical delay
    endif    
endloop
buffer = GC.buffer
buffer						# Buffer stored
sendBuffer(&GC)				# Actual sending of the bulk-load
```
