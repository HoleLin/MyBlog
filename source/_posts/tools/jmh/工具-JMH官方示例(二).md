---
title: 工具-JMH官方示例(二)
date: 2022-03-17 15:29:16
index_img: /img/cover/Tools.jpg
cover: /img/cover/Tools.jpg
tags:
- JMH
- Simple
categories:
- 工具
updated:
type:
comments:
description:
keywords:
top_img:
mathjax:
katex:
aside:
aplayer:
highlight_shrink:
---

### 参考文献

* [Simples](http://hg.openjdk.java.net/code-tools/jmh/file/tip/jmh-samples/src/main/java/org/openjdk/jmh/samples/)

### 官方样例(13~end)

#### RunToRun

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.Fork;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.Setup;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.concurrent.TimeUnit;

@State(Scope.Thread)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
public class JMHSample_13_RunToRun {

    /*
     * Forking also allows to estimate run-to-run variance.
     *
     * JVMs are complex systems, and the non-determinism is inherent for them.
     * This requires us to always account the run-to-run variance as the one
     * of the effects in our experiments.
     *
     * Luckily, forking aggregates the results across several JVM launches.
     */

    /*
     * In order to introduce readily measurable run-to-run variance, we build
     * the workload which performance differs from run to run. Note that many workloads
     * will have the similar behavior, but we do that artificially to make a point.
     */

    @State(Scope.Thread)
    public static class SleepyState {
        public long sleepTime;

        @Setup
        public void setup() {
            sleepTime = (long) (Math.random() * 1000);
        }
    }

    /*
     * Now, we will run this different number of times.
     */

    @Benchmark
    @Fork(1)
    public void baseline(SleepyState s) throws InterruptedException {
        TimeUnit.MILLISECONDS.sleep(s.sleepTime);
    }

    @Benchmark
    @Fork(5)
    public void fork_1(SleepyState s) throws InterruptedException {
        TimeUnit.MILLISECONDS.sleep(s.sleepTime);
    }

    @Benchmark
    @Fork(20)
    public void fork_2(SleepyState s) throws InterruptedException {
        TimeUnit.MILLISECONDS.sleep(s.sleepTime);
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * Note the baseline is random within [0..1000] msec; and both forked runs
     * are estimating the average 500 msec with some confidence.
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_13 -wi 0 -i 3
     *    (we requested no warmup, 3 measurement iterations; there are also other options, see -h)
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_13_RunToRun.class.getSimpleName())
                .warmupIterations(0)
                .measurementIterations(3)
                .build();

        new Runner(opt).run();
    }

}
```

#### Asymmetric

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.Group;
import org.openjdk.jmh.annotations.GroupThreads;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.Setup;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

@State(Scope.Group)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public class JMHSample_15_Asymmetric {

    /*
     * So far all the tests were symmetric: the same code was executed in all the threads.
     * At times, you need the asymmetric test. JMH provides this with the notion of @Group,
     * which can bind several methods together, and all the threads are distributed among
     * the test methods.
     *
     * Each execution group contains of one or more threads. Each thread within a particular
     * execution group executes one of @Group-annotated @Benchmark methods. Multiple execution
     * groups may participate in the run. The total thread count in the run is rounded to the
     * execution group size, which will only allow the full execution groups.
     *
     * Note that two state scopes: Scope.Benchmark and Scope.Thread are not covering all
     * the use cases here -- you either share everything in the state, or share nothing.
     * To break this, we have the middle ground Scope.Group, which marks the state to be
     * shared within the execution group, but not among the execution groups.
     *
     * Putting this all together, the example below means:
     *  a) define the execution group "g", with 3 threads executing inc(), and 1 thread
     *     executing get(), 4 threads per group in total;
     *  b) if we run this test case with 4 threads, then we will have a single execution
     *     group. Generally, running with 4*N threads will create N execution groups, etc.;
     *  c) each execution group has one @State instance to share: that is, execution groups
     *     share the counter within the group, but not across the groups.
     */

    private AtomicInteger counter;

    @Setup
    public void up() {
        counter = new AtomicInteger();
    }

    @Benchmark
    @Group("g")
    @GroupThreads(3)
    public int inc() {
        return counter.incrementAndGet();
    }

    @Benchmark
    @Group("g")
    @GroupThreads(1)
    public int get() {
        return counter.get();
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * You will have the distinct metrics for inc() and get() from this run.
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_15 -f 1
     *    (we requested single fork; there are also other options, see -h)
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_15_Asymmetric.class.getSimpleName())
                .forks(1)
                .build();

        new Runner(opt).run();
    }

}
```

#### CompilerControl

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.CompilerControl;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.concurrent.TimeUnit;

@State(Scope.Thread)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public class JMHSample_16_CompilerControl {

    /*
     * We can use HotSpot-specific functionality to tell the compiler what
     * do we want to do with particular methods. To demonstrate the effects,
     * we end up with 3 methods in this sample.
     */

    /**
     * These are our targets:
     *   - first method is prohibited from inlining
     *   - second method is forced to inline
     *   - third method is prohibited from compiling
     *
     * We might even place the annotations directly to the benchmarked
     * methods, but this expresses the intent more clearly.
     */

    public void target_blank() {
        // this method was intentionally left blank
    }

    @CompilerControl(CompilerControl.Mode.DONT_INLINE)
    public void target_dontInline() {
        // this method was intentionally left blank
    }

    @CompilerControl(CompilerControl.Mode.INLINE)
    public void target_inline() {
        // this method was intentionally left blank
    }

    @CompilerControl(CompilerControl.Mode.EXCLUDE)
    public void target_exclude() {
        // this method was intentionally left blank
    }

    /*
     * These method measures the calls performance.
     */

    @Benchmark
    public void baseline() {
        // this method was intentionally left blank
    }

    @Benchmark
    public void blank() {
        target_blank();
    }

    @Benchmark
    public void dontinline() {
        target_dontInline();
    }

    @Benchmark
    public void inline() {
        target_inline();
    }

    @Benchmark
    public void exclude() {
        target_exclude();
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * Note the performance of the baseline, blank, and inline methods are the same.
     * dontinline differs a bit, because we are making the proper call.
     * exclude is severely slower, becase we are not compiling it at all.
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_16 -wi 0 -i 3 -f 1
     *    (we requested no warmup iterations, 3 iterations, single fork; there are also other options, see -h)
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_16_CompilerControl.class.getSimpleName())
                .warmupIterations(0)
                .measurementIterations(3)
                .forks(1)
                .build();

        new Runner(opt).run();
    }

}
```

#### SyncIterations

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;
import org.openjdk.jmh.runner.options.TimeValue;

import java.util.concurrent.TimeUnit;

@State(Scope.Thread)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
public class JMHSample_17_SyncIterations {

    /*
     * This is the another thing that is enabled in JMH by default.
     *
     * Suppose we have this simple benchmark.
     */

    private double src;

    @Benchmark
    public double test() {
        double s = src;
        for (int i = 0; i < 1000; i++) {
            s = Math.sin(s);
        }
        return s;
    }

    /*
     * It turns out if you run the benchmark with multiple threads,
     * the way you start and stop the worker threads seriously affects
     * performance.
     *
     * The natural way would be to park all the threads on some sort
     * of barrier, and the let them go "at once". However, that does
     * not work: there are no guarantees the worker threads will start
     * at the same time, meaning other worker threads are working
     * in better conditions, skewing the result.
     *
     * The better solution would be to introduce bogus iterations,
     * ramp up the threads executing the iterations, and then atomically
     * shift the system to measuring stuff. The same thing can be done
     * during the rampdown. This sounds complicated, but JMH already
     * handles that for you.
     *

     */

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * You will need to oversubscribe the system to make this effect
     * clearly visible; however, this effect can also be shown on the
     * unsaturated systems.*
     *
     * Note the performance of -si false version is more flaky, even
     * though it is "better". This is the false improvement, granted by
     * some of the threads executing in solo. The -si true version more stable
     * and coherent.
     *
     * -si true is enabled by default.
     *
     * Say, $CPU is the number of CPUs on your machine.
     *
     * You can run this test with:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_17 \
     *        -w 1s -r 1s -f 1 -t ${CPU*16} -si {true|false}
     *    (we requested shorter warmup/measurement iterations, single fork,
     *     lots of threads, and changeable "synchronize iterations" option)
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_17_SyncIterations.class.getSimpleName())
                .warmupTime(TimeValue.seconds(1))
                .measurementTime(TimeValue.seconds(1))
                .threads(Runtime.getRuntime().availableProcessors()*16)
                .forks(1)
                .syncIterations(true) // try to switch to "false"
                .build();

        new Runner(opt).run();
    }

}
```

#### Control

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.Group;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.infra.Control;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.concurrent.atomic.AtomicBoolean;

@State(Scope.Group)
public class JMHSample_18_Control {

    /*
     * Sometimes you need the tap into the harness mind to get the info
     * on the transition change. For this, we have the experimental state object,
     * Control, which is updated by JMH as we go.
     */

    /*
     * In this example, we want to estimate the ping-pong speed for the simple
     * AtomicBoolean. Unfortunately, doing that in naive manner will livelock
     * one of the threads, because the executions of ping/pong are not paired
     * perfectly. We need the escape hatch to terminate the loop if threads
     * are about to leave the measurement.
     */

    public final AtomicBoolean flag = new AtomicBoolean();

    @Benchmark
    @Group("pingpong")
    public void ping(Control cnt) {
        while (!cnt.stopMeasurement && !flag.compareAndSet(false, true)) {
            // this body is intentionally left blank
        }
    }

    @Benchmark
    @Group("pingpong")
    public void pong(Control cnt) {
        while (!cnt.stopMeasurement && !flag.compareAndSet(true, false)) {
            // this body is intentionally left blank
        }
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_18 -t 2 -f 1
     *    (we requested 2 threads and single fork; there are also other options, see -h)
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_18_Control.class.getSimpleName())
                .threads(2)
                .forks(1)
                .build();

        new Runner(opt).run();
    }

}
```

#### Annotations

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.Fork;
import org.openjdk.jmh.annotations.Measurement;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.annotations.Warmup;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.concurrent.TimeUnit;

@State(Scope.Thread)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@Fork(1)
public class JMHSample_20_Annotations {

    double x1 = Math.PI;

    /*
     * In addition to all the command line options usable at run time,
     * we have the annotations which can provide the reasonable defaults
     * for the some of the benchmarks. This is very useful when you are
     * dealing with lots of benchmarks, and some of them require
     * special treatment.
     *
     * Annotation can also be placed on class, to have the effect over
     * all the benchmark methods in the same class. The rule is, the
     * annotation in the closest scope takes the precedence: i.e.
     * the method-based annotation overrides class-based annotation,
     * etc.
     */

    @Benchmark
    @Warmup(iterations = 5, time = 100, timeUnit = TimeUnit.MILLISECONDS)
    @Measurement(iterations = 5, time = 100, timeUnit = TimeUnit.MILLISECONDS)
    public double measure() {
        return Math.log(x1);
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * Note JMH honors the default annotation settings. You can always override
     * the defaults via the command line or API.
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_20
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_20_Annotations.class.getSimpleName())
                .build();

        new Runner(opt).run();
    }

}
```

