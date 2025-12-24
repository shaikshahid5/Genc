Perfect 👍
Below is a FULL ASSIGNMENT-READY SPRING JDBC PROJECT with ALL FILES you asked for.
You can directly import into Eclipse, run the SQL, and execute the main() method.

I have strictly followed:

❌ No package changes

❌ No method name changes

❌ No signature changes

✅ Spring Annotations

✅ JdbcTemplate

✅ TransactionManager

✅ VarArgs

✅ Rollback & Commit



---

📁 Project Structure

ElectricityBill
│
├── pom.xml
├── create-schema.sql
│
└── src/main
    ├── java
    │   └── com/cts/handson
    │       ├── AppConfiguration.java
    │       ├── ElectricityBillApplication.java
    │       ├── dao
    │       │   └── EBillDAO.java
    │       ├── model
    │       │   └── EBill.java
    │       └── util
    │           └── DateUtil.java
    │
    └── resources
        └── connection.properties


---

1️⃣ create-schema.sql

CREATE DATABASE electricitydb;
USE electricitydb;

CREATE TABLE EBILL (
    BILL_NUMBER BIGINT PRIMARY KEY,
    CONSUMER_NAME VARCHAR(50),
    PREVIOUS_READING INT,
    CURRENT_READING INT,
    UNITS INT,
    BILL_DATE DATE
);

INSERT INTO EBILL VALUES
(100, 'Ramesh', 1200, 1300, 100, '2024-01-10'),
(101, 'Suresh', 2000, 2150, 150, '2024-01-12'),
(102, 'Mahesh', 3000, 3200, 200, '2024-01-15');


---

2️⃣ pom.xml

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.cts.handson</groupId>
    <artifactId>ElectricityBill</artifactId>
    <version>1.0</version>

    <dependencies>

        <!-- Spring Core -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.30</version>
        </dependency>

        <!-- Spring JDBC -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.3.30</version>
        </dependency>

        <!-- Spring TX -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.3.30</version>
        </dependency>

        <!-- MySQL Driver -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.33</version>
        </dependency>

    </dependencies>
</project>


---

3️⃣ connection.properties

jdbc.driverClassName=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/electricitydb
jdbc.username=root
jdbc.password=root

(👉 Change username/password if needed)


---

4️⃣ AppConfiguration.java

package com.cts.handson;

import javax.sql.DataSource;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;

@Configuration
@ComponentScan(basePackages = "com.cts.handson")
@PropertySource("classpath:connection.properties")
public class AppConfiguration {

    @Autowired
    private Environment env;

    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource ds = new DriverManagerDataSource();
        ds.setDriverClassName(env.getProperty("jdbc.driverClassName"));
        ds.setUrl(env.getProperty("jdbc.url"));
        ds.setUsername(env.getProperty("jdbc.username"));
        ds.setPassword(env.getProperty("jdbc.password"));
        return ds;
    }

    @Bean
    public JdbcTemplate jdbcTemplate() {
        return new JdbcTemplate(dataSource());
    }

    @Bean
    public PlatformTransactionManager transactionManager() {
        return new DataSourceTransactionManager(dataSource());
    }
}


---

5️⃣ EBillDAO.java

package com.cts.handson.dao;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;

import com.cts.handson.model.EBill;

@Repository
public class EBillDAO {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Autowired
    private PlatformTransactionManager transactionManager;

    public void deleteBill(long... billNumbers) throws Exception {

        TransactionStatus status =
                transactionManager.getTransaction(new DefaultTransactionDefinition());

        try {
            for (long billNo : billNumbers) {

                if (billNo < 100) {
                    throw new Exception("Invalid Bill Number");
                }

                jdbcTemplate.update(
                        "DELETE FROM EBILL WHERE BILL_NUMBER = ?",
                        billNo
                );
            }

            transactionManager.commit(status);

        } catch (Exception e) {
            transactionManager.rollback(status);
            throw e;
        }
    }

    public List<EBill> getAllBill() {
        return jdbcTemplate.query(
                "SELECT * FROM EBILL",
                new BeanPropertyRowMapper<>(EBill.class)
        );
    }
}


---

6️⃣ ElectricityBillApplication.java

package com.cts.handson;

import java.util.List;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import com.cts.handson.dao.EBillDAO;
import com.cts.handson.model.EBill;

public class ElectricityBillApplication {

    public static void main(String[] args) {

        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(AppConfiguration.class);

        EBillDAO dao = context.getBean(EBillDAO.class);

        try {
            dao.deleteBill(100, 99); // rollback
        } catch (Exception e) {
            System.out.println("Transaction rolled back");
        }

        List<EBill> bills = dao.getAllBill();
        bills.forEach(System.out::println);

        context.close();
    }
}


---

7️⃣ EBill.java

package com.cts.handson.model;

import java.sql.Date;

public class EBill {

    private long billNumber;
    private String consumerName;
    private int previousReading;
    private int currentReading;
    private int units;
    private Date billDate;

    public long getBillNumber() {
        return billNumber;
    }

    public void setBillNumber(long billNumber) {
        this.billNumber = billNumber;
    }

    public String getConsumerName() {
        return consumerName;
    }

    public void setConsumerName(String consumerName) {
        this.consumerName = consumerName;
    }

    public int getPreviousReading() {
        return previousReading;
    }

    public void setPreviousReading(int previousReading) {
        this.previousReading = previousReading;
    }

    public int getCurrentReading() {
        return currentReading;
    }

    public void setCurrentReading(int currentReading) {
        this.currentReading = currentReading;
    }

    public int getUnits() {
        return units;
    }

    public void setUnits(int units) {
        this.units = units;
    }

    public Date getBillDate() {
        return billDate;
    }

    public void setBillDate(Date billDate) {
        this.billDate = billDate;
    }

    @Override
    public String toString() {
        return "EBill [billNumber=" + billNumber +
               ", consumerName=" + consumerName +
               ", units=" + units +
               ", billDate=" + billDate + "]";
    }
}


---

8️⃣ DateUtil.java

package com.cts.handson.util;

import java.sql.Date;
import java.time.LocalDate;

public class DateUtil {

    public static Date getCurrentDate() {
        return Date.valueOf(LocalDate.now());
    }
}


---

✅ FINAL RESULT (What Examiner Expects)

deleteBill(100, 99) → ❌ Rollback

No record deleted

getAllBill() → returns all rows

Uses JdbcTemplate + BeanPropertyRowMapper + TransactionManager



---

If you want:

📄 PDF notes

🧠 Viva answers

✍️ Short explanation version

🧪 Output screenshot explanation


Just tell me 🔥
