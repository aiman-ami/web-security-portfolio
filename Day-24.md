# Day 23: SQL Injection with Filter Bypass via XML Encoding

- Lab: SQL injection with filter bypass via XML encoding, solved
- Intercepted the stock check request, a POST to /product/stock with an XML body
- Sent it to Repeater and tested a plain UNION payload, got "Attack detected" from the WAF
- Used Hackvertor hex entities to encode the payload, the query executed through the WAF
- Column count test showed one column only, so I concatenated username and password with || and a ~ separator
- Retrieved the admin credentials from the users table and logged in

Roadmap: 2B UNION attacks with filter bypass done.

## Writeup

```
Bug: storeId value inside the XML body is injectable, and a WAF blocks raw SQL
keywords like union and select before the request reaches the application
Technique: WAF bypass through XML character references. The WAF inspects the raw
request text and sees only harmless encoded entities like u, while the server's
XML parser decodes them back into real characters before the value reaches the SQL
query, so the fully decoded payload lands inside the query
Payload pattern: <storeId>
Why it worked: one plain UNION test showed "Attack detected", confirming the WAF.
The encoded version returned data, confirming the bypass. A two column UNION
returned nothing, so the query has one column. Concatenation with || '~' || carried
both values in that single column, and the response printed them as "username~password"
```
