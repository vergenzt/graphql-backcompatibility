# graphql-backcompatibility

What changes can you make to your GraphQL schema if you can't break backcompat?

## String => Enum

If you've got a field that should be an enum but was created as a string, then for the most part it's safe to change it to an enum. There are three cases I’ve come up with, and backcompat is maintained for two of the three--but there is one case that breaks. (All of the following assume that 

 1. :white_check_mark: Queries for a field do not break on string => enum type change. The value gets returned as a JSON string either way.
 
    E.g. `{ myColorField }` will return identically if the field changes from type `String` with manually-validated options "Red", "Blue", or "Green" to type `MyColorEnum { Red, Blue, Green }`.

 2. :white_check_mark: Mutations with an input field whose type changes from string to enum do not break if the field in question is passed through a variable.

    E.g. `mutation($input: String!) { putMyColorField(value: $input) }` with variables `{ "input": "Red" }` has identical behavior to `mutation($input: MyColorEnum) { putMyColorField(value: $input) }` with variables `{ "input": "Red" }`.

 3. :x: Mutations with an input field whose type changes from string to enum _break_ if that enum is specified directly in the query.

    E.g. if `putMyColorField` expects a String argument, then the inline mutation _must_ be written as `putMyColorField(value: "Red")`, i.e. with quotes. If it expects an enum value argument, then the value _must_ be written without quotes or the mutation fails, e.g. `putMyColorField(value: Red)`.
    
    If your schema has clients that use behavior (3) then you cannot change the string to an enum without breaking those clients.
