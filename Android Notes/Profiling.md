# Profiling

## Android Profiler

CPU Profiler, Memory Profiler, Network Profiler, Energy Profiler en Android Studio

## Benchmark

```kotlin
dependencies {
    androidTestImplementation("androidx.benchmark:benchmark-junit4:1.2.2")
}

@RunWith(AndroidJUnit4::class)
class MyBenchmark {
    @get:Rule
    val benchmarkRule = BenchmarkRule()
    
    @Test
    fun benchmarkSomeWork() {
        benchmarkRule.measureRepeated {
            // Code to benchmark
            doWork()
        }
    }
}
```

## Recursos
- [Profiling](https://developer.android.com/studio/profile)
