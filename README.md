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

## Test cases

| ID | Endpoint | Type | What it verifies |
|----|----------|------|------------------|
| TC_API_01 | `GET /v1/images/search?limit=2&mime_types=jpg` | Functional | Returns exactly 2 JPG images, with fields such as `id`, `url`, `width`, `height` |
| TC_API_02 | `GET /v1/breeds/search?q=abcxyz` | Negative | Invalid query returns `200 OK` with an empty array `[]`, no crash |
| TC_API_03 | `POST /v1/favourites` | Functional | A favourite is added; response contains `message` and a generated `id` |
| TC_API_04 | `DELETE /v1/favourites/{favourite_id}` | Functional + negative | Existing favourite is deleted; a non-existing one is handled without a 500 error |

## Requests executed

- **Breed search**: `GET {{URL}}/breeds/search?q=sib` returned the Siberian breed (id `sibe`, origin Russia, life span 12 to 15 years).
- **Add favourite**: `POST {{URL}}/favourites` returned `200 OK` with `"message": "SUCCESS"` and a new favourite `id`.
- **Vote**: `POST {{URL}}/votes` returned `201 Created` with `"message": "SUCCESS"`.

## Automated tests (Postman)

Request: `GET {{URL}}/images/search?mime_types=gif&limit=1`

```javascript
pm.test("Response returns exactly 1 result", function () {
    pm.expect(pm.response.json()).to.be.an('array');
    pm.expect(pm.response.json().length).to.eql(1);
});

pm.test("The image URL has a .gif extension", function () {
    pm.expect(pm.response.json()[0].url).to.match(/\.gif$/);
});
```

Result: both tests **passed**.

## Boundary test: `limit=1000`

Request: `GET {{URL}}/images/search?limit=1000`

- Status code: `200 OK`
- Items returned: **10**

The API does not reject the oversized value. It silently caps it to a safe maximum. This is input sanitisation rather than strict validation: it protects performance and stability and gives the client a valid response, but a tester should know the behaviour is "adjust" and not "error".

## Key takeaways

- A good API test checks more than the status code: also the body, headers, data types and error handling.
- Negative and boundary tests reveal how robust an API really is.
- Different APIs handle bad input differently (empty result, error code, or silent correction), so the expected result must be defined per endpoint.

## Repository structure

```
.
├── README.md
├── docs/
│   └── Kaniz_APITestAssignment.pdf
├── postman/
│   └── APITestAssignment.postman_collection.json
└── screenshots/
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

## Author

**Kaniz**: university student, studying computer science, AI and data systems.
