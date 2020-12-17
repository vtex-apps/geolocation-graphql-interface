📢 Use this project, [contribute](https://github.com/vtex-apps/geolocation-graphql-interface) to it or open issues to help evolve it using [Store Discussion](https://github.com/vtex-apps/store-discussion).

# Geolocation GraphQL Interface

> :warning: *Although ready to be installed, the Geolocation GraphQL Interface app is currently an **open-beta project**. Bear in mind that due to that status, you can expect an accelerated state of evolution.*

The Geolocation GraphQL interface provides autosuggestions for address inputs and autocompletion for address forms.

>ℹ️ The Geolocation GraphQL interface allows you to choose which geolocation provider (e.g., [Google Maps](https://www.google.com.br/maps/), [Algolia Places](https://community.algolia.com/places/), [VTEX Places](https://github.com/vtex-apps/places-graphql), etc.) suits better for your store.

It's possible to use the Geolocation GraphQL interface to connect with any front-end component, adapting the queries according to the UI needs.

Check the following sections for more information on how to use the data fetched from the Geolocation GraphQL interface in your React components.

## Usage examples

>ℹ️ The following examples are based on the [Location Search](https://github.com/vtex-apps/place-components/#locationfield-search) component used during the checkout process in the [Place Components](https://github.com/vtex-apps/place-components) app.

### Using autosuggestions for address inputs

Take the following example: imagine you want to develop a component that suggests a list of addresses based on a search term.

![location2](https://user-images.githubusercontent.com/60782333/100448177-ad8c2980-3090-11eb-8f9e-d01cfc7eef8f.png)

>💡 Autosuggest related operations are commonly used along with text-based inputs.

For that, we can implement the Geolocation GraphQL interface in our React component in a way that, for each keystroke, our component sends a query to the Geolocation GraphQL interface as in the following:

```graphql
query {
  suggestAddresses(searchTerm: "Upper") {
    description
    externalId
  }
}
```

In the sequence, according to the resolver chosen, the query returns address suggestions.

```graphql
{
  "data": {
    "suggestAddresses": [
      {
        "description": "Upper Saddle River, NJ, USA",
        "externalId": "SAMPLE_ID_2718"
      },
      {
        "description": "Upper East Side, Manhattan, New York, NY, USA",
        "externalId": "SAMPLE_ID_8888"
      },
      {
        "description": "Upper West Side, Manhattan, New York, NY, USA",
        "externalId": "SAMPLE_ID_0042"
      },
      {
        "description": "Upper Manhattan, Manhattan, New York, NY, USA",
        "externalId": "SAMPLE_ID_1001"
      },
      {
        "description": "Upper Nyack, NY, USA",
        "externalId": "SAMPLE_ID_1123"
      }      
    ]
  }
}
```

>ℹ️ *Notice that the number of suggestions is handled by the resolver itself.*

### Using autocompletion for address forms

After providing a list of suggestions for the address input (as in the previous example), we must guarantee that when the user selects the desired address, the address form will be automatically completed.

For that, we must use the `externalId` related to the desired address to make a `getAddressByExternalId` query.

For example, imagine the user selects the first suggested address from the previous example, we would need to perform the following query:

```graphql
query {
  getAddressByExternalId(id: "SAMPLE_ID_2718") {
    city
    neighborhood
    country
  }
}
```

As a result, this query would return an address object that could be used to autocomplete the address form.

```graphql
{
  "data": {
    "getAddressByExternalId": {
      "city": "NJ",
      "neighborhood": "Upper Saddle River",
      "country": "USA"
    }
  }
}
```

## Fetching data using GraphQL queries with React and Apollo

### Requirements

To successfully develop front-end components that fetch data from the Geolocation GraphQL interface, you must install the following apps through the terminal and [Toolbelt](https://developers.vtex.com/docs/vtex-io-documentation-vtex-io-cli-installation-and-command-reference) in the VTEX account you're using to develop your app:

- The Geolocation GraphQL interface `vtex.geolocation-graphql-interface.`
- The Google Geolocation Resolver [`vtex.google-geolocation-resolver`](https://github.com/vtex-apps/google-geolocation-resolver)

>⚠️ *Having zero or more than one geolocation resolver installed will result in an error.*

>ℹ️️ *Currently [Google Geolocation](https://github.com/vtex-apps/google-geolocation-resolver) is the only GraphQL resolver available.*

>💡 *Install the VTEX GraphQL IDE app (`vtex.admin-graphql-ide@3.x`) and access it through the admin's side-bar menu. From the dropdown list, choose the `vtex.geolocation-graphql-interface` app. Then, try running and debugging your queries there!*

### Step by step 

>ℹ️ *To successfully perform this step by step, you may need previous knowledge of [Apollo Client](https://www.apollographql.com/docs/tutorial/introduction/?_ga=2.219865305.1929567151.1606434433-711418368.1606434433). Apollo Client is a library for JavaScript that works as a GraphQL client. It includes built-in features that allow declarative data fetching from a GraphQL server.*

1. Open your React app in any code editor of your preference, and edit the `manifest.json` file, including the `vtex.geolocation-graphql-interface` app as a dependency.

```json
  "dependencies": {
    "vtex.geolocation-graphql-interface": "0.x"
  },
```

2. Using the terminal, access your app's directory and run the command below to install the [React Apollo](https://www.npmjs.com/package/react-apollo) npm library:

```
yarn add react-apollo
```

3. In your front-end app's repository, create a folder named `graphql` inside `react`.
4. Save all the necessary queries your UI components rely on in `.graphql` files. 

Take the [Location Search](https://github.com/vtex-apps/place-components/blob/37f42e94588e2cef897a7a0ff1e0df9e30ce2305/react/LocationSearch.tsx) component from the [Place Components](https://github.com/vtex-apps/place-components) app as an example. Notice that the [Location Search](https://github.com/vtex-apps/place-components/blob/37f42e94588e2cef897a7a0ff1e0df9e30ce2305/react/LocationSearch.tsx) component relies on the [`suggestAddresses.graphql`](https://github.com/vtex-apps/place-components/blob/master/react/graphql/suggestAddresses.graphql) query, defined as:

```graphql
query SuggestAddresses($searchTerm: String!) {
  suggestAddresses(searchTerm: $searchTerm)
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

>⚠️ You must use the `@context` [directive](https://www.apollographql.com/docs/apollo-server/schema/directives/) to prevent any issues related to ambiguity in the query definition, specifying which provider will be used to fetch the data for your component.

5. In your front-end components, declared in the `/react` folder, define how your component will consume the data fetched from the Geolocation API using [Apollo Client](https://www.apollographql.com/docs/tutorial/introduction/?_ga=2.219865305.1929567151.1606434433-711418368.1606434433).

>ℹ️ *For more information on how to develop React components using VTEX IO and GraphQL, check our documentation on [Developing components using VTEX IO and React - Fetching data](https://vtex.io/docs/getting-started/desenvolva-componentes-usando-vtex-io-e-react/6/).*

## Schema description

### Query

![geolocation](https://user-images.githubusercontent.com/60782333/100455655-d0710a80-309d-11eb-91d8-b926554b2a32.png)

<table>
<thead>
<tr>
<th align="left">Field</th>
<th align="left">Type</th>
<th align="left">Description</th>
</tr>
</thead>
<tbody>
<tr>
<td colspan="1" valign="top"><strong>suggestAddresses</strong></td>
<td valign="top"><a href="#addresssuggestion">[AddressSuggestion!]!</a></td>
<td>

</td>
</tr>
<tr>
<td colspan="1" valign="top"><strong>getAddressByExternalId</strong></td>
<td valign="top"><a href="#address">Address!</td>
<td>

</td>
</tr>
</tbody>
</table>

### `AddressSuggestion`

<table>
<thead>
<tr>
<th align="left">Field</th>
<th align="left">Type</th>
<th align="left">Description</th>
</tr>
</thead>
<tbody>
<tr>
<td colspan="1" valign="top"><strong>description</strong></td>
<td valign="top">String!</td>
<td></td>
</tr>
<tr>
<td colspan="1" valign="top"><strong>mainText</strong></td>
<td valign="top">String!</td>
<td></td>
</tr>


<tr>
<td colspan="1" valign="top"><strong>mainTextMatchInterval</strong></td>
<td valign="top"><a href="#matchinterval">MatchInterval!</a></td>
<td></td>
</tr>

<tr>
<td colspan="1" valign="top"><strong>secondaryText</strong></td>
<td valign="top">String!</td>
<td></td>
</tr>

<tr>
<td colspan="1" valign="top"><strong>externalId</strong></td>
<td valign="top">String!</td>
<td></td>
</tr>
</tbody>
</table>

### `Address`


<table>
<thead>
<tr>
<th align="left">Field</th>
<th align="left">Type</th>
<th align="left">Description</th>
</tr>
</thead>
<tbody>
<tr>
<td colspan="1" valign="top"><strong>addressId</strong></td>
<td valign="top">ID</td>
<td></td>
</tr>

<tr>
<td colspan="1" valign="top"><strong>addressType</strong></td>
<td valign="top"><a href="#addresstype">AddressType</a></td>
<td></td>
</tr>


<tr>
<td colspan="1" valign="top"><strong>city</strong></td>
<td valign="top">String</td>
<td>

</td>
</tr>

<tr>
<td colspan="1" valign="top"><strong>complement</strong></td>
<td valign="top">String</td>
<td>

</td>
</tr>

<tr>
<td colspan="1" valign="top"><strong>country</strong></td>
<td valign="top">String</td>
<td>

</td>
</tr>

<tr>
<td colspan="1" valign="top"><strong>geoCoordinates</strong></td>
<td valign="top">[Float!]</td>
<td>

</td>
</tr>

<tr>
<td colspan="1" valign="top"><strong>neighborhood</strong></td>
<td valign="top">String</td>
<td>

</td>
</tr>

<tr>
<td colspan="1" valign="top"><strong>number</strong></td>
<td valign="top">String</td>
<td>

</td>
</tr>

<tr>
<td colspan="1" valign="top"><strong>postalCode</strong></td>
<td valign="top">String</td>
<td>

</td>
</tr>

<tr>
<td colspan="1" valign="top"><strong>receiverName</strong></td>
<td valign="top">String</td>
<td>

</td>
</tr>

<tr>
<td colspan="1" valign="top"><strong>reference</strong></td>
<td valign="top">String</td>
<td>

</td>
</tr>


<tr>
<td colspan="1" valign="top"><strong>state</strong></td>
<td valign="top">String</td>
<td>

</td>
</tr>


<tr>
<td colspan="1" valign="top"><strong>street</strong></td>
<td valign="top">String</td>
<td>

</td>
</tr>
</tbody>
</table>

### `MatchInterval`

<table>
<thead>
<tr>
<th align="left">Field</th>
<th align="left">Type</th>
<th align="left">Description</th>
</tr>
</thead>
<tbody>
<tr>
<td colspan="1" valign="top"><strong>offset</strong></td>
<td valign="top">Int!</td>
<td>

</td>
</tr>
<tr>
<td colspan="1" valign="top"><strong>length</strong></td>
<td valign="top">Int!</td>
<td>


</td>
</tr>
</tbody>
</table>

### `AddressType`

<table>
<thead>
<tr>
<th align="left">Field</th>
<th align="left">Type</th>
<th align="left">Description</th>
</tr>
</thead>
<tbody>
<tr>
<td colspan="1" valign="top"><strong>offset</strong></td>
<td valign="top">Int!</td>
<td>

</td>
</tr>
<tr>
<td colspan="1" valign="top"><strong>length</strong></td>
<td valign="top">Int!</td>
<td>


</td>
</tr>
</tbody>
</table>
