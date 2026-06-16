# 大数据考试与项目答辩常用命令速查

本文档整理大数据考试、项目演示和答辩时可能用到的常用命令。  
项目默认目录：

```bash
~/MeiliDuolikun/lenta_final
```

项目使用的主要 HDFS 路径：

```text
/datasets/news_lenta.csv
/MeiliDuolikun/lenta_search/models/w2v_100k_v2
/MeiliDuolikun/lenta_search/index/docs_100k_v2_text
/MeiliDuolikun/lenta_search/stats/bm25_100k_v2
```

---

## 1. 进入项目目录

### 命令

```bash
cd ~/MeiliDuolikun/lenta_final
ls
```

### 含义

进入最终项目目录，并查看项目文件。

### 预期文件

```text
add_text.py
ask.sh
find.py
find.sh
train.py
train.sh
```

---

## 2. 查看 HDFS 数据集

### 查看 Lenta.ru 数据集是否存在

```bash
hdfs dfs -ls /datasets/news_lenta.csv
```

### 查看数据集大小

```bash
hdfs dfs -du -h /datasets/news_lenta.csv
```

### 查看数据前几行

```bash
hdfs dfs -cat /datasets/news_lenta.csv | head -5
```

### 含义

这些命令用于证明项目使用的是 bigdata 服务器 HDFS 中的 Lenta.ru 数据集。

---

## 3. 查看 HDFS 中保存的模型、索引和统计

### 查看 Word2Vec 模型

```bash
hdfs dfs -ls /MeiliDuolikun/lenta_search/models/w2v_100k_v2
```

### 查看文档索引

```bash
hdfs dfs -ls /MeiliDuolikun/lenta_search/index/docs_100k_v2_text
```

### 查看 BM25 统计

```bash
hdfs dfs -ls /MeiliDuolikun/lenta_search/stats/bm25_100k_v2
```

### 一次性查看三个目录

```bash
hdfs dfs -ls /MeiliDuolikun/lenta_search/models
hdfs dfs -ls /MeiliDuolikun/lenta_search/index
hdfs dfs -ls /MeiliDuolikun/lenta_search/stats
```

### 含义

这些命令用于答辩时证明：

```text
模型已经训练并保存到 HDFS
索引已经保存到 HDFS
BM25 统计已经保存到 HDFS
```

---

## 4. 运行文件查询

### 创建查询文件

```bash
echo "футбол чемпионат" > q.txt
```

### 运行查询

```bash
./find.sh q.txt 10
```

### 另一个查询示例

```bash
echo "илон маск" > q_ilon.txt
./find.sh q_ilon.txt 10
```

### 含义

`find.sh` 从文件中读取用户 query，并返回 Top-10 新闻。

参数含义：

```text
q.txt   查询文本文件
10      返回结果数量
```

---

## 5. 运行交互式查询

### 命令

```bash
./ask.sh 10
```

然后输入：

```text
футбол чемпионат
```

或：

```text
илон маск
```

### 含义

`ask.sh` 允许现场手动输入 query，然后直接返回搜索结果。

---

## 6. 保存查询输出到文件

### 命令

```bash
./find.sh q.txt 10 > out.txt 2>&1
```

### 查看最后输出

```bash
tail -80 out.txt
```

### 查找关键结果

```bash
grep -n "Query terms\|rank\|ERROR\|Exception\|Traceback" out.txt | head -80
```

### 含义

这个方法适合答辩前检查结果，也适合保留运行日志。

---

## 7. 查看脚本内容

### 查看训练脚本

```bash
cat train.sh
```

### 查看查询脚本

```bash
cat find.sh
```

### 查看交互脚本

```bash
cat ask.sh
```

### 含义

答辩时如果老师问“怎么运行”，可以直接展示这些脚本。

---

## 8. 查看 Python 代码中的模型部分

### 在 train.py 中查找 Word2Vec

```bash
grep -n "Word2Vec\|model.write\|train_word2vec" train.py
```

### 在 find.py 中查找模型加载

```bash
grep -n "Word2VecModel\|load" find.py
```

### 查找 BM25 相关代码

```bash
grep -n "bm25\|term_df\|avg_doc_len\|n_docs" train.py
grep -n "bm25\|term_df\|avg_doc_len\|n_docs" find.py
```

### 含义

这些命令用于快速定位：

```text
Word2Vec 神经网络训练代码
模型保存代码
模型加载代码
BM25 统计和打分代码
```

---

## 9. 检查 query 文件编码

### 命令

```bash
xxd -g 1 q.txt
```

### 含义

用于检查俄语 query 是否是正常 UTF-8 编码。

如果看到类似：

```text
d1 84 d1 83 ...
```

说明俄语 UTF-8 正常。

如果看到很多：

```text
3f
```

说明可能变成了问号，编码有问题。

---

## 10. 查看 Linux 当前语言环境

