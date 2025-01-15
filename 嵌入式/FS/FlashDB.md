# 参考链接

[FlashDB/README_zh.md at master · armink/FlashDB (github.com)](https://github.com/armink/FlashDB/blob/master/README_zh.md)

# 基本概念

- **键值数据库（KVDB）**：是一种非关系数据库，它将数据存储为键值（Key-Value）对集合，其中键作为唯一标识符。KVDB 操作简洁，可扩展性强。
- **时序数据（TSDB）** ：时间序列数据库 （Time Series Database , 简称 TSDB），它将数据按照 **时间顺序存储** 。TSDB 数据具有时间戳，数据存储量大，插入及查询性能高。
- **时序记录（TSL）** ：TSL (Time series log)，是 TSDB 中每条记录的简称。
- **Blob** ：在计算机中，blob 常常是数据库中用来存储二进制文件的字段类型。在 FlashDB 中， KV 和 TSL 都使用 blob 类型来存储，该类型可以兼容任意变量类型。
- **迭代器（iterator）**：它可以让用户透过特定的接口巡访容器中的每一个元素，而不用了解底层的实现。 TSDB 和 KVDB 都支持通过迭代器对数据库进行遍历访问。

![框架结构](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/25_10_24_50_202412251024824.png)







# KVDB

## 字符串类型

使用一个名为 `"temp"` 的 KV 来存储温度值，分别演示了字符串 KV 从 `创建->读取->修改->删除` 的全过程。大致内容如下：

```c
void kvdb_type_string_sample(fdb_kvdb_t kvdb)
{
    FDB_INFO("==================== kvdb_type_string_sample ====================\n");
    
    { /* CREATE new Key-Value */
        char temp_data[10] = "36C";
        /* It will create new KV node when "temp" KV not in database. */
        fdb_kv_set(kvdb, "temp", temp_data);
        FDB_INFO("create the 'temp' string KV, value is: %s\n", temp_data);
    }
    
    { /* GET the KV value */
        char *return_value, temp_data[10] = { 0 };
        /* Get the "temp" KV value.
         * NOTE: The return value saved in fdb_kv_get's buffer. Please copy away as soon as possible.
         */
        return_value = fdb_kv_get(kvdb, "temp");
        /* the return value is NULL when get the value failed */
        if (return_value != NULL) {
            strncpy(temp_data, return_value, sizeof(temp_data));
            FDB_INFO("get the 'temp' value is: %s\n", temp_data);
        }
    }
    
    { /* CHANGE the KV value */
        char temp_data[10] = "38C";
        /* change the "temp" KV's value to "38C" */
        fdb_kv_set(kvdb, "temp", temp_data);
        FDB_INFO("set 'temp' value to %s\n", temp_data);
    }
    
    { /* DELETE the KV by name */
        fdb_kv_del(kvdb, "temp");
        FDB_INFO("delete the 'temp' finish\n");
    }
    
    FDB_INFO("===========================================================\n");
}
```

- 首先创建了一个 KV 名为 `"temp"` ，并给予初值 36℃
- 读取 `"temp"` KV 当前的值，发现与初值相同
- 修改 `"temp"` KV 的值为 38℃
- 最后删除 `"temp"` KV

## Blob类型

blob KV 是一个比较常用类型，其 value 是一个没有类型限制的二进制类型。在功能上，blob KV 也兼容字符串 KV 。在 API 的使用上， blob KV 拥有一套独立的 API ，可以很快速的实现各种类型 KV 到 KVDB 中的存储，比如：基本类型，数组以及结构体等。

下面的代码使用一个名为 `"temp"` 的 KV 来存储温度值，分别演示了 blob KV 从 `创建->读取->修改->删除` 的全过程。

```c
void kvdb_type_blob_sample(fdb_kvdb_t kvdb)
{
    struct fdb_blob blob;
    
    FDB_INFO("==================== kvdb_type_blob_sample ====================\n");
    
    { /* CREATE new Key-Value */
        int temp_data = 36;
        /* It will create new KV node when "temp" KV not in database.
         * fdb_blob_make: It's a blob make function, and it will return the blob when make finish.
         */
        fdb_kv_set_blob(kvdb, "temp", fdb_blob_make(&blob, &temp_data, sizeof(temp_data)));
        FDB_INFO("create the 'temp' blob KV, value is: %d\n", temp_data);
    }
    
    { /* GET the KV value */
        int temp_data = 0;
        /* get the "temp" KV value */
        fdb_kv_get_blob(kvdb, "temp", fdb_blob_make(&blob, &temp_data, sizeof(temp_data)));
        /* the blob.saved.len is more than 0 when get the value successful */
        if (blob.saved.len > 0) {
            FDB_INFO("get the 'temp' value is: %d\n", temp_data);
        }
    }

    { /* CHANGE the KV value */
        int temp_data = 38;
        /* change the "temp" KV's value to 38 */
        fdb_kv_set_blob(kvdb, "temp", fdb_blob_make(&blob, &temp_data, sizeof(temp_data)));
        FDB_INFO("set 'temp' value to %d\n", temp_data);
    }
    
    { /* DELETE the KV by name */
        fdb_kv_del(kvdb, "temp");
        FDB_INFO("delete the 'temp' finish\n");
    }
    
    FDB_INFO("===========================================================\n");
}
```

- 首先创建了一个 KV 名为 `"temp"` ，并给予初值 36℃
- 读取 `"temp"` KV 当前的值，发现与初值相同
- 修改 `"temp"` KV 的值为 38℃
- 最后删除 `"temp"` KV

## 遍历（迭代器）

下面的示例代码中，首先初始化了 KVDB 的迭代器，然后使用迭代器 API ，将 KVDB 的所有 KV 逐一遍历出来。

遍历出来的 KV 对象含有 KV 的一些属性，包括：key name, value saved addr, value length 等，用户通过 `fdb_blob_read` 配合 `fdb_kv_to_blob` 读取出来，做一些自己的业务处理。

```c
void kvdb_tarversal_sample(fdb_kvdb_t kvdb)
{
    struct fdb_kv_iterator iterator;
    fdb_kv_t cur_kv;
    struct fdb_blob blob;
    size_t data_size;
    uint8_t *data_buf;
    
    fdb_kv_iterator_init(kvdb, &iterator);
    
    while (fdb_kv_iterate(kvdb, &iterator)) {
        cur_kv = &(iterator.curr_kv);
        data_size = (size_t) cur_kv->value_len;
        data_buf = (uint8_t *) malloc(data_size);
        if (data_buf == NULL) {
            FDB_INFO("Error: malloc failed.\n");
            break;
        }
        fdb_blob_read((fdb_db_t) kvdb, fdb_kv_to_blob(cur_kv, fdb_blob_make(&blob, data_buf, data_size)));
        /*
         * balabala do what ever you like with blob...
         */
        free(data_buf);
    }
}
```

# TSDB



```c
void tsdb_sample(fdb_tsdb_t tsdb)
{
    struct fdb_blob blob;
    
    FDB_INFO("==================== tsdb_sample ====================\n");
    
    { /* APPEND new TSL (time series log) */
        struct env_status status;
        /* append new log to TSDB */
        status.temp = 36;
        status.humi = 85;
        fdb_tsl_append(tsdb, fdb_blob_make(&blob, &status, sizeof(status)));
        FDB_INFO("append the new status.temp (%d) and status.humi (%d)\n", status.temp, status.humi);
        status.temp = 38;
        status.humi = 90;
        fdb_tsl_append(tsdb, fdb_blob_make(&blob, &status, sizeof(status)));
        FDB_INFO("append the new status.temp (%d) and status.humi (%d)\n", status.temp, status.humi);
    }
    
    { /* QUERY the TSDB */
        /* query all TSL in TSDB by iterator */
        fdb_tsl_iter(tsdb, query_cb, tsdb);
    }
    
    { /* QUERY the TSDB by time */
        /* prepare query time (from 1970-01-01 00:00:00 to 2020-05-05 00:00:00) */
        struct tm tm_from = { .tm_year = 1970 - 1900, 
                             .tm_mon = 0, 
                             .tm_mday = 1, 
                             .tm_hour = 0, 
                             .tm_min = 0, 
                             .tm_sec = 0 
                            };
        struct tm tm_to = { .tm_year = 2020 - 1900, 
                           .tm_mon = 4, 
                           .tm_mday = 5, 
                           .tm_hour = 0, 
                           .tm_min = 0, 
                           .tm_sec = 0 
                          };
        time_t from_time = mktime(&tm_from), to_time = mktime(&tm_to);
        size_t count;
        /* query all TSL in TSDB by time */
        fdb_tsl_iter_by_time(tsdb, from_time, to_time, query_by_time_cb, tsdb);
        /* query all FDB_TSL_WRITE status TSL's count in TSDB by time */
        count = fdb_tsl_query_count(tsdb, from_time, to_time, FDB_TSL_WRITE);
        FDB_INFO("query count is: %u\n", count);
    }
    
    { /* SET the TSL status */
        /* Change the TSL status by iterator or time iterator
         * set_status_cb: the change operation will in this callback
         *
         * NOTE: The actions to modify the state must be in orderC.
         *       like: FDB_TSL_WRITE -> FDB_TSL_USER_STATUS1 -> FDB_TSL_DELETED -> FDB_TSL_USER_STATUS2
         *       The intermediate states can also be ignored.
         *       such as: FDB_TSL_WRITE -> FDB_TSL_DELETED
         */
        fdb_tsl_iter(tsdb, set_status_cb, tsdb);
    }
    
    FDB_INFO("===========================================================\n");
}
```

分别来看下这几个过程

- **追加**：分两次修改结构体对象 `status` 的值然后追加到 TSDB 中；

- **查询**：通过 TSDB 的迭代器 API ，在每次迭代时会自动执行 `query_cb` 回调函数，实现对 TSDB 中所有记录的查询，回调函数内容如下：

  ```c
  static bool query_cb(fdb_tsl_t tsl, void *arg)
  {
      struct fdb_blob blob;
      struct env_status status;
      fdb_tsdb_t db = arg;
      fdb_blob_read((fdb_db_t) db, fdb_tsl_to_blob(tsl, fdb_blob_make(&blob, &status, sizeof(status))));
      FDB_INFO("[query_cb] queried a TSL: time: %ld, temp: %d, humi: %d\n", tsl->time, status.temp, status.humi);
      return false;
  }
  ```

- **按时间查询**：TSDB 还提供了按时间迭代的 API : `fdb_tsl_iter_by_time` ，可以传入起始和截至时间，此时迭代器会按照传入的时间段，对时序记录进行迭代。每次迭代时会执行 `query_by_time_cb` 回调，在回调中读取当前记录的内容，并打印出来。

  ```c
  static bool query_by_time_cb(fdb_tsl_t tsl, void *arg)
  {
      struct fdb_blob blob;
      struct env_status status;
      fdb_tsdb_t db = arg;
      fdb_blob_read((fdb_db_t) db, fdb_tsl_to_blob(tsl, fdb_blob_make(&blob, &status, sizeof(status))));
      FDB_INFO("[query_by_time_cb] queried a TSL: time: %ld, temp: %d, humi: %d\n", tsl->time, status.temp, status.humi);
      return false;
  }
  ```

- **修改状态**：每条 TSL 在被追加到 TSDB 后，都可以修改其状态，状态共有 4 种：

  - `FDB_TSL_WRITE`：已写入状态，TSL 被追加到 TSDB 中后的默认状态；

  - `FDB_TSL_USER_STATUS1`：该状态介于写入与删除之间，用户可自定义其状态含义，比如：数据已被同步至云端；

  - `FDB_TSL_DELETED` ：已删除状态，当 TSL 需要删除时，修改 TSL 的状态为该状态即可；

    > 提示：在 FlashDB 中，为了提升 Flash 寿命，删除动作并不会真正的将数据从 Flash 从擦除，而是将其标记为删除状态，用户可以通过状态对不同的数据记录进行区分。

  - `FDB_TSL_USER_STATUS2`：删除状态之后的自定义状态，预留给用户使用；

  修改状态时只能按照 `FDB_TSL_WRITE -> FDB_TSL_USER_STATUS1 -> FDB_TSL_DELETED -> FDB_TSL_USER_STATUS2` 顺序进行修改，不能逆序修改。也可以跳过中间状态，例如：从 `FDB_TSL_WRITE` 直接修改为`FDB_TSL_DELETED` 状态，跳过 `FDB_TSL_USER_STATUS1` 状态。

  示例中通过迭代器将当前所有的 TSL 都修改为 `FDB_TSL_USER_STATUS1` 状态，迭代器中的回调代码如下：

  ```c
  static bool set_status_cb(fdb_tsl_t tsl, void *arg)
  {
      fdb_tsdb_t db = arg;
      FDB_INFO("set the TSL (time %ld) status from %d to %d\n", tsl->time, tsl->status, FDB_TSL_USER_STATUS1);
      fdb_tsl_set_status(db, tsl, FDB_TSL_USER_STATUS1);
      return false;
  }
  ```

# 移植

## 介绍

![框架图](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/12/25_13_44_14_202412251344836.png)



移植的主要工作都在 FAL 这边，其他接口并不是强依赖，可以根据自己的情况进行对接。

移植前建议先了解下 FAL 功能介绍，详见：http://packages.rt-thread.org/detail.html?package=fal

FAL 底层将不同的 Flash 存储介质进行了统一封装，并提供了分区表机制，暴露给上层用户。

FlashDB 的每个数据库就是基于 FAL 提供的分区机制，每个数据库都坐落在某个 FAL 的分区上，相当于一个分区对应一个数据库。

## 定义Flash设备

在定义 Flash 设备表前，需要先定义 Flash 设备。可以是片内 flash, 也可以是片外基于 SFUD 的 spi flash：

- 定义片内 flash 设备可以参考 [`fal_flash_stm32f2_port.c`](https://github.com/RT-Thread-packages/fal/blob/master/samples/porting/fal_flash_stm32f2_port.c) 。
- 定义片外 spi flash 设备可以参考 [`fal_flash_sfud_port.c`](https://github.com/RT-Thread-packages/fal/blob/master/samples/porting/fal_flash_sfud_port.c) 。

定义具体的 Flash 设备对象，用户需要根据自己的 Flash 情况分别实现 `init`、 `read`、 `write`、 `erase` 这些操作函数：

- `static int init(void)`：**可选** 的初始化操作。
- `static int read(long offset, uint8_t *buf, size_t size)`：读取操作。

| 参数   | 描述                      |
| :----- | :------------------------ |
| offset | 读取数据的 Flash 偏移地址 |
| buf    | 存放待读取数据的缓冲区    |
| size   | 待读取数据的大小          |
| return | 返回实际读取的数据大小    |

- `static int write(long offset, const uint8_t *buf, size_t size)` ：写入操作。

| 参数   | 描述                      |
| :----- | :------------------------ |
| offset | 写入数据的 Flash 偏移地址 |
| buf    | 存放待写入数据的缓冲区    |
| size   | 待写入数据的大小          |
| return | 返回实际写入的数据大小    |

- `static int erase(long offset, size_t size)` ：擦除操作。

| 参数   | 描述                      |
| :----- | :------------------------ |
| offset | 擦除区域的 Flash 偏移地址 |
| size   | 擦除区域的大小            |
| return | 返回实际擦除的区域大小    |

用户需要根据自己的 Flash 情况分别实现这些操作函数。在文件最底部定义了具体的 Flash 设备对象 ，如下示例定义了 stm32f2 片上 flash：stm32f2_onchip_flash

```c
const struct fal_flash_dev stm32f2_onchip_flash =
{
    .name       = "stm32_onchip",
    .addr       = 0x08000000,
    .len        = 1024*1024,
    .blk_size   = 128*1024,
    .ops        = {init, read, write, erase},
    .write_gran = 8
};
```

- `"stm32_onchip"` : Flash 设备的名字。
- `0x08000000`: 对 Flash 操作的起始地址。
- `1024*1024`：Flash 的总大小（1MB）。
- `128*1024`：Flash 块/扇区大小（因为 STM32F2 各块大小不均匀，所以擦除粒度为最大块的大小：128K）。
- `{init, read, write, erase}` ：Flash 的操作函数。 如果没有 init 初始化过程，第一个操作函数位置可以置空。
- `8` : 设置写粒度，单位 bit， 0 表示未生效（默认值为 0 ），该成员是 fal 版本大于 0.4.0 的新增成员。各个 flash 写入粒度不尽相同，可通过该成员进行设置，以下列举几种常见 Flash 写粒度：
  - nor flash: 1 bit
  - stm32f2/f4: 8 bit
  - stm32f1: 32 bit
  - stm32l4: 64 bit

## 定义Flash设备表

Flash 设备表定义在 `fal_cfg.h` 头文件中，定义分区表前需 **新建 `fal_cfg.h` 文件** ，请将该文件统一放在对应 BSP 或工程目录的 port 文件夹下，并将该头文件路径加入到工程。fal_cfg.h 可以参考 [示例文件 fal/samples/porting/fal_cfg.h](https://github.com/RT-Thread-packages/fal/blob/master/samples/porting/fal_cfg.h) 完成。

设备表示例：

```c
/* ===================== Flash device Configuration ========================= */
extern const struct fal_flash_dev stm32f2_onchip_flash;
extern struct fal_flash_dev nor_flash0;
/* flash device table */
#define FAL_FLASH_DEV_TABLE                                          \
{                                                                    \
    &stm32f2_onchip_flash,                                           \
    &nor_flash0,                                                     \
}
```

Flash 设备表中，有两个 Flash 对象，一个为 STM32F2 的片内 Flash ，一个为片外的 Nor Flash。



## 定义Flash分区表

分区表也定义在 `fal_cfg.h` 头文件中。Flash 分区基于 Flash 设备，每个 Flash 设备又可以有 N 个分区，这些分区的集合就是分区表。在配置分区表前，务必保证已定义好 **Flash 设备** 及 **设备表**。fal_cfg.h 可以参考 [示例文件 fal/samples/porting/fal_cfg.h](https://github.com/RT-Thread-packages/fal/blob/master/samples/porting/fal_cfg.h) 完成。

分区表示例：

```c
#define NOR_FLASH_DEV_NAME             "norflash0"
/* ====================== Partition Configuration ========================== */
#ifdef FAL_PART_HAS_TABLE_CFG
/* partition table */
#define FAL_PART_TABLE                                                               \
{                                                                                    \
    {FAL_PART_MAGIC_WORD,        "bl",     "stm32_onchip",         0,   64*1024, 0}, \
    {FAL_PART_MAGIC_WORD,       "app",     "stm32_onchip",   64*1024,  704*1024, 0}, \
    {FAL_PART_MAGIC_WORD, "easyflash", NOR_FLASH_DEV_NAME,         0, 1024*1024, 0}, \
    {FAL_PART_MAGIC_WORD,  "download", NOR_FLASH_DEV_NAME, 1024*1024, 1024*1024, 0}, \
}
#endif /* FAL_PART_HAS_TABLE_CFG */
```

上面这个分区表详细描述信息如下：

| 分区名      | Flash 设备名   | 偏移地址  | 大小  | 说明               |
| :---------- | :------------- | :-------- | :---- | :----------------- |
| “bl”        | “stm32_onchip” | 0         | 64KB  | 引导程序           |
| “app”       | “stm32_onchip” | 64*1024   | 704KB | 应用程序           |
| “easyflash” | “norflash0”    | 0         | 1MB   | EasyFlash 参数存储 |
| “download”  | “norflash0”    | 1024*1024 | 1MB   | OTA 下载区         |

用户需要修改的分区参数包括：分区名称、关联的 Flash 设备名、偏移地址（相对 Flash 设备内部）、大小，需要注意以下几点：

- 分区名保证 **不能重复**；
- 关联的 Flash 设备 **务必已经在 Flash 设备表中定义好** ，并且 **名称一致** ，否则会出现无法找到 Flash 设备的错误；
- 分区的起始地址和大小 **不能超过 Flash 设备的地址范围** ，否则会导致包初始化错误；

> [!tip]
>
> 注意：每个分区定义时，除了填写上面介绍的参数属性外，需在前面增加 `FAL_PART_MAGIC_WORD` 属性，末尾增加 `0` （目前用于保留功能）

# 配置

FlashDB 的使用时，可以通过 fdb_cfg.h 对其进行功能配置。

1. FDB_USING_KVDB：使能 KVDB 功能

2. FDB_KV_AUTO_UPDATE：使能 KV 自动升级功能。该功能使能后， `fdb_kvdb.ver_num` 存储了当前数据库的版本，如果版本发生变化时，会自动触发升级动作，将更新新的默认 KV 集合至当前数据库中。

3. FDB_USING_TSDB:使能 TSDB 功能

4. FDB_USING_FAL_MODE:使能 FAL 模式，FAL 里的分区用于存储数据库。该模式下，FlashDB 直接操作 Flash，所以性能较好

5. FDB_USING_FILE_POSIX_MODE：使用 POSIX 的文件模式，需要系统提供 open/read/write/close 相关文件访问接口。

6. FDB_USING_FILE_LIBC_MODE：使用 C 标准库的文件模式，需要系统提供 fopen/fread/fwrite/fclose 相关文件访问接口。

   > [!tip]
   >
   > FDB_USING_FILE_LIBC_MODE 与 FDB_USING_FILE_POSIX_MODE 模式只能二选一。相比 FAL 模式，文件模式下数据库的存储位置、大小及数量没有限制。

7. FDB_WRITE_GRAN:Flash 写粒度，单位为 bit。目前支持：

   - 1: nor flash
   - 8: stm32f2/f4 片上 Flash
   - 32: stm32f1 片上 Flash

   > [!tip] 
   >
   > 如果数据库中使用了多种 Flash 规格，例如：既有 nor flash，也有 stm32f4 片上 Flash ，此时取最大值作为配置项，即：8 bit

8. FDB_BIG_ENDIAN：MCU 大小端配置，默认不配置时，系统自动使用小端配置

9. FDB_PRINT(…)：打印函数宏定义配置，默认不配置时，使用 printf 作为打印日志是输出函数。用户也可以自定义新的打印函数宏定义，例如：

   ```c
   #define FDB_PRINT(...)              my_printf(__VA_ARGS__)
   ```

10. FDB_DEBUG_ENABLE：使能调试信息输出。关闭该配置时，系统将不会输出用于调试的日志。

# API

## Blob

### 构造 blob 对象

构造 blob 对象的过程，其内部是对 blob 结构体初始化赋值的过程，将传入的参数写入 blob 结构体中，并进行返回

```c
fdb_blob_t fdb_blob_make(fdb_blob_t blob, const void *value_buf, size_t buf_len)
```

| 参数      | 描述                   |
| :-------- | :--------------------- |
| blob      | blob 初始对象          |
| value_buf | 存放数据的缓冲区       |
| buf_len   | 缓冲区的大小           |
| 返回      | 创建完成后的 blob 对象 |

### 读取 blob 数据

通过 KVDB 和 TSDB 的 API 可以返回 blob 对象，此时返回的 blob 对象里存放了 blob 数据的存储地址。该 API 可以将数据库里存放的 blob 数据读取出来，并存放至 `blob->buf` 。

```c
size_t fdb_blob_read(fdb_db_t db, fdb_blob_t blob)
```

| 参数 | 描述                       |
| :--- | :------------------------- |
| db   | 数据库对象                 |
| blob | blob 对象                  |
| 返回 | 实际读取到的 blob 数据长度 |

## KVDB

### 初始化 KVDB

```c
fdb_err_t fdb_kvdb_init(fdb_kvdb_t db, const char *name, const char *path, struct fdb_default_kv *default_kv, void *user_data)
```

| 参数       | 描述                                                     |
| :--------- | :------------------------------------------------------- |
| db         | 数据库对象                                               |
| name       | 数据库名称                                               |
| path       | FAL 模式：分区表中的分区名，文件模式：数据库保存的路径   |
| default_kv | 默认 KV 集合，第一次初始化时，将会把默认 KV 写入数据库中 |
| user_data  | 用户自定义数据，没有时传入 NULL                          |
| 返回       | 错误码                                                   |

### 控制 KVDB

通过命令控制字，用户可以对数据库进行一些控制操作

```c
void fdb_kvdb_control(fdb_kvdb_t db, int cmd, void *arg)
```

| 参数 | 描述       |
| :--- | :--------- |
| db   | 数据库对象 |
| cmd  | 命令控制字 |
| arg  | 控制的参数 |
| 返回 | 错误码     |

支持的命令控制字如下：

```c
#define FDB_KVDB_CTRL_SET_SEC_SIZE     0x00             /**< 设置扇区大小，需要在数据库初始化前配置 */
#define FDB_KVDB_CTRL_GET_SEC_SIZE     0x01             /**< 获取扇区大小 */
#define FDB_KVDB_CTRL_SET_LOCK         0x02             /**< 设置加锁函数 */
#define FDB_KVDB_CTRL_SET_UNLOCK       0x03             /**< 设置解锁函数 */
#define FDB_KVDB_CTRL_SET_FILE_MODE    0x09             /**< 设置文件模式，需要在数据库初始化前配置 */
#define FDB_KVDB_CTRL_SET_MAX_SIZE     0x0A             /**< 在文件模式下，设置数据库最大大小，需要在数据库初始化前配置 */
#define FDB_KVDB_CTRL_SET_NOT_FORMAT   0x0B             /**< 设置初始化时不进行格式化，需要在数据库初始化前配置 */
```

#### 扇区大小与块大小

FlashDB 内部存储结构由 N 个扇区组成，每次格式化时是以扇区作为最小单位。而一个扇区通常是 Flash 块大小的 N 倍，比如： Nor Flash 的块大小一般为 4096。

默认 KVDB 会使用 1倍 的块大小作为扇区大小，即：4096。此时，该 KVDB 无法存入超过 4096 长度的 KV 。如果想要存入比如：10K 长度的 KV ，可以通过 control 函数，设置扇区大小为 12K，或者更大大小即可。

### 反初始化 KVDB

```c
fdb_err_t fdb_kvdb_deinit(fdb_kvdb_t db)
```

| 参数 | 描述       |
| :--- | :--------- |
| db   | 数据库对象 |

### 设置 KV

使用此方法可以实现对 KV 的增加和修改功能。

- **增加** ：当 KVDB 中不存在该名称的 KV 时，则会执行新增操作；

- **修改** ：入参中的 KV 名称在当前 KVDB 中存在，则把该 KV 值修改为入参中的值；

  > 在 KVDB 内部实现中，修改 KV 会先删除旧的 KV ，再增加新的 KV，所以修改后数据库剩余容量会变小

通过 KV 的名字来获取其对应的值。支持两种接口

#### 设置 blob 类型KV

```c
fdb_err_t fdb_kv_set_blob(fdb_kvdb_t db, const char *key, fdb_blob_t blob)
```

| 参数 | 描述                        |
| :--- | :-------------------------- |
| db   | 数据库对象                  |
| key  | KV 的名称                   |
| blob | blob 对象，做为 KV 的 value |
| 返回 | 错误码                      |

示例：

```c
struct fdb_blob blob;
int temp_data = 36;
/* 通过 fdb_blob_make 构造 blob 对象，作为 "temp" KV 的 value */
fdb_kv_set_blob(kvdb, "temp", fdb_blob_make(&blob, &temp_data, sizeof(temp_data)));
```



#### 设置 string 类型KV

```c
fdb_err_t fdb_kv_set(fdb_kvdb_t db, const char *key, const char *value)
```

| 参数  | 描述        |
| :---- | :---------- |
| db    | 数据库对象  |
| key   | KV 的名称   |
| value | KV 的 value |
| 返回  | 错误码      |

### 获取 KV

#### 获取 blob 类型 KV

```c
size_t fdb_kv_get_blob(fdb_kvdb_t db, const char *key, fdb_blob_t blob)
```

| 参数 | 描述                                  |
| :--- | :------------------------------------ |
| db   | 数据库对象                            |
| key  | KV 的名称                             |
| blob | 通过 blob 对象，返回 KV 的 blob value |
| 返回 | 错误码                                |

示例：

```c
struct fdb_blob blob;
int temp_data = 0;
/* 构造 blob 对象，用于存储 get 回来的 "temp" KV 的 value 数据 */
fdb_kv_get_blob(kvdb, "temp", fdb_blob_make(&blob, &temp_data, sizeof(temp_data)));
/* 如果有需要，可以检查 blob.saved.len 是否大于 0 ，确保 get 成功 */
if (blob.saved.len > 0) {
    FDB_INFO("get the 'temp' value is: %d\n", temp_data);
}
```

#### 获取 KV 对象

与 `fdb_kv_get_blob` API 不同，该 API 在 get 过程中并不会执行 value 数据的读取动作。返回的 KV 对象中存放了读取回来的 KV 属性，该 API 适用于 value 长度不确定，或 value 长度过长，需要分段读取的场景。

```c
fdb_kv_t fdb_kv_get_obj(fdb_kvdb_t db, const char *key, fdb_kv_t kv)
```

| 参数 | 描述                                                         |
| :--- | :----------------------------------------------------------- |
| db   | 数据库对象                                                   |
| key  | KV 的名称                                                    |
| kv   | 通过 KV 对象，返回 KV 的属性，可以再用 `fdb_kv_to_blob` 转换为 blob 对象，再进行数据读取 |
| 返回 | 错误码                                                       |

#### 获取字符串类型 KV

**注意** ：

- 该函数不允许连续使用，使用时需使用 strdup 包裹，确保每次返回回来的字符串内存空间独立；
- 该函数不支持可重入，返回的值位于函数内部缓冲区，出于安全考虑，请加锁保护。

```c
char *fdb_kv_get(fdb_kvdb_t db, const char *key)
```

| 参数 | 描述                                |
| :--- | :---------------------------------- |
| db   | 数据库对象                          |
| key  | KV 的名称                           |
| 返回 | !=NULL: KV 的 value；NULL: 获取失败 |

### 删除 KV

> 在 KVDB 内部实现中，删除 KV 并不会完全从 KVDB 中移除，而是标记为了删除状态，所以删除后数据库剩余容量不会有变化

```c
fdb_err_t fdb_kv_del(fdb_kvdb_t db, const char *key)
```

| 参数 | 描述       |
| :--- | :--------- |
| db   | 数据库对象 |
| key  | KV 的名称  |
| 返回 | 错误码     |

### 重置 KVDB

将 KVDB 中的 KV 重置为 **首次初始时** 的默认值

```c
fdb_err_t fdb_kv_set_default(fdb_kvdb_t db)
```

| 参数 | 描述       |
| :--- | :--------- |
| db   | 数据库对象 |
| 返回 | 错误码     |

### 打印 KVDB 中的 KV 信息

```c
void fdb_kv_print(fdb_kvdb_t db)
```

| 参数 | 描述       |
| :--- | :--------- |
| db   | 数据库对象 |
| 返回 | 错误码     |

### KV 对象转换为 blob 对象

```c
fdb_blob_t fdb_kv_to_blob(fdb_kv_t kv, fdb_blob_t blob)
```

| 参数 | 描述               |
| :--- | :----------------- |
| kv   | 待转换的 KV 对象   |
| blob | 转换前的 blob 对象 |
| 返回 | 转换后的 blob 对象 |

### 初始化 KV 迭代器

```c
fdb_kv_iterator_t fdb_kv_iterator_init(fdb_kvdb_t db, fdb_kv_iterator_t itr)
```

| 参数 | 描述                 |
| :--- | :------------------- |
| db   | 数据库对象           |
| itr  | 待初始化的迭代器对象 |
| 返回 | 初始化后的迭代器对象 |

### 迭代 KV

使用该迭代器 API，可以遍历整个 KVDB 中的所有 KV。

> [!important]
>
> **注意**：使用前请先初始化迭代器



```c
bool fdb_kv_iterate(fdb_kvdb_t db, fdb_kv_iterator_t itr)
```

## TSDB

### 初始化 TSDB

```c
fdb_err_t fdb_tsdb_init(fdb_tsdb_t db, const char *name, const char *path, fdb_get_time get_time, size_t max_len, void *user_data)
```

| 参数      | 描述                                                   |
| :-------- | :----------------------------------------------------- |
| db        | 数据库对象                                             |
| name      | 数据库名称                                             |
| path      | FAL 模式：分区表中的分区名，文件模式：数据库保存的路径 |
| get_time  | 获取当前时间戳的函数                                   |
| max_len   | 每条 TSL 的最大长度                                    |
| user_data | 用户自定义数据，没有时传入 NULL                        |
| 返回      | 错误码                                                 |

### 控制 TSDB

通过命令控制字，用户可以对数据库进行一些控制操作

```c
void fdb_tsdb_control(fdb_tsdb_t db, int cmd, void *arg)
```



| 参数 | 描述       |
| :--- | :--------- |
| db   | 数据库对象 |
| cmd  | 命令控制字 |
| arg  | 控制的参数 |
| 返回 | 错误码     |

支持的命令控制字如下：

```c
#define FDB_TSDB_CTRL_SET_SEC_SIZE     0x00             /**< 设置扇区大小，需要在数据库初始化前配置 */
#define FDB_TSDB_CTRL_GET_SEC_SIZE     0x01             /**< 获取扇区大小 */
#define FDB_TSDB_CTRL_SET_LOCK         0x02             /**< 设置加锁函数 */
#define FDB_TSDB_CTRL_SET_UNLOCK       0x03             /**< 设置解锁函数 */
#define FDB_TSDB_CTRL_SET_ROLLOVER     0x04             /**< 设置是否滚动写入，默认滚动。设置非滚动时，如果数据库写满，无法再追加写入。需要在数据库初始化后配置 */
#define FDB_TSDB_CTRL_GET_ROLLOVER     0x05             /**< 获取是否滚动写入 */
#define FDB_TSDB_CTRL_GET_LAST_TIME    0x06             /**< 获取上次追加 TSL 时的时间戳  */
#define FDB_TSDB_CTRL_SET_FILE_MODE    0x09             /**< 设置文件模式，需要在数据库初始化前配置，需要在数据库初始化前配置 */
#define FDB_TSDB_CTRL_SET_MAX_SIZE     0x0A             /**< 在文件模式下，设置数据库最大大小，需要在数据库初始化前配置 */
#define FDB_TSDB_CTRL_SET_NOT_FORMAT   0x0B             /**< 设置初始化时不进行格式化，需要在数据库初始化前配置 */
```

### 反初始化 TSDB

```c
fdb_err_t fdb_tsdb_deinit(fdb_tsdb_t db)
```

| 参数 | 描述       |
| :--- | :--------- |
| db   | 数据库对象 |

### 追加 TSL

对于 TSDB ，新增 TSL 的过程，就是往 TSDB 末尾追加新 TSL 的过程

```c
fdb_err_t fdb_tsl_append(fdb_tsdb_t db, fdb_blob_t blob)
```

| 参数 | 描述                       |
| :--- | :------------------------- |
| db   | 数据库对象                 |
| blob | blob 对象，做为 TSL 的数据 |
| 返回 | 错误码                     |

### 迭代 TSL

遍历整个 TSDB 并执行迭代回调

```c
void fdb_tsl_iter(fdb_tsdb_t db, fdb_tsl_cb cb, void *cb_arg)
```

| 参数   | 描述                                    |
| :----- | :-------------------------------------- |
| db     | 数据库对象                              |
| cb     | 回调函数，每次遍历到 TSL 时会执行该回调 |
| cb_arg | 回调函数的参数                          |
| 返回   | 错误码                                  |

### 逆序迭代 TSL

逆序遍历整个 TSDB 并执行迭代回调

```c
void fdb_tsl_iter_reverse(fdb_tsdb_t db, fdb_tsl_cb cb, void *cb_arg)
```

| 参数   | 描述                                    |
| :----- | :-------------------------------------- |
| db     | 数据库对象                              |
| cb     | 回调函数，每次遍历到 TSL 时会执行该回调 |
| cb_arg | 回调函数的参数                          |
| 返回   | 错误码                                  |

### 按时间段迭代 TSL

按时间段范围，遍历整个 TSDB 并执行迭代回调

```c
void fdb_tsl_iter_by_time(fdb_tsdb_t db, fdb_time_t from, fdb_time_t to, fdb_tsl_cb cb, void *cb_arg)
```

| 参数   | 描述                                                         |
| :----- | :----------------------------------------------------------- |
| db     | 数据库对象                                                   |
| from   | 开始时间戳。如果结束时间戳比开始时间戳要小，此时将会执行逆序迭代。 |
| to     | 结束时间戳                                                   |
| cb     | 回调函数，每次遍历到 TSL 时会执行该回调                      |
| cb_arg | 回调函数的参数                                               |
| 返回   | 错误码                                                       |

### 查询 TSL 的数量

按照传入的时间段，查询符合状态的 TSL 数量 

```c
size_t fdb_tsl_query_count(fdb_tsdb_t db, fdb_time_t from, fdb_time_t to, fdb_tsl_status_t status)
```



| 参数   | 描述           |
| :----- | :------------- |
| db     | 数据库对象     |
| from   | 开始时间戳     |
| to     | 结束时间戳     |
| status | TSL 的状态条件 |
| 返回   | 数量           |

### 设置 TSL 状态

TSL 状态详见 `enum fdb_tsl_status` ，必须按照顺序设置 TSL 状态。

```c
fdb_err_t fdb_tsl_set_status(fdb_tsdb_t db, fdb_tsl_t tsl, fdb_tsl_status_t status)
```

| 参数   | 描述         |
| :----- | :----------- |
| db     | 数据库对象   |
| tsl    | TSL 对象     |
| status | TSL 的新状态 |
| 返回   | 错误码       |

### 清空 TSDB

```c
void fdb_tsl_clean(fdb_tsdb_t db)
```

| 参数 | 描述       |
| :--- | :--------- |
| db   | 数据库对象 |
| 返回 | 错误码     |

### TSL 对象转换为 blob 对象

```c
fdb_blob_t fdb_tsl_to_blob(fdb_tsl_t tsl, fdb_blob_t blob)
```

| 参数 | 描述               |
| :--- | :----------------- |
| tsl  | 待转换的 TSL 对象  |
| blob | 转换前的 blob 对象 |
| 返回 | 转换后的 blob 对象 |









































