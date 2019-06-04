---
title: "Composable function in Client"
description: ""

author: Khairunj
ms.author: Khairunj
ms.date: 02/19/2019
ms.topic: article
ms.service: multiple
---

Composable function (function import) can have additional path segments and query options as appropriate for the returned type.

## Unbound composable function
For example, we have model:

``` csharp
<Function Name="GetAllProducts" IsComposable="true">
  <ReturnType Type="Collection(NS.Product)" Nullable="false" />
</Function>
<Action Name="Discount" IsBound="true" EntitySetPath="products">
  <Parameter Name="products" Type="Collection(NS.Product)" Nullable="false" />
  <Parameter Name="percentage" Type="Edm.Int32" Nullable="false" />
  <ReturnType Type="Collection(NS.Product)" Nullable="false" />
</Action>
...
<FunctionImport Name="GetAllProducts" Function="NS.GetAllProducts" EntitySet="Products" IncludeInServiceDocument="true" />
```

`GetAllProducts` is a function import and it is composable. And since action `Discount` accepts what `GetAllProducts` returns, we can query `Discount` after `GetAllProducts`.

<strong>1. Create function query</strong>

``` csharp
var products = Context.CreateFunctionQuery<Product>("", "GetAllProducts", true).Execute();
```

And we can append query option to the function. For example:

``` csharp
var products = Context.CreateFunctionQuery<ProductPlus>("", "GetAllProducts", true).AddQueryOption("$select", "Name").Execute();
```
The actual query would be:
``` csharp
GET https://localhost/GetAllProducts()?$select=Name
```

<strong>2. With codegen</strong>

With [OData client generator](https://blogs.msdn.com/b/odatateam/archive/2014/03/12/how-to-use-odata-client-code-generator-to-generate-client-side-proxy-class.aspx), proxy class for function and action would be auto generated.
For example:

``` csharp
var getAllProductsFunction = Context.GetAllProductsPlus();
var products = getAllProductsFunction.Execute();   // Get products 
var discdProducts = getAllProductsFunction.DiscountPlus(50).Execute();   // Call action on function
var filteredProducts = getAllProductsFunction.Where(p => p.SkinColorPlus == ColorPlus.RedPlus).Execute();   //Add query option 
```

## Bound composable function

Bound composable function has similiar usage, except that it is tied to a resource.

For example, we have model:
``` csharp
<Function Name="GetSeniorEmployees" IsBound="true" EntitySetPath="People" IsComposable="true">
    <Parameter Name="employees" Type="Collection(NS.Employee)" Nullable="false" />
    <ReturnType Type="NS.Employee" />
</Function>
<Function Name="GetHomeAddress" IsBound="true" IsComposable="true">
    <Parameter Name="person" Type="NS.Person" Nullable="false" />
    <ReturnType Type="NS.HomeAddress" Nullable="false" />
</Function>
```

Person is the base type of Employee. 
Then a sample query is:
``` csharp
(Context.People.OfType<Employee>() as DataServiceQuery<Employee>).GetSeniorEmployees().GetHomeAddress().GetValue();
```