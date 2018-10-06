

# Perform a GET request using fetch() & promises



````javascript
let  options = {
  method: "GET"
  headers: {
        Authorization: 'Client-ID a0579c853711683ef69c0b'
    }
}
fetch(`https://someurl.com/someresourse.json`, options) // perform GET 
  .then( response => response.json() ) 					// get the JSON from the responce
  .then( processResponce )								// process the responce.
  .catch( e => dealWithError(e, 'some message'));		// deal with any errors.

function processResponce( responceJSON ){
  // do something with the JSON 
}

function dealWithError (e, errorMessage) {
  console.log("The error was "+ e +" message was "+ errorMessage);
}
````

