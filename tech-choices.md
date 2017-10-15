## Technology choices

### Database

Things to consider

* Do you need transaction ?
* Do you need write replicas ?
* Do you need cross-region HA ?
* Do you have expertise to operate those database?

Tips:
1. When using NoSql, try to make all operation visible to user, this will remove the need of transaction as backend does not need to abstract multiple operations into 1 atomic op.

#### Dynamo

AWS Dynamo DB brings a lot of issues when you need **atomic operation**.
It gives great scaling ability (multi-region), but there is no straight-forward way to achieve consistency.
The strong-consistent read provided by Dynamo only prevents reading different value from replica for same row.


### Distributed Queue

Normally you use queue for async semantic, ie publisher and consumer is de-coupled.

Things to consider

1. What delivery semantic do you need ? (at-least 1, at-most 1, only-1)
2. What latency do you need ?
3. Do you need ordering guarantees of the queue ?

#### Kinesis

Using kinesis (KPL) does not gives at-least-1 semantic
