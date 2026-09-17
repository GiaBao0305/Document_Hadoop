# Hadoop Installation Guide

## 1. Tổng thể các công việc

| STT | Nội dung công việc | Các công việc chi tiết | Phân công |
| :-- | :----------------- | :--------------------- | :-------- |
| 1 | Chuẩn bị môi trường | wsl --install → cài Ubuntu → cài Java 17 → tải/giải nén Hadoop → cấu hình JAVA_HOME, HADOOP_HOME, PATH → kiểm tra hadoop version | Thùy Dung |
| 2 | Cài đặt và cấu hình SSH | Cài SSH → cài rsync → tạo SSH Key → thêm Key vào authorized_keys → phân quyền → kiểm tra ssh localhost | Thùy Dung |
| 3 | Cấu hình và thực hành HDFS | Cấu hình core-site.xml → hdfs-site.xml → tạo thư mục NameNode/DataNode → format NameNode → start-dfs.sh → kiểm tra bằng jps → tạo thư mục trên HDFS → đưa file lên HDFS → kiểm tra HDFS Web UI | |
| 4 | Chạy chương trình WordCount mẫu | Chuẩn bị file input → đưa input lên HDFS → sử dụng chương trình WordCount có sẵn của Hadoop → chạy chương trình → kiểm tra kết quả | |
| 5 | Cấu hình YARN và kiểm tra | Cấu hình mapred-site.xml → cấu hình yarn-site.xml → start-yarn.sh → kiểm tra bằng jps → mở localhost:8088 → kiểm tra Job MapReduce | Gia Bảo |
| 6 | Tự viết chương trình WordCount | Viết Mapper → viết Reducer → viết Driver → biên dịch chương trình → đóng gói thành file .jar | Gia Bảo |
| 7 | Chạy chương trình WordCount tự viết và kiểm tra kết quả | Chuẩn bị input → đưa input lên HDFS → chạy file .jar → kiểm tra output trên HDFS → kiểm tra Job trên YARN Web UI | |

## 1. Chuẩn bị môi trường:

- Cài WSL
```bash
wsl –install
```

- Cài ubuntu cho wsl
```bash
wsl --install -d Ubuntu-24.04  
wsl --install -d Ubuntu-22.04
```

- Tạo tên tài khoản và mật khẩu (lưu ý khi tạo nhập vào nó sẽ không hiện gì, nhưng vẫn đang nhận dữ liệu từ bàn phím)
- Cập nhật danh sách các phần mềm trên ubuntu
```bash
sudo apt update
```

- Cài Java 17
```bash
sudo apt install openjdk-17-jdk
```

- Kiểm tra phiên bản Java
```bash
java -version
```

- Tải và giải nén Hadoop 3.5.0.
```bash
tar -xzf hadoop-3.5.0.tar.gz
```

- Di chuyển Hadoop vào thư mục /opt/hadoop
```bash
sudo mv hadoop-3.5.0 /opt/hadoop
```

- Cấu hình biến môi trường
```bash
nano ~/.bashrc
```

- Thêm vào cuối file:
```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export HADOOP_HOME=/opt/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

- Cập nhật lại cấu hình
```bash
source ~/.bashrc
```

-  Kiểm tra Hadoop
```bash
hadoop version
```

Kết quả: Hoàn thành việc cài đặt WSL, Ubuntu và Java, Hadoop.

## 2. Cài đặt và cấu hình SSH:

- Cài đặt SSH
```bash
sudo apt install ssh
```

- Cài đặt rsync
```bash
sudo apt install rsync
```

- Tạo SSH Key
```bash
ssh-keygen -t ed25519
```

- Nhấn Enter để sử dụng đường dẫn mặc định và để trống mật khẩu.
- Thêm SSH Key vào danh sách được phép
```bash
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
```

- Phân quyền cho file
```bash
chmod 600 ~/.ssh/authorized_keys
```

- Kiểm tra kết nối SSH đến localhost
```bash
ssh localhost
```

- Nếu kết nối thành công mà không yêu cầu nhập mật khẩu thì cấu hình SSH đã hoàn tất.
- Thoát khỏi SSH
exit

## 3. Cấu hình và thực hành HDFS:

- Di chuyển vào thư mục cấu hình Hadoop
```bash
cd /opt/hadoop/etc/hadoop
```

- Cấu hình core-site.xml
```bash
sudo nano core-site.xml
```

Thêm:

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

- Cấu hình hdfs-site.xml
```bash
sudo nano hdfs-site.xml
```

Thêm:

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
```

```xml
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///opt/hadoop/data/namenode</value>
    </property>
```

```xml
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///opt/hadoop/data/datanode</value>
    </property>
</configuration>
```

- Cấu hình cho hadoop biết java đang ở đâu
```bash
sudo nano /opt/hadoop/etc/hadoop/hadoop-env.sh
```