#### ConsumeCPU

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.infra.Blackhole;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public class JMHSample_21_ConsumeCPU {

    /*
     * At times you require the test to burn some of the cycles doing nothing.
     * In many cases, you *do* want to burn the cycles instead of waiting.
     *
     * For these occasions, we have the infrastructure support. Blackholes
     * can not only consume the values, but also the time! Run this test
     * to get familiar with this part of JMH.
     *
     * (Note we use static method because most of the use cases are deep
     * within the testing code, and propagating blackholes is tedious).
     */

    @Benchmark
    public void consume_0000() {
        Blackhole.consumeCPU(0);
    }

    @Benchmark
    public void consume_0001() {
        Blackhole.consumeCPU(1);
    }

    @Benchmark
    public void consume_0002() {
        Blackhole.consumeCPU(2);
    }

    @Benchmark
    public void consume_0004() {
        Blackhole.consumeCPU(4);
    }

    @Benchmark
    public void consume_0008() {
        Blackhole.consumeCPU(8);
    }

    @Benchmark
    public void consume_0016() {
        Blackhole.consumeCPU(16);
    }

    @Benchmark
    public void consume_0032() {
        Blackhole.consumeCPU(32);
    }

    @Benchmark
    public void consume_0064() {
        Blackhole.consumeCPU(64);
    }

    @Benchmark
    public void consume_0128() {
        Blackhole.consumeCPU(128);
    }

    @Benchmark
    public void consume_0256() {
        Blackhole.consumeCPU(256);
    }

    @Benchmark
    public void consume_0512() {
        Blackhole.consumeCPU(512);
    }

    @Benchmark
    public void consume_1024() {
        Blackhole.consumeCPU(1024);
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * Note the single token is just a few cycles, and the more tokens
     * you request, then more work is spent (almost linearly)
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_21 -f 1
     *    (we requested single fork; there are also other options, see -h)
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_21_ConsumeCPU.class.getSimpleName())
                .forks(1)
                .build();

        new Runner(opt).run();
    }

}
```

#### FalseSharing

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.Fork;
import org.openjdk.jmh.annotations.Group;
import org.openjdk.jmh.annotations.Measurement;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.annotations.Warmup;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.Throughput)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Fork(5)
public class JMHSample_22_FalseSharing {

    /*
     * One of the unusual thing that can bite you back is false sharing.
     * If two threads access (and possibly modify) the adjacent values
     * in memory, chances are, they are modifying the values on the same
     * cache line. This can yield significant (artificial) slowdowns.
     *
     * JMH helps you to alleviate this: @States are automatically padded.
     * This padding does not extend to the State internals though,
     * as we will see in this example. You have to take care of this on
     * your own.
     */

    /*
     * Suppose we have two threads:
     *   a) innocuous reader which blindly reads its own field
     *   b) furious writer which updates its own field
     */

    /*
     * BASELINE EXPERIMENT:
     * Because of the false sharing, both reader and writer will experience
     * penalties.
     */

    @State(Scope.Group)
    public static class StateBaseline {
        int readOnly;
        int writeOnly;
    }

    @Benchmark
    @Group("baseline")
    public int reader(StateBaseline s) {
        return s.readOnly;
    }

    @Benchmark
    @Group("baseline")
    public void writer(StateBaseline s) {
        s.writeOnly++;
    }

    /*
     * APPROACH 1: PADDING
     *
     * We can try to alleviate some of the effects with padding.
     * This is not versatile because JVMs can freely rearrange the
     * field order, even of the same type.
     */

    @State(Scope.Group)
    public static class StatePadded {
        int readOnly;
        int p01, p02, p03, p04, p05, p06, p07, p08;
        int p11, p12, p13, p14, p15, p16, p17, p18;
        int writeOnly;
        int q01, q02, q03, q04, q05, q06, q07, q08;
        int q11, q12, q13, q14, q15, q16, q17, q18;
    }

    @Benchmark
    @Group("padded")
    public int reader(StatePadded s) {
        return s.readOnly;
    }

    @Benchmark
    @Group("padded")
    public void writer(StatePadded s) {
        s.writeOnly++;
    }

    /*
     * APPROACH 2: CLASS HIERARCHY TRICK
     *
     * We can alleviate false sharing with this convoluted hierarchy trick,
     * using the fact that superclass fields are usually laid out first.
     * In this construction, the protected field will be squashed between
     * paddings.

     * It is important to use the smallest data type, so that layouter would
     * not generate any gaps that can be taken by later protected subclasses
     * fields. Depending on the actual field layout of classes that bear the
     * protected fields, we might need more padding to account for "lost"
     * padding fields pulled into in their superclass gaps.
     */

    public static class StateHierarchy_1 {
        int readOnly;
    }

    public static class StateHierarchy_2 extends StateHierarchy_1 {
        byte p01, p02, p03, p04, p05, p06, p07, p08;
        byte p11, p12, p13, p14, p15, p16, p17, p18;
        byte p21, p22, p23, p24, p25, p26, p27, p28;
        byte p31, p32, p33, p34, p35, p36, p37, p38;
        byte p41, p42, p43, p44, p45, p46, p47, p48;
        byte p51, p52, p53, p54, p55, p56, p57, p58;
        byte p61, p62, p63, p64, p65, p66, p67, p68;
        byte p71, p72, p73, p74, p75, p76, p77, p78;
    }

    public static class StateHierarchy_3 extends StateHierarchy_2 {
        int writeOnly;
    }

    public static class StateHierarchy_4 extends StateHierarchy_3 {
        byte q01, q02, q03, q04, q05, q06, q07, q08;
        byte q11, q12, q13, q14, q15, q16, q17, q18;
        byte q21, q22, q23, q24, q25, q26, q27, q28;
        byte q31, q32, q33, q34, q35, q36, q37, q38;
        byte q41, q42, q43, q44, q45, q46, q47, q48;
        byte q51, q52, q53, q54, q55, q56, q57, q58;
        byte q61, q62, q63, q64, q65, q66, q67, q68;
        byte q71, q72, q73, q74, q75, q76, q77, q78;
    }

    @State(Scope.Group)
    public static class StateHierarchy extends StateHierarchy_4 {
    }

    @Benchmark
    @Group("hierarchy")
    public int reader(StateHierarchy s) {
        return s.readOnly;
    }

    @Benchmark
    @Group("hierarchy")
    public void writer(StateHierarchy s) {
        s.writeOnly++;
    }

    /*
     * APPROACH 3: ARRAY TRICK
     *
     * This trick relies on the contiguous allocation of an array.
     * Instead of placing the fields in the class, we mangle them
     * into the array at very sparse offsets.
     */

    @State(Scope.Group)
    public static class StateArray {
        int[] arr = new int[128];
    }

    @Benchmark
    @Group("sparse")
    public int reader(StateArray s) {
        return s.arr[0];
    }

    @Benchmark
    @Group("sparse")
    public void writer(StateArray s) {
        s.arr[64]++;
    }

    /*
     * APPROACH 4:
     *
     * @Contended (since JDK 8):
     *  Uncomment the annotation if building with JDK 8.
     *  Remember to flip -XX:-RestrictContended to enable.
     */

    @State(Scope.Group)
    public static class StateContended {
        int readOnly;

//        @sun.misc.Contended
        int writeOnly;
    }

    @Benchmark
    @Group("contended")
    public int reader(StateContended s) {
        return s.readOnly;
    }

    @Benchmark
    @Group("contended")
    public void writer(StateContended s) {
        s.writeOnly++;
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * Note the slowdowns.
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_22 -t $CPU
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_22_FalseSharing.class.getSimpleName())
                .threads(Runtime.getRuntime().availableProcessors())
                .build();

        new Runner(opt).run();
    }

}
```

#### AuxCounters

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.AuxCounters;
import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.Fork;
import org.openjdk.jmh.annotations.Measurement;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.annotations.Warmup;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.concurrent.TimeUnit;

@OutputTimeUnit(TimeUnit.SECONDS)
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Fork(1)
public class JMHSample_23_AuxCounters {

    /*
     * In some weird cases you need to get the separate throughput/time
     * metrics for the benchmarked code depending on the outcome of the
     * current code. Trying to accommodate the cases like this, JMH optionally
     * provides the special annotation which treats @State objects
     * as the object bearing user counters. See @AuxCounters javadoc for
     * the limitations.
     */

    @State(Scope.Thread)
    @AuxCounters(AuxCounters.Type.OPERATIONS)
    public static class OpCounters {
        // These fields would be counted as metrics
        public int case1;
        public int case2;

        // This accessor will also produce a metric
        public int total() {
            return case1 + case2;
        }
    }

    @State(Scope.Thread)
    @AuxCounters(AuxCounters.Type.EVENTS)
    public static class EventCounters {
        // This field would be counted as metric
        public int wows;
    }

    /*
     * This code measures the "throughput" in two parts of the branch.
     * The @AuxCounters state above holds the counters which we increment
     * ourselves, and then let JMH to use their values in the performance
     * calculations.
     */

    @Benchmark
    public void splitBranch(OpCounters counters) {
        if (Math.random() < 0.1) {
            counters.case1++;
        } else {
            counters.case2++;
        }
    }

    @Benchmark
    public void runSETI(EventCounters counters) {
        float random = (float) Math.random();
        float wowSignal = (float) Math.PI / 4;
        if (random == wowSignal) {
            // WOW, that's unusual.
            counters.wows++;
        }
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_23
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_23_AuxCounters.class.getSimpleName())
                .build();

        new Runner(opt).run();
    }

}
```

#### Inheritance

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.Fork;
import org.openjdk.jmh.annotations.Measurement;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.Setup;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.annotations.Warmup;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.concurrent.TimeUnit;

public class JMHSample_24_Inheritance {

    /*
     * In very special circumstances, you might want to provide the benchmark
     * body in the (abstract) superclass, and specialize it with the concrete
     * pieces in the subclasses.
     *
     * The rule of thumb is: if some class has @Benchmark method, then all the subclasses
     * are also having the "synthetic" @Benchmark method. The caveat is, because we only
     * know the type hierarchy during the compilation, it is only possible during
     * the same compilation session. That is, mixing in the subclass extending your
     * benchmark class *after* the JMH compilation would have no effect.
     *
     * Note how annotations now have two possible places. The closest annotation
     * in the hierarchy wins.
     */

    @BenchmarkMode(Mode.AverageTime)
    @Fork(1)
    @State(Scope.Thread)
    @OutputTimeUnit(TimeUnit.NANOSECONDS)
    public static abstract class AbstractBenchmark {
        int x;

        @Setup
        public void setup() {
            x = 42;
        }

        @Benchmark
        @Warmup(iterations = 5, time = 100, timeUnit = TimeUnit.MILLISECONDS)
        @Measurement(iterations = 5, time = 100, timeUnit = TimeUnit.MILLISECONDS)
        public double bench() {
            return doWork() * doWork();
        }

        protected abstract double doWork();
    }

    public static class BenchmarkLog extends AbstractBenchmark {
        @Override
        protected double doWork() {
            return Math.log(x);
        }
    }

    public static class BenchmarkSin extends AbstractBenchmark {
        @Override
        protected double doWork() {
            return Math.sin(x);
        }
    }

    public static class BenchmarkCos extends AbstractBenchmark {
        @Override
        protected double doWork() {
            return Math.cos(x);
        }
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * You can run this test, and observe the three distinct benchmarks running the squares
     * of Math.log, Math.sin, and Math.cos, accordingly.
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_24
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_24_Inheritance.class.getSimpleName())
                .build();

        new Runner(opt).run();
    }

}
```

#### API_GA

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.results.Result;
import org.openjdk.jmh.results.RunResult;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;
import org.openjdk.jmh.runner.options.TimeValue;
import org.openjdk.jmh.runner.options.VerboseMode;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

@State(Scope.Thread)
public class JMHSample_25_API_GA {

    /**
     * This example shows the rather convoluted, but fun way to exploit
     * JMH API in complex scenarios. Up to this point, we haven't consumed
     * the results programmatically, and hence we are missing all the fun.
     *
     * Let's consider this naive code, which obviously suffers from the
     * performance anomalies, since current HotSpot is resistant to make
     * the tail-call optimizations.
     */

    private int v;

    @Benchmark
    public int test() {
        return veryImportantCode(1000, v);
    }

    public int veryImportantCode(int d, int v) {
        if (d == 0) {
            return v;
        } else {
            return veryImportantCode(d - 1, v);
        }
    }

    /*
     * We could probably make up for the absence of TCO with better inlining
     * policy. But hand-tuning the policy requires knowing a lot about VM
     * internals. Let's instead construct the layman's genetic algorithm
     * which sifts through inlining settings trying to find the better policy.
     *
     * If you are not familiar with the concept of Genetic Algorithms,
     * read the Wikipedia article first:
     *    http://en.wikipedia.org/wiki/Genetic_algorithm
     *
     * VM experts can guess which option should be tuned to get the max
     * performance. Try to run the sample and see if it improves performance.
     */

