# Testing logstash pipelines
Guide to setup a logstash container to test pipelines. Once setup, you can edit the config file and send POST requests to the logstash container to see the output. It refreshes the configuration automatically.


Create a testing directory
```bash
mkdir logstash_testing
cd logstash_testing
```
Create a pipeline
```bash
cat <<EOF > test.conf
input {
    http {
       port => 8080
    }
}

filter {
    grok {
        match => { "message" => "%{GREEDYDATA:allofit}"}
    }
}


output {
    stdout {
        codec => rubydebug
    }
}
EOF
```
Run the logstash container in a speparate terminal
```bash
docker run --network=host -it -v ./:/mnt --rm logstash:8.16.1 logstash -f /mnt/test.conf --config.reload.automatic | grep -vF "logstash.licensechecker.licensereader"
```
Send a POST request to the logstash container
```bash
curl -X POST -d 'thisiswheretheloglinegoes' http://localhost:8080
```
Watch the logs of the container to see the output, for example:
```
{
       "allofit" => "thisiswheretheloglinegoes",
       "message" => "thisiswheretheloglinegoes",
          "host" => {
        "ip" => "0:0:0:0:0:0:0:1"
    },
           "url" => {
        "domain" => "localhost",
          "port" => 8080,
          "path" => "/"
    },
         "event" => {
        "original" => "thisiswheretheloglinegoes"
    },
    "user_agent" => {
        "original" => "curl/8.5.0"
    },
    "@timestamp" => 2024-11-26T17:18:54.526337616Z,
      "@version" => "1",
          "http" => {
         "method" => "POST",
        "version" => "HTTP/1.1",
        "request" => {
                 "body" => {
                "bytes" => "25"
            },
            "mime_type" => "application/x-www-form-urlencoded"
        }
    }
}
```
