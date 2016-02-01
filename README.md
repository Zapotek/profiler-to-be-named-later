# Codename Mzr (pronounced "measure")

Hello, I'm [Tasos Laskos](https://twitter.com/Zap0tek), founder and lead developer of the [Arachni Web Application Security Scaner Framework](http://www.arachni-scanner.com), and this is a call for contributors for a Ruby profiler project codenamed Mzr.

## Basic idea

Based on my experiences while working on Arachni, the most effective way to identify memory leaks has been the simplest and most naive one, i.e. going over the `ObjectSpace` and gathering stats on live objects based on class type.

The way I've done this so far is by dumping these stats to a file at regular intervals and then monitoring that file from a terminal window.

Here's what that looks like:

```
RAM: 97.59MB (0.15%) - CPU: 56.94%

String:                                       129986 (-335) -- 12.671 (2.899)
Array:                                        21479 (374) -- 1.332 (0.02)
Hash:                                         5523 (223) -- 2.314 (0.093)
Arachni::HTTP::Headers:                       352 (12) -- 0.135 (0.007)
Ethon::Easy::DebugInfo::Message:              334 (2) -- 0.013 (0.0)
Set:                                          278 (6) -- 0.011 (0.0)
Arachni::HTTP::Response:                      215 (10) -- 0.032 (0.002)
Ethon::Easy::DebugInfo:                       172 (9) -- 0.007 (0.0)
Typhoeus::Response:                           171 (9) -- 0.01 (0.001)
Arachni::HTTP::Request:                       137 (2) -- 0.03 (0.0)
Typhoeus::Request:                            129 (2) -- 0.014 (0.0)
Arachni::URI:                                 119 (46) -- 0.014 (0.005)
Typhoeus::EasyFactory:                        110 (2) -- 0.004 (0.0)
Typhoeus::Response::Header:                   108 (9) -- 0.081 (0.008)
Arachni::Element::Form::DOM:                  102 (0) -- 0.015 (0.0)
Arachni::Element::Form:                       102 (0) -- 0.024 (0.0)
Arachni::HTTP::Response::Scope:               97 (1) -- 0.004 (0.0)
Nokogiri::XML::Attr:                          56 (28) -- 0.002 (0.001)
Arachni::URI::Scope:                          55 (31) -- 0.002 (0.001)
Arachni::Page::DOM::Transition:               46 (3) -- 0.004 (0.0)
Ethon::Easy:                                  45 (0) -- 0.008 (0.0)
Nokogiri::XML::Element:                       43 (8) -- 0.002 (0.0)
Arachni::HTTP::ProxyServer::Connection:       38 (0) -- 0.006 (0.0)
Arachni::HTTP::Request::Scope:                37 (2) -- 0.001 (0.0)
Arachni::Platform::List:                      35 (0) -- 0.001 (0.0)
Arachni::Support::LookUp::HashSet:            35 (0) -- 0.001 (0.0)
Thread:                                       25 (-8) -- 25.031 (-8.009)
Arachni::Support::Cache::LeastRecentlyPushed: 24 (0) -- 0.001 (0.0)
Arachni::Reactor::Tasks:                      18 (0) -- 0.001 (0.0)
Thread::Queue:                                17 (0) -- 0.001 (0.0)
Thread::ConditionVariable:                    16 (0) -- 0.001 (0.0)
Nokogiri::XML::Comment:                       13 (0) -- 0.0 (0.0)
Arachni::Browser::Javascript::Proxy::Stub:    12 (0) -- 0.0 (0.0)
Arachni::Element::Header:                     8 (0) -- 0.001 (0.0)
Arachni::Platform::Manager:                   7 (0) -- 0.0 (0.0)
Ethon::Easy::Mirror:                          7 (0) -- 0.0 (0.0)
Selenium::WebDriver::Remote::Bridge:          6 (0) -- 0.0 (0.0)
Arachni::Reactor::Tasks::Persistent:          6 (0) -- 0.0 (0.0)
Arachni::BrowserCluster::Worker:              6 (0) -- 0.002 (0.0)
Arachni::HTTP::ProxyServer:                   6 (0) -- 0.0 (0.0)
Arachni::Reactor:                             6 (0) -- 0.001 (0.0)
Selenium::WebDriver::Remote::Capabilities:    6 (0) -- 0.0 (0.0)
Selenium::WebDriver::Proxy:                   6 (0) -- 0.0 (0.0)
Selenium::WebDriver::Driver:                  6 (0) -- 0.0 (0.0)
Watir::Browser:                               6 (0) -- 0.0 (0.0)
Arachni::Support::Cache::LeastRecentlyUsed:   6 (0) -- 0.0 (0.0)
Arachni::Browser::Javascript:                 6 (0) -- 0.0 (0.0)
Arachni::Browser::Javascript::TaintTracer:    6 (0) -- 0.0 (0.0)
Arachni::Browser::Javascript::DOMMonitor:     6 (0) -- 0.0 (0.0)
Selenium::WebDriver::Remote::Http::Default:   6 (0) -- 0.0 (0.0)
Nokogiri::XML::SyntaxError:                   6 (6) -- 0.001 (0.001)
```

The format is fairly simple:

```
Class: total-count (delta from previous iteration) -- allocated memory in MB (delta from previous iteration)
```

Over time, the culprit becomes evident as its live object count keeps rising, regardless of garbage collection.

Still, this presentation is tiring and very limited as at any given time you only see the current and previous (based on deltas) states, making it hard to get a clear picture of any issues.

## Suggested project

Presenting the same data as live charts via a nice Rails-based user-interface would make identifying issues immensly easier.

In addition, the above runtime information would persist in a DB and be available for later review and comparison, allowing one to easily gauge the impact of implemented optimizations.

## Implementation

### Server -- WebUI (_Collaborators wanted_)

A fairly simple Rails application to accept (REST), store (PostgresSQL) and present (live charts) the profiling data for each application.

* `Application has_many :configurations`
* `Configuration has_many :runs`
* `Run has_many :performance_points`
* `PerformancePoint has_many :objects, :ram_usage, :ram_utilization, :cpu_utilization`
* etc.

### Client -- Profiler

Gem that includes the existing profiling code but transmits (REST) the data to the WebUI instead of writting to a file.
This can be done in regular intervals, on-demand or a combination.

For example, background sampling every 0.5s and on-demand creation of special events to appear alongside the data in the charts.

For a less abstract example, in a system like Arachni, you could sample in the background and create data points for each page audit, titled for the page URL; this would allow you to spot problem areas in the handling of pages with special characteristics -- complex page with lots of inputs resulting in more objects/RAM/CPU being used.

It sort of goes on and on from here but even at this simple state a system like this can be really helpful.

## Get in touch

If you're interested in the project and good with charting and Rails app development [please get in touch](https://github.com/Zapotek/profiler-to-be-named-later/issues/1).
