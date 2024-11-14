# API Design

API Design is distinct from systems design. It can be considered as a sibling of a system design. There are some definite similarities between them:

- An product or software service, there is always an API which backs them and it really is the core of these API services. Hence these should be well designed. There is a sense of permanence to these APIs because almost all the services or product may consume this API to make changes each of their processes. 

Lets look at some examples of interviews:

System Design for Twitter
- What part of twitter are we design?
- What functionality are we supporting ?
- What should be the scale of the system ?
- What region are we supporting ?

The API design question are similar for Twitter:
- What part of twitter are we designing the API for? Maybe only for homepage of twitter?
- What functionality should the API perform ?
- Who is going to consume the API ?


Lets look at some examples of API Design

### Entity Definition

#### Charge:
- id: uuid
- customer_id: uuid
- amount: integer
- currency: string ( or currency-code enum )
- status: enum ["succeeeded","pending","failed"]

#### Customer:
- id: uuid
- name: string
- address: string
- email: string
- card: card

#### Card
- Endpoint Definition
CreateCharge(Charge: Charge)
	=> Charge
GetCharge(id: uuid)
	=> Charge
UpdateCharge(id:uuid, updateCharge: Charge)
 => Charge
ListCharge(offset: integer, limit:integer)
 => Charge[]
CaptureCharge(id:uuid)
 => Charge

Usually API are described in swagger file, in .yml format.

[back](../SystemDesign.md)
