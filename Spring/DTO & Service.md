
DTO (Data transfer objects) is useful for several things and it's a widely used pattern in Spring. The main objective of them is that they are supposed to be the carriers of data instead of the entities. This results in more code but it allows you to isolate data you don't want to be exposed externally. It can reduce the overhead as DTO's can contain only the required data needed etc.

DTO are used with a service layer. Service classes gets annotated with **@Service**. The service layer is between the controller layer and the repositories.
Controller classes call on the service classes whom in turn got access to the repositories. ^8221d2

The service classes got two tasks. First convert entity to DTO and vice versa. And they should also handle all the business logic. Controller classes should just have responsibility of sending responses and receiving requests.