    public static void main(String[] args) throws RunnerException {
        // These are our base options. We will mix these options into the
        // measurement runs. That is, all measurement runs will inherit these,
        // see how it's done below.
        Options baseOpts = new OptionsBuilder()
                .include(JMHSample_25_API_GA.class.getName())
                .warmupTime(TimeValue.milliseconds(200))
                .measurementTime(TimeValue.milliseconds(200))
                .warmupIterations(5)
                .measurementIterations(5)
                .forks(1)
                .verbosity(VerboseMode.SILENT)
                .build();

        // Initial population
        Population pop = new Population();
        final int POPULATION = 10;
        for (int c = 0; c < POPULATION; c++) {
            pop.addChromosome(new Chromosome(baseOpts));
        }

        // Make a few rounds of optimization:
        final int GENERATIONS = 100;
        for (int g = 0; g < GENERATIONS; g++) {
            System.out.println("Entering generation " + g);

            // Get the baseline score.
            // We opt to remeasure it in order to get reliable current estimate.
            RunResult runner = new Runner(baseOpts).runSingle();
            Result baseResult = runner.getPrimaryResult();

            // Printing a nice table...
            System.out.println("---------------------------------------");
            System.out.printf("Baseline score: %10.2f %s%n",
                    baseResult.getScore(),
                    baseResult.getScoreUnit()
            );

            for (Chromosome c : pop.getAll()) {
                System.out.printf("%10.2f %s (%+10.2f%%) %s%n",
                        c.getScore(),
                        baseResult.getScoreUnit(),
                        (c.getScore() / baseResult.getScore() - 1) * 100,
                        c.toString()
                );
            }
            System.out.println();

            Population newPop = new Population();

            // Copy out elite solutions
            final int ELITE = 2;
            for (Chromosome c : pop.getAll().subList(0, ELITE)) {
                newPop.addChromosome(c);
            }

            // Cross-breed the rest of new population
            while (newPop.size() < pop.size()) {
                Chromosome p1 = pop.selectToBreed();
                Chromosome p2 = pop.selectToBreed();

                newPop.addChromosome(p1.crossover(p2).mutate());
                newPop.addChromosome(p2.crossover(p1).mutate());
            }

            pop = newPop;
        }

    }

    /**
     * Population.
     */
    public static class Population {
        private final List<Chromosome> list = new ArrayList<>();

        public void addChromosome(Chromosome c) {
            list.add(c);
            Collections.sort(list);
        }

        /**
         * Select the breeding material.
         * Solutions with better score have better chance to be selected.
         * @return breed
         */
        public Chromosome selectToBreed() {
            double totalScore = 0D;
            for (Chromosome c : list) {
                totalScore += c.score();
            }

            double thresh = Math.random() * totalScore;
            for (Chromosome c : list) {
                if (thresh < 0) return c;
                thresh =- c.score();
            }

            throw new IllegalStateException("Can not choose");
        }

        public int size() {
            return list.size();
        }

        public List<Chromosome> getAll() {
            return list;
        }
    }

    /**
     * Chromosome: encodes solution.
     */
    public static class Chromosome implements Comparable<Chromosome> {

        // Current score is not yet computed.
        double score = Double.NEGATIVE_INFINITY;

        // Base options to mix in
        final Options baseOpts;

        // These are current HotSpot defaults.
        int freqInlineSize = 325;
        int inlineSmallCode = 1000;
        int maxInlineLevel = 9;
        int maxInlineSize = 35;
        int maxRecursiveInlineLevel = 1;
        int minInliningThreshold = 250;

        public Chromosome(Options baseOpts) {
            this.baseOpts = baseOpts;
        }

        public double score() {
            if (score != Double.NEGATIVE_INFINITY) {
                // Already got the score, shortcutting
                return score;
            }

            try {
                // Add the options encoded by this solution:
                //  a) Mix in base options.
                //  b) Add JVM arguments: we opt to parse the
                //     stringly representation to make the example
                //     shorter. There are, of course, cleaner ways
                //     to do this.
                Options theseOpts = new OptionsBuilder()
                        .parent(baseOpts)
                        .jvmArgs(toString().split("[ ]"))
                        .build();

                // Run through JMH and get the result back.
                RunResult runResult = new Runner(theseOpts).runSingle();
                score = runResult.getPrimaryResult().getScore();
            } catch (RunnerException e) {
                // Something went wrong, the solution is defective
                score = Double.MIN_VALUE;
            }

            return score;
        }

        @Override
        public int compareTo(Chromosome o) {
            // Order by score, descending.
            return -Double.compare(score(), o.score());
        }

        @Override
        public String toString() {
            return "-XX:FreqInlineSize=" + freqInlineSize +
                    " -XX:InlineSmallCode=" + inlineSmallCode +
                    " -XX:MaxInlineLevel=" + maxInlineLevel +
                    " -XX:MaxInlineSize=" + maxInlineSize +
                    " -XX:MaxRecursiveInlineLevel=" + maxRecursiveInlineLevel +
                    " -XX:MinInliningThreshold=" + minInliningThreshold;
        }

        public Chromosome crossover(Chromosome other) {
            // Perform crossover:
            // While this is a very naive way to perform crossover, it still works.

            final double CROSSOVER_PROB = 0.1;

            Chromosome result = new Chromosome(baseOpts);

            result.freqInlineSize = (Math.random() < CROSSOVER_PROB) ?
                    this.freqInlineSize : other.freqInlineSize;

            result.inlineSmallCode = (Math.random() < CROSSOVER_PROB) ?
                    this.inlineSmallCode : other.inlineSmallCode;

            result.maxInlineLevel = (Math.random() < CROSSOVER_PROB) ?
                    this.maxInlineLevel : other.maxInlineLevel;

            result.maxInlineSize = (Math.random() < CROSSOVER_PROB) ?
                    this.maxInlineSize : other.maxInlineSize;

            result.maxRecursiveInlineLevel = (Math.random() < CROSSOVER_PROB) ?
                    this.maxRecursiveInlineLevel : other.maxRecursiveInlineLevel;

            result.minInliningThreshold = (Math.random() < CROSSOVER_PROB) ?
                    this.minInliningThreshold : other.minInliningThreshold;

            return result;
        }

        public Chromosome mutate() {
            // Perform mutation:
            //  Again, this is a naive way to do mutation, but it still works.

            Chromosome result = new Chromosome(baseOpts);

            result.freqInlineSize = (int) randomChange(freqInlineSize);
            result.inlineSmallCode = (int) randomChange(inlineSmallCode);
            result.maxInlineLevel = (int) randomChange(maxInlineLevel);
            result.maxInlineSize = (int) randomChange(maxInlineSize);
            result.maxRecursiveInlineLevel = (int) randomChange(maxRecursiveInlineLevel);
            result.minInliningThreshold = (int) randomChange(minInliningThreshold);

            return result;
        }

        private double randomChange(double v) {
            final double MUTATE_PROB = 0.5;
            if (Math.random() < MUTATE_PROB) {
                if (Math.random() < 0.5) {
                    return v / (Math.random() * 2);
                } else {
                    return v * (Math.random() * 2);
                }
            } else {
                return v;
            }
        }

        public double getScore() {
            return score;
        }
    }

}
```

#### BatchSize

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.Level;
import org.openjdk.jmh.annotations.Measurement;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.Setup;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.annotations.Warmup;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.LinkedList;
import java.util.List;

@State(Scope.Thread)
public class JMHSample_26_BatchSize {

    /*
     * Sometimes you need to evaluate operation which doesn't have
     * the steady state. The cost of a benchmarked operation may
     * significantly vary from invocation to invocation.
     *
     * In this case, using the timed measurements is not a good idea,
     * and the only acceptable benchmark mode is a single shot. On the
     * other hand, the operation may be too small for reliable single
     * shot measurement.
     *
     * We can use "batch size" parameter to describe the number of
     * @Benchmark invocations to do per one "shot" without looping the method
     * manually and protect from problems described in JMHSample_11_Loops.
     * If there are any @Setup/@TearDown(Level.Invocation), they still run
     * per each @Benchmark invocation.
     */

    /*
     * Suppose we want to measure insertion in the middle of the list.
     */

    List<String> list = new LinkedList<>();

    @Benchmark
    @Warmup(iterations = 5, time = 1)
    @Measurement(iterations = 5, time = 1)
    @BenchmarkMode(Mode.AverageTime)
    public List<String> measureWrong_1() {
        list.add(list.size() / 2, "something");
        return list;
    }

    @Benchmark
    @Warmup(iterations = 5, time = 5)
    @Measurement(iterations = 5, time = 5)
    @BenchmarkMode(Mode.AverageTime)
    public List<String> measureWrong_5() {
        list.add(list.size() / 2, "something");
        return list;
    }

    /*
     * This is what you do with JMH.
     */
    @Benchmark
    @Warmup(iterations = 5, batchSize = 5000)
    @Measurement(iterations = 5, batchSize = 5000)
    @BenchmarkMode(Mode.SingleShotTime)
    public List<String> measureRight() {
        list.add(list.size() / 2, "something");
        return list;
    }

    @Setup(Level.Iteration)
    public void setup(){
        list.clear();
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * You can see completely different results for measureWrong_1 and measureWrong_5; this
     * is because the workload has no steady state. The result of the workload is dependent
     * on the measurement time. measureRight does not have this drawback, because it measures
     * the N invocations of the test method and measures it's time.
     *
     * We measure batch of 5000 invocations and consider the batch as the single operation.
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_26 -f 1
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_26_BatchSize.class.getSimpleName())
                .forks(1)
                .build();

        new Runner(opt).run();
    }

}
```

#### Params

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.Fork;
import org.openjdk.jmh.annotations.Measurement;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Param;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.annotations.Warmup;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.math.BigInteger;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Fork(1)
@State(Scope.Benchmark)
public class JMHSample_27_Params {

    /**
     * In many cases, the experiments require walking the configuration space
     * for a benchmark. This is needed for additional control, or investigating
     * how the workload performance changes with different settings.
     */

    @Param({"1", "31", "65", "101", "103"})
    public int arg;

    @Param({"0", "1", "2", "4", "8", "16", "32"})
    public int certainty;

    @Benchmark
    public boolean bench() {
        return BigInteger.valueOf(arg).isProbablePrime(certainty);
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * Note the performance is different with different parameters.
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_27
     *
     *    You can juggle parameters through the command line, e.g. with "-p arg=41,42"
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_27_Params.class.getSimpleName())
//                .param("arg", "41", "42") // Use this to selectively constrain/override parameters
                .build();

        new Runner(opt).run();
    }

}
```

#### BlackholeHelpers

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.Fork;
import org.openjdk.jmh.annotations.Measurement;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.Setup;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.annotations.Warmup;
import org.openjdk.jmh.infra.Blackhole;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Fork(1)
@State(Scope.Thread)
public class JMHSample_28_BlackholeHelpers {

    /**
     * Sometimes you need the black hole not in @Benchmark method, but in
     * helper methods, because you want to pass it through to the concrete
     * implementation which is instantiated in helper methods. In this case,
     * you can request the black hole straight in the helper method signature.
     * This applies to both @Setup and @TearDown methods, and also to other
     * JMH infrastructure objects, like Control.
     *
     * Below is the variant of {@link org.openjdk.jmh.samples.JMHSample_08_DeadCode}
     * test, but wrapped in the anonymous classes.
     */

    public interface Worker {
        void work();
    }

    private Worker workerBaseline;
    private Worker workerRight;
    private Worker workerWrong;

    @Setup
    public void setup(final Blackhole bh) {
        workerBaseline = new Worker() {
            double x;

            @Override
            public void work() {
                // do nothing
            }
        };

        workerWrong = new Worker() {
            double x;

            @Override
            public void work() {
                Math.log(x);
            }
        };

        workerRight = new Worker() {
            double x;

            @Override
            public void work() {
                bh.consume(Math.log(x));
            }
        };

    }

    @Benchmark
    public void baseline() {
        workerBaseline.work();
    }

    @Benchmark
    public void measureWrong() {
        workerWrong.work();
    }

    @Benchmark
    public void measureRight() {
        workerRight.work();
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * You will see measureWrong() running on-par with baseline().
     * Both measureRight() are measuring twice the baseline, so the logs are intact.
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_28
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_28_BlackholeHelpers.class.getSimpleName())
                .build();

        new Runner(opt).run();
    }

}
```

#### StatesDAG

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.Fork;
import org.openjdk.jmh.annotations.Measurement;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.Setup;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.annotations.TearDown;
import org.openjdk.jmh.annotations.Warmup;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;
import java.util.Queue;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Fork(1)
@State(Scope.Thread)
public class JMHSample_29_StatesDAG {

    /**
     * WARNING:
     * THIS IS AN EXPERIMENTAL FEATURE, BE READY FOR IT BECOME REMOVED WITHOUT NOTICE!
     */

    /**
     * There are weird cases when the benchmark state is more cleanly described
     * by the set of @States, and those @States reference each other. JMH allows
     * linking @States in directed acyclic graphs (DAGs) by referencing @States
     * in helper method signatures. (Note that {@link org.openjdk.jmh.samples.JMHSample_28_BlackholeHelpers}
     * is just a special case of that.
     *
     * Following the interface for @Benchmark calls, all @Setups for
     * referenced @State-s are fired before it becomes accessible to current @State.
     * Similarly, no @TearDown methods are fired for referenced @State before
     * current @State is done with it.
     */

    /*
     * This is a model case, and it might not be a good benchmark.
     * // TODO: Replace it with the benchmark which does something useful.
     */