### 命令

```bash
locale
```

### 临时设置 UTF-8

```bash
export LANG=C.UTF-8
export LC_ALL=C.UTF-8
```

### 含义

如果交互输入俄语出问题，可以检查终端 locale。  
文件查询通常更稳定。

---

## 11. 查看 Spark 是否可用

### 查看 Spark 命令位置

```bash
which spark-submit
```

### 查看 Spark 默认配置

```bash
cat $SPARK_HOME/conf/spark-defaults.conf
```

### 查看 Java/Spark 相关进程

```bash
jps
```

### 含义

这些命令用于检查 Spark 环境。

如果 `jps` 里没有：

```text
Master
Worker
```

说明 Spark standalone master/worker 当前可能没有启动。

---

## 12. Spark master 不可用时的查询方式

### 本项目查询脚本中使用

```bash
--master local[2]
```

### 含义

表示用本地两个线程运行 Spark 查询任务。  
当前服务器 Spark standalone master 可能不可用，所以演示查询时可以使用 local mode。  
数据、模型和索引仍然来自 HDFS。

---

## 13. 查看 HDFS 目录

### 查看自己的 HDFS 目录

```bash
hdfs dfs -ls /MeiliDuolikun
```

### 查看 Lenta 项目 HDFS 目录

```bash
hdfs dfs -ls /MeiliDuolikun/lenta_search
```

### 查看全部数据集

```bash
hdfs dfs -ls /datasets
```

### 含义

这些命令用于查看 HDFS 目录结构。

---

## 14. HDFS 常用文件操作

### 创建目录

```bash
hdfs dfs -mkdir -p /MeiliDuolikun/test_dir
```

### 上传文件

```bash
hdfs dfs -put local_file.txt /MeiliDuolikun/test_dir/
```

### 覆盖上传

```bash
hdfs dfs -put -f local_file.txt /MeiliDuolikun/test_dir/
```

### 下载文件

```bash
hdfs dfs -get /MeiliDuolikun/test_dir/local_file.txt .
```

### 删除目录

```bash
hdfs dfs -rm -r /MeiliDuolikun/test_dir
```

### 查看文件内容

```bash
hdfs dfs -cat /path/to/file | head
```

### 查看大小

```bash
hdfs dfs -du -h /path/to/file_or_dir
```

---

## 15. 训练命令

### 训练脚本

```bash
./train.sh
```

### 含义

该命令会重新训练 Word2Vec、构建索引并保存到 HDFS。

### 注意

训练比较慢，不建议答辩现场重新跑。  
答辩现场建议展示 HDFS 中已经保存好的模型和索引。

---

## 16. 推荐答辩演示命令顺序

### 1. 进入项目

```bash
cd ~/MeiliDuolikun/lenta_final
ls
```

### 2. 展示 HDFS 数据集

```bash
hdfs dfs -ls /datasets/news_lenta.csv
```

### 3. 展示模型、索引、统计

```bash
hdfs dfs -ls /MeiliDuolikun/lenta_search/models/w2v_100k_v2
hdfs dfs -ls /MeiliDuolikun/lenta_search/index/docs_100k_v2_text
hdfs dfs -ls /MeiliDuolikun/lenta_search/stats/bm25_100k_v2
```

### 4. 运行查询

```bash
echo "футбол чемпионат" > q.txt
./find.sh q.txt 10
```

### 5. 再演示一个查询

```bash
echo "илон маск" > q_ilon.txt
./find.sh q_ilon.txt 10
```

### 6. 如果要现场输入

```bash
./ask.sh 10
```

---

## 17. 常见错误与处理

### 错误：Failed to connect to /127.0.0.1:7077

含义：

```text
Spark standalone master 当前不可用。
```

解决：

```text
查询时使用 --master local[2]
```

本项目的 `find.sh` 和 `ask.sh` 已经使用 local mode。

### 错误：Query has no valid tokens after preprocessing

可能原因：

```text
query 为空
query 太短
query 编码出错
query 只有停用词
```

解决：

```bash
echo "футбол чемпионат" > q.txt
./find.sh q.txt 10
```

### 错误：No such file or directory

可能原因：

```text
当前目录不对
文件名写错
脚本没有复制到当前目录
```

解决：

```bash
cd ~/MeiliDuolikun/lenta_final
ls
```

### 错误：Permission denied

可能原因：

```text
脚本没有执行权限
```

解决：

```bash
chmod +x train.sh find.sh ask.sh
```

---

## 18. 一句话说明这些命令的用途

### Русский

Эти команды показывают структуру проекта, наличие данных в HDFS, сохраненную модель Word2Vec, поисковый индекс, статистику BM25 и позволяют запустить поиск по запросу пользователя.

### 中文

这些命令用于展示项目结构、HDFS 中的数据、保存好的 Word2Vec 模型、搜索索引、BM25 统计，并运行用户查询。

