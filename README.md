[![Clojars Project](https://img.shields.io/clojars/v/jmh-clojure.svg)](https://clojars.org/jmh-clojure)
[![Travis CI](https://travis-ci.org/jgpc42/jmh-clojure.svg?branch=master)](https://travis-ci.org/jgpc42/jmh-clojure)

### Dependency information

Leiningen

``` clojure
[jmh-clojure "0.1.3"]
```

Maven

``` xml
<dependency>
  <groupId>jmh-clojure</groupId>
  <artifactId>jmh-clojure</artifactId>
  <version>0.1.3</version>
</dependency>
```

### What is it?

This library provides a data-oriented API to [JMH][jmh], the Java Microbenchmark Harness. It can be used directly or via Leiningen with [lein-jmh][lein-jmh].

JMH is developed by OpenJDK JVM experts and goes to great lengths to ensure accurate benchmarks. Benchmarking on the JVM is a complex beast and, by extension, JMH takes some effort to learn and use properly. That being said, JMH is very robust and configurable. If you are new to JMH, I would recommend browsing the [sample][samples] code and javadocs before using this library.

If you a need simpler, less strenuous tool, I would suggest looking at the popular [criterium][criterium] library.

### Quick start

As a simple example, let's say we want to benchmark our fn that gets the value at an arbitrary indexed type. Of course, the built in `nth` already does this. But we can't extend `nth` to existing types like `java.nio.ByteBuffer`, etc.

```clojure
(ns demo.core)

(defprotocol ValueAt
  (value-at [x idx]))

(extend-protocol ValueAt
  clojure.lang.Indexed
  (value-at [i idx]
    (.nth i idx))
  CharSequence
  (value-at [s idx]
    (.charAt s idx))
  #_...)
```

Benchmarks are described in data and are fully separated from definitions. The reason for this is twofold. First, decoupling is generally good design practice. And second, it allows us to easily take advantage of JMH process isolation (forking) for reliability and accurracy. More on this later.

For repeatability, we'll place the following data in a `benchmarks.edn` resource file in our project. (Note that using a file is not a requirement, we could also specify the same data in Clojure.)

```clojure
{:benchmarks
 [{:name :str, :fn demo.core/value-at, :args [:state/string, :state/index]}
  {:name :vec, :fn demo.core/value-at, :args [:state/vector, :state/index]}]

 :states
 {:index {:fn (partial * 0.5), :args [:param/count]}
  :string {:fn demo.utils/make-str, :args [:param/count]}
  :vector {:fn demo.utils/make-vec, :args [:param/count]}}

 :params {:count 10}}
```

I have omitted showing the `demo.utils` namespace for brevity, it is defined [here][utils] if interested.

The above data should be fairly easy to understand. It is also a limited view of what can be specified. The [sample file][sample] provides a complete reference and explanation.

Now to run the benchmarks. We'll start a REPL in our project and evaluate the following. Note that we could instead use [lein-jmh][lein-jmh] to automate this entire process.

```clojure
(require '[jmh.core :as jmh]
         '[clojure.java.io :as io]
         '[clojure.edn :as edn])

(def bench-env
  (-> "benchmarks.edn" io/resource slurp edn/read-string))

(def bench-opts
  {:type :quick
   :params {:count [31 100000]}
   :profilers ["gc"]})

(jmh/run bench-env bench-opts)
;; => ({:name :str, :params {:count 31},     :score [1.44959801438209E8 "ops/s"], #_...}
;;     {:name :str, :params {:count 100000}, :score [1.45485370497829E8 "ops/s"]}
;;     {:name :vec, :params {:count 31},     :score [1.45550038851249E8 "ops/s"]}
;;     {:name :vec, :params {:count 100000}, :score [8.5783753539823E7 "ops/s"]})
```

The `run` fn takes a benchmark environment and an optional map. We select the `:quick` type: an alias for some common JMH options. We override our default `:count` parameter sequence to measure our fn against small and large inputs. We also enable the gc profiler.

Notice how we have four results: one for each combination of parameter and benchmark fn. For this example, we have omitted lots of additional result map data.

Note that the above results were taken from multiple [runs][result], which is always a good practice when benchmarking.

Benchmarking expressions or fns manually without the data specification is also supported. For example, the `run-expr` macro provides an interface similar to [criterium][criterium]. However, this forgoes JMH process isolation. For more on why benchmarking this way can be sub-optimal, see [here][extended].

### More information

As previously mentioned, please see the [sample file][sample] for the complete benchmark environment reference. For `run` options, see the [docs][run-doc]. Also, see the [wiki][wiki] for additional topics.

### Running the tests

```bash
lein test
```

Or, `lein test-all` for all supported Clojure versions.

### License

Copyright © 2017 Justin Conklin

Distributed under the Eclipse Public License, the same as Clojure.



[criterium]:  https://github.com/hugoduncan/criterium
[extended]:   https://github.com/jgpc42/jmh-clojure/wiki/Extended
[jmh]:        http://openjdk.java.net/projects/code-tools/jmh/
[lein-jmh]:   https://github.com/jgpc42/lein-jmh
[result]:     https://gist.github.com/jgpc42/4d8a828f8d0739748afa71035f2b2c9c#file-results-edn
[run-doc]:    https://jgpc42.github.io/jmh-clojure/doc/jmh.core.html#var-run
[sample]:     https://github.com/jgpc42/jmh-clojure/blob/master/resources/sample.jmh.edn
[samples]:    http://hg.openjdk.java.net/code-tools/jmh/file/1ddf31f810a3/jmh-samples/src/main/java/org/openjdk/jmh/samples/
[utils]:      https://gist.github.com/jgpc42/4d8a828f8d0739748afa71035f2b2c9c#file-utils-clj
[wiki]:       https://github.com/jgpc42/jmh-clojure/wiki