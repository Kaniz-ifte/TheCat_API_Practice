# API Testing with Postman: TheCatAPI

A hands-on REST API testing project. I designed test cases, executed them in **Postman**, and automated assertions with JavaScript, using [TheCatAPI](https://thecatapi.com/) as the system under test.

## What this project covers

| Area | What I did |
|------|-----------|
| **Theory** | Key parts of a REST API response (status codes, body, headers); types of API test cases; why testing missing/invalid parameters matters |
| **Test case design** | Wrote structured test cases with ID, summary, pre-conditions, steps, expected result, status code and validation logic |
| **Execution** | Ran requests in Postman: GET, POST, DELETE |
| **Automation** | Wrote Postman post-response test scripts (Chai assertions) |
| **Boundary testing** | Checked how the API behaves with an oversized `limit` value |


## Key takeaways

- A good API test checks more than the status code: also the body, headers, data types and error handling.
- Negative and boundary tests reveal how robust an API really is.
- Different APIs handle bad input differently (empty result, error code, or silent correction), so the expected result must be defined per endpoint.

## Repository structure

```
.
├── APITestAssignment.postman_collection.json
├── README.md
├── Test_cases.md
```

## How to run the Postman collection

1. Install [Postman](https://www.postman.com/downloads/).
2. Get a free API key from [thecatapi.com](https://thecatapi.com/signup).
3. Import `postman/APITestAssignment.postman_collection.json`.
4. Create an environment with these variables:
   - `URL` = `https://api.thecatapi.com/v1`
   - `API_KEY` = *your own key*
5. Make sure requests that need authentication send the header `x-api-key: {{API_KEY}}`.
6. Run requests individually, or use the **Collection Runner** to run everything.

> **Never commit your real API key.** Use environment variables and keep the key out of exported files.

## Tools

Postman · JavaScript (Chai assertions) · TheCatAPI · REST/JSON