    public static class Counter {
        int x;

        public int inc() {
            return x++;
        }

        public void dispose() {
            // pretend this is something really useful
        }
    }

    /*
     * Shared state maintains the set of Counters, and worker threads should
     * poll their own instances of Counter to work with. However, it should only
     * be done once, and therefore, Local state caches it after requesting the
     * counter from Shared state.
     */

    @State(Scope.Benchmark)
    public static class Shared {
        List<Counter> all;
        Queue<Counter> available;

        @Setup
        public synchronized void setup() {
            all = new ArrayList<>();
            for (int c = 0; c < 10; c++) {
                all.add(new Counter());
            }

            available = new LinkedList<>();
            available.addAll(all);
        }

        @TearDown
        public synchronized void tearDown() {
            for (Counter c : all) {
                c.dispose();
            }
        }

        public synchronized Counter getMine() {
            return available.poll();
        }
    }

    @State(Scope.Thread)
    public static class Local {
        Counter cnt;

        @Setup
        public void setup(Shared shared) {
            cnt = shared.getMine();
        }
    }

    @Benchmark
    public int test(Local local) {
        return local.cnt.inc();
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_29
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_29_StatesDAG.class.getSimpleName())
                .build();

        new Runner(opt).run();
    }
}
```

#### Interrupts

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.Group;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.Setup;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;
import org.openjdk.jmh.runner.options.TimeValue;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@State(Scope.Group)
public class JMHSample_30_Interrupts {

    /*
     * JMH can also detect when threads are stuck in the benchmarks, and try
     * to forcefully interrupt the benchmark thread. JMH tries to do that
     * when it is arguably sure it would not affect the measurement.
     */

    /*
     * In this example, we want to measure the simple performance characteristics
     * of the ArrayBlockingQueue. Unfortunately, doing that without a harness
     * support will deadlock one of the threads, because the executions of
     * take/put are not paired perfectly. Fortunately for us, both methods react
     * to interrupts well, and therefore we can rely on JMH to terminate the
     * measurement for us. JMH will notify users about the interrupt actions
     * nevertheless, so users can see if those interrupts affected the measurement.
     * JMH will start issuing interrupts after the default or user-specified timeout
     * had been reached.
     *
     * This is a variant of org.openjdk.jmh.samples.JMHSample_18_Control, but without
     * the explicit control objects. This example is suitable for the methods which
     * react to interrupts gracefully.
     */

    private BlockingQueue<Integer> q;

    @Setup
    public void setup() {
        q = new ArrayBlockingQueue<>(1);
    }

    @Group("Q")
    @Benchmark
    public Integer take() throws InterruptedException {
        return q.take();
    }

    @Group("Q")
    @Benchmark
    public void put() throws InterruptedException {
        q.put(42);
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_30 -t 2 -f 5 -to 10
     *    (we requested 2 threads, 5 forks, and 10 sec timeout)
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_30_Interrupts.class.getSimpleName())
                .threads(2)
                .forks(5)
                .timeout(TimeValue.seconds(10))
                .build();

        new Runner(opt).run();
    }

}
```

#### InfraParams

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.Setup;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.infra.BenchmarkParams;
import org.openjdk.jmh.infra.ThreadParams;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.Throughput)
@OutputTimeUnit(TimeUnit.SECONDS)
@State(Scope.Benchmark)
public class JMHSample_31_InfraParams {

    /*
     * There is a way to query JMH about the current running mode. This is
     * possible with three infrastructure objects we can request to be injected:
     *   - BenchmarkParams: covers the benchmark-global configuration
     *   - IterationParams: covers the current iteration configuration
     *   - ThreadParams: covers the specifics about threading
     *
     * Suppose we want to check how the ConcurrentHashMap scales under different
     * parallelism levels. We can put concurrencyLevel in @Param, but it sometimes
     * inconvenient if, say, we want it to follow the @Threads count. Here is
     * how we can query JMH about how many threads was requested for the current run,
     * and put that into concurrencyLevel argument for CHM constructor.
     */

    static final int THREAD_SLICE = 1000;

    private ConcurrentHashMap<String, String> mapSingle;
    private ConcurrentHashMap<String, String> mapFollowThreads;

    @Setup
    public void setup(BenchmarkParams params) {
        int capacity = 16 * THREAD_SLICE * params.getThreads();
        mapSingle        = new ConcurrentHashMap<>(capacity, 0.75f, 1);
        mapFollowThreads = new ConcurrentHashMap<>(capacity, 0.75f, params.getThreads());
    }

    /*
     * Here is another neat trick. Generate the distinct set of keys for all threads:
     */

    @State(Scope.Thread)
    public static class Ids {
        private List<String> ids;

        @Setup
        public void setup(ThreadParams threads) {
            ids = new ArrayList<>();
            for (int c = 0; c < THREAD_SLICE; c++) {
                ids.add("ID" + (THREAD_SLICE * threads.getThreadIndex() + c));
            }
        }
    }

    @Benchmark
    public void measureDefault(Ids ids) {
        for (String s : ids.ids) {
            mapSingle.remove(s);
            mapSingle.put(s, s);
        }
    }

    @Benchmark
    public void measureFollowThreads(Ids ids) {
        for (String s : ids.ids) {
            mapFollowThreads.remove(s);
            mapFollowThreads.put(s, s);
        }
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_31 -t 4 -f 5
     *    (we requested 4 threads, and 5 forks; there are also other options, see -h)
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_31_InfraParams.class.getSimpleName())
                .threads(4)
                .forks(5)
                .build();

        new Runner(opt).run();
    }

}
```

#### BulkWarmup

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.CompilerControl;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;
import org.openjdk.jmh.runner.options.WarmupMode;

import java.util.concurrent.TimeUnit;

@State(Scope.Thread)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public class JMHSample_32_BulkWarmup {

    /*
     * This is an addendum to JMHSample_12_Forking test.
     *
     * Sometimes you want an opposite configuration: instead of separating the profiles
     * for different benchmarks, you want to mix them together to test the worst-case
     * scenario.
     *
     * JMH has a bulk warmup feature for that: it does the warmups for all the tests
     * first, and then measures them. JMH still forks the JVM for each test, but once the
     * new JVM has started, all the warmups are being run there, before running the
     * measurement. This helps to dodge the type profile skews, as each test is still
     * executed in a different JVM, and we only "mix" the warmup code we want.
     */

    /*
     * These test classes are borrowed verbatim from JMHSample_12_Forking.
     */

    public interface Counter {
        int inc();
    }

    public static class Counter1 implements Counter {
        private int x;

        @Override
        public int inc() {
            return x++;
        }
    }

    public static class Counter2 implements Counter {
        private int x;

        @Override
        public int inc() {
            return x++;
        }
    }

    Counter c1 = new Counter1();
    Counter c2 = new Counter2();

    /*
     * And this is our test payload. Notice we have to break the inlining of the payload,
     * so that in could not be inlined in either measure_c1() or measure_c2() below, and
     * specialized for that only call.
     */

    @CompilerControl(CompilerControl.Mode.DONT_INLINE)
    public int measure(Counter c) {
        int s = 0;
        for (int i = 0; i < 10; i++) {
            s += c.inc();
        }
        return s;
    }

    @Benchmark
    public int measure_c1() {
        return measure(c1);
    }

    @Benchmark
    public int measure_c2() {
        return measure(c2);
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * Note how JMH runs the warmups first, and only then a given test. Note how JMH re-warmups
     * the JVM for each test. The scores for C1 and C2 cases are equally bad, compare them to
     * the scores from JMHSample_12_Forking.
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_32 -f 1 -wm BULK
     *    (we requested a single fork, and bulk warmup mode; there are also other options, see -h)
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_32_BulkWarmup.class.getSimpleName())
                // .includeWarmup(...) <-- this may include other benchmarks into warmup
                .warmupMode(WarmupMode.BULK) // see other WarmupMode.* as well
                .forks(1)
                .build();

        new Runner(opt).run();
    }

}
```

#### SecurityManager

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.Setup;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.annotations.TearDown;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;
import java.security.NoSuchAlgorithmException;
import java.security.Policy;
import java.security.URIParameter;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public class JMHSample_33_SecurityManager {

    /*
     * Some targeted tests may care about SecurityManager being installed.
     * Since JMH itself needs to do privileged actions, it is not enough
     * to blindly install the SecurityManager, as JMH infrastructure will fail.
     */

    /*
     * In this example, we want to measure the performance of System.getProperty
     * with SecurityManager installed or not. To do this, we have two state classes
     * with helper methods. One that reads the default JMH security policy (we ship one
     * with JMH), and installs the security manager; another one that makes sure
     * the SecurityManager is not installed.
     *
     * If you need a restricted security policy for the tests, you are advised to
     * get /jmh-security-minimal.policy, that contains the minimal permissions
     * required for JMH benchmark to run, merge the new permissions there, produce new
     * policy file in a temporary location, and load that policy file instead.
     * There is also /jmh-security-minimal-runner.policy, that contains the minimal
     * permissions for the JMH harness to run, if you want to use JVM args to arm
     * the SecurityManager.
     */

    @State(Scope.Benchmark)
    public static class SecurityManagerInstalled {
        @Setup
        public void setup() throws IOException, NoSuchAlgorithmException, URISyntaxException {
            URI policyFile = JMHSample_33_SecurityManager.class.getResource("/jmh-security.policy").toURI();
            Policy.setPolicy(Policy.getInstance("JavaPolicy", new URIParameter(policyFile)));
            System.setSecurityManager(new SecurityManager());
        }

        @TearDown
        public void tearDown() {
            System.setSecurityManager(null);
        }
    }

    @State(Scope.Benchmark)
    public static class SecurityManagerEmpty {
        @Setup
        public void setup() throws IOException, NoSuchAlgorithmException, URISyntaxException {
            System.setSecurityManager(null);
        }
    }

    @Benchmark
    public String testWithSM(SecurityManagerInstalled s) throws InterruptedException {
        return System.getProperty("java.home");
    }

    @Benchmark
    public String testWithoutSM(SecurityManagerEmpty s) throws InterruptedException {
        return System.getProperty("java.home");
    }

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_33 -f 1
     *    (we requested 5 warmup iterations, 5 forks; there are also other options, see -h))
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_33_SecurityManager.class.getSimpleName())
                .warmupIterations(5)
                .measurementIterations(5)
                .forks(1)
                .build();

        new Runner(opt).run();
    }

}
```

#### SafeLooping

```java
/*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.CompilerControl;
import org.openjdk.jmh.annotations.Fork;
import org.openjdk.jmh.annotations.Measurement;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;
import org.openjdk.jmh.annotations.Param;
import org.openjdk.jmh.annotations.Scope;
import org.openjdk.jmh.annotations.Setup;
import org.openjdk.jmh.annotations.State;
import org.openjdk.jmh.annotations.Warmup;
import org.openjdk.jmh.infra.Blackhole;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.concurrent.TimeUnit;

@State(Scope.Thread)
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Fork(3)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
public class JMHSample_34_SafeLooping {

    /*
     * JMHSample_11_Loops warns about the dangers of using loops in @Benchmark methods.
     * Sometimes, however, one needs to traverse through several elements in a dataset.
     * This is hard to do without loops, and therefore we need to devise a scheme for
     * safe looping.
     */

    /*
     * Suppose we want to measure how much it takes to execute work() with different
     * arguments. This mimics a frequent use case when multiple instances with the same
     * implementation, but different data, is measured.
     */

    static final int BASE = 42;

    static int work(int x) {
        return BASE + x;
    }

    /*
     * Every benchmark requires control. We do a trivial control for our benchmarks
     * by checking the benchmark costs are growing linearly with increased task size.
     * If it doesn't, then something wrong is happening.
     */

    @Param({"1", "10", "100", "1000"})
    int size;

    int[] xs;

    @Setup
    public void setup() {
        xs = new int[size];
        for (int c = 0; c < size; c++) {
            xs[c] = c;
        }
    }

    /*
     * First, the obviously wrong way: "saving" the result into a local variable would not
     * work. A sufficiently smart compiler will inline work(), and figure out only the last
     * work() call needs to be evaluated. Indeed, if you run it with varying $size, the score
     * will stay the same!
     */

    @Benchmark
    public int measureWrong_1() {
        int acc = 0;
        for (int x : xs) {
            acc = work(x);
        }
        return acc;
    }

    /*
     * Second, another wrong way: "accumulating" the result into a local variable. While
     * it would force the computation of each work() method, there are software pipelining
     * effects in action, that can merge the operations between two otherwise distinct work()
     * bodies. This will obliterate the benchmark setup.
     *
     * In this example, HotSpot does the unrolled loop, merges the $BASE operands into a single
     * addition to $acc, and then does a bunch of very tight stores of $x-s. The final performance
     * depends on how much of the loop unrolling happened *and* how much data is available to make
     * the large strides.
     */

    @Benchmark
    public int measureWrong_2() {
        int acc = 0;
        for (int x : xs) {
            acc += work(x);
        }
        return acc;
    }

    /*
     * Now, let's see how to measure these things properly. A very straight-forward way to
     * break the merging is to sink each result to Blackhole. This will force runtime to compute
     * every work() call in full. (We would normally like to care about several concurrent work()
     * computations at once, but the memory effects from Blackhole.consume() prevent those optimization
     * on most runtimes).
     */

    @Benchmark
    public void measureRight_1(Blackhole bh) {
        for (int x : xs) {
            bh.consume(work(x));
        }
    }

    /*
     * DANGEROUS AREA, PLEASE READ THE DESCRIPTION BELOW.
     *
     * Sometimes, the cost of sinking the value into a Blackhole is dominating the nano-benchmark score.
     * In these cases, one may try to do a make-shift "sinker" with non-inlineable method. This trick is
     * *very* VM-specific, and can only be used if you are verifying the generated code (that's a good
     * strategy when dealing with nano-benchmarks anyway).
     *
     * You SHOULD NOT use this trick in most cases. Apply only where needed.
     */

    @Benchmark
    public void measureRight_2() {
        for (int x : xs) {
            sink(work(x));
        }
    }

    @CompilerControl(CompilerControl.Mode.DONT_INLINE)
    public static void sink(int v) {
        // IT IS VERY IMPORTANT TO MATCH THE SIGNATURE TO AVOID AUTOBOXING.
        // The method intentionally does nothing.
    }


    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * You might notice measureWrong_1 does not depend on $size, measureWrong_2 has troubles with
     * linearity, and otherwise much faster than both measureRight_*. You can also see measureRight_2
     * is marginally faster than measureRight_1.
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_34
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(JMHSample_34_SafeLooping.class.getSimpleName())
                .forks(3)
                .build();

        new Runner(opt).run();
    }

}
```

#### Profilers

```java
*
 * Copyright (c) 2014, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.infra.Blackhole;
