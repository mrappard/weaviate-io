---
title: Java
sidebar_position: 3
image: og/docs/client-libraries.jpg
# tags: ['java', 'client library']
---
import Badges from '/_includes/badges.mdx';

<Badges/>

:::note Java client version
The current Java client version is `v||site.java_client_version||`.
:::

## Installation and setup
To get the latest stable version of the Java client library, add this dependency to your project:

```xml
<dependency>
  <groupId>technology.semi.weaviate</groupId>
  <artifactId>client</artifactId>
  <version>3.6.2</version>  <!-- Check latest version -->
</dependency>
```

This API client is compatible with Java 8 and beyond.

You can use the client in your project as follows:

```clike
import technology.semi.weaviate.client.Config;
import technology.semi.weaviate.client.WeaviateClient;
import technology.semi.weaviate.client.base.Result;
import technology.semi.weaviate.client.v1.misc.model.Meta;

public class App {
  public static void main(String[] args) {
    Config config = new Config("http", "localhost:8080");
    WeaviateClient client = new WeaviateClient(config);
    Result<Meta> meta = client.misc().metaGetter().run();
    if (meta.getError() == null) {
      System.out.printf("meta.hostname: %s\n", meta.getResult().getHostname());
      System.out.printf("meta.version: %s\n", meta.getResult().getVersion());
      System.out.printf("meta.modules: %s\n", meta.getResult().getModules());
    } else {
      System.out.printf("Error: %s\n", meta.getError().getMessages());
    }
  }
}
```

## Authentication

import ClientAuthIntro from '/developers/weaviate/client-libraries/_components/client.auth.introduction.mdx'

<ClientAuthIntro clientName="Java"/>

### WCS authentication

import ClientAuthWCS from '/developers/weaviate/client-libraries/_components/client.auth.wcs.mdx'

<ClientAuthWCS />

### Resource Owner Password Flow

import ClientAuthFlowResourceOwnerPassword from '/developers/weaviate/client-libraries/_components/client.auth.flow.resource.owner.password.mdx'

<ClientAuthFlowResourceOwnerPassword />

```java
import technology.semi.weaviate.client.Config;
import technology.semi.weaviate.client.WeaviateAuthClient;

Config config = new Config("http", "weaviate.example.com:8080");
WeaviateAuthClient.clientPassword(
    config,
    "Your user",
    "Your password",
    Arrays.asList("scope1", "scope2") // optional, depends on the configuration of your identity provider (not required with WCS)
);
```

### Client credentials flow

import ClientAuthFlowClientCredentials from '/developers/weaviate/client-libraries/_components/client.auth.flow.client.credentials.mdx'

<ClientAuthFlowClientCredentials />

```java
import technology.semi.weaviate.client.Config;
import technology.semi.weaviate.client.WeaviateAuthClient;

Config config = new Config("http", "weaviate.example.com:8080");
WeaviateAuthClient.clientCredentials(
    config,
    "your_client_secret",
    Arrays.asList("scope1" ,"scope2") // optional, depends on the configuration of your identity provider
);
```

### Refresh Token flow

import ClientAuthBearerToken from '/developers/weaviate/client-libraries/_components/client.auth.bearer.token.mdx'

<ClientAuthBearerToken />

```java
import technology.semi.weaviate.client.Config;
import technology.semi.weaviate.client.WeaviateAuthClient;

Config config = new Config("http", "weaviate.example.com:8080");
WeaviateAuthClient.bearerToken(
    config,
    "Your_access_token",
    500,  // lifetime in seconds
    "Your_refresh_token",
);
```

## Custom headers 

You can pass custom headers to the client, which are added at initialization:

```java
import technology.semi.weaviate.client.Config;
import technology.semi.weaviate.client.WeaviateClient;

public class App {
  public static void main(String[] args) {
    Map<String, String> headers = new HashMap<String, String>() { {
      put("header_key", "value");
    } };
    Config config = new Config("http", "localhost:8080", headers);
    WeaviateClient client = new WeaviateClient(config);
  }
}
```

## References

All [RESTful endpoints](../api/rest/index.md) and [GraphQL functions](../api/graphql/index.md) references covered by the Java client, and explained on those reference pages in the code blocks.

