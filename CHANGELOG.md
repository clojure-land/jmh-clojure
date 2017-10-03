## 0.1.3 (2017-10-02)

* BUGFIX: exception when passing global time options (e.g., `:timeout`) to `jmh.core/run`
* Use `:expr` not `"expr"` as the default name for benchmark `:fn` expressions

## 0.1.2 (2017-10-02)

* BUGFIX: [#1](https://github.com/jgpc42/jmh-clojure/issues/1) NPE when no benchmarks are defined/selected

## 0.1.1 (2017-09-27)

* Compile java files with `-target 1.6` for older JVM compatibility