import org.openjdk.jmh.profile.ClassloaderProfiler;
import org.openjdk.jmh.profile.DTraceAsmProfiler;
import org.openjdk.jmh.profile.LinuxPerfProfiler;
import org.openjdk.jmh.profile.StackProfiler;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.net.URL;
import java.net.URLClassLoader;
import java.util.HashMap;
import java.util.Map;
import java.util.TreeMap;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicLong;

public class JMHSample_35_Profilers {

    /*
     * This sample serves as the profiler overview.
     *
     * JMH has a few very handy profilers that help to understand your benchmarks. While
     * these profilers are not the substitute for full-fledged external profilers, in many
     * cases, these are handy to quickly dig into the benchmark behavior. When you are
     * doing many cycles of tuning up the benchmark code itself, it is important to have
     * a quick turnaround for the results.
     *
     * Use -lprof to list the profilers. There are quite a few profilers, and this sample
     * would expand on a handful of most useful ones. Many profilers have their own options,
     * usually accessible via -prof <profiler-name>:help.
     *
     * Since profilers are reporting on different things, it is hard to construct a single
     * benchmark sample that will show all profilers in action. Therefore, we have a couple
     * of benchmarks in this sample.
     */

    /*
     * ================================ MAPS BENCHMARK ================================
     */

    @State(Scope.Thread)
    @Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
    @Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
    @Fork(3)
    @BenchmarkMode(Mode.AverageTime)
    @OutputTimeUnit(TimeUnit.NANOSECONDS)
    public static class Maps {
        private Map<Integer, Integer> map;

        @Param({"hashmap", "treemap"})
        private String type;

        private int begin;
        private int end;

        @Setup
        public void setup() {
            switch (type) {
                case "hashmap":
                    map = new HashMap<>();
                    break;
                case "treemap":
                    map = new TreeMap<>();
                    break;
                default:
                    throw new IllegalStateException("Unknown type: " + type);
            }

            begin = 1;
            end = 256;
            for (int i = begin; i < end; i++) {
                map.put(i, i);
            }
        }

        @Benchmark
        public void test(Blackhole bh) {
            for (int i = begin; i < end; i++) {
                bh.consume(map.get(i));
            }
        }

        /*
         * ============================== HOW TO RUN THIS TEST: ====================================
         *
         * You can run this test:
         *
         * a) Via the command line:
         *    $ mvn clean install
         *    $ java -jar target/benchmarks.jar JMHSample_35.*Maps -prof stack
         *    $ java -jar target/benchmarks.jar JMHSample_35.*Maps -prof gc
         *
         * b) Via the Java API:
         *    (see the JMH homepage for possible caveats when running from IDE:
         *      http://openjdk.java.net/projects/code-tools/jmh/)
         */

        public static void main(String[] args) throws RunnerException {
            Options opt = new OptionsBuilder()
                    .include(JMHSample_35_Profilers.Maps.class.getSimpleName())
                    .addProfiler(StackProfiler.class)
//                    .addProfiler(GCProfiler.class)
                    .build();

            new Runner(opt).run();
        }

        /*
            Running this benchmark will yield something like:

              Benchmark                              (type)  Mode  Cnt     Score    Error   Units
              JMHSample_35_Profilers.Maps.test     hashmap  avgt    5  1553.201 Â±   6.199   ns/op
              JMHSample_35_Profilers.Maps.test     treemap  avgt    5  5177.065 Â± 361.278   ns/op

            Running with -prof stack will yield:

              ....[Thread state: RUNNABLE]........................................................................
               99.0%  99.0% org.openjdk.jmh.samples.JMHSample_35_Profilers$Maps.test
                0.4%   0.4% org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Maps_test.test_avgt_jmhStub
                0.2%   0.2% sun.reflect.NativeMethodAccessorImpl.invoke0
                0.2%   0.2% java.lang.Integer.valueOf
                0.2%   0.2% sun.misc.Unsafe.compareAndSwapInt

              ....[Thread state: RUNNABLE]........................................................................
               78.0%  78.0% java.util.TreeMap.getEntry
               21.2%  21.2% org.openjdk.jmh.samples.JMHSample_35_Profilers$Maps.test
                0.4%   0.4% java.lang.Integer.valueOf
                0.2%   0.2% sun.reflect.NativeMethodAccessorImpl.invoke0
                0.2%   0.2% org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Maps_test.test_avgt_jmhStub

            Stack profiler is useful to quickly see if the code we are stressing actually executes. As many other
            sampling profilers, it is susceptible for sampling bias: it can fail to notice quickly executing methods,
            for example. In the benchmark above, it does not notice HashMap.get.

            Next up, GC profiler. Running with -prof gc will yield:

              Benchmark                                                            (type)  Mode  Cnt    Score     Error   Units

              JMHSample_35_Profilers.Maps.test                                   hashmap  avgt    5  1553.201 Â±   6.199   ns/op
              JMHSample_35_Profilers.Maps.test:Â·gc.alloc.rate                    hashmap  avgt    5  1257.046 Â±   5.675  MB/sec
              JMHSample_35_Profilers.Maps.test:Â·gc.alloc.rate.norm               hashmap  avgt    5  2048.001 Â±   0.001    B/op
              JMHSample_35_Profilers.Maps.test:Â·gc.churn.PS_Eden_Space           hashmap  avgt    5  1259.148 Â± 315.277  MB/sec
              JMHSample_35_Profilers.Maps.test:Â·gc.churn.PS_Eden_Space.norm      hashmap  avgt    5  2051.519 Â± 520.324    B/op
              JMHSample_35_Profilers.Maps.test:Â·gc.churn.PS_Survivor_Space       hashmap  avgt    5     0.175 Â±   0.386  MB/sec
              JMHSample_35_Profilers.Maps.test:Â·gc.churn.PS_Survivor_Space.norm  hashmap  avgt    5     0.285 Â±   0.629    B/op
              JMHSample_35_Profilers.Maps.test:Â·gc.count                         hashmap  avgt    5    29.000            counts
              JMHSample_35_Profilers.Maps.test:Â·gc.time                          hashmap  avgt    5    16.000                ms

              JMHSample_35_Profilers.Maps.test                                   treemap  avgt    5  5177.065 Â± 361.278   ns/op
              JMHSample_35_Profilers.Maps.test:Â·gc.alloc.rate                    treemap  avgt    5   377.251 Â±  26.188  MB/sec
              JMHSample_35_Profilers.Maps.test:Â·gc.alloc.rate.norm               treemap  avgt    5  2048.003 Â±   0.001    B/op
              JMHSample_35_Profilers.Maps.test:Â·gc.churn.PS_Eden_Space           treemap  avgt    5   392.743 Â± 174.156  MB/sec
              JMHSample_35_Profilers.Maps.test:Â·gc.churn.PS_Eden_Space.norm      treemap  avgt    5  2131.767 Â± 913.941    B/op
              JMHSample_35_Profilers.Maps.test:Â·gc.churn.PS_Survivor_Space       treemap  avgt    5     0.131 Â±   0.215  MB/sec
              JMHSample_35_Profilers.Maps.test:Â·gc.churn.PS_Survivor_Space.norm  treemap  avgt    5     0.709 Â±   1.125    B/op
              JMHSample_35_Profilers.Maps.test:Â·gc.count                         treemap  avgt    5    25.000            counts
              JMHSample_35_Profilers.Maps.test:Â·gc.time                          treemap  avgt    5    26.000                ms

            There, we can see that the tests are producing quite some garbage. "gc.alloc" would say we are allocating 1257
            and 377 MB of objects per second, or 2048 bytes per benchmark operation. "gc.churn" would say that GC removes
            the same amount of garbage from Eden space every second. In other words, we are producing 2048 bytes of garbage per
            benchmark operation.

            If you look closely at the test, you can get a (correct) hypothesis this is due to Integer autoboxing.

            Note that "gc.alloc" counters generally produce more accurate data, but they can also fail when threads come and
            go over the course of the benchmark. "gc.churn" values are updated on each GC event, and so if you want a more accurate
            data, running longer and/or with small heap would help. But anyhow, always cross-reference "gc.alloc" and "gc.churn"
            values with each other to get a complete picture.

            It is also worth noticing that non-normalized counters are dependent on benchmark performance! Here, "treemap"
            tests are 3x slower, and thus both allocation and churn rates are also comparably lower. It is often useful to look
            into non-normalized counters to see if the test is allocation/GC-bound (figure the allocation pressure "ceiling"
            for your configuration!), and normalized counters to see the more precise benchmark behavior.

            As most profilers, both "stack" and "gc" profile are able to aggregate samples from multiple forks. It is a good
            idea to run multiple forks with the profilers enabled, as it improves results error estimates.
        */
    }

    /*
     * ================================ CLASSLOADER BENCHMARK ================================
     */


    @State(Scope.Thread)
    @Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
    @Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
    @Fork(3)
    @BenchmarkMode(Mode.AverageTime)
    @OutputTimeUnit(TimeUnit.NANOSECONDS)
    public static class Classy {

        /**
         * Our own crippled classloader, that can only load a simple class over and over again.
         */
        public static class XLoader extends URLClassLoader {
            private static final byte[] X_BYTECODE = new byte[]{
                    (byte) 0xCA, (byte) 0xFE, (byte) 0xBA, (byte) 0xBE, 0x00, 0x00, 0x00, 0x34, 0x00, 0x0D, 0x0A, 0x00, 0x03, 0x00,
                    0x0A, 0x07, 0x00, 0x0B, 0x07, 0x00, 0x0C, 0x01, 0x00, 0x06, 0x3C, 0x69, 0x6E, 0x69, 0x74, 0x3E, 0x01, 0x00, 0x03,
                    0x28, 0x29, 0x56, 0x01, 0x00, 0x04, 0x43, 0x6F, 0x64, 0x65, 0x01, 0x00, 0x0F, 0x4C, 0x69, 0x6E, 0x65, 0x4E, 0x75,
                    0x6D, 0x62, 0x65, 0x72, 0x54, 0x61, 0x62, 0x6C, 0x65, 0x01, 0x00, 0x0A, 0x53, 0x6F, 0x75, 0x72, 0x63, 0x65, 0x46,
                    0x69, 0x6C, 0x65, 0x01, 0x00, 0x06, 0x58, 0x2E, 0x6A, 0x61, 0x76, 0x61, 0x0C, 0x00, 0x04, 0x00, 0x05, 0x01, 0x00,
                    0x01, 0x58, 0x01, 0x00, 0x10, 0x6A, 0x61, 0x76, 0x61, 0x2F, 0x6C, 0x61, 0x6E, 0x67, 0x2F, 0x4F, 0x62, 0x6A, 0x65,
                    0x63, 0x74, 0x00, 0x20, 0x00, 0x02, 0x00, 0x03, 0x00, 0x00, 0x00, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 0x04, 0x00,
                    0x05, 0x00, 0x01, 0x00, 0x06, 0x00, 0x00, 0x00, 0x1D, 0x00, 0x01, 0x00, 0x01, 0x00, 0x00, 0x00, 0x05, 0x2A,
                    (byte) 0xB7, 0x00, 0x01, (byte) 0xB1, 0x00, 0x00, 0x00, 0x01, 0x00, 0x07, 0x00, 0x00, 0x00, 0x06, 0x00, 0x01, 0x00,
                    0x00, 0x00, 0x01, 0x00, 0x01, 0x00, 0x08, 0x00, 0x00, 0x00, 0x02, 0x00, 0x09,
            };