Thêm:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
```

- Tạo thư mục lưu dữ liệu cho NameNode và DataNode
```bash
sudo mkdir -p /opt/hadoop/data/namenode
sudo mkdir -p /opt/hadoop/data/datanode
```

- Cấp quyền cho thư mục Hadoop
```bash
sudo chown -R $USER:$USER /opt/hadoop
```

- Format NameNode
```bash
hdfs namenode -format
```

- Khởi động HDFS
```bash
start-dfs.sh
```

Tắt khi làm xong

```bash
stop-dfs.sh
```

- Kiểm tra các tiến trình Hadoop
```bash
jps
```

- Nếu thành công, thường thấy:
```bash
NameNode
DataNode
SecondaryNameNode
```

- Tạo thư mục trên HDFS
```bash
hdfs dfs -mkdir -p /data/input
```

- Kiểm tra thư mục
```bash
hdfs dfs -ls /data
```

-Tạo một file văn bản để đưa lên HDFS
```bash
echo "Hello Hadoop" > input.txt
```

- Đưa file lên HDFS
```bash
hdfs dfs -put input.txt /data/input/
```

- Kiểm tra file đã được đưa lên HDFS
```bash
hdfs dfs -ls /data/input
```

- Đọc nội dung file trên HDFS
```bash
hdfs dfs -cat /data/input/input.txt
```

- Mở giao diện quản lý HDFS trên trình duyệt
`http://localhost:9870/`

- Nếu truy cập được giao diện HDFS và thấy các thông tin về NameNode/DataNode thì HDFS đã hoạt động thành công.
## 4. Chạy chương trình WordCount mẫu trên Hadoop

- Chuẩn bị file dữ liệu đầu vào
Tạo một file văn bản để làm dữ liệu đầu vào:

```bash
echo "Hello Hadoop Hadoop MapReduce" > test.txt
```

- Kiểm tra nội dung file
```bash
cat test.txt
```

- Đưa file dữ liệu lên HDFS
```bash
hdfs dfs -put test.txt /data/input/
```

- Kiểm tra file đã được đưa lên HDFS
```bash
hdfs dfs -ls /data/input
```

- Kiểm tra nội dung file trên HDFS
```bash
hdfs dfs -cat /data/input/test.txt
```

- Chạy chương trình WordCount mẫu có sẵn trong Hadoop
```bash
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.5.0.jar wordcount /data/input/test.txt /output
```

Trong đó:

```bash
hadoop jar: chạy chương trình Java dạng .jar trên Hadoop.
```

hadoop-mapreduce-examples-3.5.0.jar: file chứa các chương trình MapReduce mẫu của Hadoop.

wordcount: chương trình đếm số lần xuất hiện của từng từ.

/data/input/thonguyen.txt: file dữ liệu đầu vào trên HDFS.

/output: thư mục chứa kết quả trên HDFS.

- Kiểm tra thư mục kết quả
```bash
hdfs dfs -ls /output
```

- Xem kết quả WordCount
```bash
hdfs dfs -cat /output/part-r-00000      (Bao and Thanh da toi day)
```

- Kết quả sẽ có dạng tương tự:
```bash
Hadoop    2
Hello     1
MapReduce 1
```

Mỗi dòng gồm:

từ    số_lần_xuất_hiện

- Nếu muốn chạy lại WordCount với cùng thư mục /output, cần xóa kết quả cũ trước:
```bash
hdfs dfs -rm -r /output
```

- Sau đó chạy lại lệnh WordCount.
Kiểm tra toàn bộ quá trình

- Sau khi chạy WordCount thành công, có thể kiểm tra:
```bash
hdfs dfs -ls /output
```

và:

```bash
hdfs dfs -cat /output/part-r-00000
```

Kết quả của phần này

Đã sử dụng chương trình WordCount mẫu có sẵn trong Hadoop để thực hiện một tác vụ MapReduce đơn giản trên dữ liệu được lưu trong HDFS.

## 5. Cấu hình YARN và kiểm tra

- Di chuyển vào thư mục cấu hình Hadoop
```bash
cd /opt/hadoop/etc/hadoop
```

- Cấu hình mapred-site.xml
- Tạo file cấu hình từ file mẫu:
```bash
cp mapred-site.xml.template mapred-site.xml
```

Mở file:

```bash
nano mapred-site.xml
```

Thêm nội dung:

```xml
<configuration>
```

```xml
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
```

```xml
    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=/opt/hadoop</value>
    </property>
```

```xml
    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=/opt/hadoop</value>
    </property>
```

```xml
    <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=/opt/hadoop</value>
    </property>
```

```xml
</configuration>
```

Cấu hình này cho Hadoop biết MapReduce sẽ sử dụng YARN để quản lý và thực thi job.

