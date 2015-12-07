# Errors

> Example Error Response:

```json
{
    "error":
    {
        "error_code":"1001",
        "error_desc":"Missing app_secret or app_id"
    }
}

```

In the event of an error, HoneyLedger will return an `error` response with an `error_code` and `error_desc` to describe what went wrong.

The HoneyLedger API uses the following error codes:

Error Code | Meaning
---------- | -------
1001 | Missing Parameter
1002 | Incorrect Parameter
1003 | Data does not exist