## Design

### Builder pattern

The Java client functions are designed with a 'Builder pattern'. A pattern is used to build complex query objects. This means that a function (for example to retrieve data from Weaviate with a request similar to a RESTful GET request, or a more complex GraphQL query) is built with single objects to reduce complexity. Some builder objects are optional, others are required to perform specific functions. All is documented on the [RESTful API reference pages](../api/rest/index.md) and the [GraphQL reference pages](../api/graphql/index.md).

The code snippet above shows a simple query similar to `RESTful GET /v1/meta`. The client is initiated by requiring the package and connecting to the running instance. Then, a query is constructed by using the `.metaGetter()` on `.misc()`. The query will be sent with the `.run()` function, this object is thus required for every function you want to build and execute. 

## Migration Guides

### From `2.4.0` to `3.0.0`

#### Removed @Deprecated method `Aggregate::withFields(Fields fields)`

Before:
```java
// import technology.semi.weaviate.client.v1.graphql.query.fields.Field;
// import technology.semi.weaviate.client.v1.graphql.query.fields.Fields;

Fields fields = Fields.builder().fields(new Field[]{name, description}).build();
client.graphQL().aggregate().withFields(fields)...
```

After:
```java
client.graphQL().aggregate().withFields(name, description)...
```

#### Removed @Deprecated method `Get::withFields(Fields fields)`

Before:
```java
// import technology.semi.weaviate.client.v1.graphql.query.fields.Field;
// import technology.semi.weaviate.client.v1.graphql.query.fields.Fields;

Fields fields = Fields.builder().fields(new Field[]{name, description}).build();
client.graphQL().get().withFields(fields)...
```

After:
```java
client.graphQL().get().withFields(name, description)...
```

#### Removed @Deprecated method `Get::withNearVector(Float[] vector)`

Before:
```java
client.graphQL().get().withNearVector(new Float[]{ 0f, 1f, 0.8f })...
```

After:
```java
// import technology.semi.weaviate.client.v1.graphql.query.argument.NearVectorArgument;

NearVectorArgument nearVector = NearVectorArgument.builder().vector(new Float[]{ 0f, 1f, 0.8f }).certainty(0.8f).build();
client.graphQL().get().withNearVector(nearVector)...
```

#### All `where` filters use the same implementation

With `batch delete` feature, unified `filters.WhereFilter` implementation is introduced, which replaces `classifications.WhereFilter`, `graphql.query.argument.WhereArgument` and `graphql.query.argument.WhereFilter`.

##### GraphQL

Before:
```java
// import technology.semi.weaviate.client.v1.graphql.query.argument.GeoCoordinatesParameter;
// import technology.semi.weaviate.client.v1.graphql.query.argument.WhereArgument;
// import technology.semi.weaviate.client.v1.graphql.query.argument.WhereOperator;

GeoCoordinatesParameter geo = GeoCoordinatesParameter.builder()
    .latitude(50.51f)
    .longitude(0.11f)
    .maxDistance(3000f)
    .build();
WhereArgument where = WhereArgument.builder()
    .valueGeoRange(geo)
    .operator(WhereOperator.WithinGeoRange)
    .path(new String[]{ "add "})
    .build();

client.graphQL().aggregate().withWhere(where)...
```

After:
```java
// import technology.semi.weaviate.client.v1.filters.Operator;
// import technology.semi.weaviate.client.v1.filters.WhereFilter;

WhereFilter where = WhereFilter.builder()
    .valueGeoRange(WhereFilter.GeoRange.builder()
        .geoCoordinates(WhereFilter.GeoCoordinates.builder()
            .latitude(50.51f)
            .longitude(0.11f)
            .build()
        )
        .distance(WhereFilter.GeoDistance.builder()
            .max(3000f)
            .build()
        )
        .build()
    )
    .operator(Operator.WithinGeoRange)
    .path(new String[]{ "add" })
    .build();

client.graphQL().aggregate().withWhere(where)...    
```