- Cấu hình yarn-site.xml
```bash
nano yarn-site.xml
```

Thêm nội dung:

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

Cấu hình này cho phép NodeManager hỗ trợ quá trình Shuffle trong MapReduce.

- Khởi động YARN
```bash
start-yarn.sh
```

- Kiểm tra các tiến trình Hadoop
```bash
jps
```

Kết quả dự kiến sẽ có:

```bash
NameNode
DataNode
SecondaryNameNode
ResourceManager
NodeManager
Jps
```

- Kiểm tra YARN Web UI
Mở trình duyệt và truy cập:

`http://localhost:8088/`

Giao diện YARN cho phép theo dõi các thông tin như:

```bash
ResourceManager
NodeManager
```

Các ứng dụng MapReduce

Trạng thái và tiến trình của Job

- Kiểm tra lại HDFS Web UI
```bash
HDFS vẫn hoạt động song song với YARN. Có thể truy cập:
```

`http://localhost:9870/`

- Kiểm tra hệ thống sau khi cấu hình
Có thể sử dụng:

```bash
jps
```

để đảm bảo cả các tiến trình HDFS và YARN đều đang chạy.

- Kết quả của phần này
Đã cấu hình YARN để quản lý tài nguyên và hỗ trợ thực thi các chương trình MapReduce.

Sau khi hoàn thành phần này, hệ thống Hadoop có thể sử dụng:

```bash
HDFS → lưu trữ dữ liệu.
YARN → quản lý tài nguyên và job.
MapReduce → xử lý dữ liệu.
```

## 6. Tự viết chương trình WordCount

- Tạo thư mục chứa mã nguồn
```bash
mkdir -p ~/WordCount
cd ~/WordCount
```

- Tạo chương trình Mapper
```bash
nano WordCountMapper.java
```

Thêm nội dung:

```java
import java.io.IOException;
```

```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
```

```java
public class WordCountMapper
        extends Mapper<Object, Text, Text, IntWritable> {
```

