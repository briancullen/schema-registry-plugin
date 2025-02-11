![Github Actions](https://github.com/ImFlog/schema-registry-plugin/workflows/Master/badge.svg)

# Schema-registry-plugin
The aim of this plugin is to adapt the [Confluent schema registry maven plugin](https://docs.confluent.io/current/schema-registry/docs/maven-plugin.html) for Gradle builds.

## Usage
See [gradle plugins portal](https://plugins.gradle.org/plugin/com.github.imflog.kafka-schema-registry-gradle-plugin)
for instructions about how to add the plugin to your build configuration.

Also, do not forget to add the confluent repository in your buildscript:
```groovy
buildscript {
    repositories {
        maven {
            url "https://plugins.gradle.org/m2/"
        }
        maven {
            url "http://packages.confluent.io/maven/"
        }
        maven {
            url = uri("https://jitpack.io")
        }
  }
    dependencies {
        classpath "com.github.imflog:kafka-schema-registry-gradle-plugin:X.X.X"
    }
}
```

## Tasks
When you install the plugin, four tasks are added under `registry` group:
* downloadSchemasTask
* testSchemasTask
* registerSchemasTask
* configSubjectsTask

What these tasks do and how to configure them is described in the following sections.

### Download schemas
Like the name of the task imply, this task is responsible for retrieving schemas from a schema registry.

A DSL is available to configure the task:
```groovy
schemaRegistry {
    url = 'http://registry-url:8081/'
    download {
        // extension of the output file depends on the the schema type
        subject('avroSubject', '/absolutPath/src/main/avro')
        subject('protoSubject', 'src/main/proto')
        subject('jsonSubject', 'src/main/json')
        
        // You can use a regex to download multiple schemas at once
        subjectPattern('avro.*', 'src/main/avro')
    }
}
```
You have to put the url where the script can reach the Schema Registry.

You need to specify the pairs (subjectName, outputDir) for all the schemas you want to download. 

### Test schemas compatibility
This task test compatibility between local schemas and schemas stored in the Schema Registry.

A DSL is available to specify what to test:
```groovy
schemaRegistry {
    url = 'http://registry-url:8081'
    compatibility {
        subject('avroWithDependencies', '/absolutPath/dependent/path.avsc', "AVRO").addReference('avroSubject', 'avroSubjectType', 1)
        subject('protoWithDependencies', 'dependent/path.proto', "PROTOBUF").addReference('protoSubject', 'protoSubjectType', 1)
        subject('jsonWithDependencies', 'dependent/path.json', "JSON").addReference('jsonSubject', 'jsonSubjectType', 1)
    }
}
```
You have to put the url where the script can reach the Schema Registry.

You have to list all the (subject, avsc file path) pairs that you want to test. 

If you have dependencies with other schemas required before the compatibility check,
you can call the `addReference("name", "subject", version)`, this will add a reference to fetch dynamically from the registry.
The addReference calls can be chained.

### Register schemas
Once again the name speaks for itself.
This task register schemas from a local path to a Schema Registry.

A DSL is available to specify what to register:
```groovy
schemaRegistry {
    url = 'http://registry-url:8081'
    register {
        subject('avroWithDependencies', '/absolutPath/dependent/path.avsc', "AVRO").addReference('avroSubject', 'avroSubjectType', 1)
        subject('protoWithDependencies', 'dependent/path.proto', "PROTOBUF").addReference('protoSubject', 'protoSubjectType', 1)
        subject('jsonWithDependencies', 'dependent/path.json', "JSON").addReference('jsonSubject', 'jsonSubjectType', 1)
    }
}
```
You have to put the url where the script can reach the Schema Registry.

You have to list all the (subject, avsc file path) pairs that you want to send.

If you have dependencies with other schemas required before the compatibility check,
you can call the `addReference("name", "subject", version)`, this will add a reference to fetch dynamically from the registry.
The addReference calls can be chained.

### Configure subjects
This task sets the schema compatibility level for registered subjects.

A DSL is available to specify which subjects to configure:
```groovy
schemaRegistry {
    url = 'http://registry-url:8081'
    config {
        subject('mySubject', 'FULL_TRANSITIVE')
        subject('otherSubject', 'FORWARD')
    }
}
```

See the Confluent
[Schema Registry documentation](https://docs.confluent.io/current/schema-registry/avro.html#compatibility-types)
for more information on valid compatibility levels.

You have to put the URL where the script can reach the Schema Registry.

You have to list the (subject, compatibility-level) 

## Security
According to how your schema registry instance security configuration,
you can configure the plugin to access it securely.

### Basic authentication
An extension allow you to specify the basic authentication like so:
```groovy
schemaRegistry {
    url = 'http://registry-url:8081'
    credentials {
        username = '$USERNAME'
        password = '$PASSWORD'
    }
}
```

### Encryption (SSL)
If you want to encrypt the traffic in transit (using SSL), use the following extension:
```groovy
schemaRegistry {
    url = 'https://registry-url:8081'
    ssl {
        configs = [
            "ssl.truststore.location": "/path/to/registry.truststore.jks",
            "ssl.truststore.password": "truststorePassword",
            "ssl.keystore.location": "/path/to/registry.keystore.jks",
            "ssl.keystore.password": "keystorePassword"
        ]
    }
}
```
Valid key values are listed here: [org.apache.kafka.common.config.SslConfigs](https://github.com/confluentinc/kafka/blob/master/clients/src/main/java/org/apache/kafka/common/config/SslConfigs.java)

### Examples
See the [examples](examples) directory to see the plugin in action !

## Version compatibility
When using the plugin, a default version of the confluent Schema registry is use.
The 5.5.X version of the schema-registry introduced changes that made the older version of the plugin obsolete.

It was easier to introduce all the changes in one shot instead of supporting both version. Here is what it implies for users:
* plugin versions above 1.X.X support the confluent version 5.5.X (Avro / Json / Protobuf)
* plugin versions should supports anything below 5.4.X

We are not strictly following confluent version so if you need to change the confluent version for some reason,
take a look at [examples/override-confluent-version](examples/override-confluent-version).

## Developing
In order to build the plugin locally, you can run the following commands:
```bash
./gradlew build # To compile and test the code
./gradlew publishToMavenLocal # To push the plugin to your mavenLocal
```

Once the plugin is pushed into your mavenLocal, you can use it by 
adding the `mavenLocal` to the buildscript repositories like so:
```groovy
buildscript {
    repositories {
        // The new repository to import, you may not want this in your final gradle configuration.
        mavenLocal()
        maven {
            url "http://packages.confluent.io/maven/"
        }
  }
    dependencies {
        classpath "com.github.imflog:kafka-schema-registry-gradle-plugin:X.X.X-SNAPSHOT"
    }
}
```

