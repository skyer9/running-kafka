# filebeat with IIS 와 kafka 연동하기

## 1. filebeat 설치하기

- https://www.elastic.co/kr/downloads/beats/filebeat 에서 filebeat 를 다운받는다.

- C:\Program Files 에 압출을 풀고, ```filebeat-<version>-windows``` 을 ```Filebeat``` 로 이름을 변경한다.

- PowerShell 을 관리자권한으로 연다.

- 다음 코드를 실행한다.

```powershell
PS > cd 'C:\Program Files\Filebeat'
PS C:\Program Files\Filebeat> .\install-service-filebeat.ps1
```

```이 시스템에서 스크립트를 실행할 수 없으므로``` 오류가 발생할 경우 아래 내용을 실행한다.

```powershell

PS C:\Program Files\Filebeat> Get-ExecutionPolicy
Restricted
PS C:\Program Files\Filebeat> Set-ExecutionPolicy RemoteSigned
PS C:\Program Files\Filebeat> .\install-service-filebeat.ps1
PS C:\Program Files\Filebeat> Set-ExecutionPolicy Restricted
PS C:\Program Files\Filebeat> Start-Service filebeat

```

## 2. filebeat with IIS 와 kafka 연동하기

- filebeat.yml 을 수정한다.

```powershell

filebeat.inputs:
- type: log
  enabled: true
  paths:
    - C:\inetpub\logs\LogFiles\W3SVC2\*
  ignore_older: 2h
  document_type: iis_log
  fields:
    server_domain_name: "www.test.co.kr"
- type: log
  enabled: false
  paths:
    - "/var/log/apache2/*"
  fields:
    apache: true
  fields_under_root: true
output.kafka:
  hosts: ["192.168.56.103:9092"]
  topic: 'apache_logs'
  partition.round_robin:
    reachable_only: false
  required_acks: 1
  compression: gzip
  max_message_bytes: 1000000
#output.elasticsearch:
#  hosts: ["172.111.10.10:9200"]
#output.logstash:
#  hosts: ["61.222.133.17:5044"]

```

서비스를 시작한다.

```powershell

PS C:\Program Files\Filebeat> Start-Service filebeat

```

## 3. kafka 와 logstash 연동하기

- logstash 설정파일을 생성한다.

```sh

$ vi test.conf
------------------------------------------------------------
input {
  kafka {
    zk_connect => "hostname:port"
    topic_id => "apache_logs"
  }
  #beats {
  #  port => "5044"
  #}
}

filter {
  grok {
    # IIS log 파일 상단에 로그 포멧이 있다.
    match => [
      "message", "%{TIMESTAMP_ISO8601:log_timestamp} %{IP:hostip} %{URIPROTO:method} %{URIPATH:request} (?:%{NOTSPACE:queryparam}|-) %{NUMBER:port} (?:%{WORD:username}|-) %{IP:clientip} %{NOTSPACE:user-agent} (?:%{NOTSPACE:referer}|-) %{NUMBER:status} %{NUMBER:sub-status} %{NUMBER:win32-status} %{NUMBER:time-taken}",
      "message", "%{TIMESTAMP_ISO8601:log_timestamp} %{IP:hostip} %{URIPROTO:method} %{URIPATH:request} (?:%{NOTSPACE:queryparam}|-) %{NUMBER:port} (?:%{WORD:username}|-) %{IP:clientip} %{NOTSPACE:user-agent} %{NUMBER:status} %{NUMBER:sub-status} %{NUMBER:win32-status} %{NUMBER:time-taken}"
    ]
  }

  date {
    match => [ "log_timestamp", "YYYY-MM-dd HH:mm:ss" ]
    # http://joda-time.sourceforge.net/timezones.html
    timezone => "GMT"
    # timezone => "Asia/Seoul"
    locale => "en"
  }
}

output {
  if [fields.server_domain_name] == "www.test.co.kr" {
    elasticsearch {
      hosts => ["61.222.133.17:9200"]
      index => "www.test.co.kr-%{+YYYY.MM.dd}"
      #template => "/home/docker/logstash/conf/template-weblog-one.json"
      #template_name => "testscmlog-one-*"
    }
  } else if [fields.server_domain_name] == "admin.test.co.kr" {
    elasticsearch {
      hosts => ["61.222.133.17:9200"]
      index => "admin.test.co.kr-%{+YYYY.MM.dd}"
      #template => "/home/docker/logstash/conf/template-weblog-one.json"
      #template_name => "testscmlog-one-*"
    }
  }
  #stdout { codec => rubydebug }
}
------------------------------------------------------------

```
