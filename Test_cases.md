## Test scenarios

| Test Scenario | HTTP Method | Purpose | Test Case |
|---------------|-------------|---------|-----------|
| Search for cat images | GET | Validate image count and format | TC_API_01, TC_API_07 |
| Search for an invalid breed | GET | Verify empty-result handling | TC_API_02 |
| Add a favorite image | POST | Verify successful creation | TC_API_03 |
| Delete a favorite image | DELETE | Verify deletion behavior | TC_API_04 |
| Search for a breed | GET | Validate response fields and data types | TC_API_05 |
| Validate response limits | GET | Check API limit behavior | TC_API_08 |

## Test cases

### Summary

| ID | Endpoint | Type | What it verifies |
|----|----------|------|------------------|
| TC_API_01 | `GET /v1/images/search?limit=2&mime_types=jpg` | Functional | Returns exactly 2 JPG images |
| TC_API_02 | `GET /v1/breeds/search?q=abcxyz` | Negative | Invalid query returns an empty array, no crash |
| TC_API_03 | `POST /v1/favourites` | Functional | A favourite is added and an `id` is generated |
| TC_API_04 | `DELETE /v1/favourites/{favourite_id}` | Functional + negative | Existing favourite is deleted; missing one handled gracefully |
| TC_API_05 | `GET /v1/breeds/search?q=sib` | Functional | Valid search returns matching breed data |
| TC_API_06 | `POST /v1/votes` | Functional | A vote is created and echoed back correctly |
| TC_API_07 | `GET /v1/images/search?mime_types=gif&limit=1` | Automated | Exactly 1 result, URL ends in `.gif` |
| TC_API_08 | `GET /v1/images/search?limit=1000` | Boundary | Oversized limit is handled without error |

---

### TC_API_01: Verify the API returns 2 JPG images

**Pre-conditions**
- API is accessible
- Internet connection available

**Steps**
1. Send `GET https://api.thecatapi.com/v1/images/search?limit=2&mime_types=jpg`
2. Observe the response

**Expected result**
- API returns exactly 2 images
- All images are in JPG format

**Status code:** `200 OK`

**Validation logic**
- Response is a JSON array
- Array length = 2
- Each object contains fields such as `id`, `url`, `height`, `width`
- Image URLs end with `.jpg` (or MIME type = jpg)

---

### TC_API_02: Verify the API handles an invalid search query

**Pre-conditions**
- API is accessible
- Internet connection available

**Steps**
1. Send `GET https://api.thecatapi.com/v1/breeds/search?q=abcxyz`
2. Check the response

**Expected result**
- API returns an empty array `[]`
- No error or crash

**Status code:** `200 OK`

**Validation logic**
- Response is valid JSON
- Array is empty
- No unexpected fields or errors

---

### TC_API_03: Verify a user can add a favourite image

**Pre-conditions**
- Valid API key available
- Valid `image_id` from `https://api.thecatapi.com/v1/images/search`

**Steps**
1. Send `POST https://api.thecatapi.com/v1/favourites`
2. Add header: `x-api-key`
3. Body:
   ```json
   {
     "image_id": "VYfvpfuU7"
   }
   ```

**Expected result**
- Favourite is successfully added
- Response contains `message` and `id`

**Status code:** `200 OK`

**Validation logic**
- Response contains the correct `id`
- Favourite ID is generated
- Data is stored successfully

---

### TC_API_04: Verify a favourite image can be deleted

**Pre-conditions**
- Valid API key is available
- A valid or invalid `favourite_id` is known

**Steps**
1. Send `DELETE https://api.thecatapi.com/v1/favourites/{favourite_id}`
2. Include header: `x-api-key`
3. Observe the response

**Expected result**
- If the favourite exists: it is deleted successfully
- If the favourite does not exist: the API handles it gracefully without crashing

**Status code**
- `200 OK` for successful deletion
- `404 Not Found`, or `200` with a message, if it does not exist (depends on API design)

**Validation logic**
- For an existing favourite:
  - Verify the response indicates success
  - Call `GET https://api.thecatapi.com/v1/favourites` to confirm the item is no longer present
- For a non-existing favourite:
  - Ensure the API does not return a server error (no `500`)
  - Response is safe (e.g. `404` or an empty success response)
  - System remains stable and consistent

---

### TC_API_05: Verify breed search returns matching results

**Pre-conditions**
- API is accessible
- Internet connection available

**Steps**
1. Send `GET {{URL}}/breeds/search?q=sib`
2. Observe the response

**Expected result**
- Response contains the Siberian breed (`id: "sibe"`, `name: "Siberian"`, `origin: "Russia"`)

**Status code:** `200 OK`

**Validation logic**
- Response is a JSON array with relevant results matching the query
- Required fields are present with correct data types (e.g. `weight` is an object, `life_span` is a string, rating fields like `intelligence` are numbers)
- Edge case: a query with no matches returns an empty array (see TC_API_02)

---

### TC_API_06: Verify a vote can be created

**Pre-conditions**
- Valid API key available
- A valid `image_id`

**Steps**
1. Send `POST {{URL}}/votes`
2. Add header: `x-api-key`
3. Body:
   ```json
   {
     "image_id": "img3",
     "value": 1
   }
   ```

**Expected result**
- Vote is created successfully
- Response echoes `image_id` and `value` and includes a generated `id`

**Status code:** `201 Created`

**Validation logic**
- `message` equals `"SUCCESS"`
- `id` is generated
- `image_id` and `value` in the response match the request

---

### TC_API_07: Verify the API returns exactly 1 GIF image (automated)

**Pre-conditions**
- API is accessible
- Postman post-response script added to the request

**Steps**
1. Send `GET {{URL}}/images/search?mime_types=gif&limit=1`
2. Run the post-response script (see [Automated tests](#automated-tests-postman))

**Expected result**
- Response is an array containing exactly 1 item
- The image URL ends with `.gif`

**Status code:** `200 OK`

**Validation logic**
- `pm.expect(pm.response.json()).to.be.an('array')`
- `pm.expect(pm.response.json().length).to.eql(1)`
- `pm.expect(pm.response.json()[0].url).to.match(/\.gif$/)`

**Actual result:** both tests passed

---

### TC_API_08: Verify the API handles an oversized `limit` value

**Pre-conditions**
- API is accessible
- Internet connection available

**Steps**
1. Send `GET {{URL}}/images/search?limit=1000`
2. Count the items in the response

**Expected result**
- API does not crash or return a server error
- Response is capped to a safe maximum

**Status code:** `200 OK`

**Validation logic**
- Response is a valid JSON array
- Array length is at most the API's maximum (observed: **10**)
- No `500` error

**Actual result:** `200 OK` with 10 items. The API sanitises the value instead of rejecting it.


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


## How to run the Postman collection

1. Install [Postman](https://www.postman.com/downloads/).
2. Get a free API key from [thecatapi.com](https://thecatapi.com/signup).
3. Import `postman/APITestAssignment.postman_collection.json`.
4. Create an environment with these variables:
   - `URL` = `https://api.thecatapi.com/v1`
   - `API_KEY` = *your own key*
5. Make sure requests that need authentication send the header `x-api-key: {{API_KEY}}`.
6. Run requests individually, or use the **Collection Runner** to run everything.