            public XLoader() {
                super(new URL[0], ClassLoader.getSystemClassLoader());
            }

            @Override
            protected Class<?> findClass(final String name) throws ClassNotFoundException {
                return defineClass(name, X_BYTECODE, 0, X_BYTECODE.length);
            }

        }

        @Benchmark
        public Class<?> load() throws ClassNotFoundException {
            return Class.forName("X", true, new XLoader());
        }

        /*
         * ============================== HOW TO RUN THIS TEST: ====================================
         *
         * You can run this test:
         *
         * a) Via the command line:
         *    $ mvn clean install
         *    $ java -jar target/benchmarks.jar JMHSample_35.*Classy -prof cl
         *    $ java -jar target/benchmarks.jar JMHSample_35.*Classy -prof comp
         *
         * b) Via the Java API:
         *    (see the JMH homepage for possible caveats when running from IDE:
         *      http://openjdk.java.net/projects/code-tools/jmh/)
         */

        public static void main(String[] args) throws RunnerException {
            Options opt = new OptionsBuilder()
                    .include(JMHSample_35_Profilers.Classy.class.getSimpleName())
                    .addProfiler(ClassloaderProfiler.class)
//                    .addProfiler(CompilerProfiler.class)
                    .build();

            new Runner(opt).run();
        }

        /*
            Running with -prof cl will yield:

                Benchmark                                              Mode  Cnt      Score      Error        Units
                JMHSample_35_Profilers.Classy.load                     avgt   15  34215.363 Â±  545.892        ns/op
                JMHSample_35_Profilers.Classy.load:Â·class.load         avgt   15  29374.097 Â±  716.743  classes/sec
                JMHSample_35_Profilers.Classy.load:Â·class.load.norm    avgt   15      1.000 Â±    0.001   classes/op
                JMHSample_35_Profilers.Classy.load:Â·class.unload       avgt   15  29598.233 Â± 3420.181  classes/sec
                JMHSample_35_Profilers.Classy.load:Â·class.unload.norm  avgt   15      1.008 Â±    0.119   classes/op

            Here, we can see the benchmark indeed load class per benchmark op, and this adds up to more than 29K classloads
            per second. We can also see the runtime is able to successfully keep the number of loaded classes at bay,
            since the class unloading happens at the same rate.

            This profiler is handy when doing the classloading performance work, because it says if the classes
            were actually loaded, and not reused across the Class.forName calls. It also helps to see if the benchmark
            performs any classloading in the measurement phase. For example, if you have non-classloading benchmark,
            you would expect these metrics be zero.

            Another useful profiler that could tell if compiler is doing a heavy work in background, and thus interfering
            with measurement, -prof comp:

                Benchmark                                                   Mode  Cnt      Score      Error  Units
                JMHSample_35_Profilers.Classy.load                          avgt    5  33523.875 Â± 3026.025  ns/op
                JMHSample_35_Profilers.Classy.load:Â·compiler.time.profiled  avgt    5      5.000                ms
                JMHSample_35_Profilers.Classy.load:Â·compiler.time.total     avgt    5    479.000                ms

            We seem to be at proper steady state: out of 479 ms of total compiler work, only 5 ms happen during the
            measurement window. It is expected to have some level of background compilation even at steady state.

            As most profilers, both "cl" and "comp" are able to aggregate samples from multiple forks. It is a good
            idea to run multiple forks with the profilers enabled, as it improves results error estimates.
         */
    }

    /*
     * ================================ ATOMIC LONG BENCHMARK ================================
     */

    @State(Scope.Benchmark)
    @Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
    @Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
    @Fork(1)
    @BenchmarkMode(Mode.AverageTime)
    @OutputTimeUnit(TimeUnit.NANOSECONDS)
    public static class Atomic {
        private AtomicLong n;

        @Setup
        public void setup() {
            n = new AtomicLong();
        }

        @Benchmark
        public long test() {
            return n.incrementAndGet();
        }

        /*
         * ============================== HOW TO RUN THIS TEST: ====================================
         *
         * You can run this test:
         *
         * a) Via the command line:
         *    $ mvn clean install
         *    $ java -jar target/benchmarks.jar JMHSample_35.*Atomic -prof perf     -f 1 (Linux)
         *    $ java -jar target/benchmarks.jar JMHSample_35.*Atomic -prof perfnorm -f 3 (Linux)
         *    $ java -jar target/benchmarks.jar JMHSample_35.*Atomic -prof perfasm  -f 1 (Linux)
         *    $ java -jar target/benchmarks.jar JMHSample_35.*Atomic -prof xperfasm -f 1 (Windows)
         *    $ java -jar target/benchmarks.jar JMHSample_35.*Atomic -prof dtraceasm -f 1 (Mac OS X)
         * b) Via the Java API:
         *    (see the JMH homepage for possible caveats when running from IDE:
         *      http://openjdk.java.net/projects/code-tools/jmh/)
         */

        public static void main(String[] args) throws RunnerException {
            Options opt = new OptionsBuilder()
                    .include(JMHSample_35_Profilers.Atomic.class.getSimpleName())
                    .addProfiler(LinuxPerfProfiler.class)
//                    .addProfiler(LinuxPerfNormProfiler.class)
//                    .addProfiler(LinuxPerfAsmProfiler.class)
//                    .addProfiler(WinPerfAsmProfiler.class)
//                    .addProfiler(DTraceAsmProfiler.class)
                    .build();

            new Runner(opt).run();
        }

