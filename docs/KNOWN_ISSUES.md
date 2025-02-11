# Known Issues and Workarounds

No software is perfect, here is a list of known issues /gotchas that is worth noting with potential workarounds when you do encounter them (and for some, where you don’t really have a solution) 

1. Queries assigned to variables (see [Guard: Variable, Projections and Interpolations](QUERY_PROJECTION_AND_INTERPOLATION.md)) can be accessed using two forms when defining clauses, E.g. `let api_gws = Resources.*[ Type == 'AWS::ApiGateway::RestApi' ]`
    
```
%api_gws.Properties.EndpointConfiguration.Types[*] == "PRIVATE"` 
```
    
or 
    
```
%api_gws {
    Properties.EndpointConfiguration.Types[*] == "PRIVATE"
}
```

The block form iterates over all `AWS::ApiGateway::RestApi` resources found in the input. The first form short circuits and returns immediately after the first resource failure. 

> **Workaround**: use the block form to traverse all values to show all resource failures and not just the first one that failed. We are tracking to resolve this issue.
2. Need `when` guards with filter expressions- When a query uses filters like `Resources.*[ Type == 'AWS::ApiGateway::RestApi' ]`, if there are no `ApiGatway` resources, then Guard will fail the clause today when performing the check 

```
%api_gws.Properties.EndpointConfiguration.Types[*] == "PRIVATE"
``` 
    
> **Workaround**: assign filters to variables and use `when` condition check e.g. 

```
let api_gws = Resources.*[ Type == 'AWS::ApiGateway::RestApi' ]
    when %api_gws !empty { ...}
```

3. When performing `!=` comparison, if the values are incompatible like comparing a `string` to `int`, an error is thrown internally but currently suppressed and converted to `false` to satisfy the requirements of Rust’s [PartialEq](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html). We are tracking to release a fix for this issue soon.
4. `exists` and `empty` checks do not display the JSON pointer path inside the document in the error messages. Both these clauses often have retrieval errors which does not maintain this traversal information today. We are tracking to resolve this issue. 
5. When evaluating CloudFormation templates in YAML format, we do not support the short form versions of CloudFormation intrinsic functions like `!Join`, `!Sub` and others. Guard does not support these YAML extensions when evaluating.
    
> **Workaround**: use the expanded form when using these functions. 
6. Currently, for `string` literals, Guard does not support embedded escaped strings. We are tracking to resolve this issue soon.

7. Currently, for resource keys, we only support alphanumeric literals. For example, if we have the following template:

```
key-with-dash: true
``` 
and a corresponding rule:

```
key-with-dash == true
```

The Guard evaluation errors out with the following message:
```
Parsing error handling rule file = rules.guard, Error = Parser Error when parsing rules file Parsing Error Error parsing file rules.guard at line 1 at column 1, when handling , fragment key-with-dash == true
```

```
Workaround: Use quotes while referencing these keys, as follows:
```

```
"key-with-dash" == true
```

