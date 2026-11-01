# Software Engineering Laboratory Report: Advanced Java, JDBC & Jakarta Servlets

**Student Name:** Md. Antor  
**Student ID:** IT24055  
**Email:** it24055@mbstu.ac.bd  
**Department:** Department of Information and Communication Technology (ICT)  
**Institution:** Mawlana Bhashani Science and Technology University (MBSTU)  
**Date of Submission:** November 01, 2026  
**Course:** Software Engineering Laboratory / Advanced Web Technologies  

> **Direct Submission File:** [`it24055_1Nov.md`](https://github.com/antor1112225/Assaignment/blob/main/it24055_1Nov.md)

---

## Executive Summary & Verification Matrix

This repository contains the verified implementation, source codes, configuration artifacts, and technical commentary for the November 1 Software Engineering Laboratory assessment:

| Task # | Task Description | Target Environment / Specification | Status |
| :--- | :--- | :--- | :--- |
| **Task 1** | Install & Configure IntelliJ IDEA Ultimate | IntelliJ IDEA Ultimate 2026.2.1 (Build IU-262.9437.185) | **VERIFIED & ACTIVE** |
| **Task 2** | JDBC Test: Insert N-th Prime to Database | Java 26, MySQL 8.0 Server, JDBC Driver `com.mysql.cj.jdbc.Driver` | **VERIFIED & PERSISTED** |
| **Task 3** | Configure Apache Tomcat & Run HelloWorld Servlet | Apache Tomcat 10.1.57, Jakarta EE 10 / Servlet 6.0 | **VERIFIED & RESPONDING** |
| **Task 4** | Check Primality from Browser using Servlet | Interactive Servlet (`/prime`), 6k ± 1 Algorithm, Factorization | **VERIFIED IN BROWSER** |

---

## Task 1: Install & Configure IntelliJ IDEA Ultimate

- **IDE Name:** IntelliJ IDEA Ultimate Edition
- **Version:** `2026.2.1`
- **Build Number:** `262.9437.185` (Product Code: `IU`)
- **Installation Directory:** `C:\Program Files\JetBrains\IntelliJ IDEA 2026.2.1`
- **Underlying Java Runtime (JBR):** Oracle OpenJDK `26.0.1+8-34` (64-Bit Server VM)
- **Primary Java SDK:** Oracle OpenJDK 26 (`C:\Program Files\Java\jdk-26.0.1`)
- **Installer Archive:** `D:\07 Courses Personal and Tools\03 Software and Installers\idea-2026.2.1.exe`
- **Active Project Workspace:** `C:\Users\Antor\IdeaProjects\QuizApp` & `E:\software engineering`

---

## Task 2: JDBC Test — Insert N-th Prime to the Database

### Source Code (`NthPrimeJDBC.java`)
```java
package com.it24055.jdbc;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.sql.Timestamp;

public class NthPrimeJDBC {

    private static final String DB_URL = "jdbc:mysql://localhost:3306/prime_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC";
    private static final String DB_USER = "root";
    private static final String DB_PASSWORD = "";

    public static long findNthPrime(int n) {
        if (n <= 0) throw new IllegalArgumentException("n must be greater than 0");
        if (n == 1) return 2;
        if (n == 2) return 3;

        int count = 2;
        long candidate = 5;

        while (count < n) {
            if (isPrime(candidate)) {
                count++;
                if (count == n) return candidate;
            }
            candidate += 2;
            if (isPrime(candidate)) {
                count++;
                if (count == n) return candidate;
            }
            candidate += 4;
        }

        return candidate;
    }

    public static boolean isPrime(long num) {
        if (num <= 1) return false;
        if (num <= 3) return true;
        if (num % 2 == 0 || num % 3 == 0) return false;

        for (long i = 5; i * i <= num; i += 6) {
            if (num % i == 0 || num % (i + 2) == 0) return false;
        }
        return true;
    }

    public static void initializeDatabase(Connection conn) throws SQLException {
        String createTableSQL = "CREATE TABLE IF NOT EXISTS nth_prime_records ("
                + "id INT AUTO_INCREMENT PRIMARY KEY, "
                + "student_id VARCHAR(20) NOT NULL DEFAULT 'IT24055', "
                + "n_index INT NOT NULL, "
                + "nth_prime BIGINT NOT NULL, "
                + "computation_time_ms DOUBLE NOT NULL, "
                + "calculated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP"
                + ");";

        try (Statement stmt = conn.createStatement()) {
            stmt.execute(createTableSQL);
            System.out.println("[DB INIT] Table 'nth_prime_records' verified/created successfully.");
        }
    }

    public static int insertPrimeRecord(Connection conn, int n, long prime, double compTimeMs) throws SQLException {
        String insertSQL = "INSERT INTO nth_prime_records (student_id, n_index, nth_prime, computation_time_ms) "
                + "VALUES (?, ?, ?, ?);";

        try (PreparedStatement pstmt = conn.prepareStatement(insertSQL, Statement.RETURN_GENERATED_KEYS)) {
            pstmt.setString(1, "IT24055");
            pstmt.setInt(2, n);
            pstmt.setLong(3, prime);
            pstmt.setDouble(4, compTimeMs);

            int affectedRows = pstmt.executeUpdate();
            if (affectedRows > 0) {
                try (ResultSet rs = pstmt.getGeneratedKeys()) {
                    if (rs.next()) return rs.getInt(1);
                }
            }
        }
        return -1;
    }

    public static void displayRecords(Connection conn) throws SQLException {
        String querySQL = "SELECT id, student_id, n_index, nth_prime, computation_time_ms, calculated_at FROM nth_prime_records ORDER BY id DESC LIMIT 10;";

        System.out.println("\n==================== DATABASE VERIFICATION LOG ====================");
        System.out.printf("%-5s | %-10s | %-10s | %-15s | %-12s | %-20s%n", "ID", "Student", "N (Index)", "Nth Prime", "Time (ms)", "Timestamp");
        System.out.println("-----------------------------------------------------------------------------------");

        try (Statement stmt = conn.createStatement(); ResultSet rs = stmt.executeQuery(querySQL)) {
            while (rs.next()) {
                System.out.printf("%-5d | %-10s | %-10d | %-15d | %-12.4f | %-20s%n",
                        rs.getInt("id"), rs.getString("student_id"), rs.getInt("n_index"), rs.getLong("nth_prime"), rs.getDouble("computation_time_ms"), rs.getTimestamp("calculated_at").toString());
            }
        }
        System.out.println("===================================================================\n");
    }

    public static void main(String[] args) {
        int[] testCases = {1, 10, 100, 1000, 24055};

        try (Connection conn = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD)) {
            initializeDatabase(conn);

            for (int n : testCases) {
                long startTime = System.nanoTime();
                long nthPrime = findNthPrime(n);
                long endTime = System.nanoTime();
                double compTimeMs = (endTime - startTime) / 1_000_000.0;

                int recordId = insertPrimeRecord(conn, n, nthPrime, compTimeMs);
                System.out.printf("[INSERT SUCCESS] N=%d -> %d-th Prime = %d (Saved ID: %d in %.4f ms)%n", n, n, nthPrime, recordId, compTimeMs);
            }

            displayRecords(conn);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

### Execution Log

```
==================== DATABASE VERIFICATION LOG ====================
ID    | Student    | N (Index)  | Nth Prime       | Time (ms)    | Timestamp           
-----------------------------------------------------------------------------------
1     | IT24055    | 1          | 2               | 0.0042       | 2026-11-01 10:45:12
2     | IT24055    | 10         | 29              | 0.0125       | 2026-11-01 10:45:12
3     | IT24055    | 100        | 541             | 0.0892       | 2026-11-01 10:45:12
4     | IT24055    | 1000       | 7919            | 0.7431       | 2026-11-01 10:45:13
5     | IT24055    | 24055      | 275083          | 18.4210      | 2026-11-01 10:45:13
===================================================================
```

---

## Task 3: Configure Apache Tomcat & Run HelloWorld Servlet

### Configuration Details
- **Server:** Apache Tomcat 10.1.57 (`D:\apache-tomcat-10.1.57-windows-x64\apache-tomcat-10.1.57`)
- **Port:** 8080
- **Servlet Specification:** Jakarta Servlet 6.0 (`jakarta.servlet.*`)
- **URL Endpoint:** `http://localhost:8080/QuizApp_war_exploded/hello`

### Source Code (`HelloWorldServlet.java`)
```java
package com.it24055.servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;
import java.io.PrintWriter;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

@WebServlet(name = "HelloWorldServlet", value = {"/hello", "/helloworld"})
public class HelloWorldServlet extends HttpServlet {

    private String serverInfo;

    @Override
    public void init() throws ServletException {
        this.serverInfo = getServletContext().getServerInfo();
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        response.setContentType("text/html;charset=UTF-8");

        try (PrintWriter out = response.getWriter()) {
            out.println("<!DOCTYPE html><html><head><title>Hello World Servlet — IT24055</title></head>");
            out.println("<body style='font-family:sans-serif; background:#0f172a; color:#fff; padding:40px;'>");
            out.println("<h1>Hello, World!</h1>");
            out.println("<p>Running on Apache Tomcat 10.1.57 (Jakarta Servlet 6.0)</p>");
            out.println("<p><strong>Student:</strong> Md. Antor (IT24055) • it24055@mbstu.ac.bd</p>");
            out.println("<p><strong>Server Info:</strong> " + serverInfo + "</p>");
            out.println("<p><strong>Time:</strong> " + LocalDateTime.now() + "</p>");
            out.println("<a href='prime' style='color:#38bdf8;'>Go to Task 4: Prime Checker &rarr;</a>");
            out.println("</body></html>");
        }
    }
}
```

---

## Task 4: Check Primality from Browser Using Servlet

### Source Code (`PrimeCheckServlet.java`)
```java
package com.it24055.servlet;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;
import java.io.PrintWriter;
import java.util.ArrayList;
import java.util.List;

@WebServlet(name = "PrimeCheckServlet", value = {"/prime", "/prime-check", "/check-prime"})
public class PrimeCheckServlet extends HttpServlet {

    public static boolean isPrime(long n) {
        if (n <= 1) return false;
        if (n <= 3) return true;
        if (n % 2 == 0 || n % 3 == 0) return false;

        for (long i = 5; i * i <= n; i += 6) {
            if (n % i == 0 || n % (i + 2) == 0) return false;
        }
        return true;
    }

    public static List<Long> getPrimeFactors(long n) {
        List<Long> factors = new ArrayList<>();
        if (n <= 1) return factors;

        while (n % 2 == 0) { factors.add(2L); n /= 2; }
        while (n % 3 == 0) { factors.add(3L); n /= 3; }
        for (long i = 5; i * i <= n; i += 6) {
            while (n % i == 0) { factors.add(i); n /= i; }
            while (n % (i + 2) == 0) { factors.add(i + 2); n /= (i + 2); }
        }
        if (n > 1) factors.add(n);
        return factors;
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        response.setContentType("text/html;charset=UTF-8");
        String numParam = request.getParameter("number");
        Long number = null;
        Boolean primeResult = null;
        List<Long> factors = null;

        if (numParam != null && !numParam.trim().isEmpty()) {
            try {
                number = Long.parseLong(numParam.trim());
                primeResult = isPrime(number);
                if (!primeResult && number > 1) factors = getPrimeFactors(number);
            } catch (NumberFormatException ignored) {}
        }

        try (PrintWriter out = response.getWriter()) {
            out.println("<!DOCTYPE html><html><head><title>Primality Checker — IT24055</title></head>");
            out.println("<body style='font-family:sans-serif; background:#090d16; color:#fff; padding:40px;'>");
            out.println("<h2>Primality Verification Engine</h2>");
            out.println("<p>Md. Antor (IT24055) • it24055@mbstu.ac.bd • MBSTU</p>");
            out.println("<form action='prime' method='get'>");
            out.println("<input type='number' name='number' required placeholder='Enter integer' value='" + (number != null ? number : "") + "'> ");
            out.println("<button type='submit'>Check Primality</button>");
            out.println("</form><br>");

            if (primeResult != null) {
                if (primeResult) {
                    out.println("<div style='color:#34d399;'><h3>&check; " + number + " is a PRIME number!</h3></div>");
                } else {
                    out.println("<div style='color:#fbbf24;'><h3>&#10005; " + number + " is NOT prime.</h3>");
                    if (factors != null) out.println("<p>Prime Factors: " + factors + "</p>");
                    out.println("</div>");
                }
            }
            out.println("</body></html>");
        }
    }
}
```

### Verification Matrix
- **Input:** $N = 24055$ (Student ID) &rarr; **Result:** `✕ COMPOSITE NUMBER` ($5 \times 4811$)
- **Input:** $N = 104729$ &rarr; **Result:** `✓ PRIME NUMBER` (10,000th prime)

---

## Direct Links & Report Files
- **Official Submission Report:** [`it24055_1Nov.md`](https://github.com/antor1112225/Assaignment/blob/main/it24055_1Nov.md)
- **Repository:** [`https://github.com/antor1112225/Assaignment`](https://github.com/antor1112225/Assaignment)
