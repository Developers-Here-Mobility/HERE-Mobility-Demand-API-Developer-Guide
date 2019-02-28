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

Domain|	Error Code | Message | Details 
:-----|:-----------|:--------|:------
General 	| 400000	| Invalid request 	| -										
Get Ride 	| 400001	| Route is missing  | -											
Get Ride 	| 400002	| Pickup location is missing | Pickup address/place name is missing
Get Ride 	| 400003	| Pickup point is missing  | Pickup longitude and latitude are missing
Get Ride 	| 400004	| Pickup latitude is invalid. Value should be between [min range] and [max range]  | Latitude value should be between -90 and 90.    			
Get Ride 	| 400005	| Pickup longitude is invalid. Value should be between [min range] and [max range]  | Longitude value should be between -180 and 180.    		
Get Ride 	| 400007	| Drop off location is missing | Drop off address/place name is missing												
Request Ride| 400009 	| Pickup time is missing | -								
Get Ride | 400010 | Pickup cannot be booked for more than [max] days from the current date | The pickup time is too far in the future
Get Ride | 400011 | Pickup time cannot be in the past  | - 									
Get Ride | 400016 | Maximum walking distance chosen is not supported | The given limit on walking distance has an invalid format or value						
Get Offers | 400017	| Invalid number of changes | The maximum allowed number of transit changes during the route was exceeded	
Cancel Ride 	| 400018| Ride ID is invalid  | - 											
Cancel Ride     | 400019| Reason category format is invalid | - 									
Cancel Ride     | 400020| Reason category value is invalid 	| -								
Get Offers 		| 400021| Offer ID is missing or invalid  | - 									
General		 	| 400022| User ID is missing | -  												
Get Offers 		| 400023| Demander ID is missing or invalid   | -								
Get Ride 		| 400024| Drop off point is missing | Drop off longitude and latitude are missing											
Get Ride 		| 400025| Drop off latitude is invalid. Value should be between [min range] and [max range]  | Latitude value should be between -90 and 90.	
Get Ride 		| 400026| Drop off longitude is invalid. Value should be between [min range] and [max range]  | Longitude value should be between -180 and 180.  	
Authentication 	| 401001| Invalid token	| -												
Authentication 	| 401002| Token expired	| -													
General  		| 403000| Permission denied	| -												
General 		| 404000| Requested item not found 	| -										
Create Ride		| 404001| Offer timed out | The offer has expired. 									
Get Ride 		| 404002| The ride requested can't be found  | -  								
General 		| 408000| Request timeout	| -												
General 		| 424000| Failed dependency	| An error was received from a 3rd-party package
General	   		| 429000| Too many requests	| Server is busy								
General	       	| 500000| Internal server error | -												
General	       	| 501000| Not Implemented	| -															

## References ##

[https://cloud.google.com/apis/design/errors](https://cloud.google.com/apis/design/errors)

[https://github.com/grpc/grpc-go/tree/master/examples/rpc_errors](https://github.com/grpc/grpc-go/tree/master/examples/rpc_errors)

[https://jbrandhorst.com/post/grpc-errors](https://jbrandhorst.com/post/grpc-errors)
