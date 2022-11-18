# Geolocation GraphQL Interface

⚠️ **This is an ongoing, unsupported, unfinished and undocumented project. We do not guarantee any results after installation.** ⚠️

This app exports a GraphQL schema for geolocation results for Place Components.

## Usage

### Installation

To use it in your app, run the `vtex add vtex.geolocation-graphql-interface` command.

You may then use it in your front end component queries, for example, by writing the file `addressSuggestions.graphql`:

```graphql
query AddressSuggestions($searchTerm: String!) {
  addressSuggestions(searchTerm: $searchTerm)
    @context(provider: "vtex.geolocation-graphql-interface") {
    description
    mainText
    mainTextMatchInterval {
      offset
      length
    }
    secondaryText
    externalId
  }
}
```

> 9 The only difference from a standard query is the usage of a _directive_: `@context(provider: "vtex.geolocation-graphql-interface")`

To resolve this query, you must have installed in your workspace an app that implements the GraphQL interface. Currently, those are the available resolvers:

- [vtex.google-geolocation-resolver](https://github.com/vtex-apps/google-geolocation-resolver)

Having zero or more than one resolver installed will result in an error.

### Examples

Currently, this interface only supports autosuggest related operations, which is commonly used along with a text-based input like the [Location Search component](https://github.com/vtex-apps/place-components/#locationfield-search) used in `vtex.place-components`. In order to get the suggestions, each key-stroke might send a `addressSuggestions` query like:

```graphql
query {
  addressSuggestions(searchTerm: "Praia de bot") {
    description
    externalId
  }
}
```

This returns address suggestions using the resolver of your choice. Notice that the number of suggestions are handled by the resolver itself.

```graphql
{
  "data": {
    "addressSuggestions": [
      {
        "description": "Praia de Botafogo - Botafogo, Rio de Janeiro - RJ, Brasil",
        "externalId": "SAMPLE_ID_2718"
      },
      {
        "description": "Praia de Botafogo - Pista Direita - Botafogo, Rio de Janeiro - RJ, Brasil",
        "externalId": "SAMPLE_ID_0042"
      },
      {
        "description": "Alameda Praia de Botafogo - Patamares, Salvador - BA, Brasil",
        "externalId": "SAMPLE_ID_1123"
      },
      {
        "description": "Acesso Praia de Botafogo - Botafogo, Rio de Janeiro - RJ, Brasil",
        "externalId": "SAMPLE_ID_1001"
      },
      {
        "description": "Rua Praia de Botafogo - Vilatur, Saquarema - RJ, Brasil",
        "externalId": "SAMPLE_ID_8888"
      }
    ]
  }
}
```

If the user selects the first suggested address, we must use it's `externalId` to make an `address` query:

```graphql
query {
  address(id: "SAMPLE_ID_2718") {
    city
    neighborhood
    street
  }
}
```

Which returns an address object that can be used to autocomplete an address form.

```graphql
{
  "data": {
    "address": {
      "city": "Rio de Janeiro",
      "neighborhood": "Botafogo",
      "street": "Praia de Botafogo"
    }
  }
}
```


**Upcoming documentation:**

 - [CHK-435: Session token](https://github.com/vtex-apps/geolocation-graphql-interface/pull/6)
