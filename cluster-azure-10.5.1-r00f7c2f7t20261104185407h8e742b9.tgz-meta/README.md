# cluster-azure

`cluster-azure` is an app that helps create CRs for a Cluster API Azure cluster on the Giant Swarm platform.

## Configuration

See our [full list of configuration options](helm/cluster-azure/README.md).

## Maintaining `values.schema.json` and `values.yaml`

**tl;dr**: We only maintain `values.schema.json` and automatically generate `values.yaml` from it:

```
make generate
```

**Details**:

In order to provide a better UX, we validate user values against `values.schema.json`. In addition, we also use the JSON schema in our frontend to dynamically generate a UI for cluster creation from it. To succesfully do this, we have some requirements on the `values.schema.json`, which are defined in [this RFC](https://github.com/giantswarm/rfc/pull/55).

These requirements can be checked with [`schemalint`](https://github.com/giantswarm/schemalint). `schemalint` does a couple of things:

- Normalize JSON schema (indentation, white space, sorting)  
- Validate whether your schema is a valid JSON schema
- Validate whether the requirements for cluster app schemas are met
- Check whether the schema is normalized

The first point can be achieved with:

```
make normalize-schema
```

The second to fourth point can be achieved with:

```
make validate-schema
```

The JSON schema in `values.schema.json` should contain defaults defined with the `default` keyword. These defaults should be the same as those defined in `values.yaml`. This allows us to generate `values.yaml` from `values.schema.json` with:

```
make generate-values
```
