---
title: JDBC 和 MyBatis 性能比较
date: 2017-06-12 13:54:12
tags: Java
---

以下为 JDBC 和 MyBatis 的性能比较参考，MyBatis 的性能比 JDBC 大概慢三分之一，看上去相差挺大的，不过网络才是对效率影响最大的因素，局域网中有 200 多倍的影响，这么比较起来，MyBatis 和 JDBC 本身的效率差距可以忽略不计了。

测试 2 种情况 (每条记录有 7 个字段)

* 向本机的数据库插入 66720 条记录
* 向局域网中其他机器上的数据库插入 20000 条记录

![](/img/java/jdbc-mytabis-performance.png)

> 在向局域网中其他机器上不停的插入数据时，2 台机器的 CPU 占用都很小，也就百分之几，因为大多数时候都在等待 IO。<!--more-->

* 使用 JDBC 和 MyBatis 向本机的数据库插入数据

  * JDBC

    * 不使用事务: 插入 66720 个，使用了 11827 毫秒，11 秒

    * 使用事务，500 个提交 1 次: 插入 66720 个，使用了 5256 毫秒，5 秒

      ```java
      public void insertEnrollments() throws Exception {
          new Executor(enrollmentService).execute((enrollments) -> {
              Connection conn = DbUtils.getConnection();

              for (Enrollment enrollment : enrollments) {
                  insertEnrollment(enrollment, conn);
              }

              DbUtils.close(null, null, conn);
          });
      }

      public void insertEnrollmentsWithTransaction() throws Exception {
          new Executor(enrollmentService).execute((enrollments) -> {
              int length = enrollments.size();
              Connection conn = DbUtils.getConnection();

              conn.setAutoCommit(false);
              for (int i = 0; i < length; i++) {
                  insertEnrollment(enrollments.get(i), conn);

                  if (i > 0 && i % MAX_COUNT == 0) {
                      conn.commit();
                  }
              }
              conn.commit();

              DbUtils.close(null, null, conn);
          });
      }
      ```

  * MyBatis

    * 不使用事务: 插入 66720 个，使用了 12872 毫秒，12 秒

    * 使用事务，1 个提交 1 次: 插入 66720 个，使用了 23286 毫秒，23 秒

    * 使用事务，500 个提交 1 次: 插入 66720 个，使用了 6853 毫秒，6 秒

      ```java
      public void insertEnrollment(Enrollment enrollment) {
          mapper.insertEnrollment(enrollment);
      }

      @Transactional
      public void insertEnrollmentWithTransaction(Enrollment enrollment) {
          mapper.insertEnrollment(enrollment);
      }

      @Transactional
      public void insertEnrollmentsWithTransaction(List<Enrollment> enrollments) {
          for (Enrollment enrollment : enrollments) {
              mapper.insertEnrollment(enrollment);
          }
      }
      ```

* 使用 JDBC 和 MyBatis 向局域网里其他机器上的数据库插入数据

  * JDBC

    * 使用事务，500 个提交 1 次: 插入 20000 个，使用了 614386 毫秒，614 秒

      > 本机时只使用了 2307 毫秒，2 秒

  * MyBatis

    * 使用事务，500 个提交 1 次: 插入 20000 个，使用了 804749 毫秒，804 秒

      > 本机时只使用了 3216 毫秒，3 秒