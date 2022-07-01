使用程序进行数据导入 
===============================

本文将介绍如何通过编写代码的方式，将导入数据到PolarDB-X中。

假设有一操作表：

```sql
CREATE TABLE `test1` (
    `id` int(11) NOT NULL,
    `k` int(11) NOT NULL DEFAULT '0',
    `c` char(120) NOT NULL DEFAULT '',
    `pad` char(60) NOT NULL DEFAULT '',
    PRIMARY KEY (`id`),
    KEY `k_1` (`k`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 dbpartition by hash(`id`);
```



从数据库中导出源数据 
-------------------------------

源数据可以用户自行生成，也可以从数据库中导出，在数据库中导出可通过`mysql -e`命令的方式，PolarDB-X和MySQL都支持该方式，具体方法如下：

```shell
mysql -h ip  -P port -u usr -pPassword db_name -N -e "SELECT id,k,c,pad FROM test1;" >/home/data_1000w.txt
## 原始数据以制表符分隔，数据格式：188092293    27267211    59775766593-64673028018-...-09474402685    01705051424-...-54211554755
mysql -h ip  -P port -u usr -pPassword db_name -N -e "SELECT id,k,c,pad FROM test1;" | sed 's/\t/,/g' >/home/data_1000w.csv
## csv文件格式以逗号分隔，数据格式：188092293,27267211,59775766593-64673028018-...-09474402685,01705051424-...-54211554755
```



推荐对字符串进行处理，转变成csv文件格式，方便后续程序读取数据。

在PolarDB-X中创建目标表 
----------------------------------

源数据不包括建表语句，所以需要手动在PolarDB-X目标数据库上创建表，关于PolarDB-X建表语句的语法请参见[CREATE TABLE](../../dev-guide/topics/create-table.md)语句，例如：

```sql
CREATE TABLE `test1` (
    `id` int(11) NOT NULL,
    `k` int(11) NOT NULL DEFAULT '0',
    `c` char(120) NOT NULL DEFAULT '',
    `pad` char(60) NOT NULL DEFAULT '',
    PRIMARY KEY (`id`),
    KEY `k_1` (`k`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 dbpartition by hash(`id`);
```



使用程序导入数据到PolarDB-X 
------------------------------------

您可以自行编写程序，连接PolarDB-X，然后读取本地数据，通过Batch Insert语句导入PolarDB-X中。

下面是一个简单的JAVA程序示例：

```java
// 需要mysql-connector-java.jar, 详情界面：https://mvnrepository.com/artifact/mysql/mysql-connector-java
// 下载链接：https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.20/mysql-connector-java-8.0.20.jar
// 注：不同版本的mysql-connector-java.jar，可能Class.forName("com.mysql.cj.jdbc.Driver")类路径不同
// 编译 javac LoadData.java
// 运行 java -cp .:mysql-connector-java-8.0.20.jar LoadData
import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.io.IOException;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
public class LoadData {
    public static void main(String[] args) throws IOException, ClassNotFoundException, SQLException {
        File dataFile = new File("/home/data_1000w.csv");
        String sql = "insert into test1(id, k, c, pad) values(?, ?, ?, ?)";
        int batchSize = 1000;
        try (
            Connection connection = getConnection("ip", 3306, "db", "usr", "password");
            BufferedReader br = new BufferedReader(new FileReader(dataFile))) {
            String line;
            PreparedStatement st = connection.prepareStatement(sql);
            long startTime = System.currentTimeMillis();
            int batchCount = 0;
            while ((line = br.readLine()) != null) {
                String[] data = line.split(",");
                st.setInt(1, Integer.valueOf(data[0]));
                st.setInt(2, Integer.valueOf(data[1]));
                st.setObject(3, data[2]);
                st.setObject(4, data[3]);
                st.addBatch();
                if (++batchCount % batchSize == 0) {
                    st.executeBatch();
                    System.out.println(String.format("insert %d records", batchCount));
                }
            }
            if (batchCount % batchSize != 0) {
                st.executeBatch();
            }
            long cost = System.currentTimeMillis() - startTime;
            System.out.println(String.format("Take %d second，insert %d records, tps %d", cost/1000, batchCount, batchCount/(cost/1000)));
        }
    }
    /**
     * 获取数据库连接
     *
     * @param host     数据库地址
     * @param port     端口
     * @param database 数据库名称
     * @param username 用户名
     * @param password 密码
     * @return
     * @throws ClassNotFoundException
     * @throws SQLException
     */
    private static Connection getConnection(String host, int port, String database, String username, String password)
        throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        String url = String.format(
            "jdbc:mysql://%s:%d/%s?autoReconnect=true&socketTimeout=600000&rewriteBatchedStatements=true", host, port,
            database);
        Connection con = DriverManager.getConnection(url, username, password);
        return con;
    }
}
```



您可以根据实际应用场景编写程序，设置合适的batch size和多线程导入，能够加快性能。
