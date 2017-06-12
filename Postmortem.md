*Postmortem : Json Schema Type Provider*

**Rationale and history**

3 years ago, when we just started building out Jet Partner API, we faced a massive task of validating and 
(de)serializing all the incoming JSON messages. We also needed to build documentation for Partners to help them with integrations. 

Classic approach would be to have some custom logic built for each payload to validate each field and produce meaningful error
messages. We felt there’s better way to do this, with minimum amount of code, so we searched for alternatives. 

We chose [Json Schema]( http://json-schema.org/) that was on Draft 3 back then. It gave us an ability to constrain Json payloads 
declaratively. For example, instead of having a piece of code testing length of the product title field, we could have 
something like

    "product_title": {
			"type": "string",
			"required": true,
			"minLength" : 5,
			"maxLength" : 500,
			"description": "Short product description 5 to 500 alphanumeric characters"
		}

in schema.json file, and then use excellent [Newtonsoft]( http://www.newtonsoft.com/json) library (version 6.0.3 back then) 
to do all the mundane validation for us. Of course, there were many validations that still had to be done manually, 
like testing messages against stored state. Nevertheless, it cut on the amount of code we had to write and maintain immensely.

Luckily, our team included outstanding product managers that had no problem translating payload requirements directly 
into schema files, freeing developers to focus on other tasks. It was a dream of domain language shared between 
business and development come true.

Moreover, with minimal amount of coding we could turn these schemas into API documentation and deploy it to Azure API 
Management platform. Even though we since abandoned this approach in favor of richer 
[API documentation site]( https://developer.jet.com/), it allowed us to move with incredible speed when it was critical.

Still, there was all this boring code to write that would represent Json in F#, plus (de)serialization. 
That’s when we decided it might be a good idea to have Type Provider to generate POCOs for us directly from Json schemas.

**Reasons for retirement**

While implementation wasn’t particularly hard, there were several reasons this type provider was ultimately abandoned in 
favor of more traditional approach:

* POCOs are no fun to use from F#. We had to deal with nulls on attributes and this alone was enough to 
take a second look at the solution. Back then Newtonsoft didn’t have much support for F#, and even if it 
did we were limited in what types we could use for properties in type provider implementation
* Visual Studio used older version of Newtonsoft internally. It meant that we couldn’t pass types defined in Newtonsoft 
from type provider as it was built with the version 
different from what it could bind to in Visual Studio environment. It made implementation rather inefficient, 
we had to parse schema on every validation
* Relying on Newtonsoft for validation limited our ability to customize schema for our needs.

**Next step**

For reasons mentioned earlier, we were looking for more F#-friendly ways of dealing with JSON. 
[JsonValue](https://github.com/fsharp/FSharp.Data/blob/master/src/Json/JsonValue.fs) library from FSharp.Data suited us well. Certain difficulties associated with presence of type providers in the same library made us to fork relevant part of FSharp.Data directly into our code base. This also created an opportunity to build custom Json Schema validation and abandon Newtonsoft completely.
At that point it also became clear that, while validation is necessary for Partner API, (de)serialization is not.
A lot of messages coming into API are passed downstream to other systems without any business logic applied, so 
dealing with JsonValue objects directly proved to be much more efficient than supporting translation to and from F# record types.

**Conclusion**

It is easy to miss an impedance mismatch between JSON and .Net types. (De)serialization by itself is a form of validation
and as such it exposes serialization limitations foreign to JSON.

Having upfront validation based on schema allows Jet Partner API to avoid leaking out details of implementation to customers. 
While schema-based validation is limited to individual properties, it eliminates a lot of boilerplate.

Json schema creates powerful opportunity to build types with well-defined structure that would (de)serialize in transparent manner. 
I believe this type provider can be re-written to operate with JsonValue types directly and use custom schema validation.