        /*
            Dealing with nanobenchmarks like these requires looking into the abyss of runtime, hardware, and
            generated code. Luckily, JMH has a few handy tools that ease the pain. If you are running Linux,
            then perf_events are probably available as standard package. This kernel facility taps into
            hardware counters, and provides the data for user space programs like JMH. Windows has less
            sophisticated facilities, but also usable, see below.

            One can simply run "perf stat java -jar ..." to get the first idea how the workload behaves. In
            JMH case, however, this will cause perf to profile both host and forked JVMs.

            -prof perf avoids that: JMH invokes perf for the forked VM alone. For the benchmark above, it
            would print something like:

                 Perf stats:
                                --------------------------------------------------

                       4172.776137 task-clock (msec)         #    0.411 CPUs utilized
                               612 context-switches          #    0.147 K/sec
                                31 cpu-migrations            #    0.007 K/sec
                               195 page-faults               #    0.047 K/sec
                    16,599,643,026 cycles                    #    3.978 GHz                     [30.80%]
                   <not supported> stalled-cycles-frontend
                   <not supported> stalled-cycles-backend
                    17,815,084,879 instructions              #    1.07  insns per cycle         [38.49%]
                     3,813,373,583 branches                  #  913.870 M/sec                   [38.56%]
                         1,212,788 branch-misses             #    0.03% of all branches         [38.91%]
                     7,582,256,427 L1-dcache-loads           # 1817.077 M/sec                   [39.07%]
                           312,913 L1-dcache-load-misses     #    0.00% of all L1-dcache hits   [38.66%]
                            35,688 LLC-loads                 #    0.009 M/sec                   [32.58%]
                   <not supported> LLC-load-misses:HG
                   <not supported> L1-icache-loads:HG
                           161,436 L1-icache-load-misses:HG  #    0.00% of all L1-icache hits   [32.81%]
                     7,200,981,198 dTLB-loads:HG             # 1725.705 M/sec                   [32.68%]
                             3,360 dTLB-load-misses:HG       #    0.00% of all dTLB cache hits  [32.65%]
                           193,874 iTLB-loads:HG             #    0.046 M/sec                   [32.56%]
                             4,193 iTLB-load-misses:HG       #    2.16% of all iTLB cache hits  [32.44%]
                   <not supported> L1-dcache-prefetches:HG
                                 0 L1-dcache-prefetch-misses:HG #    0.000 K/sec                   [32.33%]

                      10.159432892 seconds time elapsed

            We can already see this benchmark goes with good IPC, does lots of loads and lots of stores,
            all of them are more or less fulfilled without misses. The data like this is not handy though:
            you would like to normalize the counters per benchmark op.

            This is exactly what -prof perfnorm does:

                Benchmark                                                   Mode  Cnt   Score    Error  Units
                JMHSample_35_Profilers.Atomic.test                          avgt   15   6.551 Â±  0.023  ns/op
                JMHSample_35_Profilers.Atomic.test:Â·CPI                     avgt    3   0.933 Â±  0.026   #/op
                JMHSample_35_Profilers.Atomic.test:Â·L1-dcache-load-misses   avgt    3   0.001 Â±  0.022   #/op
                JMHSample_35_Profilers.Atomic.test:Â·L1-dcache-loads         avgt    3  12.267 Â±  1.324   #/op
                JMHSample_35_Profilers.Atomic.test:Â·L1-dcache-store-misses  avgt    3   0.001 Â±  0.006   #/op
                JMHSample_35_Profilers.Atomic.test:Â·L1-dcache-stores        avgt    3   4.090 Â±  0.402   #/op
                JMHSample_35_Profilers.Atomic.test:Â·L1-icache-load-misses   avgt    3   0.001 Â±  0.011   #/op
                JMHSample_35_Profilers.Atomic.test:Â·LLC-loads               avgt    3   0.001 Â±  0.004   #/op
                JMHSample_35_Profilers.Atomic.test:Â·LLC-stores              avgt    3  â‰ˆ 10â»â´            #/op
                JMHSample_35_Profilers.Atomic.test:Â·branch-misses           avgt    3  â‰ˆ 10â»â´            #/op
                JMHSample_35_Profilers.Atomic.test:Â·branches                avgt    3   6.152 Â±  0.385   #/op
                JMHSample_35_Profilers.Atomic.test:Â·bus-cycles              avgt    3   0.670 Â±  0.048   #/op
                JMHSample_35_Profilers.Atomic.test:Â·context-switches        avgt    3  â‰ˆ 10â»â¶            #/op
                JMHSample_35_Profilers.Atomic.test:Â·cpu-migrations          avgt    3  â‰ˆ 10â»â·            #/op
                JMHSample_35_Profilers.Atomic.test:Â·cycles                  avgt    3  26.790 Â±  1.393   #/op
                JMHSample_35_Profilers.Atomic.test:Â·dTLB-load-misses        avgt    3  â‰ˆ 10â»â´            #/op
                JMHSample_35_Profilers.Atomic.test:Â·dTLB-loads              avgt    3  12.278 Â±  0.277   #/op
                JMHSample_35_Profilers.Atomic.test:Â·dTLB-store-misses       avgt    3  â‰ˆ 10â»âµ            #/op
                JMHSample_35_Profilers.Atomic.test:Â·dTLB-stores             avgt    3   4.113 Â±  0.437   #/op
                JMHSample_35_Profilers.Atomic.test:Â·iTLB-load-misses        avgt    3  â‰ˆ 10â»âµ            #/op
                JMHSample_35_Profilers.Atomic.test:Â·iTLB-loads              avgt    3   0.001 Â±  0.034   #/op
                JMHSample_35_Profilers.Atomic.test:Â·instructions            avgt    3  28.729 Â±  1.297   #/op
                JMHSample_35_Profilers.Atomic.test:Â·minor-faults            avgt    3  â‰ˆ 10â»â·            #/op
                JMHSample_35_Profilers.Atomic.test:Â·page-faults             avgt    3  â‰ˆ 10â»â·            #/op
                JMHSample_35_Profilers.Atomic.test:Â·ref-cycles              avgt    3  26.734 Â±  2.081   #/op

            It is customary to trim the lines irrelevant to the particular benchmark. We show all of them here for
            completeness.

            We can see that the benchmark does ~12 loads per benchmark op, and about ~4 stores per op, most of
            them fitting in the cache. There are also ~6 branches per benchmark op, all are predicted as well.
            It is also easy to see the benchmark op takes ~28 instructions executed in ~27 cycles.

            The output would get more interesting when we run with more threads, say, -t 8:

                Benchmark                                                   Mode  Cnt    Score     Error  Units
                JMHSample_35_Profilers.Atomic.test                          avgt   15  143.595 Â±   1.968  ns/op
                JMHSample_35_Profilers.Atomic.test:Â·CPI                     avgt    3   17.741 Â±  28.761   #/op
                JMHSample_35_Profilers.Atomic.test:Â·L1-dcache-load-misses   avgt    3    0.175 Â±   0.406   #/op
                JMHSample_35_Profilers.Atomic.test:Â·L1-dcache-loads         avgt    3   11.872 Â±   0.786   #/op
                JMHSample_35_Profilers.Atomic.test:Â·L1-dcache-store-misses  avgt    3    0.184 Â±   0.505   #/op
                JMHSample_35_Profilers.Atomic.test:Â·L1-dcache-stores        avgt    3    4.422 Â±   0.561   #/op
                JMHSample_35_Profilers.Atomic.test:Â·L1-icache-load-misses   avgt    3    0.015 Â±   0.083   #/op
                JMHSample_35_Profilers.Atomic.test:Â·LLC-loads               avgt    3    0.015 Â±   0.128   #/op
                JMHSample_35_Profilers.Atomic.test:Â·LLC-stores              avgt    3    1.036 Â±   0.045   #/op
                JMHSample_35_Profilers.Atomic.test:Â·branch-misses           avgt    3    0.224 Â±   0.492   #/op
                JMHSample_35_Profilers.Atomic.test:Â·branches                avgt    3    6.524 Â±   2.873   #/op
                JMHSample_35_Profilers.Atomic.test:Â·bus-cycles              avgt    3   13.475 Â±  14.502   #/op
                JMHSample_35_Profilers.Atomic.test:Â·context-switches        avgt    3   â‰ˆ 10â»â´             #/op
                JMHSample_35_Profilers.Atomic.test:Â·cpu-migrations          avgt    3   â‰ˆ 10â»â¶             #/op
                JMHSample_35_Profilers.Atomic.test:Â·cycles                  avgt    3  537.874 Â± 595.723   #/op
                JMHSample_35_Profilers.Atomic.test:Â·dTLB-load-misses        avgt    3    0.001 Â±   0.006   #/op
                JMHSample_35_Profilers.Atomic.test:Â·dTLB-loads              avgt    3   12.032 Â±   2.430   #/op
                JMHSample_35_Profilers.Atomic.test:Â·dTLB-store-misses       avgt    3   â‰ˆ 10â»â´             #/op
                JMHSample_35_Profilers.Atomic.test:Â·dTLB-stores             avgt    3    4.557 Â±   0.948   #/op
                JMHSample_35_Profilers.Atomic.test:Â·iTLB-load-misses        avgt    3   â‰ˆ 10â»Â³             #/op
                JMHSample_35_Profilers.Atomic.test:Â·iTLB-loads              avgt    3    0.016 Â±   0.052   #/op
                JMHSample_35_Profilers.Atomic.test:Â·instructions            avgt    3   30.367 Â±  15.052   #/op
                JMHSample_35_Profilers.Atomic.test:Â·minor-faults            avgt    3   â‰ˆ 10â»âµ             #/op
                JMHSample_35_Profilers.Atomic.test:Â·page-faults             avgt    3   â‰ˆ 10â»âµ             #/op
                JMHSample_35_Profilers.Atomic.test:Â·ref-cycles              avgt    3  538.697 Â± 590.183   #/op

            Note how this time the CPI is awfully high: 17 cycles per instruction! Indeed, we are making almost the
            same ~30 instructions, but now they take >530 cycles. Other counters highlight why: we now have cache
            misses on both loads and stores, on all levels of cache hierarchy. With a simple constant-footprint
            like ours, that's an indication of sharing problems. Indeed, our AtomicLong is heavily-contended
            with 8 threads.

            "perfnorm", again, can (and should!) be used with multiple forks, to properly estimate the metrics.

            The last, but not the least player on our field is -prof perfasm. It is important to follow up on
            generated code when dealing with fine-grained benchmarks. We could employ PrintAssembly to dump the
            generated code, but it will dump *all* the generated code, and figuring out what is related to our
            benchmark is a daunting task. But we have "perf" that can tell what program addresses are really hot!
            This enables us to contrast the assembly output.

            -prof perfasm would indeed contrast out the hottest loop in the generated code! It will also point
            fingers at "lock xadd" as the hottest instruction in our code. Hardware counters are not very precise
            about the instruction addresses, so sometimes they attribute the events to the adjacent code lines.

                Hottest code regions (>10.00% "cycles" events):
                ....[Hottest Region 1]..............................................................................
                 [0x7f1824f87c45:0x7f1824f87c79] in org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Atomic_test::test_avgt_jmhStub

                                                                                  ; - org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Atomic_test::test_avgt_jmhStub@29 (line 201)
                                                                                  ; implicit exception: dispatches to 0x00007f1824f87d21
                                    0x00007f1824f87c25: test   %r11d,%r11d
                                    0x00007f1824f87c28: jne    0x00007f1824f87cbd  ;*ifeq
                                                                                  ; - org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Atomic_test::test_avgt_jmhStub@32 (line 201)
                                    0x00007f1824f87c2e: mov    $0x1,%ebp
                                    0x00007f1824f87c33: nopw   0x0(%rax,%rax,1)
                                    0x00007f1824f87c3c: xchg   %ax,%ax            ;*aload
                                                                                  ; - org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Atomic_test::test_avgt_jmhStub@13 (line 199)
                                    0x00007f1824f87c40: mov    0x8(%rsp),%r10
                  0.00%             0x00007f1824f87c45: mov    0xc(%r10),%r11d    ;*getfield n
                                                                                  ; - org.openjdk.jmh.samples.JMHSample_35_Profilers$Atomic::test@1 (line 280)
                                                                                  ; - org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Atomic_test::test_avgt_jmhStub@16 (line 199)
                  0.19%    0.02%    0x00007f1824f87c49: test   %r11d,%r11d
                                    0x00007f1824f87c4c: je     0x00007f1824f87cad
                                    0x00007f1824f87c4e: mov    $0x1,%edx
                                    0x00007f1824f87c53: lock xadd %rdx,0x10(%r12,%r11,8)
                                                                                  ;*invokevirtual getAndAddLong
                                                                                  ; - java.util.concurrent.atomic.AtomicLong::incrementAndGet@8 (line 200)
                                                                                  ; - org.openjdk.jmh.samples.JMHSample_35_Profilers$Atomic::test@4 (line 280)
                                                                                  ; - org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Atomic_test::test_avgt_jmhStub@16 (line 199)
                 95.20%   95.06%    0x00007f1824f87c5a: add    $0x1,%rdx          ;*ladd
                                                                                  ; - java.util.concurrent.atomic.AtomicLong::incrementAndGet@12 (line 200)
                                                                                  ; - org.openjdk.jmh.samples.JMHSample_35_Profilers$Atomic::test@4 (line 280)
                                                                                  ; - org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Atomic_test::test_avgt_jmhStub@16 (line 199)
                  0.24%    0.00%    0x00007f1824f87c5e: mov    0x10(%rsp),%rsi
                                    0x00007f1824f87c63: callq  0x00007f1824e2b020  ; OopMap{[0]=Oop [8]=Oop [16]=Oop [24]=Oop off=232}
                                                                                  ;*invokevirtual consume
                                                                                  ; - org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Atomic_test::test_avgt_jmhStub@19 (line 199)
                                                                                  ;   {optimized virtual_call}
                  0.20%    0.01%    0x00007f1824f87c68: mov    0x18(%rsp),%r10
                                    0x00007f1824f87c6d: movzbl 0x94(%r10),%r11d   ;*getfield isDone
                                                                                  ; - org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Atomic_test::test_avgt_jmhStub@29 (line 201)
                  0.00%             0x00007f1824f87c75: add    $0x1,%rbp          ; OopMap{r10=Oop [0]=Oop [8]=Oop [16]=Oop [24]=Oop off=249}
                                                                                  ;*ifeq
                                                                                  ; - org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Atomic_test::test_avgt_jmhStub@32 (line 201)
                  0.20%    0.01%    0x00007f1824f87c79: test   %eax,0x15f36381(%rip)        # 0x00007f183aebe000
                                                                                  ;   {poll}
                                    0x00007f1824f87c7f: test   %r11d,%r11d
                                    0x00007f1824f87c82: je     0x00007f1824f87c40  ;*aload_2
                                                                                  ; - org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Atomic_test::test_avgt_jmhStub@35 (line 202)
                                    0x00007f1824f87c84: mov    $0x7f1839be4220,%r10
                                    0x00007f1824f87c8e: callq  *%r10              ;*invokestatic nanoTime
                                                                                  ; - org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Atomic_test::test_avgt_jmhStub@36 (line 202)
                                    0x00007f1824f87c91: mov    (%rsp),%r10
                ....................................................................................................
                 96.03%   95.10%  <total for region 1>

            perfasm would also print the hottest methods to show if we indeed spending time in our benchmark. Most of the time,
            it can demangle VM and kernel symbols as well:

                ....[Hottest Methods (after inlining)]..............................................................
                 96.03%   95.10%  org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Atomic_test::test_avgt_jmhStub
                  0.73%    0.78%  org.openjdk.jmh.samples.generated.JMHSample_35_Profilers_Atomic_test::test_AverageTime
                  0.63%    0.00%  org.openjdk.jmh.infra.Blackhole::consume
                  0.23%    0.25%  native_write_msr_safe ([kernel.kallsyms])
                  0.09%    0.05%  _raw_spin_unlock ([kernel.kallsyms])
                  0.09%    0.00%  [unknown] (libpthread-2.19.so)
                  0.06%    0.07%  _raw_spin_lock ([kernel.kallsyms])
                  0.06%    0.04%  _raw_spin_unlock_irqrestore ([kernel.kallsyms])
                  0.06%    0.05%  _IO_fwrite (libc-2.19.so)
                  0.05%    0.03%  __srcu_read_lock; __srcu_read_unlock ([kernel.kallsyms])
                  0.04%    0.05%  _raw_spin_lock_irqsave ([kernel.kallsyms])
                  0.04%    0.06%  vfprintf (libc-2.19.so)
                  0.04%    0.01%  mutex_unlock ([kernel.kallsyms])
                  0.04%    0.01%  _nv014306rm ([nvidia])
                  0.04%    0.04%  rcu_eqs_enter_common.isra.47 ([kernel.kallsyms])
                  0.04%    0.02%  mutex_lock ([kernel.kallsyms])
                  0.03%    0.07%  __acct_update_integrals ([kernel.kallsyms])
                  0.03%    0.02%  fget_light ([kernel.kallsyms])
                  0.03%    0.01%  fput ([kernel.kallsyms])
                  0.03%    0.04%  rcu_eqs_exit_common.isra.48 ([kernel.kallsyms])
                  1.63%    2.26%  <...other 319 warm methods...>
                ....................................................................................................
                100.00%   98.97%  <totals>

                ....[Distribution by Area]..........................................................................
                 97.44%   95.99%  <generated code>
                  1.60%    2.42%  <native code in ([kernel.kallsyms])>
                  0.47%    0.78%  <native code in (libjvm.so)>
                  0.22%    0.29%  <native code in (libc-2.19.so)>
                  0.15%    0.07%  <native code in (libpthread-2.19.so)>
                  0.07%    0.38%  <native code in ([nvidia])>
                  0.05%    0.06%  <native code in (libhsdis-amd64.so)>
                  0.00%    0.00%  <native code in (nf_conntrack.ko)>
                  0.00%    0.00%  <native code in (hid.ko)>
                ....................................................................................................
                100.00%  100.00%  <totals>

            Since program addresses change from fork to fork, it does not make sense to run perfasm with more than
            a single fork.
        */
    }
}
```

#### BranchPrediction

```java
/*
 * Copyright (c) 2015, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.infra.Blackhole;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.Arrays;
import java.util.Random;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Fork(5)
@State(Scope.Benchmark)
public class JMHSample_36_BranchPrediction {

    /*
     * This sample serves as a warning against regular data sets.
     *
     * It is very tempting to present a regular data set to benchmark, either due to
     * naive generation strategy, or just from feeling better about regular data sets.
     * Unfortunately, it frequently backfires: the regular datasets are known to be
     * optimized well by software and hardware. This example exploits one of these
     * optimizations: branch prediction.
     *
     * Imagine our benchmark selects the branch based on the array contents, as
     * we are streaming through it:
     */

    private static final int COUNT = 1024 * 1024;

    private byte[] sorted;
    private byte[] unsorted;

    @Setup
    public void setup() {
        sorted = new byte[COUNT];
        unsorted = new byte[COUNT];
        Random random = new Random(1234);
        random.nextBytes(sorted);
        random.nextBytes(unsorted);
        Arrays.sort(sorted);
    }

    @Benchmark
    @OperationsPerInvocation(COUNT)
    public void sorted(Blackhole bh1, Blackhole bh2) {
        for (byte v : sorted) {
            if (v > 0) {
                bh1.consume(v);
            } else {
                bh2.consume(v);
            }
        }
    }

    @Benchmark
    @OperationsPerInvocation(COUNT)
    public void unsorted(Blackhole bh1, Blackhole bh2) {
        for (byte v : unsorted) {
            if (v > 0) {
                bh1.consume(v);
            } else {
                bh2.consume(v);
            }
        }
    }

    /*
        There is a substantial difference in performance for these benchmarks!

        It is explained by good branch prediction in "sorted" case, and branch mispredicts in "unsorted"
        case. -prof perfnorm conveniently highlights that, with larger "branch-misses", and larger "CPI"
        for "unsorted" case:

        Benchmark                                                       Mode  Cnt   Score    Error  Units
        JMHSample_36_BranchPrediction.sorted                            avgt   25   2.160 Â±  0.049  ns/op
        JMHSample_36_BranchPrediction.sorted:Â·CPI                       avgt    5   0.286 Â±  0.025   #/op
        JMHSample_36_BranchPrediction.sorted:Â·branch-misses             avgt    5  â‰ˆ 10â»â´            #/op
        JMHSample_36_BranchPrediction.sorted:Â·branches                  avgt    5   7.606 Â±  1.742   #/op
        JMHSample_36_BranchPrediction.sorted:Â·cycles                    avgt    5   8.998 Â±  1.081   #/op
        JMHSample_36_BranchPrediction.sorted:Â·instructions              avgt    5  31.442 Â±  4.899   #/op

        JMHSample_36_BranchPrediction.unsorted                          avgt   25   5.943 Â±  0.018  ns/op
        JMHSample_36_BranchPrediction.unsorted:Â·CPI                     avgt    5   0.775 Â±  0.052   #/op
        JMHSample_36_BranchPrediction.unsorted:Â·branch-misses           avgt    5   0.529 Â±  0.026   #/op  <--- OOPS
        JMHSample_36_BranchPrediction.unsorted:Â·branches                avgt    5   7.841 Â±  0.046   #/op
        JMHSample_36_BranchPrediction.unsorted:Â·cycles                  avgt    5  24.793 Â±  0.434   #/op
        JMHSample_36_BranchPrediction.unsorted:Â·instructions            avgt    5  31.994 Â±  2.342   #/op

        It is an open question if you want to measure only one of these tests. In many cases, you have to measure
        both to get the proper best-case and worst-case estimate!
     */


    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_36
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */
    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(".*" + JMHSample_36_BranchPrediction.class.getSimpleName() + ".*")
                .build();

        new Runner(opt).run();
    }

}
```

#### CacheAccess

```java
/*
 * Copyright (c) 2015, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.infra.Blackhole;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.Random;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Fork(5)
@State(Scope.Benchmark)
public class JMHSample_37_CacheAccess {

    /*
     * This sample serves as a warning against subtle differences in cache access patterns.
     *
     * Many performance differences may be explained by the way tests are accessing memory.
     * In the example below, we walk the matrix either row-first, or col-first:
     */

    private final static int COUNT = 4096;
    private final static int MATRIX_SIZE = COUNT * COUNT;

    private int[][] matrix;

    @Setup
    public void setup() {
        matrix = new int[COUNT][COUNT];
        Random random = new Random(1234);
        for (int i = 0; i < COUNT; i++) {
            for (int j = 0; j < COUNT; j++) {
                matrix[i][j] = random.nextInt();
            }
        }
    }

    @Benchmark
    @OperationsPerInvocation(MATRIX_SIZE)
    public void colFirst(Blackhole bh) {
        for (int c = 0; c < COUNT; c++) {
            for (int r = 0; r < COUNT; r++) {
                bh.consume(matrix[r][c]);
            }
        }
    }

    @Benchmark
    @OperationsPerInvocation(MATRIX_SIZE)
    public void rowFirst(Blackhole bh) {
        for (int r = 0; r < COUNT; r++) {
            for (int c = 0; c < COUNT; c++) {
                bh.consume(matrix[r][c]);
            }
        }
    }

    /*
        Notably, colFirst accesses are much slower, and that's not a surprise: Java's multidimensional
        arrays are actually rigged, being one-dimensional arrays of one-dimensional arrays. Therefore,
        pulling n-th element from each of the inner array induces more cache misses, when matrix is large.
        -prof perfnorm conveniently highlights that, with >2 cache misses per one benchmark op:

        Benchmark                                                 Mode  Cnt   Score    Error  Units
        JMHSample_37_MatrixCopy.colFirst                          avgt   25   5.306 Â±  0.020  ns/op
        JMHSample_37_MatrixCopy.colFirst:Â·CPI                     avgt    5   0.621 Â±  0.011   #/op
        JMHSample_37_MatrixCopy.colFirst:Â·L1-dcache-load-misses   avgt    5   2.177 Â±  0.044   #/op <-- OOPS
        JMHSample_37_MatrixCopy.colFirst:Â·L1-dcache-loads         avgt    5  14.804 Â±  0.261   #/op
        JMHSample_37_MatrixCopy.colFirst:Â·LLC-loads               avgt    5   2.165 Â±  0.091   #/op
        JMHSample_37_MatrixCopy.colFirst:Â·cycles                  avgt    5  22.272 Â±  0.372   #/op
        JMHSample_37_MatrixCopy.colFirst:Â·instructions            avgt    5  35.888 Â±  1.215   #/op

        JMHSample_37_MatrixCopy.rowFirst                          avgt   25   2.662 Â±  0.003  ns/op
        JMHSample_37_MatrixCopy.rowFirst:Â·CPI                     avgt    5   0.312 Â±  0.003   #/op
        JMHSample_37_MatrixCopy.rowFirst:Â·L1-dcache-load-misses   avgt    5   0.066 Â±  0.001   #/op
        JMHSample_37_MatrixCopy.rowFirst:Â·L1-dcache-loads         avgt    5  14.570 Â±  0.400   #/op
        JMHSample_37_MatrixCopy.rowFirst:Â·LLC-loads               avgt    5   0.002 Â±  0.001   #/op
        JMHSample_37_MatrixCopy.rowFirst:Â·cycles                  avgt    5  11.046 Â±  0.343   #/op
        JMHSample_37_MatrixCopy.rowFirst:Â·instructions            avgt    5  35.416 Â±  1.248   #/op

        So, when comparing two different benchmarks, you have to follow up if the difference is caused
        by the memory locality issues.
     */

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_37
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */
    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(".*" + JMHSample_37_CacheAccess.class.getSimpleName() + ".*")
                .build();

        new Runner(opt).run();
    }

}
```

#### PerInvokeSetup

```java
/*
 * Copyright (c) 2015, Oracle America, Inc.
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 *  * Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 *  * Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 *  * Neither the name of Oracle nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without
 *    specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF
 * THE POSSIBILITY OF SUCH DAMAGE.
 */
