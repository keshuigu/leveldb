# Note 1 - 从`db_bench.cc`开始

## `main(int argc, char** argv)`

程序的执行入口`main`主要做了以下几件事情：

1. 设置`write_buffer_size`为4MB，该值控制内存中维护的数据缓冲大小
2. 设置`max_file_size`为2MB，该值控制leveldb存储文件的大小。leveldb会在文件超过该值后切换到新文件
3. 设置`block_size`为4KB，该值控制leveldb每个数据块的大小
4. 设置`max_open_files`为1000，该值控制最多由level控制的文件的数量
5. 根据输入的参数设置一系列全局值，暂时先不管
6. 根据当前操作系统设置`g_env`为默认的leveldb环境
7. 从`g_env`中获得测试时数据库文件的存放目录
8. 初始化`Benchmark`类，并开始执行测试

默认的`Benchmark`包含：`fillseq`, `fillsync`, `fillrandom`, `overwrite`, `readrandom`, `readrandom`, `readseq`, `readreverse`, `compact`, `readrandom`, `readseq`, `readreverse`, `fill100K`, `crc32c`, `snappycomp`, `snappyuncomp`, `zstdcomp`, `zstduncomp`,

## `Benchmark`类

### 成员变量

| 名称                | 类型                | 用途 |
| ------------------- | ------------------- | ---- |
| cache_              | Cache*              | TODO |
| filter_policy_      | const FilterPolicy* | TODO |
| db_                 | DB*                 | TODO |
| num_                | int                 | TODO |
| value_size_         | int                 | TODO |
| entries_per_batch_  | int                 | TODO |
| write_options_      | WriteOptions        | TODO |
| reads_              | int                 | TODO |
| heap_counter_       | int                 | TODO |
| count_comparator_   | CountComparator     | TODO |
| total_thread_count_ | int                 | TODO |

### 构造函数

```cpp
Benchmark()
      : cache_(FLAGS_cache_size >= 0 ? NewLRUCache(FLAGS_cache_size) : nullptr),
        filter_policy_(FLAGS_bloom_bits >= 0
                           ? NewBloomFilterPolicy(FLAGS_bloom_bits)
                           : nullptr),
        db_(nullptr),
        num_(FLAGS_num),
        value_size_(FLAGS_value_size),
        entries_per_batch_(1),
        reads_(FLAGS_reads < 0 ? FLAGS_num : FLAGS_reads),
        heap_counter_(0),
        count_comparator_(BytewiseComparator()),
        total_thread_count_(0) {
    std::vector<std::string> files;
    g_env->GetChildren(FLAGS_db, &files);
    for (size_t i = 0; i < files.size(); i++) {
      if (Slice(files[i]).starts_with("heap-")) {
        g_env->RemoveFile(std::string(FLAGS_db) + "/" + files[i]);
      }
    }
    if (!FLAGS_use_existing_db) {
      DestroyDB(FLAGS_db, Options());
    }
  }
```
简单的做了些清理工作

### `Run()`函数

```cpp
void Run() {
  PrintHeader(); // 打印环境信息
  Open(); // 打开数据库

  const char* benchmarks = FLAGS_benchmarks;

  // 执行每一个测试
  while (benchmarks != nullptr) {
    const char* sep = strchr(benchmarks, ',');
    Slice name;
    if (sep == nullptr) {
      name = benchmarks;
      benchmarks = nullptr;
    } else {
      name = Slice(benchmarks, sep - benchmarks);
      benchmarks = sep + 1;
    }

    // Reset parameters that may be overridden below
    num_ = FLAGS_num;
    reads_ = (FLAGS_reads < 0 ? FLAGS_num : FLAGS_reads);
    value_size_ = FLAGS_value_size;
    entries_per_batch_ = 1;
    write_options_ = WriteOptions();
    /**
     *声明了一个名为method的指针，这个指针指向Benchmark类的一个成员函数，
      *这个成员函数的参数是ThreadState类型的指针，返回类型是void。
    */
    void (Benchmark::*method)(ThreadState*) = nullptr;
    bool fresh_db = false;
    int num_threads = FLAGS_threads;

    // 设置benchmark方法
    ...

    // 如果需要刷新数据库，删除数据库重新打开
    if (fresh_db) {
      if (FLAGS_use_existing_db) {
        std::fprintf(stdout, "%-12s : skipped (--use_existing_db is true)\n",
                      name.ToString().c_str());
        method = nullptr;
      } else {
        delete db_;
        db_ = nullptr;
        DestroyDB(FLAGS_db, Options());
        Open();
      }
    }
    // 根据method 执行对应的Benchmark
    if (method != nullptr) {
      RunBenchmark(num_threads, name, method);
    }
  }
}
```

TODO: 分析每个Benchmark具体的执行逻辑