```java
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();
```

    public void map(Object key, Text value, Context context)

            throws IOException, InterruptedException {

        String[] words = value.toString().split("\\s+");

        for (String w : words) {

            word.set(w);

            context.write(word, one);

```bash
        }
    }
}
Mapper có nhiệm vụ đọc từng dòng dữ liệu, tách thành các từ và tạo ra cặp:
```

(từ, 1)

- Tạo chương trình Reducer
```bash
nano WordCountReducer.java
```

Thêm nội dung:

```java
import java.io.IOException;
```

```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;
```

```java
public class WordCountReducer
        extends Reducer<Text, IntWritable, Text, IntWritable> {
```

    public void reduce(Text key, Iterable<IntWritable> values,

                       Context context)

            throws IOException, InterruptedException {

        int sum = 0;

        for (IntWritable value : values) {

            sum += value.get();

```bash
        }
```

        context.write(key, new IntWritable(sum));

```bash
    }
}
Reducer có nhiệm vụ cộng các giá trị của cùng một từ:
Hadoop → 1 + 1 → 2
```

- Tạo chương trình Driver
```bash
nano WordCount.java
```

Thêm nội dung:

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
```

```java
public class WordCount {
```

    public static void main(String[] args) throws Exception {

        Configuration conf = new Configuration();

        Job job = Job.getInstance(conf, "Word Count");

        job.setJarByClass(WordCount.class);

        job.setMapperClass(WordCountMapper.class);

        job.setReducerClass(WordCountReducer.class);

        job.setOutputKeyClass(Text.class);

        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job, new Path(args[0]));

        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        System.exit(job.waitForCompletion(true) ? 0 : 1);

```bash
    }
}
```

Driver có nhiệm vụ cấu hình và kết nối Mapper → Reducer, đồng thời xác định đường dẫn input và output.

- Kiểm tra các file mã nguồn
```bash
ls
```

Kết quả cần có:

```bash
WordCount.java
WordCountMapper.java
WordCountReducer.java
```

- Biên dịch chương trình
```bash
mkdir -p classes
javac -classpath "$(hadoop classpath)" -d classes *.java
```

Lệnh này biên dịch các file .java thành các file .class để Hadoop có thể sử dụng.

- Kiểm tra file sau khi biên dịch
```bash
find classes -type f
```

Kết quả sẽ có các file tương tự:

classes/WordCount.class

classes/WordCountMapper.class

classes/WordCountReducer.class

- Đóng gói chương trình thành file .jar
```bash
jar -cvf wordcount.jar -C classes .
```

Sau khi thực hiện, kiểm tra:

```bash
ls
```

Sẽ có:

wordcount.jar

- Kiểm tra file .jar
```bash
jar tf wordcount.jar
```

Kết quả cần có các class:

```bash
WordCount.class
WordCountMapper.class
WordCountReducer.class
```

- Kết quả của phần này
Đã tự xây dựng chương trình WordCount MapReduce gồm:

```bash
Input
```

  ↓

```bash
Mapper
```

  ↓

```bash
Shuffle
```

  ↓

```bash
Reducer
```

  ↓

```bash
Output
```

Chương trình đã được biên dịch và đóng gói thành:

wordcount.jar

File .jar này sẽ được sử dụng để chạy chương trình WordCount trên Hadoop ở Phần 7.

## 7. Chạy chương trình WordCount tự viết và kiểm tra kết quả

### Bước 1. Kiểm tra Hadoop và YARN đang chạy

- Kiểm tra các tiến trình:
```bash
jps
```

- Kết quả cần có:
```bash
NameNode
DataNode
SecondaryNameNode
ResourceManager
NodeManager
Jps
```

Trong đó:

```bash
HDFS: NameNode, DataNode, SecondaryNameNode.
YARN: ResourceManager, NodeManager.
```

### Bước 2. Kiểm tra file input trên HDFS

- Kiểm tra thư mục /data/input:
```bash
hdfs dfs -ls /data/input
```

- Kiểm tra nội dung test.txt:
```bash
hdfs dfs -cat /data/input/test.txt
```

File của bạn hiện có nội dung:

```bash
Hello Hadoop Hadoop MapReduce
```

### Bước 3. Xóa thư mục output cũ

- Nếu đã tồn tại /output từ lần chạy WordCount mẫu trước đó:
```bash
hdfs dfs -rm -r /output
```

- Kiểm tra:
```bash
hdfs dfs -ls /
```

Nếu không còn /output thì có thể chạy chương trình mới.

```bash
Hadoop MapReduce không cho ghi kết quả vào thư mục output đã tồn tại.
```

### Bước 4. Chạy chương trình WordCount tự viết

- Di chuyển vào thư mục chứa chương trình:
```bash
cd ~/WordCount
```

- Kiểm tra file JAR:
```bash
ls
```

Cần có:

wordcount.jar

- Chạy chương trình:
```bash
hadoop jar wordcount.jar WordCount /data/input/test.txt /output
```

Trong đó:

```bash
hadoop jar: chạy chương trình Java trên Hadoop.
```

wordcount.jar: chương trình WordCount do chúng ta tự viết.

```bash
WordCount: lớp Driver chứa hàm main.
```

/data/input/test.txt: dữ liệu đầu vào trên HDFS.

/output: thư mục lưu kết quả trên HDFS.

Luồng xử lý:

test.txt

   ↓

```bash
Mapper
```

   ↓

```bash
Shuffle
```

   ↓

```bash
Reducer
```

   ↓

/output

### Bước 5. Kiểm tra kết quả trên HDFS

- Kiểm tra thư mục output:
```bash
hdfs dfs -ls /output
```

Có thể thấy:

```bash
_SUCCESS
part-r-00000
```

- Xem kết quả:
```bash
hdfs dfs -cat /output/part-r-00000
```

Với dữ liệu:

```bash
Hello Hadoop Hadoop MapReduce
```

kết quả sẽ tương tự:

```bash
Hadoop  2
Hello   1
MapReduce       1
```

Điều này cho thấy chương trình đã đếm số lần xuất hiện của từng từ.

### Bước 6. Kiểm tra Job trên YARN Web UI

Mở trình duyệt:

`http://localhost:8088/`

Tại YARN Web UI có thể kiểm tra:

Job MapReduce đã chạy.

Trạng thái Job.

Thời gian chạy.

```bash
Mapper.
Reducer.
```

Trạng thái hoàn thành.

Nếu Job chạy thành công, trạng thái sẽ là SUCCEEDED.

### Bước 7. Kiểm tra dữ liệu trên HDFS Web UI

Mở:

`http://localhost:9870/`

Có thể kiểm tra:

```bash
HDFS đang hoạt động.
DataNode.
```

Dung lượng lưu trữ.

Các file/thư mục trên HDFS.

Kết quả của phần này

Đã thực hiện thành công chương trình WordCount tự viết trên Hadoop.

Quy trình:

File test.txt

      ↓

```bash
    HDFS
```

      ↓

```bash
   Mapper
```

      ↓

```bash
   Shuffle
```

      ↓

```bash
   Reducer
```

      ↓

  /output

      ↓

Kết quả WordCount

Chương trình tự viết gồm:

```bash
WordCountMapper.java
WordCountReducer.java
WordCount.java
```

        ↓

   wordcount.jar

        ↓

```bash
     Hadoop
```

        ↓

```bash
      YARN
```

        ↓

```bash
      HDFS
```

Như vậy đã hoàn thành quá trình từ chuẩn bị môi trường → HDFS → WordCount mẫu → YARN → tự viết WordCount → chạy và kiểm tra kết quả.