package org.openjdk.jmh.samples;

import org.openjdk.jmh.annotations.*;
import org.openjdk.jmh.runner.Runner;
import org.openjdk.jmh.runner.RunnerException;
import org.openjdk.jmh.runner.options.Options;
import org.openjdk.jmh.runner.options.OptionsBuilder;

import java.util.Arrays;
import java.util.Random;
import java.util.concurrent.TimeUnit;

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Fork(5)
public class JMHSample_38_PerInvokeSetup {

    /*
     * This example highlights the usual mistake in non-steady-state benchmarks.
     *
     * Suppose we want to test how long it takes to bubble sort an array. Naively,
     * we could make the test that populates an array with random (unsorted) values,
     * and calls sort on it over and over again:
     */

    private void bubbleSort(byte[] b) {
        boolean changed = true;
        while (changed) {
            changed = false;
            for (int c = 0; c < b.length - 1; c++) {
                if (b[c] > b[c + 1]) {
                    byte t = b[c];
                    b[c] = b[c + 1];
                    b[c + 1] = t;
                    changed = true;
                }
            }
        }
    }

    // Could be an implicit State instead, but we are going to use it
    // as the dependency in one of the tests below
    @State(Scope.Benchmark)
    public static class Data {

        @Param({"1", "16", "256"})
        int count;

        byte[] arr;

        @Setup
        public void setup() {
            arr = new byte[count];
            Random random = new Random(1234);
            random.nextBytes(arr);
        }
    }

    @Benchmark
    public byte[] measureWrong(Data d) {
        bubbleSort(d.arr);
        return d.arr;
    }

    /*
     * The method above is subtly wrong: it sorts the random array on the first invocation
     * only. Every subsequent call will "sort" the already sorted array. With bubble sort,
     * that operation would be significantly faster!
     *
     * This is how we might *try* to measure it right by making a copy in Level.Invocation
     * setup. However, this is susceptible to the problems described in Level.Invocation
     * Javadocs, READ AND UNDERSTAND THOSE DOCS BEFORE USING THIS APPROACH.
     */

    @State(Scope.Thread)
    public static class DataCopy {
        byte[] copy;

        @Setup(Level.Invocation)
        public void setup2(Data d) {
           copy = Arrays.copyOf(d.arr, d.arr.length);
        }
    }

    @Benchmark
    public byte[] measureNeutral(DataCopy d) {
        bubbleSort(d.copy);
        return d.copy;
    }

    /*
     * In an overwhelming majority of cases, the only sensible thing to do is to suck up
     * the per-invocation setup costs into a benchmark itself. This work well in practice,
     * especially when the payload costs dominate the setup costs.
     */

    @Benchmark
    public byte[] measureRight(Data d) {
        byte[] c = Arrays.copyOf(d.arr, d.arr.length);
        bubbleSort(c);
        return c;
    }

    /*
        Benchmark                                   (count)  Mode  Cnt      Score     Error  Units

        JMHSample_38_PerInvokeSetup.measureWrong          1  avgt   25      2.408 Â±   0.011  ns/op
        JMHSample_38_PerInvokeSetup.measureWrong         16  avgt   25      8.286 Â±   0.023  ns/op
        JMHSample_38_PerInvokeSetup.measureWrong        256  avgt   25     73.405 Â±   0.018  ns/op

        JMHSample_38_PerInvokeSetup.measureNeutral        1  avgt   25     15.835 Â±   0.470  ns/op
        JMHSample_38_PerInvokeSetup.measureNeutral       16  avgt   25    112.552 Â±   0.787  ns/op
        JMHSample_38_PerInvokeSetup.measureNeutral      256  avgt   25  58343.848 Â± 991.202  ns/op

        JMHSample_38_PerInvokeSetup.measureRight          1  avgt   25      6.075 Â±   0.018  ns/op
        JMHSample_38_PerInvokeSetup.measureRight         16  avgt   25    102.390 Â±   0.676  ns/op
        JMHSample_38_PerInvokeSetup.measureRight        256  avgt   25  58812.411 Â± 997.951  ns/op

        We can clearly see that "measureWrong" provides a very weird result: it "sorts" way too fast.
        "measureNeutral" is neither good or bad: while it prepares the data for each invocation correctly,
        the timing overheads are clearly visible. These overheads can be overwhelming, depending on
        the thread count and/or OS flavor.
     */

    /*
     * ============================== HOW TO RUN THIS TEST: ====================================
     *
     * You can run this test:
     *
     * a) Via the command line:
     *    $ mvn clean install
     *    $ java -jar target/benchmarks.jar JMHSample_38
     *
     * b) Via the Java API:
     *    (see the JMH homepage for possible caveats when running from IDE:
     *      http://openjdk.java.net/projects/code-tools/jmh/)
     */
    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(".*" + JMHSample_38_PerInvokeSetup.class.getSimpleName() + ".*")
                .build();

        new Runner(opt).run();
    }

}
```

