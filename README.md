Logback RollingPolicy with S3 upload
==========

logback-s3 is a Logback RollingPolicy that automaitcally uploads rolled log files to S3.
As S3FixedWindowRollingPolicy extends FixedWindowRollingPolicy, it works exactly same as FixedWindowRollingPolicy apart from uploading log files to S3.

logback-s3 is implemented based on Logpic (https://github.com/mweagle/Logpig) but simplified for my needs.

Example logback.xml
----------
An example logback.xml that uses `S3FixedWindowRollingPolicy` with `RollingFileAppender`.

```
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
  <file>/var/log/myapp.log</file>
  <encoder>
    <pattern>%d\t%thread\t%level\t%logger\t%msg%n</pattern>
  </encoder>

  <!--
    Policy to upload a log file into S3 on log rolling or JVM exit.
    * On each log rolling, a rolled log file is created locally and uploaded to S3
    * When <rollingOnExit> is true, log rolling occurs on JVM exit and a rolled log is uploaded (default)
    * When <rollingOnExit> is false, the active log file is uploaded as it is
  -->
  <rollingPolicy class="ch.qos.logback.core.rolling.S3FixedWindowRollingPolicy">
    <fileNamePattern>/var/log/myapp.log.%i.gz</fileNamePattern>
        <awsAccessKey>accesskey</awsAccessKey>
        <awsSecretKey>secretkey</awsSecretKey>
        <s3BucketName>com.mybucket</s3BucketName>
        <s3FolderName>log</s3FolderName>

        <rollingOnExit>true</rollingOnExit>
    </rollingPolicy>

    <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
        <maxFileSize>10MB</maxFileSize>
    </triggeringPolicy>
</appender>
```

By suppressing the log rolling as follows, the appender works exactly same as `FileAppender` plus S3 upload.

```
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">

  ...

  <rollingPolicy class="ch.qos.logback.core.rolling.S3FixedWindowRollingPolicy">
    <!-- this file name won't be used since log rolling is suppressed -->
    <fileNamePattern>/var/log/myapp.log.%i.gz</fileNamePattern>

        ...

        <rollingOnExit>false</rollingOnExit>

        <!--
            Suppress rolling.
            Note that rollingOnExit = true does not work with this config.
         -->
        <maxIndex>-1</maxIndex>
        <minIndex>-1</minIndex>
    </rollingPolicy>

    ...

</appender>
```


It is a good idea to create an IAM user only allowed to upload S3 object to a specific S3 bucket.
It improves the control and reduces the risk of unauthorized access to your S3 bucket.

The following is an example IAM policy.
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "s3:PutObject"
      ],
      "Sid": "Stmt1378251801000",
      "Resource": [
        "arn:aws:s3:::com.mybucket/log/*"
      ],
      "Effect": "Allow"
    }
  ]
}
```