Before:
```java
// import technology.semi.weaviate.client.v1.graphql.query.argument.WhereArgument;
// import technology.semi.weaviate.client.v1.graphql.query.argument.WhereOperator;

WhereArgument where = WhereArgument.builder()
    .valueString("txt")
    .operator(WhereOperator.Equal)
    .path(new String[]{ "add" })
    .build();

client.graphQL().aggregate().withWhere(where)...
```

After:
```java
// import technology.semi.weaviate.client.v1.filters.Operator;
// import technology.semi.weaviate.client.v1.filters.WhereFilter;

WhereFilter where = WhereFilter.builder()
    .valueString("txt")
    .operator(Operator.Equal)
    .path(new String[]{ "add" })
    .build();

client.graphQL().aggregate().withWhere(where)...
```

Before:
```java
// import technology.semi.weaviate.client.v1.graphql.query.argument.WhereArgument;
// import technology.semi.weaviate.client.v1.graphql.query.argument.WhereFilter;
// import technology.semi.weaviate.client.v1.graphql.query.argument.WhereOperator;

WhereArgument where = WhereArgument.builder()
    .operands(new WhereFilter[]{
        WhereFilter.builder()
            .valueInt(10)
            .path(new String[]{ "wordCount" })
            .operator(WhereOperator.LessThanEqual)
            .build(),
        WhereFilter.builder()
            .valueString("word")
            .path(new String[]{ "word" })
            .operator(WhereOperator.LessThan)
            .build()
    })
    .operator(WhereOperator.And)
    .build();

client.graphQL().aggregate().withWhere(where)...
```

After:
```java
// import technology.semi.weaviate.client.v1.filters.Operator;
// import technology.semi.weaviate.client.v1.filters.WhereFilter;

WhereFilter where = WhereFilter.builder()
    .operands(new WhereFilter[]{
        WhereFilter.builder()
            .valueInt(10)
            .path(new String[]{ "wordCount" })
            .operator(Operator.LessThanEqual)
            .build(),
        WhereFilter.builder()
            .valueString("word")
            .path(new String[]{ "word" })
            .operator(Operator.LessThan)
            .build(),
    })
    .operator(Operator.And)
    .build();

client.graphQL().aggregate().withWhere(where)...
```

##### Classification

Before:
```java
// import technology.semi.weaviate.client.v1.classifications.model.GeoCoordinates;
// import technology.semi.weaviate.client.v1.classifications.model.Operator;
// import technology.semi.weaviate.client.v1.classifications.model.WhereFilter;
// import technology.semi.weaviate.client.v1.classifications.model.WhereFilterGeoRange;
// import technology.semi.weaviate.client.v1.classifications.model.WhereFilterGeoRangeDistance;

WhereFilter where = WhereFilter.builder()
    .valueGeoRange(WhereFilterGeoRange.builder()
        .geoCoordinates(GeoCoordinates.builder()
            .latitude(50.51f)
            .longitude(0.11f)
            .build())
        .distance(WhereFilterGeoRangeDistance.builder()
            .max(3000d)
            .build())
        .build())
    .operator(Operator.WithinGeoRange)
    .path(new String[]{ "geo" })
    .build();

client.classifications().scheduler().withTrainingSetWhereFilter(where)...
```

After:
```java
// import technology.semi.weaviate.client.v1.filters.Operator;
// import technology.semi.weaviate.client.v1.filters.WhereFilter;

WhereFilter where = WhereFilter.builder()
    .valueGeoRange(WhereFilter.GeoRange.builder()
        .geoCoordinates(WhereFilter.GeoCoordinates.builder()
            .latitude(50.51f)
            .longitude(0.11f)
            .build())
        .distance(WhereFilter.GeoDistance.builder()
            .max(3000f)
            .build())
        .build())
    .operator(Operator.WithinGeoRange)
    .path(new String[]{ "geo" })
    .build();

client.classifications().scheduler().withTrainingSetWhereFilter(where)...
```


## Change logs


Check the [change logs on GitHub](https://github.com/weaviate/weaviate-java-client/releases) for updates on the latest `Java client` changes.

## More Resources

import DocsMoreResources from '/_includes/more-resources-docs.md';

<DocsMoreResources />
