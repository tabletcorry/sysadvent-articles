All The Metrics! Or How You Too Can Graph Everything.
=====================================================

As your company grows, you may find that your existing metric
collection system(s) cannot keep up. Alternately, you may find that the
interface used to read those metrics does not work with everything that you
want to collect metrics for.

The problems that I have seen with existing solutions that I have used are:

* Munin: Fails to scale in all respects. Collection is more complicated than it
should be, and graphs are _pre rendered_ for every possible time window.
Needless to say, this does not scale well.
* Collectd: While this system is excellent at collecting data, the project does
not officially support any frontend. This has lead to a proliferation of
frontend projects that, if taken together, have all of the features you need,
but no one frontend does everything.
* XYMon: It's been some time since I used this, and I have not used it on a
large sest of systems. My guess is that it would suffer from some of Munin's
issues.


Enter Graphite
--------------

Graphite is a collection of services, written in python, that can replace or 
enchance your existing metric collection setup. The major components are:

* Whisper: Replaces RRD with a vastly simpler _storage only_ format. Unlike RRD
it cannot graph data directly.
* Carbon: Listens for data on a variety of interfaces (TCP, UDP, Pickle, AMQP)
and stores the data into whisper.
* Graphite-webapp: Graphs data from whisper or RRD files.

The best thing about the three components being independent is that you can run
graphite on your existing RRD data with no hassle. While there are advantages
to using whisper, it is not required to get the power of graphite.


The Power of Aggregation & Functions
------------------------------------

As wonderful as whisper and carbon are (and they really are worth using), the
true power of graphite lies in its web interface. Unlike some other interfaces,
graphite treats each metric as an independent data series. So long as you have
and understanding of the system, you can apply functions (think `sum`, `avg`,
`stddev`, etc.) to the metrics either by themselves, or more often, in 
aggregate.

As an example, if I wanted to find overloaded webservers I could construct a
query like: 
`highestAverage(webservers.*.load.longterm, 3)`
This would graph something like the following:

** Insert Image **

Another example, would be to graph the amount of unused memory on the 
webservers (time for more memcached if so!)
`sumSeries(movingAverage(webservers.*.memory.free, 10))`
Note that I am also creating a moving average over 10 datapoints here.

** Insert Image **

And this is only a small selection of the functions available to you. Moreover,
_you can write your own_! And easily too! Here is an example function in
graphite:

`
# A function to scale up all datapoints by a given factor
def scale(requestContext, seriesList, factor):
  for series in seriesList:
    series.name = "scale(%s,%.1f)" % (series.name,float(factor))
    for i,value in enumerate(series):
      series[i] = safeMul(value,factor)
  return seriesList
`

Yes, yes, its in python... But ruby is making too much progress in DevOps, we
need more variety.