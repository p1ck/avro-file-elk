# Get Avro files into ELK
I’ve been researching the best way to get our stored Avro data files into Elasticsearch via Logstash.  These are some notes on that.

First, there is no obvious way to have Logstash open arbitrary Avro files and read them.  This seems technically straightforward since Avro files have self-contained schemas, but I believe we would need a custom file input to accomplish this.

There was a Logstash [issue](https://github.com/elastic/logstash/issues/1566) discussing this, which resulted in / was obviated by an [Avro codec plugin](https://github.com/logstash-plugins/logstash-codec-avro).  This is great, but it assumes you are publishing your data on Kafka, with one topic per schema, not reading from files.  The Logstash file plugin assumes you are reading from a delimited file.

## Avro codec via Kafka 
### Pros
* can use Avro schema resolution to filter fields
* architecturally sexy
  * you can do many other things with message queues
* exists

### Cons
* must know the schema in advance
  * What do you think this is, Protobuf?
* must republish files into Kafka
* overhead
  * must maintain Logstash config and Avro schema files
  * must run Kafka

## Avro file reader plugin
### Pros
* zero config!
  * read the schema from the file on the fly
* could (optionally) use Avro schema resolution to filter fields

### Cons
* does not exist

## Give up and convert to JSON/CSV first 
### Pros
* exists
  * logstash-plugin-stdin
  * logstash-plugin-file

### Cons
* self loathing

## Hacks considered
* use the multiline file codec plugin to trick the file reader into not looking for delimiters in Avro files
  * still requires changes to the Avro codec
  * bad for large files (no delimiters = try to load whole file in memory?)
* external file reader could peek at schemas and dynamically configure Logstash
  * still requires Kafka
  * unnecessarily complex

## Other things
* Logstash is changing the way [contrib plugins](https://www.elastic.co/blog/plugin-ecosystem-changes) work in 1.5, so plugins are in transition.  I was only able to get the Avro codec to work in the 1.5 release candidate 3, and not in the latest 1.4 stable release.  This may be a problem for existing workflows.