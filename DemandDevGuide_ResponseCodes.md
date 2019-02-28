# Here Demand API Response Codes #

## Introduction ##

While working with one of the HERE API or SDK packages, you may receive error responses from the HERE Server. The following sections describe the error structures, codes and messages you may receive.

<a name="GRPCErrorStructure"></a>
## GRPC Error Structures ##

In error situations, all endpoints return a GRPC Status object as defined below.

    type Status struct {
        // The error code (a value from the google.rpc.Code enum)
        Code int32
        // A textual, informative, English-language error message
        Message string
        // A list of messages describing the error details
        Details []*google_protobuf.Any
    }

>**NOTE**: If the GRPC-gateway fails to identify the server error, it creates a new GRPC error with an UNKNOWN error code (which is mapped to the HTTP 500 Internal Server Error).

The **Details** field of the **Status** structure contains the following nested structure:

    type PublicError struct {
       // The public error code consists of a 3-digit HTTP error code followed by a 3-digit proprietary error code.
       Code int32 `protobuf:"varint,1,opt,name=code" json:"code,omitempty"`
       
       // Client's locale as received by the HERE Marketplace
       // - ISO-639-1 standard language codes, defaults to "en".
       // - ISO-3166-1-alpha-2 country codes
       // Example:  "en-US" (US is the ISO 3166‑1 country code for the United States)
       // Based on IETF language tag best practice as specified by https://tools.ietf.org/html/rfc5646
       Locale string `protobuf:"bytes,2,opt,name=locale" json:"locale,omitempty"`
       
       // The localized error message for the above locale.
       Message string `protobuf:"bytes,3,opt,name=message" json:"message,omitempty"`
    }

## RESTful JSON Error Structure ##

The GRPC error is translated to its RESTful JSON representation automatically by the GRPC-gateway package. The JSON body contains the error code, locale and message values from the Status structure shown above.

Here is an example of a JSON-formatted error:

    Header Code: 404000
    Body:
    {
        "code": 404000,
        "locale: "en-US"
        "message": "Requested item not found"
    }
 
The GRPC error code is translated to an HTTP header status code, according to the mapping in the following table:

GRPC Error Code	| HTTP Error Code
:---------------|:----------------
InvalidArgument	|StatusBadRequest (400000)
OutOfRange	    |StatusBadRequest (400000)
PermissionDenied	|StatusForbidden (403000)
Unauthenticated	|StatusUnauthorized (401000)
NotFound	    |StatusNotFound (404000)
AlreadyExists	|StatusConflict (409000)
FailedPrecondition	|StatusPreconditionFailed (412000)
Internal	    |StatusInternalServerError (500000)
Unknown	        |StatusInternalServerError (500000)
Unavailable     | StatusServiceUnavailable(503000)
DeadlineExceeded | StatusGatewayTimeout (504000)
 
## Error Categories ##

All external errors have one of the categories described in the following table:

Error Category	| Description
:---------------|:------------
**Input**	|	Error in request input.<br/><br/>Possible GRPC codes:<br/><br/>InvalidArgument(3) – missing, malformed or otherwise invalid argument<br/><br/>OutOfRange(11) – value has the correct type but is out of the valid range
**Permissions**|Authorization or authentication error.<br/><br/>Possible GRPC codes:<br/><br/>**PermissionDenied(7)** – attempt to perform an action for which the user has no permissions<br/><br/>**Unauthenticated(16)** – the user failed to be authenticated
**Communication** | Connection error or timeout when accessing other services.<br/><br/>Possible GRPC codes:<br/><br/>**Unavailable(14)** – the service is unavailable
**Business Logic** | The request or its data is incompatible with the current server state.<br/><br/>Possible GRPC codes:<br/><br/>**NotFound(5)** – requested item was not found (for example, a ride ID)<br/><br/>**AlreadyExists(6)** – attempt to create an entity that already exists<br/><br/>**FailedPrecondition(9)** – the request does not meet a required condition (for example, a delete was requested when delete is not allowed)<br/><br/>**Internal(13)** – other error in business logic
**Other** 	|	Miscellaneous errors.<br/><br/>Possible GRPC codes:<br/><br/>**Internal(13)** – error in business logic

## Error Codes ##

The following table describes the error codes that HERE API clients can receive:

Domain|	Message (& Details) | 	Code
:-----|:--------|:-------
General 		| Invalid request 																| 400000
Get Ride 		| Route is missing   															| 400001	
Get Ride 		| Pickup location is missing													| 400002	
Get Ride 		| Pickup point is missing   													| 400003	
Get Ride 		| Pickup latitude is invalid. Value should be between -90 and 90.    			| 400004
Get Ride 		| Pickup longitude is invalid. Value should be between -180 and 180.    		| 400005
Get Ride 		| Drop off location is missing   												| 400007	
Request Ride 	| Pickup time missing from ride request 										| 400009
Get Ride 		| Pickup cannot be booked for more than <MaxDays> days from the current date (the pickup time is too far in the future)	| 400010
Get Ride 		| Pickup time cannot be in the past   											| 400011	
Get Ride 		| Maximum walking distance chosen is not supported (the given limit on walking distance	has an invalid format or value)						| 400016
Get Offers 		| Invalid number of changes (the maximum allowed number of transit changes during the route was exceeded) 		| 400017
Cancel Ride 	| Ride ID is invalid    														| 400018	
Cancel Ride     | Invalid reason category format 												| 400019
Cancel Ride     | Invalid reason category value 												| 400020
Get Offers 		| Offer ID is missing or invalid   												| 400021	
General		 	| User ID is missing   															| 400022	
Get Offers 		| Demander ID is missing or invalid   											| 400023
Get Ride 		| Drop off point is missing														| 400024
Get Ride 		| Drop off latitude is invalid. Value should be between -90 and 90.				| 400025
Get Ride 		| Drop off longitude is invalid. Value should be between -180 and 180.  		| 400026
Authentication 	| Invalid token																	| 401001
Authentication 	| Token expired																	| 401002
General  		| Permission denied																| 403000
General 		| Requested item not found 														| 404000
Create Ride		| Offer timed out  																| 404001
Get Ride 		| The ride requested can't be found    											| 404002	
General 		| Request timeout																| 408000
General 		| Failed dependency	(an error was received from a 3rd-party package)			| 424000
General	   		| Too many requests	(server is busy)											| 429000
General	       	| Internal server error															| 500000
General	       	| Not Implemented																| 501000

## References ##

[https://cloud.google.com/apis/design/errors](https://cloud.google.com/apis/design/errors)

[https://github.com/grpc/grpc-go/tree/master/examples/rpc_errors](https://github.com/grpc/grpc-go/tree/master/examples/rpc_errors)

[https://jbrandhorst.com/post/grpc-errors](https://jbrandhorst.com/post/grpc-errors)
