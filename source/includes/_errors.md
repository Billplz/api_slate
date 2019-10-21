# Errors

The Billplz API uses the following error codes:


Error Code | Meaning
---------- | -------
401 | Unauthorized -- Your API key is wrong or you are requesting unauthorized data.
404 | Not Found -- The specified resource could not be found.
422 | Unprocessable Entity -- Your passed parameter is invalid.
429 | Too Many Requests -- Reached rate limit.
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- We're temporarily offline for maintenance. Please try again later.
