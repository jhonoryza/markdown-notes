---
title: 'Tips develop java di Intellij IDEA'
date: '2023-08-10 14:58:00'
---

Kali ini kita akan membahas beberapa tips yg berguna ketika develop java menggunakan IDE jetbrains.

## cara ubah java version
1. buka file -> project structure

![image](./jvm-1.png)

2. buka preferences -> build execution deployment -> build tools -> gradle

![image](./jvm-2.png)

## postgres application properties

```env
spring.datasource.username=${DB_USERNAME:postgres}
spring.datasource.password=${DB_PASSWORD:}
spring.datasource.url=${DB_URL:jdbc:postgresql://localhost:5432/java-api}
spring.jpa.hibernate.ddl-auto=update
```
