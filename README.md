# Smart Campus API

A RESTful API built using JAX-RS and Apache Tomcat for managing rooms and sensors across a university Smart Campus. This project was developed as part of the Coursework for 5COSC022W Client-Server Architectures module at the University of Westminster.


## How to Build and Run

1. Clone the repository: git clone https://github.com/nbCode-CS/SmartCampusAPI.git

2. Open the project in Apache NetBeans

3. Make sure Apache Tomcat 9 is configured in NetBeans under Services → Servers

4. Right-click the project → Clean and Build

5. Right-click the project → Run


## Sample curl Commands

1. Get API discovery info: curl -X GET http://localhost:8080/SmartCampusAPI/api/v1

2. Get all rooms: curl -X GET http://localhost:8080/SmartCampusAPI/api/v1/rooms

3. Get all sensors: curl -X GET http://localhost:8080/SmartCampusAPI/api/v1/sensors

4. Get all sensors filtered by type: curl -X GET "http://localhost:8080/SmartCampusAPI/api/v1/sensors?type=CO2"

5. Delete a room: curl -X DELETE http://localhost:8080/SmartCampusAPI/api/v1/rooms/HALL-201


## Report — Question Answers

### Part 1 — Service Architecture & Setup

Q1. In your report, explain the default lifecycle of a JAX-RS Resource class. Is a
new instance instantiated for every incoming request, or does the runtime treat it as a
singleton? Elaborate on how this architectural decision impacts the way you manage and
synchronize your in-memory data structures (maps/lists) to prevent data loss or race con-
ditions.

Answer: By default, JAX-RS creates a new instance of a resource class for every incoming request. This means instance variables cannot store data between requests. To solve this, a shared static DataStore class with HashMaps is used so all resource instances access the same data.

Q2. Why is the provision of ”Hypermedia” (links and navigation within responses)
considered a hallmark of advanced RESTful design (HATEOAS)? How does this approach
benefit client developers compared to static documentation?

Answer: HATEOAS allows clients to navigate the API through links embedded in responses rather than hardcoding URLs. In this project the discovery endpoint returns links to /api/v1/rooms and /api/v1/sensors. This means clients only need to know the entry point and can discover everything else from there.

### Part 2 — Room Management

Q1. When returning a list of rooms, what are the implications of returning only
IDs versus returning the full room objects? Consider network bandwidth and client side
processing.

Answer: Returning only IDs uses less bandwidth but requires the client to make extra requests to get the full details. Returning full objects gives the client everything in one response but increases payload size. This implementation returns full objects since clients typically need all the room details at once.

Q2. Is the DELETE operation idempotent in your implementation? Provide a detailed
justification by describing what happens if a client mistakenly sends the exact same DELETE
request for a room multiple times.

Answer: Yes. The first DELETE removes the room and returns 204. A second DELETE returns 404 since the room no longer exists. The server state is the same after both requests (the room is gone), which satisfies idempotency.

### Part 3 — Sensor Operations & Linking

Q1. We explicitly use the @Consumes (MediaType.APPLICATION_JSON) annotation on
the POST method. Explain the technical consequences if a client attempts to send data in
a different format, such as text/plain or application/xml. How does JAX-RS handle this
mismatch?

Answer: JAX-RS will reject the request before it reaches the method and return a 415 Unsupported Media Type response. The method code never executes.

Q2.You implemented this filtering using @QueryParam. Contrast this with an alterna-
tive design where the type is part of the URL path (e.g., /api/vl/sensors/type/CO2). Why
is the query parameter approach generally considered superior for filtering and searching
collections?

Answer: Query parameters are designed for optional filtering of collections while path parameters identify specific resources. @QueryParam is also more flexible since multiple filters can be combined like ?type=CO2&status=ACTIVE without needing extra endpoints.

### Part 4 — Deep Nesting with Sub - Resources

Q1. Discuss the architectural benefits of the Sub-Resource Locator pattern. How
does delegating logic to separate classes help manage complexity in large APIs compared
to defining every nested path (e.g., sensors/{id}/readings/{rid}) in one massive con-
troller class?

Answer: It keeps resource classes focused and manageable by splitting logic into separate classes. Instead of putting all endpoint logic in one large class, SensorResource passes reading requests to SensorReadingResource. This makes the code easier to read and maintain as the API grows.

### Part 5 — Advanced Error Handling, Exception Mapping & Logging

Q1. Why is HTTP 422 often considered more semantically accurate than a standard
404 when the issue is a missing reference inside a valid JSON payload?

Answer: 404 means the endpoint itself doesn't exist. 422 means the endpoint is valid and the JSON is correct but the data inside is logically invalid. When a sensor is created with a roomId that doesn't exist, the endpoint works fine but the payload references something that doesn't exist, so 422 is more accurate.

Q2. From a cybersecurity standpoint, explain the risks associated with exposing
internal Java stack traces to external API consumers. What specific information could an
attacker gather from such a trace?

Answer: Stack traces reveal class names, method names and library versions which attackers can use to find known vulnerabilities. They can also expose internal application structure and sensitive data. The GlobalExceptionMapper prevents this by returning a generic 500 response instead.

Q3. Why is it advantageous to use JAX-RS filters for cross-cutting concerns like
logging, rather than manually inserting Logger.info() statements inside every single re-
source method?

Answer: Putting logging in every method means duplicating the same code everywhere. If the format needs changing, every method needs updating. A filter centralises logging in one place and automatically applies to every request and response without touching the resource methods.
