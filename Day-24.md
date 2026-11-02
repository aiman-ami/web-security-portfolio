# Day 23: SQL Injection with Filter Bypass via XML Encoding

- Lab: SQL injection with filter bypass via XML encoding, solved
- Intercepted the stock check request, a POST to /product/stock with an XML body
- Sent it to Repeater and tested a plain UNION payload, got "Attack detected" from the WAF
- Used Hackvertor hex entities to encode the payload, the query executed through the WAF
- Column count test showed one column only, so I concatenated username and password with || and a ~ separator
- Retrieved the admin credentials from the users table and logged in

Roadmap: 2B UNION attacks with filter bypass done.

## Writeup
