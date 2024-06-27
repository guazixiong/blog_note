# 批量插入百万数量级的数据到mysql的解决方案

> 项目场景：需要从上报的单个文件中解析出百万数据入库，项目中无论是使用jpa 还是 [mybatis](https://so.csdn.net/so/search?q=mybatis&spm=1001.2101.3001.7020) 存入数据达到10000时速度明显变慢，达到100000时就让人难以接受。所以就考虑使用存储过程或者是使用原生jdbc实现，该案例使用原生jdbc实现。

## 单线程实现案例：

```java
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
 
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
 
@Slf4j
@Data
public abstract class BigDataInsert<T> {
    //     String driverClassName = "com.mysql.cj.jdbc.Driver";
    String driverClassName = "";
    //     String url = "jdbc:mysql://localhost:3306/bigdata?useServerPrepStmts=false&rewriteBatchedStatements=true&useUnicode=true&characterEncoding=UTF-8";
    String url = "";
    //     String user = "root";
    String user = "";
    //     String password = "123456";
    String password = "";
    //     String sql = "";
    String sql = "";
 
    public BigDataInsert() {
        init();
    }
 
    public void insertBigData(List<T> list) {
        //定义连接、statement对象
        Connection conn = null;
        PreparedStatement pstm = null;
        try {
            // 检查初始化参数
            checkInit();
            //加载jdbc驱动
            Class.forName(driverClassName);
            //连接mysql
            conn = DriverManager.getConnection(url, user, password);
            //将自动提交关闭
            conn.setAutoCommit(false);
            //预编译sql
            pstm = conn.prepareStatement(sql);
            //开始总计时
            long bTime1 = System.currentTimeMillis();
 
            List<List<T>> listList = null;
            if (list.size() > 100000) {
                listList = fixedGrouping(list, 100000);
            } else {
                listList.add(list);
            }
            //循环10次，每次十万数据，一共100万
            for (int i = 0; i < listList.size(); i++) {
                //开启分段计时，计1W数据耗时
                long bTime = System.currentTimeMillis();
                //开始循环
                for (T object : listList.get(i)) {
                    //赋值
                    pstmToSetValue(pstm, object);
                    //添加到同一个批处理中
                    pstm.addBatch();
                }
                //执行批处理
                pstm.executeBatch();
                //提交事务
                conn.commit();
                //关闭分段计时
                long eTime = System.currentTimeMillis();
                //输出
                System.out.println("成功插入" + listList.get(i).size() + "条数据耗时：" + (eTime - bTime));
            }
            //关闭总计时
            long eTime1 = System.currentTimeMillis();
            //输出
            System.out.println("插入" + list.size() + "数据共耗时：" + (eTime1 - bTime1));
        } catch (SQLException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e1) {
            e1.printStackTrace();
        }finally {
            try {
                pstm.close();
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
 
    private void checkInit() {
        if ("".equals(driverClassName)) {
            log.warn("driverClassName未初始化！");
        }
        if ("".equals(url)) {
            log.warn("url未初始化！");
        }
        if ("".equals(user)) {
            log.warn("user未初始化！");
        }
        if ("".equals(password)) {
            log.warn("password未初始化！");
        }
        if ("".equals(sql)) {
            log.warn("sql未设置！");
        }
    }
 
    /**
     * 将一组数据固定分组，每组n个元素
     *
     * @param source 要分组的数据源
     * @param n      每组n个元素
     * @param <T>
     * @return
     */
    public static <T> List<List<T>> fixedGrouping(List<T> source, int n) {
 
        if (null == source || source.size() == 0 || n <= 0)
            return null;
        List<List<T>> result = new ArrayList<List<T>>();
        int remainder = source.size() % n;
        int size = (source.size() / n);
        for (int i = 0; i < size; i++) {
            List<T> subset = null;
            subset = source.subList(i * n, (i + 1) * n);
            result.add(subset);
        }
        if (remainder > 0) {
            List<T> subset = null;
            subset = source.subList(size * n, size * n + remainder);
            result.add(subset);
        }
        return result;
    }
 
    public abstract PreparedStatement pstmToSetValue(PreparedStatement pstm, T object);
 
    public abstract void init();
 
}
```

上面代码是一个批量保存大数据的一个单线程的封装，使用方法如下：

```java
import *****.NetCollectData;
import org.springframework.stereotype.Component;
 
import java.sql.Date;
import java.sql.PreparedStatement;
import java.sql.SQLException;
 
@Component("netCollectDataBigDataInsert")
public class NetCollectDataBigDataInsertImpl extends BigDataInsert<NetCollectData> {
 
    @Override
    public void init() {
        this.driverClassName = "com.mysql.cj.jdbc.Driver";
        this.url = "jdbc:mysql://10.1.1.149:3306/cnleb139?useServerPrepStmts=false&rewriteBatchedStatements=true&useUnicode=true&characterEncoding=UTF-8";
        this.user = "root";
        this.password = "Root123!!!";
        this.sql = "INSERT INTO net_collect_data(collect_time,mac,edu_id,ip,time,TS_UP_4G,TS_UP,TS_DOWN_4G,TS_DOWN,PKG_UP_4G,PKG_UP,PKG_DOWN_4G,PKG_DOWN,status_code,created_date) " +
                "VALUES (?,?,?,?,?,?,?,?,?,?,?,?,?,?,?)";
    }
 
    @Override
    public PreparedStatement pstmToSetValue(PreparedStatement pstm, NetCollectData netCollectData) {
        try {
            pstm.setString(1, netCollectData.getCOLLECT_TIME());
            pstm.setString(2, netCollectData.getMAC());
            pstm.setString(3, netCollectData.getEduId());
            pstm.setString(4, netCollectData.getIP());
            pstm.setString(5, netCollectData.getTIME());
 
            pstm.setLong(6, netCollectData.getTS_UP_4G());
            pstm.setLong(7, netCollectData.getTS_UP());
            pstm.setLong(8, netCollectData.getTS_DOWN_4G());
            pstm.setLong(9, netCollectData.getTS_DOWN());
            pstm.setLong(10, netCollectData.getPKG_UP_4G());
            pstm.setLong(11, netCollectData.getPKG_UP());
            pstm.setLong(12, netCollectData.getPKG_DOWN_4G());
            pstm.setLong(13, netCollectData.getPKG_DOWN());
            pstm.setInt(14, netCollectData.getStatus());
            pstm.setDate(15, new Date(netCollectData.getDate().getTime()));
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return pstm;
    }
 
 
}
```

**继承BigDataInsert实现init()和pstmToSetValue()方法**

`init()`: 初始化链接信息的方法（包括插入语句），在实例被创建出来的时候会被调用

`pstmToSetValue()`: 对预编译的sql语句set值，当实例出来的对象调用insertBigData()时执行

根据检查，这里是对16个字段的表插入数据，字段越多越慢，50w数据需要15分钟完成

## 多线程实现案例

```java
import lombok.Data;
import lombok.extern.slf4j.Slf4j;
 
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
 
@Slf4j
@Data
public abstract class BigDataInsert<T> {
    //     String driverClassName = "com.mysql.cj.jdbc.Driver";
    String driverClassName = "";
    //     String url = "jdbc:mysql://localhost:3306/bigdata?useServerPrepStmts=false&rewriteBatchedStatements=true&useUnicode=true&characterEncoding=UTF-8";
    String url = "";
    //     String user = "root";
    String user = "";
    //     String password = "123456";
    String password = "";
    //     String sql = "";
    String sql = "";
 
    int groupCount = 100000;
 
    int threadPoolCount = 5;
 
    // 创建一个固定大小的线程池
    private ExecutorService service = null;
 
    public BigDataInsert() {
        init();
    }
 
    public void insertBigData(List<T> list) {
        // 检查初始化参数
        checkInit();
        // 创建线程池对象
        service = Executors.newFixedThreadPool(threadPoolCount);
        // 将需保存集合分组
        List<List<T>> listList = new ArrayList<>();
        if (list.size() > groupCount) {
            listList = fixedGrouping(list, groupCount);
        } else {
            listList.add(list);
        }
        // 计数器
        final CountDownLatch latch = new CountDownLatch(listList.size());
 
        //开始总计时
        long bTime1 = System.currentTimeMillis();
        //循环10次，每次十万数据，一共100万
        for (int i = 0; i < listList.size(); i++) {
            int finalI = i;
            List<List<T>> finalListList = listList;
            // 多线程保存
            service.execute(() -> {
                Connection conn = null;
                PreparedStatement pstm = null;
                try {
                    //加载jdbc驱动
                    Class.forName(driverClassName);
                    //连接mysql
                    conn = DriverManager.getConnection(url, user, password);
                    //将自动提交关闭
                    conn.setAutoCommit(false);
                    //预编译sql
                    pstm = conn.prepareStatement(sql);
                    //开启分段计时，计1W数据耗时
                    long bTime = System.currentTimeMillis();
                    //开始循环
                    for (T object : finalListList.get(finalI)) {
                        //赋值
                        pstmToSetValue(pstm, object);
                        //添加到同一个批处理中
                        pstm.addBatch();
                    }
                    //执行批处理
                    pstm.executeBatch();
                    //提交事务
                    conn.commit();
                    //关闭分段计时
                    long eTime = System.currentTimeMillis();
                    //输出
                    System.out.println("成功插入" + finalListList.get(finalI).size() + "条数据耗时：" + (eTime - bTime));
                } catch (Exception e) {
                    log.error("批量保存失败！");
                } finally {
                    latch.countDown();
                    try {
                        pstm.close();
                        conn.close();
                    } catch (SQLException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
 
        try {
            latch.await();
        } catch (Exception e) {
            log.error("多线程分析数据中途异常!,{}", e);
        }
        //关闭总计时
        long eTime1 = System.currentTimeMillis();
        //输出
        System.out.println("插入" + list.size() + "数据共耗时：" + (eTime1 - bTime1));
 
    }
 
    private void checkInit() {
        if ("".equals(driverClassName)) {
            log.warn("driverClassName未初始化！");
        }
        if ("".equals(url)) {
            log.warn("url未初始化！");
        }
        if ("".equals(user)) {
            log.warn("user未初始化！");
        }
        if ("".equals(password)) {
            log.warn("password未初始化！");
        }
        if ("".equals(sql)) {
            log.warn("sql未设置！");
        }
    }
 
    /**
     * 将一组数据固定分组，每组n个元素
     *
     * @param source 要分组的数据源
     * @param n      每组n个元素
     * @param <T>
     * @return
     */
    public static <T> List<List<T>> fixedGrouping(List<T> source, int n) {
 
        if (null == source || source.size() == 0 || n <= 0)
            return null;
        List<List<T>> result = new ArrayList<List<T>>();
        int remainder = source.size() % n;
        int size = (source.size() / n);
        for (int i = 0; i < size; i++) {
            List<T> subset = null;
            subset = source.subList(i * n, (i + 1) * n);
            result.add(subset);
        }
        if (remainder > 0) {
            List<T> subset = null;
            subset = source.subList(size * n, size * n + remainder);
            result.add(subset);
        }
        return result;
    }
 
    public abstract PreparedStatement pstmToSetValue(PreparedStatement pstm, T object);
 
    public abstract void init();
 
}
```

多线程版本允许在实现init()方法时，修改线程数量（默认5）以及分组保存的数量（默认100000）

经过检查：50w数据3分钟左右能够完成（本人电脑一般，电脑性能好的会更快）