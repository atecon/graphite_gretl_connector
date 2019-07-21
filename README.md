# graphite_gretl_connector

This gretl package sends metrics to the Graphite monitoring tool which collects, stores, and displays time-series data in real time.

The unix tool 'netcat' is called by gretl -- hence, this package is only working under linux or some other unix system **but not Windows**.

Example: See the sample script "graphite_sample.inp"

## Sample script (see also "./src/graphite_sample.inp")

`code'
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
`
TEX







