# Apache Kafka to HDFS Ingestion
Apache Gobblin cesitli data kaynaklarinda bulunan datalari Hadoop Ecosystem icerisine cekmek icin kullanilan bir aractir. LinkedIn tarafindan gelistirilmistir. [https://gobblin.readthedocs.io/en/latest/](https://gobblin.readthedocs.io/en/latest/)
## Requirements
### 1. Apache Gobblin
1. Apache Gobblin'i indir.

    `wget https://github.com/apache/incubator-gobblin/releases/download/gobblin_0.10.0/gobblin-distribution-0.10.0.tar.gz`
2. Gobblin'i tar'dan cikart

    `tar -xzvf gobblin-distribution-0.10.0.tar.gz`
3.  `cd gobblin-dist`
4.  Gobblin'in dogru kurulup kurulmadigini test etmek icin kolay test edilebilir(onceden konfigurasyonlari yapilmis uygulamayi test et)
    `bin/gobblin run listQuickApps`
    `bin/gobblin run wikipedia -lookback P10D LinkedIn Wikipedia:Sandbox`
Komut basarili bir sekilde calistiysa bir sonraki adimla devam edebilirsiniz. Turkiye wikipedia'nin kapali olmasi sebebiyle(25.06.2018) hata aliyor olabilirsiniz. Eger wikipedia siz test ederken kapali ise vpn kullanarak komutu tekrar calistirmayi deneyin.

### 2.Apache Gobblin Configuration Files
Kafka'dan datalari HDFS'e atmak icin bir configurasyon dosyasina ihtiyac vardir. Bu uygulama icin `GobblinKafkaQuickStart.pull` ismini kullandik. Bu dosyanin icerigi asagidaki gibi olmalidir.
```
job.name=GobblinKafkaQuickStart
job.group=GobblinKafka
job.description=Gobblin quick start job for Kafka
job.lock.enabled=false

kafka.brokers=localhost:9092

source.class=gobblin.source.extractor.extract.kafka.KafkaSimpleSource
extract.namespace=gobblin.extract.kafka

writer.builder.class=gobblin.writer.SimpleDataWriterBuilder
writer.file.path.type=tablename
writer.destination.type=HDFS
writer.output.format=txt

data.publisher.type=gobblin.publisher.BaseDataPublisher

mr.job.max.mappers=1

metrics.reporting.file.enabled=true
metrics.log.dir=/gobblin-kafka/metrics
metrics.reporting.file.suffix=txt

bootstrap.with.offset=earliest

fs.uri=hdfs://localhost:9000
writer.fs.uri=hdfs://localhost:9000
state.store.fs.uri=hdfs://localhost:9000

mr.job.root.dir=/gobblin-kafka/working
state.store.dir=/gobblin-kafka/state-store
task.data.root.dir=/jobs/kafkaetl/gobblin/gobblin-kafka/task-data
data.publisher.final.dir=/gobblintest/job-output
```
### 3.Kafka to HDFS Ingestion'un Calistirilmasi
Configurasyon dosyasinin bulundugu dosya yoluna gore asagidaki komutu duzenleyiniz.
`bin/gobblin run -jobName GobblinKafkaQuickStart -jobFile file:///home/ubuntu/Desktop/GobblinConfFiles/GobblinKafkaQuickStart.pull`

### 4.Kafka to HDFS Ingestion'un Calistirilma Zamaninin Belirlenmesi
Apache Kafka uzerinde mesajlar farkli surelerde tutulabilir. Default olarak Apache Kafka 7 gun boyunca data tutar. 7 gun sonunda datayi siler. Mesajlar silinmeden Kafka to HDFS Ingestion calistirilmali ve data HDFS'e atilmalidir.
Bu yüzden crontab kullanilabilir.
Ornek:
`crontab -e` komutunu calistirin. Acilan dosyaya:

`59 17 * * * /home/ubuntu/gobblin-dist/bin/gobblin run -jobName GobblinKafkaQuickStart -jobFile file:///home/ubuntu/Desktop/GobblinConfFiles/GobblinKafkaQuickStart.pull
`
satiri eklenmelidir.


